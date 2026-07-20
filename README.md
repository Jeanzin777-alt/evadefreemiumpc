-- Bhop Script para EVADE (Roblox) - PC VERSION (Estilo Solara)
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

local bhopEnabled = false
local modifyBounceEnabled = false
local spaceHeld = false
local lastJumpTime = 0
local jumpCooldown = 0.08
local uiVisible = true
local bounceMultiplier = 1
local lastRampBoost = 0
local rampBoostCooldown = 0.5

-- Criar UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "EvadeBhopUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local frame = Instance.new("Frame")
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 320, 0, 320)
frame.Position = UDim2.new(0.5, -160, 0.5, -160)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BackgroundTransparency = 0.15
frame.BorderSizePixel = 0
frame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

-- Adicionar stroke (borda)
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(60, 60, 80)
stroke.Thickness = 1
stroke.Parent = frame

-- ===== BARRA DE TITULO =====
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 10)
titleBarCorner.Parent = titleBar

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(220, 220, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.Text = "⚡ EVADE BHOP"
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextYAlignment = Enum.TextYAlignment.Center
title.Parent = titleBar

-- Botão Minimizar
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 32, 0, 32)
minBtn.Position = UDim2.new(1, -75, 0.5, -16)
minBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
minBtn.TextSize = 16
minBtn.Font = Enum.Font.GothamBold
minBtn.Text = "−"
minBtn.BorderSizePixel = 0
minBtn.Parent = titleBar

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 5)
minCorner.Parent = minBtn

-- Botão Fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -40, 0.5, -16)
closeBtn.BackgroundColor3 = Color3.fromRGB(70, 40, 40)
closeBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
closeBtn.TextSize = 16
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Text = "✕"
closeBtn.BorderSizePixel = 0
closeBtn.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 5)
closeCorner.Parent = closeBtn

-- Container de conteúdo
local content = Instance.new("Frame")
content.Name = "Content"
content.Size = UDim2.new(1, 0, 1, -45)
content.Position = UDim2.new(0, 0, 0, 45)
content.BackgroundTransparency = 1
content.ClipsDescendants = true
content.Parent = frame

-- ===== OPÇÃO BHOP =====
local bhopFrame = Instance.new("Frame")
bhopFrame.Size = UDim2.new(1, -20, 0, 90)
bhopFrame.Position = UDim2.new(0, 10, 0, 10)
bhopFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
bhopFrame.BorderSizePixel = 0
bhopFrame.Parent = content

local bhopCorner = Instance.new("UICorner")
bhopCorner.CornerRadius = UDim.new(0, 6)
bhopCorner.Parent = bhopFrame

local bhopStroke = Instance.new("UIStroke")
bhopStroke.Color = Color3.fromRGB(60, 60, 80)
bhopStroke.Thickness = 1
bhopStroke.Parent = bhopFrame

-- Label Bhop
local bhopLabel = Instance.new("TextLabel")
bhopLabel.Size = UDim2.new(1, -80, 0, 22)
bhopLabel.Position = UDim2.new(0, 12, 0, 8)
bhopLabel.BackgroundTransparency = 1
bhopLabel.TextColor3 = Color3.fromRGB(255, 180, 100)
bhopLabel.TextSize = 14
bhopLabel.Font = Enum.Font.GothamBold
bhopLabel.Text = "🔥 BHOP"
bhopLabel.TextXAlignment = Enum.TextXAlignment.Left
bhopLabel.Parent = bhopFrame

-- Descrição
local bhopDesc = Instance.new("TextLabel")
bhopDesc.Size = UDim2.new(1, -80, 0, 18)
bhopDesc.Position = UDim2.new(0, 12, 0, 32)
bhopDesc.BackgroundTransparency = 1
bhopDesc.TextColor3 = Color3.fromRGB(160, 160, 180)
bhopDesc.TextSize = 11
bhopDesc.Font = Enum.Font.Gotham
bhopDesc.Text = "Hold SPACE"
bhopDesc.TextXAlignment = Enum.TextXAlignment.Left
bhopDesc.Parent = bhopFrame

