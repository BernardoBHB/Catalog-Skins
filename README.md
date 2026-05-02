local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer

-- === SISTEMA DE SALVAMENTO (EXECUTOR) ===
local arquivoSave = "BHB_SkinsSalvas.json"
local skins = {}

if isfile and isfile(arquivoSave) then
	local sucesso, resultado = pcall(function()
		return HttpService:JSONDecode(readfile(arquivoSave))
	end)
	if sucesso and type(resultado) == "table" then
		skins = resultado
	end
end

local function salvarSkinsNoArquivo()
	if writefile then
		writefile(arquivoSave, HttpService:JSONEncode(skins))
	end
end

-- === PROTEÇÃO E CRIAÇÃO DA GUI ===
if CoreGui:FindFirstChild("BHB_AvatarHub") then
	CoreGui.BHB_AvatarHub:Destroy()
end
if player:WaitForChild("PlayerGui"):FindFirstChild("BHB_AvatarHub") then
	player.PlayerGui.BHB_AvatarHub:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BHB_AvatarHub"
screenGui.ResetOnSpawn = false

local success = pcall(function() screenGui.Parent = CoreGui end)
if not success then screenGui.Parent = player:WaitForChild("PlayerGui") end

-- === JANELA PRINCIPAL ===
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 350, 0, 450)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local uiCornerMain = Instance.new("UICorner")
uiCornerMain.CornerRadius = UDim.new(0, 10)
uiCornerMain.Parent = mainFrame

-- === SISTEMA DE ABRIR/FECHAR COM O "CONTROL" ===
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	-- Ignora se o jogador estiver digitando no chat ou em uma caixa de texto
	if gameProcessed then return end
	
	-- Verifica se a tecla apertada foi o Control (Esquerdo ou Direito)
	if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
		screenGui.Enabled = not screenGui.Enabled
	end
end)

-- === SISTEMA DE DRAG (ARRASTAR) ===
local dragging, dragInput, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)
mainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- === BARRA DE TÍTULO ===
local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, -40, 0, 40)
titulo.Position = UDim2.new(0, 15, 0, 0)
titulo.Text = "BHB PLATAFORM - Avatar"
titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
titulo.BackgroundTransparency = 1
titulo.TextSize = 18
titulo.Font = Enum.Font.GothamBold
titulo.TextXAlignment = Enum.TextXAlignment.Left
titulo.Parent = mainFrame

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -35, 0, 5)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 60, 60)
closeBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.Parent = mainFrame
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

-- O botão X agora apenas esconde a interface, igual a tecla Control
closeBtn.MouseButton1Click:Connect(function() 
	screenGui.Enabled = false 
end)

-- === FUNÇÃO DE APLICAR SKIN ===
local function aplicarSkin(userId)
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	
	if humanoid then
		local s, e = pcall(function()
			local humanoidDescription = Players:GetHumanoidDescriptionFromUserId(userId)
			humanoid:ApplyDescription(humanoidDescription)
		end)
		
		if not s then
			-- Mostra o erro no console (F9) para você saber o motivo exato
			warn("FALHA AO APLICAR SKIN: Seu executor não tem permissão de Client-Side para o ApplyDescription.")
			warn("Erro gerado: " .. tostring(e))
			
			StarterGui:SetCore("SendNotification", {
				Title = "Bloqueado pelo Executor",
				Text = "Olhe o console (F9). O executor não tem permissão para isso.",
				Duration = 5
			})
		end
	end
end

-- === ÁREA DE ADICIONAR E SALVAR SKINS ===
local nomeInput = Instance.new("TextBox")
nomeInput.Size = UDim2.new(0.45, 0, 0, 35)
nomeInput.Position = UDim2.new(0, 15, 0, 50)
nomeInput.PlaceholderText = "Nome da Skin"
nomeInput.Text = ""
nomeInput.TextColor3 = Color3.fromRGB(255, 255, 255)
nomeInput.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
nomeInput.Font = Enum.Font.Gotham
nomeInput.TextSize = 14
nomeInput.Parent = mainFrame
Instance.new("UICorner", nomeInput).CornerRadius = UDim.new(0, 6)

