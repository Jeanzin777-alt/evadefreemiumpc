-- Script Auto Jump para Evade com UI Customizada
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Variáveis
local autoJumpEnabled = false
local mobileAutoJumpEnabled = false
local switchLagEnabled = false
local mobileButtonEnabled = false
local autoCrouchEnabled = false
local autoCrouchButtonEnabled = false
local switchLagKey = Enum.KeyCode.X
local autoCrouchKey = Enum.KeyCode.C
local autoCrouchActive = false
local isMinimized = false

-- Função para criar a UI
local function createUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "EvadeScriptUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game.CoreGui

    -- Frame Principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 350, 0, 500)
    mainFrame.Position = UDim2.new(0, 20, 0, 20)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = mainFrame

    -- Barra de Título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 40)
    titleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = titleBar

    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 14
    titleLabel.Text = "Evade Script - by carlin"
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar

    -- Botão Minimizar
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Name = "MinimizeBtn"
    minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
    minimizeBtn.Position = UDim2.new(1, -70, 0, 5)
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    minimizeBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
    minimizeBtn.TextSize = 12
    minimizeBtn.Text = "_"
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Parent = titleBar
    
    local minimizeCorner = Instance.new("UICorner")
    minimizeCorner.CornerRadius = UDim.new(0, 5)
    minimizeCorner.Parent = minimizeBtn

    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseBtn"
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -35, 0, 5)
    closeBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 12
    closeBtn.Text = "X"
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 5)
    closeCorner.Parent = closeBtn

    -- ScrollingFrame para conteúdo
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Name = "ScrollFrame"
    scrollFrame.Size = UDim2.new(1, 0, 1, -40)
    scrollFrame.Position = UDim2.new(0, 0, 0, 40)
    scrollFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    scrollFrame.BorderSizePixel = 0
    scrollFrame.ScrollBarThickness = 5
    scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 600)
    scrollFrame.Parent = mainFrame
    
    local scrollCorner = Instance.new("UICorner")
    scrollCorner.CornerRadius = UDim.new(0, 10)
    scrollCorner.Parent = scrollFrame

    -- UIListLayout para organizar elementos
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 10)
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Parent = scrollFrame

    -- Função para criar Toggle
    local function createToggle(name, description, callback)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Size = UDim2.new(1, -20, 0, 50)
        toggleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        toggleFrame.BorderSizePixel = 0
        toggleFrame.Parent = scrollFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 5)
        toggleCorner.Parent = toggleFrame

        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(0.75, 0, 0, 20)
        textLabel.Position = UDim2.new(0, 10, 0, 5)
        textLabel.BackgroundTransparency = 1
        textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        textLabel.TextSize = 12
        textLabel.Text = name
        textLabel.Font = Enum.Font.GothamBold
        textLabel.TextXAlignment = Enum.TextXAlignment.Left
        textLabel.Parent = toggleFrame

        local descLabel = Instance.new("TextLabel")
        descLabel.Size = UDim2.new(0.75, 0, 0, 15)
        descLabel.Position = UDim2.new(0, 10, 0, 25)
        descLabel.BackgroundTransparency = 1
        descLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        descLabel.TextSize = 10
        descLabel.Text = description
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.Parent = toggleFrame

        local toggleBtn = Instance.new("TextButton")
        toggleBtn.Size = UDim2.new(0, 40, 0, 20)
        toggleBtn.Position = UDim2.new(1, -50, 0, 15)
        toggleBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggleBtn.TextSize = 10
        toggleBtn.Text = "OFF"
        toggleBtn.Font = Enum.Font.GothamBold
        toggleBtn.BorderSizePixel = 0
        toggleBtn.Parent = toggleFrame
        
        local toggleCornerBtn = Instance.new("UICorner")
        toggleCornerBtn.CornerRadius = UDim.new(0, 5)
        toggleCornerBtn.Parent = toggleBtn

        local isActive = false
        toggleBtn.MouseButton1Click:Connect(function()
            isActive = not isActive
            if isActive then
                toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
                toggleBtn.Text = "ON"
            else
                toggleBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                toggleBtn.Text = "OFF"
            end
            callback(isActive)
        end)
    end

    -- Função para criar Keybind
    local function createKeybind(name, defaultKey, callback)
        local keybindFrame = Instance.new("Frame")
        keybindFrame.Size = UDim2.new(1, -20, 0, 40)
        keybindFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        keybindFrame.BorderSizePixel = 0
        keybindFrame.Parent = scrollFrame
        
        local keybindCorner = Instance.new("UICorner")
        keybindCorner.CornerRadius = UDim.new(0, 5)
        keybindCorner.Parent = keybindFrame

        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(0.6, 0, 1, 0)
        textLabel.Position = UDim2.new(0, 10, 0, 0)
        textLabel.BackgroundTransparency = 1
        textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        textLabel.TextSize = 12
        textLabel.Text = name
        textLabel.Font = Enum.Font.GothamBold
        textLabel.TextXAlignment = Enum.TextXAlignment.Left
        textLabel.Parent = keybindFrame

        local keyBtn = Instance.new("TextButton")
        keyBtn.Size = UDim2.new(0, 70, 0, 25)
        keyBtn.Position = UDim2.new(1, -80, 0, 7)
        keyBtn.BackgroundColor3 = Color3.fromRGB(50, 100, 200)
        keyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        keyBtn.TextSize = 11
        keyBtn.Text = defaultKey.Name
        keyBtn.Font = Enum.Font.GothamBold
        keyBtn.BorderSizePixel = 0
        keyBtn.Parent = keybindFrame
        
        local keyCorner = Instance.new("UICorner")
        keyCorner.CornerRadius = UDim.new(0, 5)
        keyCorner.Parent = keyBtn

        local listening = false
        keyBtn.MouseButton1Click:Connect(function()
            listening = true
            keyBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 0)
            keyBtn.Text = "..."
        end)

        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if listening and input.UserInputType == Enum.UserInputType.Keyboard then
                listening = false
                keyBtn.BackgroundColor3 = Color3.fromRGB(50, 100, 200)
                keyBtn.Text = input.KeyCode.Name
                callback(input.KeyCode)
            end
        end)
    end

    -- Função para criar Divider
    local function createDivider()
        local divider = Instance.new("Frame")
        divider.Size = UDim2.new(1, -20, 0, 1)
        divider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        divider.BorderSizePixel = 0
        divider.Parent = scrollFrame
    end

    -- Adicionar elementos
    createToggle("Auto Jump", "Pula automaticamente ao encostar no chão", function(value)
        autoJumpEnabled = value
        print(value and "✓ Auto Jump Ativado!" or "✗ Auto Jump Desativado!")
    end)

    createToggle("Mobile Auto Jump Button", "Mostra botão arrastável", function(value)
        mobileAutoJumpEnabled = value
        if value then
            createMobileAutoJumpButton()
        else
            if game.CoreGui:FindFirstChild("AutoJumpButton") then
                game.CoreGui:FindFirstChild("AutoJumpButton"):Destroy()
            end
        end
    end)

    createDivider()

    createToggle("Switch Lag", "Trava por 0.5 segundos", function(value)
        switchLagEnabled = value
        print(value and "✓ Switch Lag Ativado!" or "✗ Switch Lag Desativado!")
    end)

    createKeybind("Tecla Switch Lag", Enum.KeyCode.X, function(key)
        switchLagKey = key
        print("✓ Tecla alterada para: " .. key.Name)
    end)

    createToggle("Mobile Button Switch Lag", "Mostra botão arrastável", function(value)
        mobileButtonEnabled = value
        if value then
            createMobileButton()
        else
            if game.CoreGui:FindFirstChild("SwitchLagButton") then
                game.CoreGui:FindFirstChild("SwitchLagButton"):Destroy()
            end
        end
    end)

    createDivider()

    createToggle("Auto Crouch", "Spamma crouch automaticamente", function(value)
        autoCrouchEnabled = value
        if not value then
            autoCrouchActive = false
        end
        print(value and "✓ Auto Crouch Ativado!" or "✗ Auto Crouch Desativado!")
    end)

    createKeybind("Tecla Auto Crouch", Enum.KeyCode.C, function(key)
        autoCrouchKey = key
        print("✓ Tecla alterada para: " .. key.Name)
    end)

    createToggle("Mobile Button Auto Crouch", "Mostra botão arrastável", function(value)
        autoCrouchButtonEnabled = value
        if value then
            createMobileAutoCrouchButton()
        else
            if game.CoreGui:FindFirstChild("AutoCrouchButton") then
                game.CoreGui:FindFirstChild("AutoCrouchButton"):Destroy()
            end
        end
    end)

    -- Função de arrastar
    local dragging = false
    local dragStart
    local startPos

    titleBar.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
        end
    end)

    titleBar.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input, gameProcessed)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    -- Botão Minimizar
    minimizeBtn.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        if isMinimized then
            scrollFrame.Visible = false
            mainFrame.Size = UDim2.new(0, 350, 0, 40)
            minimizeBtn.Text = "+"
        else
            scrollFrame.Visible = true
            mainFrame.Size = UDim2.new(0, 350, 0, 500)
            minimizeBtn.Text = "_"
        end
    end)

    -- Botão Fechar
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)

    return screenGui