-- Status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -80, 0, 16)
statusLabel.Position = UDim2.new(0, 12, 0, 53)
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.fromRGB(130, 130, 150)
statusLabel.TextSize = 10
statusLabel.Font = Enum.Font.Gotham
statusLabel.Text = "Status: OFF"
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = bhopFrame

-- Toggle BHOP
local toggleBg = Instance.new("Frame")
toggleBg.Size = UDim2.new(0, 48, 0, 28)
toggleBg.Position = UDim2.new(1, -60, 0.5, -14)
toggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
toggleBg.BorderSizePixel = 0
toggleBg.Parent = bhopFrame

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 6)
toggleCorner.Parent = toggleBg

local toggleCircle = Instance.new("Frame")
toggleCircle.Size = UDim2.new(0, 22, 0, 22)
toggleCircle.Position = UDim2.new(0, 3, 0.5, -11)
toggleCircle.BackgroundColor3 = Color3.fromRGB(200, 200, 220)
toggleCircle.BorderSizePixel = 0
toggleCircle.Parent = toggleBg

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(0, 5)
circleCorner.Parent = toggleCircle

-- ===== OPÇÃO MODIFY BOUNCE =====
local bounceFrame = Instance.new("Frame")
bounceFrame.Size = UDim2.new(1, -20, 0, 105)
bounceFrame.Position = UDim2.new(0, 10, 0, 110)
bounceFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
bounceFrame.BorderSizePixel = 0
bounceFrame.Parent = content

local bounceCorner = Instance.new("UICorner")
bounceCorner.CornerRadius = UDim.new(0, 6)
bounceCorner.Parent = bounceFrame

local bounceStroke = Instance.new("UIStroke")
bounceStroke.Color = Color3.fromRGB(60, 60, 80)
bounceStroke.Thickness = 1
bounceStroke.Parent = bounceFrame

-- Label Modify Bounce
local bounceLabel = Instance.new("TextLabel")
bounceLabel.Size = UDim2.new(1, -80, 0, 22)
bounceLabel.Position = UDim2.new(0, 12, 0, 8)
bounceLabel.BackgroundTransparency = 1
bounceLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
bounceLabel.TextSize = 14
bounceLabel.Font = Enum.Font.GothamBold
bounceLabel.Text = "⚡ MODIFY BOUNCE"
bounceLabel.TextXAlignment = Enum.TextXAlignment.Left
bounceLabel.Parent = bounceFrame

-- Input box
local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(1, -80, 0, 26)
inputBox.Position = UDim2.new(0, 12, 0, 32)
inputBox.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
inputBox.TextColor3 = Color3.fromRGB(220, 220, 220)
inputBox.TextSize = 12
inputBox.Font = Enum.Font.Gotham
inputBox.Text = "150"
inputBox.PlaceholderText = "Velocidade"
inputBox.BorderSizePixel = 0
inputBox.Parent = bounceFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 4)
inputCorner.Parent = inputBox

local inputStroke = Instance.new("UIStroke")
inputStroke.Color = Color3.fromRGB(60, 60, 80)
inputStroke.Thickness = 1
inputStroke.Parent = inputBox

-- Info
local bounceInfo = Instance.new("TextLabel")
bounceInfo.Size = UDim2.new(1, -80, 0, 15)
bounceInfo.Position = UDim2.new(0, 12, 0, 62)
bounceInfo.BackgroundTransparency = 1
bounceInfo.TextColor3 = Color3.fromRGB(130, 130, 150)
bounceInfo.TextSize = 9
bounceInfo.Font = Enum.Font.Gotham
bounceInfo.Text = "x10 em rampas"
bounceInfo.TextXAlignment = Enum.TextXAlignment.Left
bounceInfo.Parent = bounceFrame

-- Status Bounce
local statusBounce = Instance.new("TextLabel")
statusBounce.Size = UDim2.new(1, -80, 0, 15)
statusBounce.Position = UDim2.new(0, 12, 0, 80)
statusBounce.BackgroundTransparency = 1
statusBounce.TextColor3 = Color3.fromRGB(130, 130, 150)
statusBounce.TextSize = 9
statusBounce.Font = Enum.Font.Gotham
statusBounce.Text = "Status: OFF"
statusBounce.TextXAlignment = Enum.TextXAlignment.Left
statusBounce.Parent = bounceFrame

