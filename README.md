local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

if RunService:IsServer() then
    -- ===================================================
    -- 🖥️ SERVIDOR (Executa a troca para o mapa todo)
    -- ===================================================
    local evento = ReplicatedStorage:FindFirstChild("BHB_TrocaMeshEvent")
    if not evento then
        evento = Instance.new("RemoteEvent")
        evento.Name = "BHB_TrocaMeshEvent"
        evento.Parent = ReplicatedStorage
    end

    evento.OnServerEvent:Connect(function(player, nome, mesh, tex)
        local item = workspace:FindFirstChild(nome)
        if not item then return end
        
        local prefix = "rbxassetid://"
        if item:IsA("MeshPart") then
            item.MeshId = prefix .. mesh
            if tex ~= "" then item.TextureID = prefix .. tex end
        elseif item:IsA("Part") then
            local sm = item:FindFirstChildOfClass("SpecialMesh") or Instance.new("SpecialMesh", item)
            sm.MeshId = prefix .. mesh
            if tex ~= "" then sm.TextureId = prefix .. tex end
        end
    end)

    -- Injeta a parte visual (Cliente) diretamente no jogador
    local function prepararTela(player)
        local playerGui = player:WaitForChild("PlayerGui")
        local clone = script:Clone()
        clone.Name = "BHB_PainelVisual"
        clone.RunContext = Enum.RunContext.Client
        clone.Parent = playerGui
    end

    Players.PlayerAdded:Connect(prepararTela)
    for _, p in ipairs(Players:GetPlayers()) do prepararTela(p) end

elseif RunService:IsClient() then
    -- ===================================================
    -- 🎮 CLIENTE (Gera a interface na tela do jogador)
    -- ===================================================
    local player = Players.LocalPlayer
    local PlayerGui = player:WaitForChild("PlayerGui")
    
    -- Evita clonar a tela duas vezes
    if PlayerGui:FindFirstChild("BHB_TelaSubstituicao") then return end 
    
    local evento = ReplicatedStorage:WaitForChild("BHB_TrocaMeshEvent")
    
    local gui = Instance.new("ScreenGui", PlayerGui)
    gui.Name = "BHB_TelaSubstituicao"
    gui.ResetOnSpawn = false
    
    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 300, 0, 260)
    frame.Position = UDim2.new(0.5, -150, 0.5, -130)
    frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    
    -- Função para facilitar a criação das caixas de texto
    local function criarCaixa(texto, yPos)
        local caixa = Instance.new("TextBox", frame)
        caixa.Size = UDim2.new(0.9, 0, 0, 40)
        caixa.Position = UDim2.new(0.05, 0, yPos, 0)
        caixa.PlaceholderText = texto
        caixa.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        caixa.TextColor3 = Color3.fromRGB(255, 255, 255)
        caixa.TextScaled = true
        return caixa
    end
    
    -- As 3 caixas na ordem certa
    local inputNome = criarCaixa("Nome do Item (Workspace)", 0.1)
    local inputMesh = criarCaixa("ID do Mesh", 0.3)
    local inputTextura = criarCaixa("ID da Textura (Opcional)", 0.5)
    
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0.75, 0)
    btn.Text = "SUBSTITUIR"
    btn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 20
    
    -- Botão ativando o sistema
    btn.MouseButton1Click:Connect(function()
        if inputNome.Text ~= "" and inputMesh.Text ~= "" then
            evento:FireServer(inputNome.Text, inputMesh.Text, inputTextura.Text)
        end
    end)
end
