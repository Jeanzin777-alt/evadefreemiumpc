-- Script Auto Jump para Evade com Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criar janela
local Window = Rayfield:CreateWindow({
    Name = "Evade Script",
    LoadingTitle = "Bem-vindo!",
    LoadingSubtitle = "by carlin",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "EvadeScript",
        FileName = "Config"
    },
    KeySystem = false
})

-- Aba Main
local MainTab = Window:CreateTab("Main", 4483362458)

-- Variáveis
local autoJumpEnabled = false
local switchLagEnabled = false
local mobileButtonEnabled = false
local switchLagKey = Enum.KeyCode.X
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local originalPosition = humanoidRootPart.Position

-- Toggle para Auto Jump com descrição
MainTab:CreateToggle({
    Name = "Auto Jump",
    Description = "Para pular basta segurar o botão de pular",
    CurrentValue = false,
    Flag = "AutoJump",
    Callback = function(Value)
        autoJumpEnabled = Value
        if Value then
            print("✓ Auto Jump Ativado!")
        else
            print("✗ Auto Jump Desativado!")
        end
    end,
})

-- Toggle para Switch Lag
MainTab:CreateToggle({
    Name = "Switch Lag",
    Description = "Te teleporta para frente rapidamente para enganar inimigos",
    CurrentValue = false,
    Flag = "SwitchLag",
    Callback = function(Value)
        switchLagEnabled = Value
        if Value then
            print("✓ Switch Lag Ativado!")
        else
            print("✗ Switch Lag Desativado!")
        end
    end,
})

-- Keybind para mudar a tecla de Switch Lag
MainTab:CreateKeybind({
    Name = "Tecla Switch Lag",
    CurrentKeybind = "X",
    HoldToInteract = false,
    Flag = "SwitchLagKeybind",
    Callback = function(Keybind)
        switchLagKey = Keybind
        print("✓ Tecla de Switch Lag alterada para: " .. Keybind.Name)
    end,
})

-- Toggle para Mobile Button
MainTab:CreateToggle({
    Name = "Mobile Button",
    Description = "Mostra um botão arrastável para Switch Lag (Mobile)",
    CurrentValue = false,
    Flag = "MobileButton",
    Callback = function(Value)
        mobileButtonEnabled = Value
        if Value then
            print("✓ Mobile Button Ativado!")
            createMobileButton()
        else
            print("✗ Mobile Button Desativado!")
            if game.CoreGui:FindFirstChild("SwitchLagButton") then
                game.CoreGui:FindFirstChild("SwitchLagButton"):Destroy()
            end
        end
    end,
})

-- Função para detectar se está no chão (Evade)
local function isGrounded()
    local rayOrigin = humanoidRootPart.Position + Vector3.new(0, -3, 0)
    local rayDirection = Vector3.new(0, -1, 0)
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {character}
    
    local result = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    return result ~= nil
end

-- Função Switch Lag
local function activateSwitchLag()
    if not character or not humanoidRootPart then return end
    
    local currentPosition = humanoidRootPart.Position
    local camera = workspace.CurrentCamera
    
    -- Teleporta para frente na direção que está olhando
    local direction = camera.CFrame.LookVector
    local teleportDistance = 50
    local newPosition = currentPosition + direction * teleportDistance
    
    humanoidRootPart.CFrame = CFrame.new(newPosition)
    
    print("✓ Switch Lag ativado!")
end

-- Função para criar botão Mobile
local function createMobileButton()
    if game.CoreGui:FindFirstChild("SwitchLagButton") then
        game.CoreGui:FindFirstChild("SwitchLagButton"):Destroy()
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "SwitchLagButton"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game.CoreGui
    
    local button = Instance.new("TextButton")
    button.Name = "Button"
    button.Size = UDim2.new(0, 60, 0, 60)
    button.Position = UDim2.new(0, 20, 0, 20)
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Text = "LAG\nSWITCH"
    button.BorderSizePixel = 0
    button.Parent = screenGui
    
    -- Efeito visual no hover
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    end)
    
    -- Clique para ativar
    button.MouseButton1Down:Connect(function()
        if switchLagEnabled then
            activateSwitchLag()
        end
    end)
    
    -- Sistema de arrastar
    local dragging = false
    local dragStart
    local startPos
    
    button.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = button.Position
        end
    end)
    
    button.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    game:GetService("UserInputService").TouchMoved:Connect(function(input, gameProcessed)
        if dragging then
            local delta = input.Position - dragStart
            button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Detectar quando o jogador pressiona uma tecla específica para Switch Lag
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if switchLagEnabled and input.KeyCode == switchLagKey then
        activateSwitchLag()
    end
end)

-- Loop principal de Auto Jump
game:GetService("RunService").Heartbeat:Connect(function()
    if not character or not humanoid or humanoid.Health <= 0 then return end
    
    if autoJumpEnabled and game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then
        if isGrounded() then
            -- Pular no Evade
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Atualizar referências quando o personagem respawna
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    print("✓ Personagem respawned!")
end)

-- Notificação de conclusão
Rayfield:Notify({
    Title = "Script Carregado",
    Content = "Auto Jump Script para Evade pronto! Pressione X para Switch Lag",
    Duration = 3,
    Image = 4483362458,
})
