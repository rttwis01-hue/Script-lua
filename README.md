--[[
    MM2 HUB - Murder Mystery 2 Advanced GUI
    Compatível com Roblox Studio e Delta Executor
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Variaveis globais
local settings = {
    espEnabled = true,
    autoShoot = false,
    autoKnife = false,
    speedValue = 16,
    showMurder = true,
    showSheriff = true,
    showInnocents = true,
    showDroppedGun = true
}

local playersList = {}
local droppedGuns = {}
local currentFilter = "all"
local espObjects = {}

-- Cores dos papeis
local roleColors = {
    Murder = Color3.fromRGB(255, 70, 70),
    Sheriff = Color3.fromRGB(255, 215, 0),
    Hero = Color3.fromRGB(255, 180, 80),
    Innocent = Color3.fromRGB(80, 255, 80)
}

-- Funcao para detectar o papel do jogador
local function getPlayerRole(player)
    local character = player.Character
    if character then
        local backpack = player.Backpack
        
        for _, tool in pairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                local toolName = tool.Name:lower()
                if toolName:find("knife") or toolName:find("dagger") then
                    return "Murder"
                elseif toolName:find("gun") or toolName:find("pistol") or toolName:find("revolver") then
                    if player:GetAttribute("IsHero") then
                        return "Hero"
                    end
                    return "Sheriff"
                end
            end
        end
        
        for _, tool in pairs(backpack:GetChildren()) do
            if tool:IsA("Tool") then
                local toolName = tool.Name:lower()
                if toolName:find("knife") or toolName:find("dagger") then
                    return "Murder"
                elseif toolName:find("gun") or toolName:find("pistol") or toolName:find("revolver") then
                    if player:GetAttribute("IsHero") then
                        return "Hero"
                    end
                    return "Sheriff"
                end
            end
        end
    end
    
    if player:GetAttribute("IsHero") then
        return "Hero"
    end
    
    return "Innocent"
end

-- Sistema de Toast
local function createToast(message, duration)
    duration = duration or 2
    local screenGui = Instance.new("ScreenGui")
    local frame = Instance.new("Frame")
    local text = Instance.new("TextLabel")
    
    screenGui.Name = "ToastNotification"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    
    frame.Size = UDim2.new(0, 280, 0, 40)
    frame.Position = UDim2.new(1, 0, 1, -60)
    frame.BackgroundColor3 = Color3.fromRGB(27, 18, 43)
    frame.BorderSizePixel = 0
    frame.BackgroundTransparency = 0.1
    frame.ClipsDescendants = true
    frame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = frame
    
    local leftBar = Instance.new("Frame")
    leftBar.Size = UDim2.new(0, 5, 1, 0)
    leftBar.BackgroundColor3 = Color3.fromRGB(192, 132, 252)
    leftBar.BorderSizePixel = 0
    leftBar.Parent = frame
    
    text.Text = message
    text.Size = UDim2.new(1, -15, 1, 0)
    text.Position = UDim2.new(0, 15, 0, 0)
    text.BackgroundTransparency = 1
    text.TextColor3 = Color3.fromRGB(255, 255, 255)
    text.TextXAlignment = Enum.TextXAlignment.Left
    text.Font = Enum.Font.GothamBold
    text.TextSize = 13
    text.TextWrapped = true
    text.Parent = frame
    
    frame.Position = UDim2.new(1, 0, 1, -60)
    local tween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {Position = UDim2.new(1, -290, 1, -60)})
    tween:Play()
    
    task.wait(duration)
    
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {Position = UDim2.new(1, 0, 1, -60)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function()
        screenGui:Destroy()
    end)
end

-- Sistema de velocidade
local function updateWalkSpeed()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = settings.speedValue
    end
end

LocalPlayer.CharacterAdded:Connect(function(character)
    task.wait(0.5)
    updateWalkSpeed()
end)

-- Sistema ESP
local function createESP(player)
    if espObjects[player] then
        espObjects[player]:Destroy()
    end
    
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then 
        return 
    end
    
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_" .. player.Name
    billboard.Adornee = character:FindFirstChild("HumanoidRootPart")
    billboard.Size = UDim2.new(0, 140, 0, 45)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = character
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.5
    frame.BorderSizePixel = 0
    frame.Parent = billboard
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.55, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 14
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeTransparency = 0.3
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.Parent = frame
    
    local roleLabel = Instance.new("TextLabel")
    roleLabel.Size = UDim2.new(1, 0, 0.45, 0)
    roleLabel.Position = UDim2.new(0, 0, 0.55, 0)
    roleLabel.BackgroundTransparency = 1
    roleLabel.TextSize = 11
    roleLabel.Font = Enum.Font.Gotham
    roleLabel.TextStrokeTransparency = 0.3
    roleLabel.Parent = frame
    
    local healthBar = Instance.new("Frame")
    healthBar.Size = UDim2.new(1, 0, 0.08, 0)
    healthBar.Position = UDim2.new(0, 0, 0.92, 0)
    healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    healthBar.BorderSizePixel = 0
    healthBar.Parent = frame
    
    espObjects[player] = billboard
    
    local function updateESP()
        local role = getPlayerRole(player)
        roleLabel.Text = role
        local color = roleColors[role] or roleColors.Innocent
        nameLabel.TextColor3 = color
        frame.BackgroundColor3 = color
        frame.BackgroundTransparency = 0.7
        
        if character and character:FindFirstChild("Humanoid") then
            local healthPercent = character.Humanoid.Health / character.Humanoid.MaxHealth
            healthBar.Size = UDim2.new(healthPercent, 0, 0.08, 0)
            if healthPercent > 0.5 then
                healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            elseif healthPercent > 0.2 then
                healthBar.BackgroundColor3 = Color3.fromRGB(255, 255, 0)
            else
                healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            end
        end
        
        local shouldShow = settings.espEnabled
        if shouldShow then
            if currentFilter == "murder" and role ~= "Murder" then
                shouldShow = false
            elseif currentFilter == "sheriff" and not (role == "Sheriff" or role == "Hero") then
                shouldShow = false
            elseif currentFilter == "innocent" and role ~= "Innocent" then
                shouldShow = false
            end
        end
        
        billboard.Enabled = shouldShow
    end
    
    updateESP()
    
    local updateConnection
    updateConnection = RunService.Heartbeat:Connect(function()
        if not player.Parent or not character.Parent then
            updateConnection:Disconnect()
            return
        end
        updateESP()
    end)
end

local function updateAllESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if settings.espEnabled then
                createESP(player)
            elseif espObjects[player] then
                espObjects[player]:Destroy()
                espObjects[player] = nil
            end
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(1)
        if settings.espEnabled and player ~= LocalPlayer then
            createESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if espObjects[player] then
        espObjects[player]:Destroy()
        espObjects[player] = nil
    end
end)

