local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- === SUA LISTA DE SKINS SALVAS ===
-- Aqui você pode colocar o ID da sua galera para ter acesso rápido
local skins = {
	{Nome = "Roblox Boy", Id = 1},
	{Nome = "Bacon Hair", Id = 11169651}, 
	{Nome = "Lucas (Exemplo)", Id = 156},
	{Nome = "João Pedro (Exemplo)", Id = 167},
	{Nome = "Filipe (Exemplo)", Id = 178},
	{Nome = "Slender", Id = 4397524},
	{Nome = "Korblox", Id = 132456} 
}

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
mainFrame.Size = UDim2.new(0, 350, 0, 500)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local uiCornerMain = Instance.new("UICorner")
uiCornerMain.CornerRadius = UDim.new(0, 10)
uiCornerMain.Parent = mainFrame

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
titulo.Text = "BHB PLATAFORM - Avatar Hub"
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

closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- === FUNÇÃO DE APLICAR SKIN ===
local function aplicarSkin(userId)
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		pcall(function()
			local humanoidDescription = Players:GetHumanoidDescriptionFromUserId(userId)
			humanoid:ApplyDescription(humanoidDescription)
		end)
	end
end

-- === SEÇÃO: ID CUSTOMIZADO ===
local customIdBox = Instance.new("TextBox")
customIdBox.Size = UDim2.new(0.65, 0, 0, 40)
customIdBox.Position = UDim2.new(0, 15, 0, 50)
customIdBox.PlaceholderText = "Digite qualquer ID..."
customIdBox.Text = ""
customIdBox.TextColor3 = Color3.fromRGB(255, 255, 255)
customIdBox.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
customIdBox.Font = Enum.Font.Gotham
customIdBox.TextSize = 14
customIdBox.Parent = mainFrame
Instance.new("UICorner", customIdBox).CornerRadius = UDim.new(0, 6)

local equipCustomBtn = Instance.new("TextButton")
equipCustomBtn.Size = UDim2.new(0.25, 0, 0, 40)
equipCustomBtn.Position = UDim2.new(0.65, 25, 0, 50)
equipCustomBtn.Text = "Equipar"
equipCustomBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
equipCustomBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
equipCustomBtn.Font = Enum.Font.GothamBold
equipCustomBtn.TextSize = 14
equipCustomBtn.Parent = mainFrame
Instance.new("UICorner", equipCustomBtn).CornerRadius = UDim.new(0, 6)

equipCustomBtn.MouseButton1Click:Connect(function()
	local id = tonumber(customIdBox.Text)
	if id then aplicarSkin(id) end
end)

-- === SEÇÃO: BARRA DE PESQUISA ===
local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -30, 0, 35)
searchBox.Position = UDim2.new(0, 15, 0, 100)
searchBox.PlaceholderText = "Pesquisar skin salva..."
searchBox.Text = ""
searchBox.TextColor3 = Color3.fromRGB(200, 200, 200)
searchBox.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
searchBox.Font = Enum.Font.Gotham
searchBox.TextSize = 14
searchBox.Parent = mainFrame
Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 6)

-- === LISTA DE BOTÕES (SCROLL) ===
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -30, 1, -155)
scroll.Position = UDim2.new(0, 15, 0, 145)
scroll.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
scroll.ScrollBarThickness = 5
scroll.BorderSizePixel = 0
scroll.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 8)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = scroll

local botoes = {}

-- Gera os botões da lista fixa
for i, skinData in ipairs(skins) do
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -10, 0, 45)
	btn.Text = "  " .. skinData.Nome
	btn.TextXAlignment = Enum.TextXAlignment.Left
	btn.TextSize = 15
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
	btn.Font = Enum.Font.GothamMedium
	btn.Parent = scroll
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

	btn.MouseButton1Click:Connect(function()
		aplicarSkin(skinData.Id)
	end)
	
	table.insert(botoes, {botao = btn, nome = string.lower(skinData.Nome)})
end

-- Ajusta o tamanho do scroll de acordo com a quantidade de botões
scroll.CanvasSize = UDim2.new(0, 0, 0, #skins * 53)

-- === LÓGICA DE PESQUISA ===
searchBox.Changed:Connect(function(propriedade)
	if propriedade == "Text" then
		local textoPesquisa = string.lower(searchBox.Text)
		local visiveis = 0
		
		for _, dados in ipairs(botoes) do
			if string.find(dados.nome, textoPesquisa) or textoPesquisa == "" then
				dados.botao.Visible = true
				visiveis = visiveis + 1
			else
				dados.botao.Visible = false
			end
		end
		
		scroll.CanvasSize = UDim2.new(0, 0, 0, visiveis * 53)
	end
end)
