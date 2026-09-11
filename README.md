local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LP = Players.LocalPlayer
local parentGui = LP:WaitForChild("PlayerGui")

if parentGui:FindFirstChild("InstantGrabStatusGui") then
    parentGui.InstantGrabStatusGui:Destroy()
end

if parentGui:FindFirstChild("InstantGrabToggleGui") then
    parentGui.InstantGrabToggleGui:Destroy()
end

local function getMyPlot()
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil end

    for _, plot in ipairs(plots:GetChildren()) do
        local sign = plot:FindFirstChild("PlotSign")

        if sign and sign:FindFirstChild("SurfaceGui") then
            local fr = sign.SurfaceGui:FindFirstChild("Frame")

            if fr and fr:FindFirstChild("TextLabel") then
                if fr.TextLabel.Text == LP.DisplayName .. "'s Base" then
                    return plot
                end
            end
        end
    end

    return nil
end

local PromptMemoryCache = {}

local function findNearestPromptInPlots(maxRadius)
    local char = LP.Character
    if not char then return nil, math.huge end

    local root =
        char:FindFirstChild("HumanoidRootPart")
        or char:FindFirstChild("Torso")
        or char:FindFirstChild("UpperTorso")

    if not root then return nil, math.huge end

    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil, math.huge end

    local myPlot = getMyPlot()
    local nearestPrompt, minDist = nil, maxRadius

    for _, plot in ipairs(plots:GetChildren()) do
        if myPlot and plot == myPlot then
            continue
        end

        local podiums = plot:FindFirstChild("AnimalPodiums")

        if podiums then
            for _, podium in ipairs(podiums:GetChildren()) do
                local uid = plot.Name .. "_" .. podium.Name
                local cached = PromptMemoryCache[uid]
                local targetPrompt = nil

                if cached and cached.Parent then
                    targetPrompt = cached
                else
                    local base = podium:FindFirstChild("Base")
                    local spawnPoint = base and base:FindFirstChild("Spawn")
                    local attachment =
                        spawnPoint and spawnPoint:FindFirstChild("PromptAttachment")

                    if attachment then
                        for _, child in ipairs(attachment:GetChildren()) do
                            if child:IsA("ProximityPrompt") and child.Enabled then
                                targetPrompt = child
                                PromptMemoryCache[uid] = child
                                break
                            end
                        end
                    end
                end

                if targetPrompt and targetPrompt.Enabled and targetPrompt.Parent then
                    local part = targetPrompt.Parent

                    if part:IsA("Attachment") then
                        part = part.Parent
                    end

                    if part and part:IsA("BasePart") then
                        local dist = (part.Position - root.Position).Magnitude

                        if dist < minDist then
                            minDist = dist
                            nearestPrompt = targetPrompt
                        end
                    end
                end
            end
        end
    end

    return nearestPrompt, minDist
end

local function triggerHoldBegan(prompt)
    if not prompt then return end

    if getconnections then
        for _, conn in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
            if conn.Function then
                task.spawn(conn.Function)
            end
        end
    else
        prompt:InputHoldBegan()
    end
end

local function triggerPrompt(prompt)
    if not prompt then return end

    if getconnections then
        for _, conn in ipairs(getconnections(prompt.Triggered)) do
            if conn.Function then
                task.spawn(conn.Function)
            end
        end
    else
        prompt:InputHoldEnded()
    end
end

local statusGui = Instance.new("ScreenGui")
statusGui.Name = "InstantGrabStatusGui"
statusGui.ResetOnSpawn = false
statusGui.DisplayOrder = 999
statusGui.IgnoreGuiInset = true
statusGui.Parent = parentGui

local statusFrame = Instance.new("Frame")
statusFrame.Name = "StatusFrame"

-- WIDE + SHORT
statusFrame.Size = UDim2.new(0, 320, 0, 46)

-- FIXED POSITION
statusFrame.Position = UDim2.new(0.5, -160, 0.85, 0)

statusFrame.BackgroundColor3 = Color3.fromRGB(20, 14, 28)
statusFrame.BorderSizePixel = 0
statusFrame.Active = false
statusFrame.Parent = statusGui

Instance.new("UICorner", statusFrame).CornerRadius = UDim.new(0, 10)

-- KIAN SKIDDED TITLE
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "KianSkiddedTitle"
titleLabel.Size = UDim2.new(1, -20, 0, 16)
titleLabel.Position = UDim2.new(0, 10, 0, 1)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "kian skidded"
titleLabel.TextColor3 = Color3.fromRGB(205, 95, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 13
titleLabel.TextXAlignment = Enum.TextXAlignment.Center
titleLabel.Active = false
titleLabel.Selectable = false
titleLabel.Parent = statusFrame

-- PURPLE RGB BORDER
local statusStroke = Instance.new("UIStroke")
statusStroke.Color = Color3.fromRGB(170, 70, 255)
statusStroke.Thickness = 2
statusStroke.Parent = statusFrame

local strokeGradient = Instance.new("UIGradient")
strokeGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(120, 35, 210)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(235, 120, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(170, 70, 255))
})
strokeGradient.Parent = statusStroke

