local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- === SISTEMA DE SALVAMENTO ===
local arquivoSave = "BHB_Skins_V5.json"
local skins = {}

if isfile and isfile(arquivoSave) then
	local s, r = pcall(function() return HttpService:JSONDecode(readfile(arquivoSave)) end)
	if s and type(r) == "table" then skins = r end
end

local function salvarSkins()
	if writefile then writefile(arquivoSave, HttpService:JSONEncode(skins)) end
end

-- === PROTEÇÃO DE GUI ===
if CoreGui:FindFirstChild("BHB_AvatarV5") then CoreGui.BHB_AvatarV5:Destroy() end
if player:WaitForChild("PlayerGui"):FindFirstChild("BHB_AvatarV5") then player.PlayerGui.BHB_AvatarV5:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BHB_AvatarV5"
screenGui.ResetOnSpawn = false
pcall(function() screenGui.Parent = CoreGui end)
if screenGui.Parent == nil then screenGui.Parent = player:WaitForChild("PlayerGui") end

-- === INTERFACE PRINCIPAL ===
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 380, 0, 480)
mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)
local strokeMain = Instance.new("UIStroke")
strokeMain.Color = Color3.fromRGB(0, 150, 255)
strokeMain.Thickness = 2
strokeMain.Parent = mainFrame

-- Toggle (Control)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if not gameProcessed and (input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl) then
		screenGui.Enabled = not screenGui.Enabled
	end
end)

-- Sistema de Arrastar
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

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, -40, 0, 50)
titulo.Position = UDim2.new(0, 20, 0, 0)
titulo.Text = "BHB PLATAFORM - Avatar"
titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
titulo.BackgroundTransparency = 1
titulo.TextSize = 22
titulo.Font = Enum.Font.GothamBlack
titulo.TextXAlignment = Enum.TextXAlignment.Left
titulo.Parent = mainFrame

-- === A CORREÇÃO DEFINITIVA (ROUPAS, CORES E 3D) ===
local function aplicarSkinManual(userId)
	local char = player.Character or player.CharacterAdded:Wait()
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	
	-- Baixa a aparência completa do Roblox
	local sucesso, appearance = pcall(function()
		return Players:GetCharacterAppearanceAsync(userId)
	end)
	
	if sucesso and appearance then
		-- 1. Destrói as roupas, cores e acessórios antigos do seu personagem
		for _, v in pairs(char:GetChildren()) do
			if v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic") or v:IsA("Accessory") or v:IsA("BodyColors") then
				v:Destroy()
			end
		end
		
		-- 2. Copia os itens novos para o seu personagem
		for _, item in pairs(appearance:GetChildren()) do
			-- Roupas e Cores de Pele
			if item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic") or item:IsA("BodyColors") then
				item:Clone().Parent = char
				
			-- Acessórios 3D (Cabelos, Chapéus, etc)
			elseif item:IsA("Accessory") then
				if humanoid then
					-- O AddAccessory é a forma mais segura de equipar itens 3D sem quebrar o avatar
					humanoid:AddAccessory(item:Clone())
				else
					item:Clone().Parent = char
				end
				
			-- Rosto (Face)
			elseif item:IsA("Decal") and item.Name == "face" then
				local head = char:FindFirstChild("Head")
				if head then
					local oldFace = head:FindFirstChildOfClass("Decal")
					if oldFace then oldFace:Destroy() end
					item:Clone().Parent = head
				end
			end
		end
		
		-- Limpa o lixo baixado da memória
		appearance:Destroy()
	end
end

-- === INPUTS E BOTÕES ===
local function criarInput(placeholder, posX, posY, sizeX)
	local txt = Instance.new("TextBox")
	txt.Size = UDim2.new(sizeX, 0, 0, 40)
	txt.Position = UDim2.new(0, posX, 0, posY)
	txt.PlaceholderText = placeholder
	txt.Text = ""
	txt.TextColor3 = Color3.fromRGB(255, 255, 255)
	txt.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
	txt.Font = Enum.Font.Gotham
	txt.TextSize = 14
	txt.Parent = mainFrame
	Instance.new("UICorner", txt).CornerRadius = UDim.new(0, 8)
	Instance.new("UIStroke", txt).Color = Color3.fromRGB(50, 50, 65)
	return txt
