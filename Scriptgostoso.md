--[[
    Script: Interface GUI Avançada para Delta Executor
    Biblioteca: Rayfield UI Library
    Desenvolvedor: Script para Roblox
    Compatibilidade: Delta Executor
--]]

-- Verificação de ambiente e segurança
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Carregar biblioteca Rayfield (versão estável para Delta)
local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Rayfield/main/source"))()

-- Variáveis de controle de estado
local State = {
    WalkFlingActive = false,
    FlingActive = false,
    CurrentCharacter = nil,
    WalkFlingConnection = nil,
    FlingConnection = nil
}

-- Função para obter o personagem atual de forma segura
local function GetCharacter()
    local player = game.Players.LocalPlayer
    if player and player.Character then
        return player.Character
    end
    return nil
end

-- Função para parar o WalkFling
local function StopWalkFling()
    if State.WalkFlingConnection then
        State.WalkFlingConnection:Disconnect()
        State.WalkFlingConnection = nil
    end
    State.WalkFlingActive = false
end

-- Função para iniciar WalkFling
local function StartWalkFling()
    StopWalkFling() -- Garantir que não haja conflitos
    
    local character = GetCharacter()
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart then return end
    
    -- Parâmetros otimizados para WalkFling (velocidade moderada)
    local WALK_SPEED = 85
    local VELOCITY_FORCE = Vector3.new(0, 150, 0)
    
    State.WalkFlingConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if not State.WalkFlingActive then return end
        
        local currentChar = GetCharacter()
        if currentChar ~= character or not humanoid.Parent or not rootPart.Parent then
            StopWalkFling()
            return
        end
        
        -- Aplicar WalkFling
        humanoid.WalkSpeed = WALK_SPEED
        rootPart.Velocity = VELOCITY_FORCE
        rootPart.CFrame = rootPart.CFrame + rootPart.CFrame.LookVector * 2.5
    end)
    
    State.WalkFlingActive = true
end

-- Função para parar o Fling
local function StopFling()
    if State.FlingConnection then
        State.FlingConnection:Disconnect()
        State.FlingConnection = nil
    end
    
    -- Resetar velocidades do personagem
    local character = GetCharacter()
    if character then
        local humanoid = character:FindFirstChild("Humanoid")
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        
        if humanoid then
            humanoid.WalkSpeed = 16
            humanoid.JumpPower = 50
        end
        if rootPart then
            rootPart.Velocity = Vector3.new(0, 0, 0)
        end
    end
    
    State.FlingActive = false
end

-- Função para iniciar Fling (força extrema)
local function StartFling()
    StopFling() -- Garantir que não haja conflitos
    
    local character = GetCharacter()
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart then return end
    
    -- Parâmetros ajustáveis para força do Fling
    local FLING_FORCE = Vector3.new(0, 350, 0) -- Força vertical
    local HORIZONTAL_FORCE = 95 -- Força horizontal
    
    State.FlingConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if not State.FlingActive then return end
        
        local currentChar = GetCharacter()
        if currentChar ~= character or not humanoid.Parent or not rootPart.Parent then
            StopFling()
            return
        end
        
        -- Aplicar Fling extremo
        humanoid.WalkSpeed = 0
        humanoid.JumpPower = 0
        rootPart.Velocity = FLING_FORCE
        rootPart.CFrame = rootPart.CFrame + rootPart.CFrame.LookVector * HORIZONTAL_FORCE
        rootPart.AssemblyLinearVelocity = FLING_FORCE + (rootPart.CFrame.LookVector * HORIZONTAL_FORCE)
    end)
    
    State.FlingActive = true
end

-- Criar janela principal da GUI
local Window = Rayfield:CreateWindow({
    Name = "Advanced Script Hub",
    Icon = 0,
    LoadingTitle = "Carregando Scripts...",
    LoadingSubtitle = "Por favor, aguarde",
    Theme = "Dark", -- Dark Theme moderno
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "DeltaHub",
        FileName = "Config"
    }
})

-- TABS principais
local MainTab = Window:CreateTab("Main Scripts", 4483362458)
local FlingsTab = Window:CreateTab("Fling System", 4483362458)
local InfoTab = Window:CreateTab("Informações", 4483362458)

-- Seção: Scripts Principais
local ScriptsSection = MainTab:CreateSection("Scripts Principais")

-- Botão para Infinite Yield
MainTab:CreateButton({
    Name = "Carregar Infinite Yield",
    Callback = function()
        local success, errorMsg = pcall(function()
            loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Infinite-yield-fe-37483"))()
        end)
        
        if not success then
            Rayfield:Notify({
                Title = "Erro",
                Content = "Falha ao carregar Infinite Yield: " .. tostring(errorMsg),
                Duration = 3
            })
        else
            Rayfield:Notify({
                Title = "Sucesso",
                Content = "Infinite Yield carregado com sucesso!",
                Duration = 2
            })
        end
    end
})

