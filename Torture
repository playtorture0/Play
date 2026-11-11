--[[
    ╔══════════════════════════════════════════════════════════╗
    ║   🎯  TWEEN NEAREST FOLLOWER  v1.3                       ║
    ║   Loadstring / Executor Ready                            ║
    ║   Usage:                                                 ║
    ║     loadstring(game:HttpGet("RAW_URL_HERE"))()           ║
    ╚══════════════════════════════════════════════════════════╝
]]

--// Anti double-execute
if _G.__TweenFollowerLoaded then
    pcall(function() _G.__TweenFollowerGui:Destroy() end)
end
_G.__TweenFollowerLoaded = true

--// Services
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local Players      = game:GetService("Players")
local CoreGui      = game:GetService("CoreGui")
local UserInput    = game:GetService("UserInputService")
local StarterGui   = game:GetService("StarterGui")

--// Safe notify
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title    = "🎯 Tween Follower",
        Text     = "Loaded! Pressione RightShift para abrir o menu",
        Duration = 5,
    })
end)

--// ============================================
--// CONFIG
--// ============================================
local Config = {
    Enabled      = true,
    BaseSpeed    = 25,
    MinSpeed     = 12,
    MaxSpeed     = 90,
    SpeedFalloff = 8,
    FollowRange  = 150,
    UpdateRate   = 0.05,
    EasingStyle  = Enum.EasingStyle.Sine,
    EasingDir    = Enum.EasingDirection.Out,

    Offset = {
        Type     = "Behind", -- Behind | Front | Above | Below | Left | Right | Diagonal
        Distance = 8,
        Height   = 4,
    },

    LookAtTarget = true,

    -- "Player" = move o LocalPlayer / "Model" = move um Model do workspace
    Mode        = "Player",
    ModelName   = "MyPet",   -- nome do Model (se Mode == "Model")
    PrimaryPart = nil,       -- auto-detecta se nil

    -- Seguir somente um alvo específico (nome do player). nil = mais próximo
    SpecificTarget = nil,
}

--// ============================================
--// HELPERS
--// ============================================
local function getOffsetPosition(rootCF)
    local look  = rootCF.LookVector
    local right = rootCF.RightVector
    local up    = Vector3.new(0, 1, 0)
    local p     = rootCF.Position
    local t, d, h = Config.Offset.Type, Config.Offset.Distance, Config.Offset.Height

    if     t == "Behind"   then p = p - look  * d + up * h
    elseif t == "Front"    then p = p + look  * d + up * h
    elseif t == "Above"    then p = p + up    * d
    elseif t == "Below"    then p = p - up    * d
    elseif t == "Left"     then p = p - right * d + up * h
    elseif t == "Right"    then p = p + right * d + up * h
    elseif t == "Diagonal" then p = p - look * d + right * (d * 0.6) + up * h
    end
    return p
end

local function getNearestTargetCF(fromPos)
    local bestCF, bestDist = nil, Config.FollowRange

    for _, plr in ipairs(Players:GetPlayers()) do
        local char = plr.Character
        local hum  = char and char:FindFirstChildOfClass("Humanoid")
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root and hum and hum.Health > 0 then
            if Config.SpecificTarget and plr.Name ~= Config.SpecificTarget then
                continue
            end
            local dist = (root.Position - fromPos).Magnitude
            if dist < bestDist then
                bestDist, bestCF = dist, root.CFrame
            end
        end
    end
    return bestCF
end

-- Retorna {part = BasePart, isModel = bool, model = Model?}
local function getFollowerPart()
    if Config.Mode == "Model" then
        local model = workspace:FindFirstChild(Config.ModelName, true)
        if not model or not model:IsA("Model") then return nil end
        local part = Config.PrimaryPart and model:FindFirstChild(Config.PrimaryPart)
                     or model.PrimaryPart
        if not part then return nil end
        return { part = part, isModel = true, model = model }
    else
        local plr  = Players.LocalPlayer
        local char = plr and plr.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return nil end
        return { part = root, isModel = false }
    end
end

--// ============================================
--// MAIN LOOP
--// ============================================
local activeTween, acc = nil, 0