-- Sistema AutoShoot
local function getNearestMurder()
    local nearest = nil
    local shortestDist = math.huge
    local localChar = LocalPlayer.Character
    
    if not localChar or not localChar:FindFirstChild("HumanoidRootPart") then
        return nil, math.huge
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local role = getPlayerRole(player)
            if role == "Murder" then
                local character = player.Character
                if character and character:FindFirstChild("HumanoidRootPart") then
                    local dist = (character.HumanoidRootPart.Position - localChar.HumanoidRootPart.Position).Magnitude
                    if dist < shortestDist then
                        shortestDist = dist
                        nearest = player
                    end
                end
            end
        end
    end
    
    return nearest, shortestDist
end

local function getNearestPlayer()
    local nearest = nil
    local shortestDist = math.huge
    local localChar = LocalPlayer.Character
    
    if not localChar or not localChar:FindFirstChild("HumanoidRootPart") then
        return nil, math.huge
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local dist = (character.HumanoidRootPart.Position - localChar.HumanoidRootPart.Position).Magnitude
                if dist < shortestDist then
                    shortestDist = dist
                    nearest = player
                end
            end
        end
    end
    
    return nearest, shortestDist
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if settings.autoShoot and input.UserInputType == Enum.UserInputType.MouseButton1 then
        local murder, dist = getNearestMurder()
        if murder and dist < 60 then
            local character = LocalPlayer.Character
            local tool = character and character:FindFirstChildWhichIsA("Tool")
            if tool then
                local args = {
                    murder.Character.HumanoidRootPart.CFrame.Position
                }
                if tool:FindFirstChild("Fire") then
                    tool.Fire:FireServer(unpack(args))
                    createToast("AutoShoot: Atirou no Murder", 1)
                end
            end
        end
    end
    
    if settings.autoKnife and input.KeyCode == Enum.KeyCode.Q then
        local nearest, dist = getNearestPlayer()
        if nearest and dist < 30 then
            createToast("AutoKnife: Faca arremessada em " .. nearest.Name, 1.5)
        end
    end