end

-- Função para detectar se está no chão
local function isGrounded()
    if not character or not humanoidRootPart then return false end
    
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
    
    print("✓ Switch Lag ativado!")
    
    pcall(function()
        local originalVelocity = humanoidRootPart.AssemblyLinearVelocity
        humanoidRootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        
        local originalAngularVelocity = humanoidRootPart.AssemblyAngularVelocity
        humanoidRootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        
        local camera = workspace.CurrentCamera
        local originalCFrame = camera.CFrame
        
        local connection
        connection = RunService.Heartbeat:Connect(function()
            if character and humanoidRootPart and character.Parent then
                humanoidRootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                humanoidRootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                camera.CFrame = originalCFrame
            end
        end)
        
        wait(0.5)
        
        connection:Disconnect()
        if character and humanoidRootPart and character.Parent then
            humanoidRootPart.AssemblyLinearVelocity = originalVelocity
            humanoidRootPart.AssemblyAngularVelocity = originalAngularVelocity
        end
    end)
end

-- Função para ativar Auto Crouch
local function startAutoCrouch()
    autoCrouchActive = true
    while autoCrouchActive and autoCrouchEnabled do
        keyPress(autoCrouchKey)
        wait(0.50)
    end
end

-- Função para parar Auto Crouch
local function stopAutoCrouch()
    autoCrouchActive = false