end

local nomeInput = criarInput("Nome", 20, 60, 0.42)
local idInput = criarInput("ID do Jogador", 195, 60, 0.42)

local function criarBotao(texto, posX, posY, sizeX, corBase)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(sizeX, 0, 0, 40)
	btn.Position = UDim2.new(0, posX, 0, posY)
	btn.Text = texto
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.BackgroundColor3 = corBase
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.AutoButtonColor = false
	btn.Parent = mainFrame
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
	
	btn.MouseEnter:Connect(function() TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(corBase.R*255 + 30, corBase.G*255 + 30, corBase.B*255 + 30)}):Play() end)
	btn.MouseLeave:Connect(function() TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = corBase}):Play() end)
	
	return btn
end

local equiparBtn = criarBotao("Equipar Agora", 20, 110, 0.42, Color3.fromRGB(80, 80, 100))
local salvarBtn = criarBotao("Salvar Skin", 195, 110, 0.42, Color3.fromRGB(0, 130, 255))

-- === ÁREA DA LISTA ===
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -40, 1, -180)
scroll.Position = UDim2.new(0, 20, 0, 165)
scroll.BackgroundColor3 = Color3.fromRGB(22, 22, 30)
scroll.ScrollBarThickness = 4
scroll.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
scroll.BorderSizePixel = 0
scroll.Parent = mainFrame
Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 8)

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 8)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = scroll
Instance.new("UIPadding", scroll).PaddingTop = UDim.new(0, 8)

local function atualizarLista()
	for _, child in ipairs(scroll:GetChildren()) do
		if child:IsA("Frame") then child:Destroy() end
	end
	
	for i, skinData in ipairs(skins) do
		local itemFrame = Instance.new("Frame")
		itemFrame.Size = UDim2.new(1, -15, 0, 45)
		itemFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
		itemFrame.Parent = scroll
		Instance.new("UICorner", itemFrame).CornerRadius = UDim.new(0, 8)
		
		local btnSkin = Instance.new("TextButton")
		btnSkin.Size = UDim2.new(1, -50, 1, 0)
		btnSkin.BackgroundTransparency = 1
		btnSkin.Text = "   " .. skinData.Nome
		btnSkin.TextXAlignment = Enum.TextXAlignment.Left
		btnSkin.TextColor3 = Color3.fromRGB(220, 220, 220)
		btnSkin.Font = Enum.Font.GothamMedium
		btnSkin.TextSize = 15
		btnSkin.Parent = itemFrame
		
		local btnDeletar = Instance.new("TextButton")
		btnDeletar.Size = UDim2.new(0, 35, 0, 35)
		btnDeletar.Position = UDim2.new(1, -40, 0, 5)
		btnDeletar.Text = "X"
		btnDeletar.TextColor3 = Color3.fromRGB(255, 70, 70)
		btnDeletar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
		btnDeletar.Font = Enum.Font.GothamBold
		btnDeletar.TextSize = 14
		btnDeletar.Parent = itemFrame
		Instance.new("UICorner", btnDeletar).CornerRadius = UDim.new(0, 6)
		
		btnSkin.MouseButton1Click:Connect(function() aplicarSkinManual(skinData.Id) end)
		btnDeletar.MouseButton1Click:Connect(function()
			table.remove(skins, i)
			salvarSkins()
			atualizarLista()
		end)
	end
	scroll.CanvasSize = UDim2.new(0, 0, 0, #skins * 53)
end

equiparBtn.MouseButton1Click:Connect(function()
	local id = tonumber(idInput.Text)
	if id then aplicarSkinManual(id) end
end)

salvarBtn.MouseButton1Click:Connect(function()
	local nome = nomeInput.Text
	local id = tonumber(idInput.Text)
	if nome ~= "" and id then
		table.insert(skins, {Nome = nome, Id = id})
		salvarSkins()
		atualizarLista()
		nomeInput.Text = ""
		idInput.Text = ""
	end
end)

atualizarLista()