-- Botão para Emotes/UGC
MainTab:CreateButton({
    Name = "Carregar Emotes/UGC Free",
    Callback = function()
        local success, errorMsg = pcall(function()
            loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Emot-UGC-Free-script-53262"))()
        end)
        
        if not success then
            Rayfield:Notify({
                Title = "Erro",
                Content = "Falha ao carregar Emotes: " .. tostring(errorMsg),
                Duration = 3
            })
        else
            Rayfield:Notify({
                Title = "Sucesso",
                Content = "Emotes/UGC carregado com sucesso!",
                Duration = 2
            })
        end
    end
})

-- Seção: Sistema de Fling
local FlingSection = FlingsTab:CreateSection("Controles de Fling")

-- Toggle para WalkFling
FlingsTab:CreateToggle({
    Name = "WalkFling (Moderado)",
    CurrentValue = false,
    Flag = "WalkFlingToggle",
    Callback = function(Value)
        if Value then
            -- Desativar Fling primeiro se estiver ativo
            if State.FlingActive then
                StopFling()
                Rayfield:SetToggle("FlingToggle", false)
                Rayfield:Notify({
                    Title = "Aviso",
                    Content = "Fling desativado para ativar WalkFling",
                    Duration = 1.5
                })
            end
            StartWalkFling()
            Rayfield:Notify({
                Title = "WalkFling",
                Content = "WalkFling ativado! (Velocidade moderada)",
                Duration = 1.5
            })
        else
            StopWalkFling()
            Rayfield:Notify({
                Title = "WalkFling",
                Content = "WalkFling desativado!",
                Duration = 1.5
            })
        end
    end
})

-- Toggle para Fling
FlingsTab:CreateToggle({
    Name = "Fling (Extremo)",
    CurrentValue = false,
    Flag = "FlingToggle",
    Callback = function(Value)
        if Value then
            -- Desativar WalkFling primeiro se estiver ativo
            if State.WalkFlingActive then
                StopWalkFling()
                Rayfield:SetToggle("WalkFlingToggle", false)
                Rayfield:Notify({
                    Title = "Aviso",
                    Content = "WalkFling desativado para ativar Fling",
                    Duration = 1.5
                })
            end
            StartFling()
            Rayfield:Notify({
                Title = "Fling",
                Content = "Fling extremo ativado!",
                Duration = 1.5
            })
        else
            StopFling()
            Rayfield:Notify({
                Title = "Fling",
                Content = "Fling desativado!",
                Duration = 1.5
            })
        end
    end
})

-- Botão para resetar velocidades
FlingsTab:CreateButton({
    Name = "Resetar Velocidades",
    Callback = function()
        StopWalkFling()
        StopFling()
        Rayfield:SetToggle("WalkFlingToggle", false)
        Rayfield:SetToggle("FlingToggle", false)
        
        Rayfield:Notify({
            Title = "Reset",
            Content = "Todas as velocidades foram resetadas!",
            Duration = 2
        })
    end
})

-- Seção: Informações
local InfoSection = InfoTab:CreateSection("Sobre o Script")

InfoTab:CreateLabel("Script Hub Avançado para Delta Executor")
InfoTab:CreateLabel("Versão: 1.0.0")
InfoTab:CreateLabel("Biblioteca: Rayfield UI")
InfoTab:CreateLabel("Compatível: Delta Executor")
InfoTab:CreateLabel(" ")
InfoTab:CreateLabel("⚠️ Aviso de Segurança:")
InfoTab:CreateLabel("Os scripts externos carregados via loadstring")
InfoTab:CreateLabel("são de responsabilidade de terceiros.")
InfoTab:CreateLabel("Execute apenas se confiar nas fontes!")

-- Botão de informações de performance
InfoTab:CreateButton({
    Name = "Status do Sistema",
    Callback = function()
        Rayfield:Notify({
            Title = "Status",
            Content = string.format("WalkFling: %s | Fling: %s", 
                State.WalkFlingActive and "Ativo" or "Inativo",
                State.FlingActive and "Ativo" or "Inativo"),
            Duration = 2
        })
    end
})

-- Limpeza automática quando o personagem morrer
game.Players.LocalPlayer.CharacterAdded:Connect(function(character)
    StopWalkFling()
    StopFling()
    Rayfield:SetToggle("WalkFlingToggle", false)
    Rayfield:SetToggle("FlingToggle", false)
end)

-- Notificação inicial
Rayfield:Notify({
    Title = "Script Hub Carregado",
    Content = "Interface pronta para uso!",
    Duration = 3
})

print("Script GUI carregado com sucesso! - Delta Executor Compatível")
