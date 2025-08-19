--[[
    Pigatin Hub - Aimbot EXATO na cabeça, círculo maior e aimbot gruda de qualquer distância
    - ESP Linhas
    - Highlight corpo vermelho, cabeça azul
    - AIMBOT: Círculo grande no centro da tela; gruda a mira EXATAMENTE no centro da cabeça do player/NPC dentro do círculo, independente da distância
    - Ativação/desativação por botões ou tecla "="
    - Exclusividade entre modos
    - Hub arrastável, botão fechar
    - LocalScript: StarterPlayer > StarterPlayerScripts
]]

local config = {
    HubTitle = "Pigatin",
    MainColor = Color3.fromRGB(30, 30, 30),
    AccentColor = Color3.fromRGB(255, 0, 0),
    TextColor = Color3.fromRGB(255,255,255),
    HubSize = UDim2.new(0, 430, 0, 260),
    ButtonColor = Color3.fromRGB(0,0,0),
    ButtonActiveColor = Color3.fromRGB(30,30,30),
    BodyHighlightColor = Color3.fromRGB(255, 0, 0),
    HeadHighlightColor = Color3.fromRGB(0, 140, 255),
    AimCircleColor = Color3.fromRGB(255,0,0),
    AimCircleSize = 160, -- pixels (CÍRCULO MUITO MAIOR)
    AimKey = Enum.KeyCode.Equals,
}

local ESP_Lines_Enabled = false
local ESP_Highlight_Enabled = false
local AIMBOT_Enabled = false
local lines = {}
local highlights = {}
local aimCircle = nil

local function Create(objType, props)
    local obj = Instance.new(objType)
    for k,v in pairs(props) do
        obj[k] = v
    end
    return obj
end

local function getRootPart(model)
    if model:IsA("Model") then
        return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso")
    end
    return nil
end
local function getHead(model)
    if model:IsA("Model") then
        return model:FindFirstChild("Head")
    end
    return nil
end
local function getBodyParts(model)
    local parts = {}
    if model:IsA("Model") then
        for _,obj in pairs(model:GetChildren()) do
            if obj:IsA("BasePart") then
                table.insert(parts, obj)
            end
        end
    end
    return parts
end

local function UpdateLines()
    for _,line in pairs(lines) do
        if line and line.Parent then
            line:Destroy()
        end
    end
    lines = {}
    if not ESP_Lines_Enabled then return end

    local plr = game.Players.LocalPlayer
    local char = plr.Character or plr.CharacterAdded:Wait()
    local myRoot = getRootPart(char)
    if not myRoot then return end

    for _,obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= char and obj:FindFirstChildWhichIsA("Humanoid") then
            local tgtRoot = getRootPart(obj)
            if tgtRoot then
                local line = Instance.new("Beam")
                line.Color = ColorSequence.new(config.AccentColor)
                line.FaceCamera = true
                line.Width0 = 0.2
                line.Width1 = 0.2
                line.Transparency = NumberSequence.new(0.1)
                local att0 = Instance.new("Attachment", myRoot)
                local att1 = Instance.new("Attachment", tgtRoot)
                line.Attachment0 = att0
                line.Attachment1 = att1
                line.Parent = myRoot
                table.insert(lines, line)
                table.insert(lines, att0)
                table.insert(lines, att1)
            end
        end
    end
end

local function UpdateHighlights()
    for _,hl in pairs(highlights) do
        if hl and hl.Parent then
            hl:Destroy()
        end
    end
    highlights = {}
    if not ESP_Highlight_Enabled then return end

    for _,obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") then
            for _,part in pairs(getBodyParts(obj)) do
                if part.Name ~= "Head" then
                    local highlight = Instance.new("Highlight")
                    highlight.FillColor = config.BodyHighlightColor
                    highlight.OutlineColor = config.BodyHighlightColor
                    highlight.Adornee = part
                    highlight.Parent = part
                    table.insert(highlights, highlight)
                end
            end
            local head = getHead(obj)
            if head then
                local highlight = Instance.new("Highlight")
                highlight.FillColor = config.HeadHighlightColor
                highlight.OutlineColor = config.HeadHighlightColor
                highlight.Adornee = head
                highlight.Parent = head
                table.insert(highlights, highlight)
            end
        end
    end
end

