-- LocalScript - Coloque dentro de StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ================= 1. CRIAÇÃO DA GUI INTERATIVA =================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoFarmGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Botão Toggle principal
local botao = Instance.new("TextButton")
botao.Name = "ToggleButton"
botao.Size = UDim2.new(0, 180, 0, 50)
botao.Position = UDim2.new(0.05, 0, 0.4, 0) -- Posição inicial no canto esquerdo
botao.BackgroundColor3 = Color3.fromRGB(220, 53, 69) -- Vermelho (OFF)
botao.TextColor3 = Color3.fromRGB(255, 255, 255)
botao.TextSize = 16
botao.Font = Enum.Font.SourceSansBold
botao.Text = "AUTOFARM: OFF"
botao.Parent = screenGui

-- Arredondamento do botão
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
uiCorner.Parent = botao

-- Borda estilizada
local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(255, 255, 255)
uiStroke.Transparency = 0.5
uiStroke.Parent = botao

-- ================= 2.SISTEMA DE ARRASTAR O BOTÃO (DRAGGABLE) =================
local dragging = false
local dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	botao.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

botao.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = botao.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

botao.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		update(input)
	end
end)

-- ================= 3. LÓGICA DO TOGGLE (ON / OFF) =================
local autoFarmAtivado = false

botao.MouseButton1Click:Connect(function()
	autoFarmAtivado = not autoFarmAtivado

	if autoFarmAtivado then
		botao.Text = "AUTOFARM: ON"
		botao.BackgroundColor3 = Color3.fromRGB(40, 167, 69) -- Verde (ON)
	else
		botao.Text = "AUTOFARM: OFF"
		botao.BackgroundColor3 = Color3.fromRGB(220, 53, 69) -- Vermelho (OFF)
	end
end)

-- ================= 4. LÓGICA DO AUTOFARM (TELEPORTE E COLETA) =================
task.spawn(function()
	local pastaEntregas = workspace:WaitForChild("Entregas", 10)

	if not pastaEntregas then
		warn("[Autofarm]: A pasta 'Entregas' não foi encontrada no Workspace.")
		return
	end

	while task.wait(0.5) do
		-- Se o toggle estiver desligado, não executa o teleporte
		if not autoFarmAtivado then continue end

		local character = player.Character
		local rootPart = character and character:FindFirstChild("HumanoidRootPart")

		if not rootPart then continue end

		-- Procura todas as casas dentro da pasta Entregas
		for _, casa in ipairs(pastaEntregas:GetChildren()) do
			if not autoFarmAtivado then break end

			if casa.Name == "Casa" then
				local pEntrega = casa:FindFirstChild("PEntrega")
				local guiGps = casa:FindFirstChild("GUI_GPS")

				-- Verifica se a casa possui as duas partes ao mesmo tempo
				if pEntrega and guiGps then
					-- Teleporta o personagem para o local
					rootPart.CFrame = pEntrega.CFrame + Vector3.new(0, 3, 0)

					-- Se houver um ProximityPrompt na entrega, ele interage segurando a tecla
					local prompt = pEntrega:FindFirstChildOfClass("ProximityPrompt")
					if prompt then
						prompt:InputHoldBegin()
						task.wait(prompt.HoldDuration + 0.1)
						prompt:InputHoldEnd()
					end

					-- Aguarda um tempo de segurança para concluir a entrega
					task.wait(1.5)
					break
				end
			end
		end
	end
end)
