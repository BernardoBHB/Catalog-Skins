local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

if RunService:IsServer() then
    -- ===================================================
    -- 🖥️ PARTE 1: SERVIDOR (Muda o modelo para todos)
    -- ===================================================
    local evento = ReplicatedStorage:FindFirstChild("BHB_Event_Unico") 
    if not evento then
        evento = Instance.new("RemoteEvent", ReplicatedStorage)
        evento.Name = "BHB_Event_Unico"
    end

    evento.OnServerEvent:Connect(function(plr, nome, mesh, tex)
        local item = workspace:FindFirstChild(nome)
        if not item then return end
        
        local pfx = "rbxassetid://"
        if item:IsA("MeshPart") then
            item.MeshId = pfx .. mesh
            if tex ~= "" then item.TextureID = pfx .. tex end
        elseif item:IsA("Part") then
            local sm = item:FindFirstChildOfClass("SpecialMesh") or Instance.new("SpecialMesh", item)
            sm.MeshId = pfx .. mesh
            if tex ~= "" then sm.TextureId = pfx .. tex end
        end
    end)

    -- Injeta a parte visual no jogador 
    local function injetarUi(player)
        local pg = player:WaitForChild("PlayerGui")
        local clone = script:Clone()
        clone.Name = "BHB_ClientUI"
        clone.RunContext = Enum.RunContext.Client
        clone.Parent = pg
    end

    Players.PlayerAdded:Connect(injetarUi)
    for _, p in ipairs(Players:GetPlayers()) do injetarUi(p) end

elseif RunService:IsClient() then
    -- ===================================================
    -- 🎮 PARTE 2: CLIENTE (Interface e tecla Control)
    -- ===================================================
    local player = Players.LocalPlayer
    local pg = player:WaitForChild("PlayerGui")
    
    -- Evita duplicar se der reset
    if pg:FindFirstChild("BHB_UI_Main") then pg.BHB_UI_Main:Destroy() end

    local gui = Instance.new("ScreenGui", pg)
    gui.Name = "BHB_UI_Main"
    gui.ResetOnSpawn = false

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 300, 0, 260)
    frame.Position = UDim2.new(0.5, -150, 0.5, -130)
    frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    frame.Visible = false -- O PAINEL COMEÇA INVISÍVEL AQUI

    local function criarCaixa(txt, y)
        local cx = Instance.new("TextBox", frame)
        cx.Size = UDim2.new(0.9, 0, 0, 40)
        cx.Position = UDim2.new(0.05, 0, y, 0)
        cx.PlaceholderText = txt
        cx.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        cx.TextColor3 = Color3.fromRGB(255, 255, 255)
        cx.TextScaled = true
        return cx
    end

    local inNome = criarCaixa("Nome do Item no Workspace", 0.1)
    local inMesh = criarCaixa("ID do Mesh", 0.3)
    local inTex = criarCaixa("ID da Textura (Opcional)", 0.5)

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0.75, 0)
    btn.Text = "SUBSTITUIR MODELO"
    btn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 20

    btn.MouseButton1Click:Connect(function()
        if inNome.Text ~= "" and inMesh.Text ~= "" then
            ReplicatedStorage:WaitForChild("BHB_Event_Unico"):FireServer(inNome.Text, inMesh.Text, inTex.Text)
        end
    end)

    -- LÓGICA DA TECLA CONTROL PARA ABRIR/FECHAR
    game:GetService("UserInputService").InputBegan:Connect(function(input, digitando)
        -- Se o jogador estiver digitando no chat ou em uma caixa, ignora
        if digitando then return end 
        
        if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
            frame.Visible = not frame.Visible
        end
    end)
end