-- Toggle MODIFY BOUNCE
local bounceTtoggleBg = Instance.new("Frame")
bounceTtoggleBg.Size = UDim2.new(0, 48, 0, 28)
bounceTtoggleBg.Position = UDim2.new(1, -60, 0.5, -14)
bounceTtoggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
bounceTtoggleBg.BorderSizePixel = 0
bounceTtoggleBg.Parent = bounceFrame

local bounceToggleCorner = Instance.new("UICorner")
bounceToggleCorner.CornerRadius = UDim.new(0, 6)
bounceToggleCorner.Parent = bounceTtoggleBg

local bounceToggleCircle = Instance.new("Frame")
bounceToggleCircle.Size = UDim2.new(0, 22, 0, 22)
bounceToggleCircle.Position = UDim2.new(0, 3, 0.5, -11)
bounceToggleCircle.BackgroundColor3 = Color3.fromRGB(200, 200, 220)
bounceToggleCircle.BorderSizePixel = 0
bounceToggleCircle.Parent = bounceTtoggleBg

local bounceCircleCorner = Instance.new("UICorner")
bounceCircleCorner.CornerRadius = UDim.new(0, 5)
bounceCircleCorner.Parent = bounceToggleCircle

-- Funções de atualizar toggles
local function updateBhopToggle()
    if bhopEnabled then
        toggleBg.BackgroundColor3 = Color3.fromRGB(80, 140, 80)
        toggleCircle.Position = UDim2.new(0, 23, 0.5, -11)
        statusLabel.TextColor3 = Color3.fromRGB(100, 220, 100)
        statusLabel.Text = "Status: ON"
    else
        toggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        toggleCircle.Position = UDim2.new(0, 3, 0.5, -11)
        statusLabel.TextColor3 = Color3.fromRGB(150, 100, 100)
        statusLabel.Text = "Status: OFF"
    end
end

local function updateBounceToggle()
    if modifyBounceEnabled then
        bounceTtoggleBg.BackgroundColor3 = Color3.fromRGB(80, 140, 80)
        bounceToggleCircle.Position = UDim2.new(0, 23, 0.5, -11)
        statusBounce.TextColor3 = Color3.fromRGB(100, 220, 100)
        statusBounce.Text = "Status: ON"
    else
        bounceTtoggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        bounceToggleCircle.Position = UDim2.new(0, 3, 0.5, -11)
        statusBounce.TextColor3 = Color3.fromRGB(150, 100, 100)
        statusBounce.Text = "Status: OFF"
    end
end

-- Clique no toggle BHOP
toggleBg.InputBegan:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        bhopEnabled = not bhopEnabled
        updateBhopToggle()
        print(bhopEnabled and "✓ BHOP ATIVADO!" or "✗ BHOP DESATIVADO")
    end
end)

-- Clique no toggle MODIFY BOUNCE
bounceTtoggleBg.InputBegan:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        modifyBounceEnabled = not modifyBounceEnabled
        updateBounceToggle()
        if modifyBounceEnabled then
            local vel = tonumber(inputBox.Text) or 150
            bounceMultiplier = vel
            print("✓ MODIFY BOUNCE ATIVADO! Velocidade:", bounceMultiplier)
        else
            print("✗ MODIFY BOUNCE DESATIVADO")
        end
    end
end)

-- Atualizar velocidade quando digita
inputBox.FocusLost:Connect(function(enterPressed)
    local vel = tonumber(inputBox.Text) or 150
    bounceMultiplier = vel
    if modifyBounceEnabled then
        print("Velocidade atualizada para:", bounceMultiplier)
    end
end)

-- Hover effects
minBtn.MouseEnter:Connect(function()
    minBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
end)

minBtn.MouseLeave:Connect(function()
    minBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
end)

closeBtn.MouseEnter:Connect(function()
    closeBtn.BackgroundColor3 = Color3.fromRGB(90, 50, 50)
end)

