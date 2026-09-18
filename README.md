-- LocalScript - Coloque dentro de StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ================= 1. CRIAÇÃO DA INTERFACE (GUI) =================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoFarmGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local botao = Instance.new("TextButton")
botao.Name = "ToggleButton"
botao.Size = UDim2.new(0, 180, 0, 50)
botao.Position = UDim2.new(0.05, 0, 0.4, 0)
botao.BackgroundColor3 = Color3.fromRGB(220, 53, 69) -- Vermelho (OFF)
botao.TextColor3 = Color3.fromRGB(255, 255, 255)
botao.TextSize = 16
botao.Font = Enum.Font.SourceSansBold
botao.Text = "AUTOFARM: OFF"
botao.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
uiCorner.Parent = botao

local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(255, 255, 255)
uiStroke.Transparency = 0.5
uiStroke.Parent = botao

-- ================= 2. ARRASTAR E TOGGLE SEM CONFLITO =================
local dragging = false
local moved = false
local dragStart, startPos
local autoFarmAtivado = false

botao.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		moved = false
		dragStart = input.Position
		startPos = botao.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		-- Só considera arrasto se mover mais de 5 pixels
		if delta.Magnitude > 5 then
			moved = true
			botao.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

-- Alterna ON/OFF apenas se NÃO tiver arrastado o botão
botao.Activated:Connect(function()
	if not moved then
		autoFarmAtivado = not autoFarmAtivado

		if autoFarmAtivado then
			botao.Text = "AUTOFARM: ON"
			botao.BackgroundColor3 = Color3.fromRGB(40, 167, 69) -- Verde (ON)
			print("[Autofarm]: LIGADO")
		else
			botao.Text = "AUTOFARM: OFF"
			botao.BackgroundColor3 = Color3.fromRGB(220, 53, 69) -- Vermelho (OFF)
			print("[Autofarm]: DESLIGADO")
		end
	end
end)

-- ================= 3. LÓGICA DE BUSCA E TELEPORTE =================
task.spawn(function()
	local pastaEntregas = workspace:WaitForChild("Entregas", 10)

	if not pastaEntregas then
		warn("[Autofarm]: A pasta 'Entregas' NÃO foi encontrada no Workspace!")
		return
	end

	print("[Autofarm]: Sistema carregado com sucesso.")

	while task.wait(0.5) do
		if not autoFarmAtivado then continue end

		local character = player.Character
		if not character or not character:FindFirstChild("HumanoidRootPart") then continue end

		-- Verifica todos os itens dentro da pasta Entregas (independente do nome da casa)
		for _, casa in ipairs(pastaEntregas:GetChildren()) do
			if not autoFarmAtivado then break end

			-- Busca PEntrega e GUI_GPS recursivamente (mesmo que estejam dentro de sub-grupos)
			local pEntrega = casa:FindFirstChild("PEntrega", true)
			local guiGps = casa:FindFirstChild("GUI_GPS", true)

			if pEntrega and guiGps then
				print("[Autofarm]: Casa com entrega encontrada:", casa.Name)

				local targetCFrame = nil
				if pEntrega:IsA("BasePart") then
					targetCFrame = pEntrega.CFrame
				elseif pEntrega:IsA("Model") then
					targetCFrame = pEntrega:GetPivot()
				end

				if targetCFrame then
					-- Teleporta o personagem de forma garantida
					character:PivotTo(targetCFrame + Vector3.new(0, 3, 0))

					-- Se houver ProximityPrompt na parte de entrega
					local prompt = pEntrega:FindFirstChildOfClass("ProximityPrompt") or pEntrega:FindFirstChildWhichIsA("ProximityPrompt", true)
					if prompt then
						prompt:InputHoldBegin()
						task.wait(prompt.HoldDuration + 0.1)
						prompt:InputHoldEnd()
					end

					task.wait(2)
					break
				end
			end
		end
	end
end)