end)

-- Sistema Fling
local function flingPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then 
        return 
    end
    
    local character = targetPlayer.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    
    if humanoidRootPart and humanoid then
        humanoidRootPart.Velocity = Vector3.new(0, 1000, 0)
        humanoid.PlatformStand = true
        
        task.wait(0.1)
        humanoid.Health = 0
        
        createToast(targetPlayer.Name .. " foi arremessado para fora do mapa", 2)
    end
end

local function flingMurder()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and getPlayerRole(player) == "Murder" then
            flingPlayer(player)
            return
        end
    end
    createToast("Nenhum Murder encontrado", 1.5)
end

local function flingSheriff()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local role = getPlayerRole(player)
            if role == "Sheriff" or role == "Hero" then
                flingPlayer(player)
                return
            end
        end
    end
    createToast("Nenhum Sheriff ou Hero encontrado", 1.5)
end

local function grabDroppedGun()
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    for _, item in pairs(workspace:GetDescendants()) do
        if item:IsA("Tool") and item:GetAttribute("Dropped") then
            local originalPos = humanoidRootPart.CFrame
            humanoidRootPart.CFrame = item.Handle.CFrame
            
            task.wait(0.001)
            
            humanoidRootPart.CFrame = originalPos
            
            local backpack = LocalPlayer.Backpack
            item.Parent = backpack
            
            createToast("Voce pegou a arma dropada", 1.5)
            return
        end
    end
    
    createToast("Nenhuma arma dropada encontrada", 1.5)
end