local idInput = Instance.new("TextBox")
idInput.Size = UDim2.new(0.45, 0, 0, 35)
idInput.Position = UDim2.new(0.5, 2, 0, 50)
idInput.PlaceholderText = "Somente ID (Números)"
idInput.Text = ""
idInput.TextColor3 = Color3.fromRGB(255, 255, 255)
idInput.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
idInput.Font = Enum.Font.Gotham
idInput.TextSize = 14
idInput.Parent = mainFrame
Instance.new("UICorner", idInput).CornerRadius = UDim.new(0, 6)

local equiparBtn = Instance.new("TextButton")
equiparBtn.Size = UDim2.new(0.45, 0, 0, 35)
equiparBtn.Position = UDim2.new(0, 15, 0, 95)
equiparBtn.Text = "Só Equipar"
equiparBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
equiparBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 110)
equiparBtn.Font = Enum.Font.GothamBold
equiparBtn.TextSize = 14
equiparBtn.Parent = mainFrame
Instance.new("UICorner", equiparBtn).CornerRadius = UDim.new(0, 6)

local salvarBtn = Instance.new("TextButton")
salvarBtn.Size = UDim2.new(0.45, 0, 0, 35)
salvarBtn.Position = UDim2.new(0.5, 2, 0, 95)
salvarBtn.Text = "Salvar na Lista"
salvarBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
salvarBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
salvarBtn.Font = Enum.Font.GothamBold
salvarBtn.TextSize = 14
salvarBtn.Parent = mainFrame
Instance.new("UICorner", salvarBtn).CornerRadius = UDim.new(0, 6)

-- === LISTA DE BOTÕES (SCROLL) ===
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -30, 1, -150)
scroll.Position = UDim2.new(0, 15, 0, 140)
scroll.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
scroll.ScrollBarThickness = 5
scroll.BorderSizePixel = 0
scroll.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = scroll

local function atualizarLista()
	for _, child in ipairs(scroll:GetChildren()) do
		if child:IsA("Frame") then child:Destroy() end
	end
	
	for i, skinData in ipairs(skins) do
		local itemFrame = Instance.new("Frame")
		itemFrame.Size = UDim2.new(1, -10, 0, 40)
		itemFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
		itemFrame.Parent = scroll
		Instance.new("UICorner", itemFrame).CornerRadius = UDim.new(0, 6)
		
		local btnSkin = Instance.new("TextButton")
		btnSkin.Size = UDim2.new(1, -40, 1, 0)
		btnSkin.BackgroundTransparency = 1
		btnSkin.Text = "  " .. skinData.Nome
		btnSkin.TextXAlignment = Enum.TextXAlignment.Left
		btnSkin.TextColor3 = Color3.fromRGB(255, 255, 255)
		btnSkin.Font = Enum.Font.GothamMedium
		btnSkin.TextSize = 14
		btnSkin.Parent = itemFrame
		
		local btnDeletar = Instance.new("TextButton")
		btnDeletar.Size = UDim2.new(0, 30, 0, 30)
		btnDeletar.Position = UDim2.new(1, -35, 0, 5)
		btnDeletar.Text = "X"
		btnDeletar.TextColor3 = Color3.fromRGB(255, 80, 80)
		btnDeletar.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
		btnDeletar.Font = Enum.Font.GothamBold
		btnDeletar.TextSize = 14
		btnDeletar.Parent = itemFrame
		Instance.new("UICorner", btnDeletar).CornerRadius = UDim.new(0, 6)
		
		btnSkin.MouseButton1Click:Connect(function()
			aplicarSkin(skinData.Id)
		end)
		
		btnDeletar.MouseButton1Click:Connect(function()
			table.remove(skins, i)
			salvarSkinsNoArquivo()
			atualizarLista()
		end)
	end
	
	scroll.CanvasSize = UDim2.new(0, 0, 0, #skins * 45)
end

-- === LÓGICA DOS BOTÕES SUPERIORES ===
equiparBtn.MouseButton1Click:Connect(function()
	local id = tonumber(idInput.Text)
	if id then aplicarSkin(id) end
end)

salvarBtn.MouseButton1Click:Connect(function()
	local nome = nomeInput.Text
	local id = tonumber(idInput.Text)
	
	if nome ~= "" and id then
		table.insert(skins, {Nome = nome, Id = id})
		salvarSkinsNoArquivo()
		atualizarLista()
		
		nomeInput.Text = ""
		idInput.Text = ""
	end
end)

atualizarLista()