local function CreateAimCircle()
    if aimCircle then aimCircle:Destroy() aimCircle = nil end
    local screenGui = game.Players.LocalPlayer:WaitForChild("PlayerGui"):FindFirstChild("PigatinHubScreenGui")
    if not screenGui then screenGui = game.Players.LocalPlayer:WaitForChild("PlayerGui") end
    aimCircle = Instance.new("Frame")
    aimCircle.Name = "AimCircle"
    aimCircle.Size = UDim2.new(0, config.AimCircleSize, 0, config.AimCircleSize)
    aimCircle.AnchorPoint = Vector2.new(0.5,0.5)
    aimCircle.Position = UDim2.new(0.5,0,0.5,0)
    aimCircle.BackgroundTransparency = 1
    aimCircle.BorderSizePixel = 0
    aimCircle.ZIndex = 1000
    aimCircle.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0.5,0)
    corner.Parent = aimCircle

    local stroke = Instance.new("UIStroke")
    stroke.Color = config.AimCircleColor
    stroke.Thickness = 3
    stroke.Parent = aimCircle
end

local function DestroyAimCircle()
    if aimCircle then aimCircle:Destroy() aimCircle = nil end
end

-- EXATO NA HEAD, DE QUALQUER DISTÂNCIA!
local function getAimbotTarget()
    local camera = workspace.CurrentCamera
    local screenCenter = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    local radius = config.AimCircleSize/2
    local closest, closestDist = nil, math.huge

    -- Players
    for _,plr in pairs(game.Players:GetPlayers()) do
        if plr ~= game.Players.LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                if dist <= radius and dist < closestDist then
                    closest = head
                    closestDist = dist
                end
            end
        end
    end
    -- NPCs também
    for _,obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") and obj:FindFirstChild("Head") and obj ~= game.Players.LocalPlayer.Character then
            local head = obj.Head
            local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                if dist <= radius and dist < closestDist then
                    closest = head
                    closestDist = dist
                end
            end
        end
    end
    return closest
end

game:GetService("RunService").RenderStepped:Connect(function()
    if AIMBOT_Enabled then
        if not aimCircle then CreateAimCircle() end
        local camera = workspace.CurrentCamera
        local target = getAimbotTarget()
        if target then
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
        end
    else
        DestroyAimCircle()
    end
end)

workspace.DescendantAdded:Connect(function(obj)
    if ESP_Lines_Enabled and obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") then UpdateLines() end
    if ESP_Highlight_Enabled and obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") then UpdateHighlights() end
end)
workspace.DescendantRemoving:Connect(function(obj)
    if ESP_Lines_Enabled and obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") then UpdateLines() end
    if ESP_Highlight_Enabled and obj:IsA("Model") and obj:FindFirstChildWhichIsA("Humanoid") then UpdateHighlights() end
end)
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    if ESP_Lines_Enabled then UpdateLines() end
    if ESP_Highlight_Enabled then UpdateHighlights() end
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if ESP_Lines_Enabled then UpdateLines() end
    if ESP_Highlight_Enabled then UpdateHighlights() end
end)

local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local old = playerGui:FindFirstChild("PigatinHubScreenGui")
if old then old:Destroy() end

local screenGui = Create("ScreenGui", {
    Name = "PigatinHubScreenGui",
    Parent = playerGui,
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Global,
})

local mainFrame = Create("Frame", {
    Name = "MainFrame",
    Parent = screenGui,
    BackgroundColor3 = config.MainColor,
    Size = config.HubSize,
    Position = UDim2.new(0.5, -config.HubSize.X.Offset/2, 0.5, -config.HubSize.Y.Offset/2),
    BorderSizePixel = 0,
    AnchorPoint = Vector2.new(0.5,0.5)
})

local accentBar = Create("Frame", {
    Parent = mainFrame,
    Size = UDim2.new(1,0,0,30),
    BackgroundColor3 = config.AccentColor,
    BorderSizePixel = 0
})

local titleLabel = Create("TextLabel", {
    Parent = accentBar,
    Size = UDim2.new(0.9,0,1,0),
    BackgroundTransparency = 1,
    Text = config.HubTitle,
    TextColor3 = config.TextColor,
    Font = Enum.Font.GothamBold,
    TextSize = 22,
    Position = UDim2.new(0.05,0,0,0)
})

local closeBtn = Create("TextButton", {
    Parent = accentBar,
    Size = UDim2.new(0,30,0,30),
    Position = UDim2.new(1,-30,0,0),
    BackgroundColor3 = Color3.fromRGB(255,50,50),
    Text = "X",
    Font = Enum.Font.GothamBold,
    TextSize = 18,
    TextColor3 = Color3.new(1,1,1),
    BorderSizePixel = 0
})
closeBtn.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
    ESP_Lines_Enabled = false
    ESP_Highlight_Enabled = false
    AIMBOT_Enabled = false
    UpdateLines()
    UpdateHighlights()
    DestroyAimCircle()
end)

local ESPBtn = Create("TextButton", {
    Parent = mainFrame,
    Size = UDim2.new(0,160,0,40),
    Position = UDim2.new(0,20,0,60),
    BackgroundColor3 = config.ButtonColor,
    Text = "ESP Linhas: OFF",
    Font = Enum.Font.Gotham,
    TextSize = 18,
    TextColor3 = config.TextColor,
    BorderSizePixel = 0
})