-- Interface Grafica
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MM2Hub"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    screenGui.ResetOnSpawn = false
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 580)
    mainFrame.Position = UDim2.new(0.5, -200, 0.5, -290)
    mainFrame.BackgroundColor3 = Color3.fromRGB(12, 11, 22)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 20)
    mainCorner.Parent = mainFrame
    
    local mainStroke = Instance.new("UIStroke")
    mainStroke.Color = Color3.fromRGB(128, 0, 255)
    mainStroke.Thickness = 1
    mainStroke.Transparency = 0.7
    mainStroke.Parent = mainFrame
    
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 45)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(156, 56, 255)
    titleBar.BackgroundTransparency = 0.3
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 20)
    titleCorner.Parent = titleBar
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 1, 0)
    title.BackgroundTransparency = 1
    title.Text = "MURDER MYSTERY 2 HUB"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 18
    title.Font = Enum.Font.GothamBold
    title.Parent = titleBar
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 8)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    closeBtn.BackgroundTransparency = 0.3
    closeBtn.Text = "X"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = titleBar
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    local espGroup = Instance.new("Frame")
    espGroup.Size = UDim2.new(0.95, 0, 0, 160)
    espGroup.Position = UDim2.new(0.025, 0, 0, 55)
    espGroup.BackgroundColor3 = Color3.fromRGB(20, 18, 35)
    espGroup.BackgroundTransparency = 0.5
    espGroup.BorderSizePixel = 0
    espGroup.Parent = mainFrame
    
    local espCorner = Instance.new("UICorner")
    espCorner.CornerRadius = UDim.new(0, 12)
    espCorner.Parent = espGroup
    
    local espTitle = Instance.new("TextLabel")
    espTitle.Size = UDim2.new(1, 0, 0, 30)
    espTitle.Position = UDim2.new(0, 10, 0, 5)
    espTitle.BackgroundTransparency = 1
    espTitle.Text = "ESP - VISUALIZACAO"
    espTitle.TextColor3 = Color3.fromRGB(192, 132, 252)
    espTitle.TextSize = 14
    espTitle.Font = Enum.Font.GothamBold
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.Parent = espGroup
    
    local btnY = 40
    local btnW = 90
    local btnH = 32
    local spacing = 10
    
    local murderBtn = Instance.new("TextButton")
    murderBtn.Size = UDim2.new(0, btnW, 0, btnH)
    murderBtn.Position = UDim2.new(0, 10, 0, btnY)
    murderBtn.BackgroundColor3 = Color3.fromRGB(58, 14, 28)
    murderBtn.Text = "MURDER"
    murderBtn.TextColor3 = Color3.fromRGB(255, 167, 185)
    murderBtn.TextSize = 12
    murderBtn.Font = Enum.Font.GothamBold
    murderBtn.BorderSizePixel = 0
    murderBtn.Parent = espGroup
    
    local murderCorner = Instance.new("UICorner")
    murderCorner.CornerRadius = UDim.new(0, 15)
    murderCorner.Parent = murderBtn
    
    local sheriffBtn = Instance.new("TextButton")
    sheriffBtn.Size = UDim2.new(0, btnW, 0, btnH)
    sheriffBtn.Position = UDim2.new(0, 10 + btnW + spacing, 0, btnY)
    sheriffBtn.BackgroundColor3 = Color3.fromRGB(19, 38, 59)
    sheriffBtn.Text = "SHERIFF"
    sheriffBtn.TextColor3 = Color3.fromRGB(255, 217, 102)
    sheriffBtn.TextSize = 12
    sheriffBtn.Font = Enum.Font.GothamBold
    sheriffBtn.BorderSizePixel = 0
    sheriffBtn.Parent = espGroup
    
    local sheriffCorner = Instance.new("UICorner")
    sheriffCorner.CornerRadius = UDim.new(0, 15)
    sheriffCorner.Parent = sheriffBtn
    
    local innocentBtn = Instance.new("TextButton")
    innocentBtn.Size = UDim2.new(0, btnW, 0, btnH)
    innocentBtn.Position = UDim2.new(0, 10 + (btnW + spacing) * 2, 0, btnY)
    innocentBtn.BackgroundColor3 = Color3.fromRGB(20, 56, 31)
    innocentBtn.Text = "INNOCENTS"
    innocentBtn.TextColor3 = Color3.fromRGB(163, 240, 176)
    innocentBtn.TextSize = 12
    innocentBtn.Font = Enum.Font.GothamBold
    innocentBtn.BorderSizePixel = 0
    innocentBtn.Parent = espGroup
    
    local innocentCorner = Instance.new("UICorner")
    innocentCorner.CornerRadius = UDim.new(0, 15)
    innocentCorner.Parent = innocentBtn
    
    local gunBtn = Instance.new("TextButton")
    gunBtn.Size = UDim2.new(0, btnW, 0, btnH)
    gunBtn.Position = UDim2.new(0, 10 + (btnW + spacing) * 3, 0, btnY)
    gunBtn.BackgroundColor3 = Color3.fromRGB(45, 26, 59)
    gunBtn.Text = "DROPPED GUN"
    gunBtn.TextColor3 = Color3.fromRGB(217, 180, 255)
    gunBtn.TextSize = 11
    gunBtn.Font = Enum.Font.GothamBold
    gunBtn.BorderSizePixel = 0
    gunBtn.Parent = espGroup
    
    local gunCorner = Instance.new("UICorner")
    gunCorner.CornerRadius = UDim.new(0, 15)
    gunCorner.Parent = gunBtn
    
    local espToggle = Instance.new("TextButton")
    espToggle.Size = UDim2.new(0, 80, 0, 30)
    espToggle.Position = UDim2.new(0, 10, 0, btnY + btnH + 8)
    espToggle.BackgroundColor3 = Color3.fromRGB(30, 28, 50)
    espToggle.Text = "ESP: ON"
    espToggle.TextColor3 = Color3.fromRGB(100, 255, 100)
    espToggle.TextSize = 11
    espToggle.Font = Enum.Font.GothamBold
    espToggle.BorderSizePixel = 0
    