RunService.Heartbeat:Connect(function(dt)
    if not Config.Enabled then
        if activeTween then activeTween:Cancel(); activeTween = nil end
        return
    end

    acc += dt
    if acc < Config.UpdateRate then return end
    acc = 0

    local follower = getFollowerPart()
    if not follower then return end

    local targetCF = getNearestTargetCF(follower.part.Position)
    if not targetCF then return end

    local targetPos = getOffsetPosition(targetCF)
    local dist = (targetPos - follower.part.Position).Magnitude
    if dist < 0.1 then return end

    local speed = math.clamp(
        Config.BaseSpeed + (dist / Config.SpeedFalloff) * 10,
        Config.MinSpeed,
        Config.MaxSpeed
    )
    local duration = math.max(dist / speed, 0.05)

    local goalCF = Config.LookAtTarget
        and CFrame.lookAt(targetPos, targetCF.Position)
        or  CFrame.new(targetPos) * (follower.part.CFrame - follower.part.CFrame.Position)

    if activeTween then activeTween:Cancel() end

    activeTween = TweenService:Create(
        follower.part,
        TweenInfo.new(duration, Config.EasingStyle, Config.EasingDir),
        { CFrame = goalCF }
    )
    activeTween:Play()
end)

--// ============================================
--// GUI
--// ============================================
local old = CoreGui:FindFirstChild("TweenFollowerGui")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "TweenFollowerGui"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = CoreGui
_G.__TweenFollowerGui = gui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 380)
frame.Position = UDim2.new(0, 30, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(70, 70, 90)
stroke.Thickness = 1
stroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
title.BorderSizePixel = 0
title.Text = "🎯  TWEEN FOLLOWER  v1.3"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.Parent = frame
Instance.new("UICorner", title).CornerRadius = UDim.new(0, 10)

local list = Instance.new("Frame")
list.Size = UDim2.new(1, -16, 1, -56)
list.Position = UDim2.new(0, 8, 0, 48)
list.BackgroundTransparency = 1
list.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 6)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = list

local function makeButton(text)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(230, 230, 230)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 13
    btn.AutoButtonColor = true
    btn.Parent = list
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    return btn
end

-- Toggle ON/OFF
local toggleBtn = makeButton("")
local function refreshToggle()
    toggleBtn.Text = Config.Enabled and "🟢  Status: ON" or "🔴  Status: OFF"
    toggleBtn.BackgroundColor3 = Config.Enabled
        and Color3.fromRGB(40, 120, 60)
        or  Color3.fromRGB(120, 40, 40)
end
refreshToggle()
toggleBtn.MouseButton1Click:Connect(function()
    Config.Enabled = not Config.Enabled
    refreshToggle()
end)

-- Offset
local offsetBtn = makeButton("")
local offsets = { "Behind", "Front", "Above", "Below", "Left", "Right", "Diagonal" }
local function refreshOffset() offsetBtn.Text = "Offset: " .. Config.Offset.Type end
refreshOffset()
offsetBtn.MouseButton1Click:Connect(function()
    for i, v in ipairs(offsets) do
        if v == Config.Offset.Type then
            Config.Offset.Type = offsets[(i % #offsets) + 1]
            break
        end
    end
    refreshOffset()
end)

-- BaseSpeed
local speedBtn = makeButton("")
local function refreshSpeed() speedBtn.Text = "BaseSpeed: " .. Config.BaseSpeed end
refreshSpeed()
speedBtn.MouseButton1Click:Connect(function()
    Config.BaseSpeed = Config.BaseSpeed >= 100 and 10 or Config.BaseSpeed + 10
    refreshSpeed()
end)

-- Range
local rangeBtn = makeButton("")
local function refreshRange() rangeBtn.Text = "Range: " .. Config.FollowRange end
refreshRange()
rangeBtn.MouseButton1Click:Connect(function()
    Config.FollowRange = Config.FollowRange >= 500 and 50 or Config.FollowRange + 50
    refreshRange()
end)

-- LookAt
local lookBtn = makeButton("")
local function refreshLook() lookBtn.Text = "LookAt: " .. (Config.LookAtTarget and "ON" or "OFF") end
refreshLook()
lookBtn.MouseButton1Click:Connect(function()
    Config.LookAtTarget = not Config.LookAtTarget
    refreshLook()
end)

-- Mode Player/Model
local modeBtn = makeButton("")
local function refreshMode() modeBtn.Text = "Mode: " .. Config.Mode end
refreshMode()
modeBtn.MouseButton1Click:Connect(function()
    Config.Mode = (Config.Mode == "Player") and "Model" or "Player"
    refreshMode()
end)

-- Fechar
local closeBtn = makeButton("❌  Fechar")
closeBtn.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
closeBtn.MouseButton1Click:Connect(function()
    if _G.__TweenFollowerGui then _G.__TweenFollowerGui:Destroy() end
    _G.__TweenFollowerLoaded = false
end)

-- Toggle visibilidade
UserInput.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift and gui.Parent then
        frame.Visible = not frame.Visible
    end
end)

print("[TweenFollower] v1.3 carregado com sucesso ✅")