local ESPHighBtn = Create("TextButton", {
    Parent = mainFrame,
    Size = UDim2.new(0,160,0,40),
    Position = UDim2.new(0,200,0,60),
    BackgroundColor3 = config.ButtonColor,
    Text = "Highlight: OFF",
    Font = Enum.Font.Gotham,
    TextSize = 18,
    TextColor3 = config.TextColor,
    BorderSizePixel = 0
})

local AimbotBtn = Create("TextButton", {
    Parent = mainFrame,
    Size = UDim2.new(0,350,0,40),
    Position = UDim2.new(0.5,-175,0,120),
    BackgroundColor3 = config.ButtonColor,
    Text = "Aimbot: OFF (=)",
    Font = Enum.Font.Gotham,
    TextSize = 18,
    TextColor3 = config.TextColor,
    BorderSizePixel = 0
})

ESPBtn.MouseButton1Click:Connect(function()
    ESP_Lines_Enabled = not ESP_Lines_Enabled
    ESPBtn.Text = ESP_Lines_Enabled and "ESP Linhas: ON" or "ESP Linhas: OFF"
    ESPBtn.BackgroundColor3 = ESP_Lines_Enabled and config.ButtonActiveColor or config.ButtonColor
    if ESP_Lines_Enabled then
        ESP_Highlight_Enabled = false
        AIMBOT_Enabled = false
        ESPHighBtn.Text = "Highlight: OFF"
        ESPHighBtn.BackgroundColor3 = config.ButtonColor
        AimbotBtn.Text = "Aimbot: OFF (=)"
        AimbotBtn.BackgroundColor3 = config.ButtonColor
        DestroyAimCircle()
    end
    UpdateLines()
    UpdateHighlights()
end)
ESPHighBtn.MouseButton1Click:Connect(function()
    ESP_Highlight_Enabled = not ESP_Highlight_Enabled
    ESPHighBtn.Text = ESP_Highlight_Enabled and "Highlight: ON" or "Highlight: OFF"
    ESPHighBtn.BackgroundColor3 = ESP_Highlight_Enabled and config.ButtonActiveColor or config.ButtonColor
    if ESP_Highlight_Enabled then
        ESP_Lines_Enabled = false
        AIMBOT_Enabled = false
        ESPBtn.Text = "ESP Linhas: OFF"
        ESPBtn.BackgroundColor3 = config.ButtonColor
        AimbotBtn.Text = "Aimbot: OFF (=)"
        AimbotBtn.BackgroundColor3 = config.ButtonColor
        DestroyAimCircle()
    end
    UpdateLines()
    UpdateHighlights()
end)
AimbotBtn.MouseButton1Click:Connect(function()
    AIMBOT_Enabled = not AIMBOT_Enabled
    AimbotBtn.Text = AIMBOT_Enabled and "Aimbot: ON (=)" or "Aimbot: OFF (=)"
    AimbotBtn.BackgroundColor3 = AIMBOT_Enabled and config.ButtonActiveColor or config.ButtonColor
    if AIMBOT_Enabled then
        ESP_Lines_Enabled = false
        ESP_Highlight_Enabled = false
        ESPBtn.Text = "ESP Linhas: OFF"
        ESPBtn.BackgroundColor3 = config.ButtonColor
        ESPHighBtn.Text = "Highlight: OFF"
        ESPHighBtn.BackgroundColor3 = config.ButtonColor
        CreateAimCircle()
    else
        DestroyAimCircle()
    end
    UpdateLines()
    UpdateHighlights()
end)

game:GetService("UserInputService").InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == config.AimKey then
        AIMBOT_Enabled = not AIMBOT_Enabled
        AimbotBtn.Text = AIMBOT_Enabled and "Aimbot: ON (=)" or "Aimbot: OFF (=)"
        AimbotBtn.BackgroundColor3 = AIMBOT_Enabled and config.ButtonActiveColor or config.ButtonColor
        if AIMBOT_Enabled then
            ESP_Lines_Enabled = false
            ESP_Highlight_Enabled = false
            ESPBtn.Text = "ESP Linhas: OFF"
            ESPBtn.BackgroundColor3 = config.ButtonColor
            ESPHighBtn.Text = "Highlight: OFF"
            ESPHighBtn.BackgroundColor3 = config.ButtonColor
            CreateAimCircle()
        else
            DestroyAimCircle()
        end
        UpdateLines()
        UpdateHighlights()
    end
end)

local dragger = Instance.new("Frame")
dragger.Size = accentBar.Size
dragger.Position = accentBar.Position
dragger.BackgroundTransparency = 1
dragger.Parent = accentBar

local UserInputService = game:GetService("UserInputService")
local dragging, dragInput, dragStart, startPos

dragger.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

dragger.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