end

-- Função para simular pressionamento de tecla
function keyPress(key)
    pcall(function()
        local VirtualInputManager = game:GetService("VirtualInputManager")
        VirtualInputManager:SendKeyEvent(true, key, false, game)
        wait(0.05)
        VirtualInputManager:SendKeyEvent(false, key, false, game)
    end)
end

-- Função para criar botão Mobile Auto Jump
function createMobileAutoJumpButton()
    if game.CoreGui:FindFirstChild("AutoJumpButton") then
        game.CoreGui:FindFirstChild("AutoJumpButton"):Destroy()
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutoJumpButton"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game.CoreGui
    
    local button = Instance.new("TextButton")
    button.Name = "Button"
    button.Size = UDim2.new(0, 60, 0, 60)
    button.Position = UDim2.new(0, 400, 0, 20)
    button.BackgroundColor3 = Color3.fromRGB(66, 245, 123)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 10
    button.Text = "AUTO\nJUMP"
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamBold
    button.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button
    
    local isActive = false
    
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(100, 255, 150)
    end)
    
    button.MouseLeave:Connect(function()
        if isActive then
            button.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
        else
            button.BackgroundColor3 = Color3.fromRGB(66, 245, 123)
        end
    end)
    
    button.MouseButton1Down:Connect(function()
        isActive = not isActive
        autoJumpEnabled = isActive
        if isActive then
            button.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
        else
            button.BackgroundColor3 = Color3.fromRGB(66, 245, 123)
        end
    end)
    
    local dragging = false
    local dragStart
    local startPos
    
    button.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = button.Position
        end
    end)
    
    button.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    game:GetService("UserInputService").InputChanged:Connect(function(input, gameProcessed)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = input.Position - dragStart
            button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Função para criar botão Mobile Switch Lag
function createMobileButton()
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
    button.Position = UDim2.new(0, 400, 0, 100)
    button.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Text = "LAG\nSWITCH"
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamBold
    button.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button
    
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(255, 150, 150)
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    end)
    
    button.MouseButton1Down:Connect(function()
        if switchLagEnabled then
            activateSwitchLag()
        end
    end)
    
    local dragging = false
    local dragStart
    local startPos
    
    button.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = button.Position
        end
    end)
    
    button.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    game:GetService("UserInputService").InputChanged:Connect(function(input, gameProcessed)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = input.Position - dragStart
            button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Função para criar botão Mobile Auto Crouch
function createMobileAutoCrouchButton()
    if game.CoreGui:FindFirstChild("AutoCrouchButton") then
        game.CoreGui:FindFirstChild("AutoCrouchButton"):Destroy()
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutoCrouchButton"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game.CoreGui
    
    local button = Instance.new("TextButton")
    button.Name = "Button"
    button.Size = UDim2.new(0, 60, 0, 60)
    button.Position = UDim2.new(0, 400, 0, 180)
    button.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 10
    button.Text = "AUTO\nCROUCH"
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamBold
    button.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button
    
    local isActive = false
    
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(150, 200, 255)
    end)
    
    button.MouseLeave:Connect(function()
        if isActive then
            button.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
        else
            button.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
        end
    end)
    
    button.MouseButton1Down:Connect(function()
        if autoCrouchEnabled then
            if not isActive then
                startAutoCrouch()
                isActive = true
                button.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
            else
                stopAutoCrouch()
                isActive = false
                button.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
            end
        end
    end)
    
    local dragging = false
    local dragStart
    local startPos
    
    button.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = button.Position
        end
    end)
    
    button.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    game:GetService("UserInputService").InputChanged:Connect(function(input, gameProcessed)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = input.Position - dragStart
            button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Detectar quando o jogador pressiona uma tecla
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if switchLagEnabled and input.KeyCode == switchLagKey then
        activateSwitchLag()
    end
    
    if autoCrouchEnabled and input.KeyCode == autoCrouchKey then
        startAutoCrouch()
    end
end)

-- Detectar quando o jogador solta a tecla
UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if autoCrouchEnabled and input.KeyCode == autoCrouchKey then
        stopAutoCrouch()
    end
end)

-- Loop principal de Auto Jump
RunService.Heartbeat:Connect(function()
    if not character or not humanoid or humanoid.Health <= 0 or not character.Parent then return end
    
    if autoJumpEnabled then
        if isGrounded() then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Atualizar referências quando o personagem respawna
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    autoCrouchActive = false
    print("✓ Personagem respawned!")
end)

-- Criar a UI
createUI()

print("✓ Script executado com sucesso!")