closeBtn.MouseLeave:Connect(function()
    closeBtn.BackgroundColor3 = Color3.fromRGB(70, 40, 40)
end)

bhopFrame.MouseEnter:Connect(function()
    bhopFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
end)

bhopFrame.MouseLeave:Connect(function()
    bhopFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
end)

bounceFrame.MouseEnter:Connect(function()
    bounceFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
end)

bounceFrame.MouseLeave:Connect(function()
    bounceFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
end)

-- ===== MINIMIZAR COM BOTÃO =====
local minimized = false
minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        frame.Size = UDim2.new(0, 320, 0, 45)
        minBtn.Text = "□"
    else
        frame.Size = UDim2.new(0, 320, 0, 320)
        minBtn.Text = "−"
    end
end)

-- Fechar
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- ===== MINIMIZAR COM CTRL DIREITO =====
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightControl then
        uiVisible = not uiVisible
        screenGui.Enabled = uiVisible
        if uiVisible then
            UserInputService.MouseBehavior = Enum.MouseBehavior.Default
            print("✓ UI Visível")
        else
            UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
            print("✗ UI Oculta")
        end
    end
end)

-- ===== ARRASTAR WINDOW =====
local dragging = false
local dragStart = nil
local frameStart = nil

titleBar.InputBegan:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        local mouse = game:GetService("Players").LocalPlayer:GetMouse()
        dragStart = mouse.Position
        frameStart = frame.Position
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

RunService.RenderStepped:Connect(function()
    if dragging then
        local mouse = game:GetService("Players").LocalPlayer:GetMouse()
        local delta = mouse.Position - dragStart
        frame.Position = UDim2.new(frameStart.X.Scale, frameStart.X.Offset + delta.X, frameStart.Y.Scale, frameStart.Y.Offset + delta.Y)
    end
end)

-- Função para verificar se está em uma rampa
local function isOnRamp()
    if not rootPart then return false end
    
    local rayOrigin = rootPart.Position + Vector3.new(0, 3, 0)
    local rayDirection = Vector3.new(0, -6, 0)
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.FilterDescendantsInstances = {character}
    
    local result = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    
    if result then
        local hitPart = result.Instance
        if hitPart then
            local normal = result.Normal
            local angle = math.deg(math.acos(math.min(1, math.max(-1, normal.Y))))
            return angle > 15
        end
    end
    return false
end

-- Função para verificar se está no chão
local function isGrounded()
    if not rootPart then return false end
    
    local rayOrigin = rootPart.Position + Vector3.new(0, 3, 0)
    local rayDirection = Vector3.new(0, -6, 0)
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.FilterDescendantsInstances = {character}
    
    local result = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    return result ~= nil
end

-- Detectar espaço
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space then
        spaceHeld = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Space then
        spaceHeld = false
    end
end)

-- Loop do Bhop
RunService.RenderStepped:Connect(function()
    if not bhopEnabled or not spaceHeld then return end
    
    if not humanoid or humanoid.Health <= 0 then return end
    if not rootPart then return end
    
    local currentTime = tick()
    if (currentTime - lastJumpTime) < jumpCooldown then return end
    
    if isGrounded() then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        lastJumpTime = currentTime
    end
end)

-- Loop Modify Bounce
RunService.RenderStepped:Connect(function()
    if not modifyBounceEnabled then return end
    if not rootPart or not humanoid or humanoid.Health <= 0 then return end
    
    local currentTime = tick()
    if (currentTime - lastRampBoost) < rampBoostCooldown then return end
    
    if isOnRamp() then
        local direction = rootPart.CFrame.LookVector
        local boostVelocity = direction * (bounceMultiplier * 10)
        rootPart.AssemblyLinearVelocity = rootPart.AssemblyLinearVelocity + boostVelocity
        lastRampBoost = currentTime
    end
end)

-- Reconectar ao morrer
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
end)

UserInputService.MouseBehavior = Enum.MouseBehavior.Default

print("✓ EVADE BHOP - Estilo Solara - PC VERSION"
