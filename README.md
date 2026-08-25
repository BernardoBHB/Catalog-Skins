local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

if RunService:IsServer() then
    -- =========================================================
    -- 🖥️ SERVIDOR: Cria as conexões e muda o item para todos
    -- =========================================================
    
    -- 1. Cria o canal de comunicação automaticamente (zero esforço seu)
    local eventoMudar = ReplicatedStorage:FindFirstChild("BHB_MudarMeshEvent")
    if not eventoMudar then
        eventoMudar = Instance.new("RemoteEvent")
        eventoMudar.Name = "BHB_MudarMeshEvent"
        eventoMudar.Parent = ReplicatedStorage
    end

    -- 2. Recebe a ordem do Hub e aplica no mapa
    eventoMudar.OnServerEvent:Connect(function(player, nomeItem, meshId, texturaId)
        if type(nomeItem) ~= "string" or type(meshId) ~= "string" then return end

        local item = workspace:FindFirstChild(nomeItem)
        if item then
            local prefixo = "rbxassetid://"
            local meshFormatado = prefixo .. meshId
            local texturaFormatada = (texturaId and texturaId ~= "") and (prefixo .. texturaId) or ""

            if item:IsA("MeshPart") then
                item.MeshId = meshFormatado
                if texturaFormatada ~= "" then item.TextureID = texturaFormatada end
                print("✅ [BHB System] Mesh mudado no MeshPart por:", player.Name)
                
            elseif item:IsA("Part") then
                local specialMesh = item:FindFirstChildOfClass("SpecialMesh")
                if not specialMesh then
                    specialMesh = Instance.new("SpecialMesh")
                    specialMesh.Parent = item
                end
                specialMesh.MeshId = meshFormatado
                if texturaFormatada ~= "" then specialMesh.TextureId = texturaFormatada end
                print("✅ [BHB System] Mesh mudado na Part por:", player.Name)
            end
        else
            warn("❌ [BHB System] O item '" .. nomeItem .. "' não foi encontrado no Workspace.")
        end
    end)

    -- 3. Injeta a interface na tela de quem entrar no jogo automaticamente
    local scriptCliente = script:Clone()
    scriptCliente.Name = "Hub_BHB_Interface"
    scriptCliente.RunContext = Enum.RunContext.Client
    scriptCliente.Parent = game:GetService("StarterPlayer").StarterPlayerScripts

    -- Garante que apareça para você mesmo testando sozinho no Studio
    for _, p in ipairs(Players:GetPlayers()) do
        local playerScripts = p:FindFirstChild("PlayerScripts")
        if playerScripts and not playerScripts:FindFirstChild(scriptCliente.Name) then
            scriptCliente:Clone().Parent = playerScripts
        end
    end

elseif RunService:IsClient() then
    -- =========================================================
    -- 🎮 CLIENTE: Desenha os 3 espaços (Nome, Mesh, Textura) na tela
    -- =========================================================
    local eventoMudar = ReplicatedStorage:WaitForChild("BHB_MudarMeshEvent")
    local player = Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")

    -- Previne duplicar a interface
    if playerGui:FindFirstChild("HubSubstituicaoBHB") then
        playerGui["HubSubstituicaoBHB"]:Destroy()
    end

    -- Cria o Hub visual do zero
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "HubSubstituicaoBHB"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 300, 0, 260)
    frame.Position = UDim2.new(0.5, -150, 0.5, -130)
    frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    frame.BorderSizePixel = 0
    frame.Parent = screenGui

    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 40)
    titulo.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
    titulo.Text = "Painel BHB de Modelos"
    titulo.Font = Enum.Font.SourceSansBold
    titulo.TextSize = 20
    titulo.Parent = frame

    local inputNome = Instance.new("TextBox")
    inputNome.Size = UDim2.new(0.9, 0, 0, 40)
    inputNome.Position = UDim2.new(0.05, 0, 0.2, 0)
    inputNome.PlaceholderText = "Nome do Item"
    inputNome.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    inputNome.TextColor3 = Color3.fromRGB(255, 255, 255)
    inputNome.TextScaled = true
    inputNome.Parent = frame

    local inputMesh = Instance.new("TextBox")
    inputMesh.Size = UDim2.new(0.9, 0, 0, 40)
    inputMesh.Position = UDim2.new(0.05, 0, 0.4, 0)
    inputMesh.PlaceholderText = "ID do Mesh"
    inputMesh.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    inputMesh.TextColor3 = Color3.fromRGB(255, 255, 255)
    inputMesh.TextScaled = true
    inputMesh.Parent = frame

    -- ESPAÇO DA TEXTURA AQUI
    local inputTextura = Instance.new("TextBox")
    inputTextura.Size = UDim2.new(0.9, 0, 0, 40)
    inputTextura.Position = UDim2.new(0.05, 0, 0.6, 0)
    inputTextura.PlaceholderText = "ID da Textura (Opcional)"
    inputTextura.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    inputTextura.TextColor3 = Color3.fromRGB(255, 255, 255)
    inputTextura.TextScaled = true
    inputTextura.Parent = frame

    local botaoAplicar = Instance.new("TextButton")
    botaoAplicar.Size = UDim2.new(0.9, 0, 0, 40)
    botaoAplicar.Position = UDim2.new(0.05, 0, 0.8, 0)
    botaoAplicar.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    botaoAplicar.TextColor3 = Color3.fromRGB(255, 255, 255)
    botaoAplicar.Text = "SUBSTITUIR"
    botaoAplicar.Font = Enum.Font.SourceSansBold
    botaoAplicar.TextSize = 20
    botaoAplicar.Parent = frame

    -- Ação quando você clica em substituir
    botaoAplicar.MouseButton1Click:Connect(function()
        local nomeItem = inputNome.Text
        local meshId = inputMesh.Text
        local texturaId = inputTextura.Text
        
        if nomeItem == "" or meshId == "" then
            warn("⚠️ BHB, preencha o nome do item e o ID do Mesh!")
            return
        end

        -- Manda os 3 dados (inclusive textura) para o servidor executar
        eventoMudar:FireServer(nomeItem, meshId, texturaId)
    end)
end