-- PURPLE DOT
local dot = Instance.new("Frame")
dot.Size = UDim2.new(0, 10, 0, 10)
dot.Position = UDim2.new(0, 10, 0, 10)
dot.BackgroundColor3 = Color3.fromRGB(195, 90, 255)
dot.BorderSizePixel = 0
dot.Parent = statusFrame

Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

-- TARGET NAME
local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(1, -80, 0, 16)
nameLabel.Position = UDim2.new(0, 26, 0, 18)
nameLabel.BackgroundTransparency = 1
nameLabel.Text = "Searching..."
nameLabel.TextColor3 = Color3.fromRGB(240, 240, 240)
nameLabel.Font = Enum.Font.GothamBold
nameLabel.TextSize = 12
nameLabel.TextXAlignment = Enum.TextXAlignment.Left
nameLabel.Parent = statusFrame

-- PERCENT
local percentLabel = Instance.new("TextLabel")
percentLabel.Size = UDim2.new(0, 45, 0, 18)
percentLabel.Position = UDim2.new(1, -50, 0, 18)
percentLabel.BackgroundTransparency = 1
percentLabel.Text = "0%"
percentLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
percentLabel.Font = Enum.Font.GothamBold
percentLabel.TextSize = 12
percentLabel.TextXAlignment = Enum.TextXAlignment.Right
percentLabel.Parent = statusFrame

-- PROGRESS BACKGROUND
local progressBackground = Instance.new("Frame")
progressBackground.Name = "ProgressBG"
progressBackground.Size = UDim2.new(1, -20, 0, 7)
progressBackground.Position = UDim2.new(0, 10, 0, 36)
progressBackground.BackgroundColor3 = Color3.fromRGB(24, 14, 32)
progressBackground.BorderSizePixel = 0
progressBackground.ClipsDescendants = true
progressBackground.Parent = statusFrame

Instance.new("UICorner", progressBackground).CornerRadius = UDim.new(0, 7)

-- PURPLE PROGRESS
local progressBar = Instance.new("Frame")
progressBar.Name = "ProgressBar"
progressBar.Size = UDim2.new(0, 0, 1, 0)
progressBar.BackgroundColor3 = Color3.fromRGB(155, 45, 255)
progressBar.BorderSizePixel = 0
progressBar.Parent = progressBackground

Instance.new("UICorner", progressBar).CornerRadius = UDim.new(0, 7)

-- CENTER LINE
local centerLine = Instance.new("Frame")
centerLine.Name = "CenterLine"
centerLine.Size = UDim2.new(0, 2, 1, 0)
centerLine.Position = UDim2.new(0.5, -1, 0, 0)
centerLine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
centerLine.BorderSizePixel = 0
centerLine.ZIndex = 3
centerLine.Parent = progressBackground

-- HARD LOCK POSITION
statusFrame.Active = false
statusFrame.Draggable = false
statusFrame.Position = UDim2.new(0.5, -160, 0.85, 0)

local currentTarget = nil
local currentDist = math.huge
local cycleTimer = 0

RunService.Heartbeat:Connect(function(dt)
    strokeGradient.Rotation =
        (strokeGradient.Rotation + dt * 180) % 360

    local prompt, dist = findNearestPromptInPlots(2000)

    currentTarget = prompt
    currentDist = dist

    if not currentTarget then
        nameLabel.Text = "Searching..."
        percentLabel.Text = "0%"
        progressBar.Size = UDim2.new(0, 0, 1, 0)
        cycleTimer = 0
        return
    end

    local targetName = "Unknown"

    if currentTarget.ObjectText
        and currentTarget.ObjectText ~= "" then

        targetName = currentTarget.ObjectText

    elseif currentTarget.ActionText
        and currentTarget.ActionText ~= "" then

        targetName = currentTarget.ActionText

    elseif currentTarget.Parent then
        targetName = currentTarget.Parent.Name
    end

    nameLabel.Text = targetName

    cycleTimer = cycleTimer + dt

    local fullProgress =
        math.clamp(cycleTimer / 2.6, 0, 1)

    progressBar.Size =
        UDim2.new(fullProgress, 0, 1, 0)

    percentLabel.Text =
        string.format("%d%%", math.floor(fullProgress * 100))

    if cycleTimer < 1.3 then

        progressBar.BackgroundColor3 =
            Color3.fromRGB(155, 45, 255)

    elseif cycleTimer >= 1.3
        and cycleTimer < 2.6 then

        progressBar.BackgroundColor3 =
            Color3.fromRGB(210, 80, 255)

        if currentDist <= 8 then
            triggerPrompt(currentTarget)
            triggerHoldBegan(currentTarget)
            cycleTimer = 0
        end

    elseif cycleTimer >= 2.6 then

        cycleTimer = 0

        if currentTarget then
            triggerHoldBegan(currentTarget)
        end
    end
end)
