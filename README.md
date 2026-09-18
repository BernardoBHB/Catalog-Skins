local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Caminho da pasta de entregas
local pastaEntregas = workspace:WaitForChild("Entregas")

-- Variável do Toggle (mude para false para parar o autofarm)
local toggleAtivado = true 

-- Função principal do Autofarm
local function iniciarAutoFarm()
    task.spawn(function()
        -- Loop infinito que roda enquanto o jogador estiver no jogo
        while task.wait(0.5) do 
            -- Se o toggle estiver desligado, ele ignora o código abaixo e espera o próximo ciclo
            if not toggleAtivado then continue end
            
            local character = player.Character
            local rootPart = character and character:FindFirstChild("HumanoidRootPart")
            
            if not rootPart then continue end

            -- Procura por todas as casas dentro da pasta Entregas
            for _, objeto in ipairs(pastaEntregas:GetChildren()) do
                if objeto.Name == "Casa" then
                    local pEntrega = objeto:FindFirstChild("PEntrega")
                    local guiGps = objeto:FindFirstChild("GUI_GPS")

                    -- Verifica se a casa possui as duas partes necessárias
                    if pEntrega and guiGps then
                        -- Teleporta o jogador para a parte PEntrega
                        -- Adicionamos +3 no eixo Y para o personagem não bugar dentro do chão
                        rootPart.CFrame = pEntrega.CFrame + Vector3.new(0, 3, 0)
                        
                        -- Se houver um ProximityPrompt (Segurar "E") na parte PEntrega, podemos ativá-lo via script
                        local prompt = pEntrega:FindFirstChildOfClass("ProximityPrompt")
                        if prompt then
                            prompt:InputHoldBegin()
                            task.wait(prompt.HoldDuration + 0.1)
                            prompt:InputHoldEnd()
                        end

                        -- Pausa para evitar teleportes instantâneos repetidos na mesma casa
                        task.wait(2)
                        
                        -- Quebra o loop 'for' para recomeçar a busca do zero no próximo ciclo do 'while'
                        break 
                    end
                end
            end
        end
    end)
end

-- Inicia o loop
iniciarAutoFarm()

-- Exemplo de como você conectaria isso a um botão de UI (Toggle):
--[[
local botaoToggle = script.Parent -- Supondo que o script esteja dentro do botão
botaoToggle.MouseButton1Click:Connect(function()
    toggleAtivado = not toggleAtivado
    if toggleAtivado then
        botaoToggle.Text = "Autofarm: ON"
    else
        botaoToggle.Text = "Autofarm: OFF"
    end
end)
]]--
