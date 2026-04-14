--[[
================================================================================
  👑 KING AKBARION - ULTIMATE BARISTA SCRIPT 👑
================================================================================
    [+] Developer   : Akbarion
    [+] Version     : DDS FREE EDITION (v4.6 - FULL SIZE & FIX SERVE BUG)
    [+] UI Theme    : Sodium Menu / MySuperHub
    [+] Features    : Safe Webhook, Perfect Pathing, Auto-Repair, Anti-Fail Serve
================================================================================
]]--

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

-- ============================================================================
-- // 1. SERVICES & REFERENCES
-- ============================================================================
local Services = {
    Players      = game:GetService("Players"),
    RunService   = game:GetService("RunService"),
    TweenService = game:GetService("TweenService"),
    UserInput    = game:GetService("UserInputService"),
    Stats        = game:GetService("Stats"),
    RepStorage   = game:GetService("ReplicatedStorage"),
    Workspace    = game:GetService("Workspace"),
    VIM          = game:GetService("VirtualInputManager"),
    VirtualUser  = game:GetService("VirtualUser"),
    HttpService  = game:GetService("HttpService")
}

local LocalPlayer = Services.Players.LocalPlayer
local IsMobile    = Services.UserInput.TouchEnabled and not Services.UserInput.KeyboardEnabled

local CharRef = {
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait(),
    Humanoid  = nil,
    Root      = nil
}

CharRef.Humanoid = CharRef.Character:WaitForChild("Humanoid")
CharRef.Root     = CharRef.Character:WaitForChild("HumanoidRootPart")

LocalPlayer.CharacterAdded:Connect(function(newChar)
    CharRef.Character = newChar
    CharRef.Humanoid  = newChar:WaitForChild("Humanoid")
    CharRef.Root      = newChar:WaitForChild("HumanoidRootPart")
end)

-- ============================================================================
-- // 2. STATE MANAGER & SECURITY
-- ============================================================================
local State = {
    IsFarming   = false,
    FarmThread  = nil,
    AiThread    = nil,
    StatusText  = "Lagi Santai Coy",
    OrderCount  = 0,
    ActionDelay = 2,
    AntiAFK     = true,
    AntiAdmin   = true,
    
    -- Webhook Config
    WebhookURL      = "",
    WebhookEnabled  = false,
    WebhookInterval = 10,
    UangAwal        = 0
}

LocalPlayer.Idled:Connect(function()
    if State.AntiAFK then
        Services.VirtualUser:CaptureController()
        Services.VirtualUser:ClickButton2(Vector2.new())
    end
end)

local GAME_GROUP_ID = 11378976
local MIN_STAFF_RANK = 200

local function CheckForAdmin(player)
    if not State.AntiAdmin then return end
    if player == LocalPlayer then return end
    pcall(function()
        local playerRank = player:GetRankInGroup(GAME_GROUP_ID)
        if playerRank >= MIN_STAFF_RANK then
            LocalPlayer:Kick("🚨 KABUR BRAY! Admin terdeteksi masuk server.")
        end
    end)
end

for _, player in ipairs(Services.Players:GetPlayers()) do CheckForAdmin(player) end
Services.Players.PlayerAdded:Connect(CheckForAdmin)

-- ============================================================================
-- // 3. SPLASH SCREEN (KING AKBARION - DEFAULT THEME)
-- ============================================================================
do
    local splashGui = Instance.new("ScreenGui")
    splashGui.Name           = "BaristaSplash"
    splashGui.ResetOnSpawn   = false
    splashGui.IgnoreGuiInset = true
    splashGui.DisplayOrder   = 999
    splashGui.Parent         = LocalPlayer:WaitForChild("PlayerGui")

    local bg = Instance.new("Frame", splashGui)
    bg.Size             = UDim2.fromScale(1, 1)
    bg.BackgroundColor3 = Color3.fromHex("#0a0a0a")
    bg.BorderSizePixel  = 0
    bg.ZIndex           = 1

    local uiGrad = Instance.new("UIGradient", bg)
    uiGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromHex("#0a0a0a")),
        ColorSequenceKeypoint.new(1, Color3.fromHex("#1e1e1e")),
    })
    uiGrad.Rotation = 135

    local container = Instance.new("Frame", bg)
    container.Size               = UDim2.fromOffset(500, 300)
    container.Position           = UDim2.fromScale(0.5, 0.5)
    container.AnchorPoint        = Vector2.new(0.5, 0.5)
    container.BackgroundTransparency = 1
    container.ZIndex             = 2

    local function createLabel(text, yOff, size)
        local lbl = Instance.new("TextLabel", container)
        lbl.Size                = UDim2.fromOffset(500, 70)
        lbl.Position            = UDim2.fromOffset(0, yOff)
        lbl.BackgroundTransparency = 1
        lbl.Text                = text
        lbl.TextSize            = size
        lbl.Font                = Enum.Font.GothamBold
        lbl.TextColor3          = Color3.fromHex("#ffffff")
        lbl.TextTransparency    = 1
        lbl.ZIndex              = 3
        return lbl
    end

    local iconImg = Instance.new("ImageLabel", container)
    iconImg.Size = UDim2.fromOffset(120, 120) 
    iconImg.Position = UDim2.fromOffset(190, -40) 
    iconImg.BackgroundTransparency = 1
    iconImg.Image = "rbxassetid://91115084979317" 
    iconImg.ImageTransparency = 1
    iconImg.ZIndex = 3

    local titleLbl = createLabel("King Akbarion", 70, IsMobile and 38 or 50)
    local titleGrad = Instance.new("UIGradient", titleLbl)
    titleGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0,   Color3.fromHex("#ffffff")),
        ColorSequenceKeypoint.new(0.5, Color3.fromHex("#aaaaaa")),
        ColorSequenceKeypoint.new(1,   Color3.fromHex("#555555")),
    })
    titleGrad.Rotation = 45

    local statLbl = createLabel("Memuat DDS Free Script...", 200, 12)
    statLbl.Font             = Enum.Font.Gotham
    statLbl.TextColor3       = Color3.fromHex("#555555")
    statLbl.TextXAlignment   = Enum.TextXAlignment.Left
    statLbl.Position         = UDim2.fromOffset(50, 200)

    local line = Instance.new("Frame", container)
    line.Size             = UDim2.fromOffset(0, 2)
    line.Position         = UDim2.fromOffset(250, 152)
    line.AnchorPoint      = Vector2.new(0.5, 0)
    line.BackgroundColor3 = Color3.fromHex("#444444")
    line.BorderSizePixel  = 0
    line.ZIndex           = 3

    local barBg = Instance.new("Frame", container)
    barBg.Size                   = UDim2.fromOffset(400, 5)
    barBg.Position               = UDim2.fromOffset(50, 190)
    barBg.BackgroundColor3       = Color3.fromHex("#222222")
    barBg.BackgroundTransparency = 1
    barBg.BorderSizePixel        = 0
    barBg.ZIndex                 = 3
    Instance.new("UICorner", barBg).CornerRadius = UDim.new(1, 0)

    local bar = Instance.new("Frame", barBg)
    bar.Size             = UDim2.fromOffset(0, 5)
    bar.BackgroundColor3 = Color3.fromHex("#ffffff")
    bar.BorderSizePixel  = 0
    bar.ZIndex           = 4
    Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

    local function tween(obj, props, t)
        Services.TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props):Play()
    end

    local loadSteps = {
        { "Fixing Serve Bug...",           0.30 },
        { "Menyiapkan AI Barista...",      0.60 },
        { "Welcome, Akbarion!",            1.00 },
    }

    task.spawn(function()
        tween(iconImg,  { ImageTransparency = 0 }, 0.5)
        task.wait(0.15)
        tween(titleLbl, { TextTransparency = 0 }, 0.6)
        task.wait(0.35)
        tween(line,     { Size = UDim2.fromOffset(400, 2) }, 0.7)
        task.wait(0.4)
        tween(barBg,    { BackgroundTransparency = 0 }, 0.3)
        tween(statLbl,  { TextTransparency = 0 }, 0.3)

        for _, step in ipairs(loadSteps) do
            statLbl.Text = step[1]
            tween(bar, { Size = UDim2.fromOffset(400 * step[2], 5) }, 0.5)
            task.wait(0.55)
        end

        task.wait(0.3)
        tween(bg,       { BackgroundTransparency = 1 }, 0.6)
        tween(iconImg,  { ImageTransparency = 1 }, 0.4)
        tween(titleLbl, { TextTransparency = 1 }, 0.4)
        tween(line,     { BackgroundTransparency = 1 }, 0.4)
        tween(barBg,    { BackgroundTransparency = 1 }, 0.4)
        tween(bar,      { BackgroundTransparency = 1 }, 0.4)
        tween(statLbl,  { TextTransparency = 1 }, 0.4)
        task.wait(0.8)
        splashGui:Destroy()
    end)
    task.wait(3)
end

-- ============================================================================
-- // 4. CONSTANTS & WAYPOINTS
-- ============================================================================
local Constants = {
    START_SHIFT  = Vector3.new(-4991.23, 4.29, -715.26),
    COLOR_ORANGE = Color3.fromRGB(230, 150, 30),
    COLOR_GREEN  = Color3.fromRGB(30, 180, 60),
}

local Paths = {
    START_TO_MACHINE = {
        Vector3.new(-4991.23, 4.29, -715.26),
        Vector3.new(-5004.86, 4.29, -718.90),
        Vector3.new(-5006.28, 4.29, -802.11),
        Vector3.new(-4994.18, 4.29, -801.66),
        Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4998.16, 4.29, -794.80),
    },
    MACHINE_TO_CASHIER = {
        Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4995.56, 4.29, -759.78),
    },
    CASHIER_TO_MACHINE = {
        Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4998.16, 4.29, -794.80),
    },
    MACHINE_TO_START = {
        Vector3.new(-4998.16, 4.29, -794.80),
        Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4994.18, 4.29, -801.66),
        Vector3.new(-5006.28, 4.29, -802.11),
        Vector3.new(-5004.86, 4.29, -718.90),
        Vector3.new(-4991.23, 4.29, -715.26),
    },
    CASHIER_TO_START = {
        Vector3.new(-4995.56, 4.29, -759.78),
        Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4994.18, 4.29, -801.66),
        Vector3.new(-5006.28, 4.29, -802.11),
        Vector3.new(-5004.86, 4.29, -718.90),
        Vector3.new(-4991.23, 4.29, -715.26),
    },
    MACHINE_TO_FIX = {
        Vector3.new(-4998.14, 4.29, -795.38),
        Vector3.new(-4997.02, 4.29, -802.18),
        Vector3.new(-5006.31, 4.29, -802.30),
        Vector3.new(-5003.75, 4.29, -711.60),
        Vector3.new(-5004.43, 3.19, -670.40),
        Vector3.new(-5114.86, 3.19, -670.41),
    },
    FIX_TO_MACHINE = {
        Vector3.new(-5114.86, 3.19, -670.41),
        Vector3.new(-5004.43, 3.19, -670.40),
        Vector3.new(-5003.75, 4.29, -711.60),
        Vector3.new(-5006.31, 4.29, -802.30),
        Vector3.new(-4997.02, 4.29, -802.18),
        Vector3.new(-4998.14, 4.29, -795.38),
    }
}

-- ============================================================================
-- // 5. MONEY TRACKER & DDS WEBHOOK
-- ============================================================================
local function GetPlayerMoney()
    local money = 0
    pcall(function()
        if LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Money") then
            money = LocalPlayer.leaderstats.Money.Value
        elseif LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Money") then
            money = LocalPlayer.Data.Money.Value
        else
            -- UI Scraper Fallback
            for _, v in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if v:IsA("TextLabel") and v.Visible and string.find(v.Text, "Rp%.") then
                    local numStr = string.gsub(v.Text, "[^%d]", "")
                    local m = tonumber(numStr)
                    if m and m > money then money = m end
                end
            end
        end
    end)
    return money
end

local function FormatMoney(amount)
    local formatted = tostring(amount)
    local k
    while true do
        formatted, k = string.gsub(formatted, "^(-?%d+)(%d%d%d)", '%1.%2')
        if k == 0 then break end
    end
    return "Rp. " .. formatted
end

local function SendDiscordReport()
    if not State.WebhookEnabled or State.WebhookURL == "" or State.WebhookURL == "Paste Link Disini" then return end
    
    local success, err = pcall(function()
        local request = (syn and syn.request) or (http and http.request) or http_request or request
        if request then
            local uangSekarang = GetPlayerMoney()
            local profit = uangSekarang - State.UangAwal
            local timeStr = os.date("%H:%M:%S")
            
            -- SENSOR NAMA
            local pName = LocalPlayer.Name
            local safeName = string.sub(pName, 1, 4) .. "...."
            
            -- FORMAT DDS SCRIPT (FREE)
            local description = string.format(
                "👤 **Akun:** %s\n" ..
                "Status: 🟢 ` Aman Lancar Jaya `\n\n" ..
                "💰 Uang Awal\n" ..
                "**%s**\n" ..
                "💵 Uang Sekarang\n" ..
                "**%s**\n" ..
                "📈 Total Profit\n" ..
                "```diff\n" ..
                "+ %s\n" ..
                "```\n" ..
                "☕ Kopi Terjual\n" ..
                "```\n" ..
                "%dx\n" ..
                "```",
                safeName,
                FormatMoney(State.UangAwal),
                FormatMoney(uangSekarang),
                FormatMoney(profit),
                State.OrderCount
            )

            local data = {
                ["embeds"] = {{
                    ["title"] = "⚙️ King Akbarion - Monitoring Profit",
                    ["description"] = description,
                    ["color"] = 16766720, -- Warna Kuning Bar
                    ["footer"] = {["text"] = "DDS Free Script • Time: " .. timeStr}
                }}
            }
            
            request({
                Url = State.WebhookURL,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = Services.HttpService:JSONEncode(data)
            })
        end
    end)
end

-- Timer Webhook berkala
task.spawn(function()
    while true do
        task.wait(60)
        if State.WebhookEnabled and State.IsFarming then
            for i = 1, State.WebhookInterval - 1 do
                if not State.IsFarming then break end
                task.wait(60)
            end
            if State.IsFarming then SendDiscordReport() end
        end
    end
end)


-- ============================================================================
-- // 6. UTILITY METHODS (LIVE SCAN + UI SENSOR)
-- ============================================================================
local function WalkToPoint(targetPos)
    if not CharRef.Character or not CharRef.Humanoid or not CharRef.Root then return end
    CharRef.Humanoid.Sit = false
    CharRef.Humanoid:MoveTo(targetPos)
    
    local timeOut = 10 
    while timeOut > 0 and State.IsFarming do
        local flatDist = Vector3.new(CharRef.Root.Position.X, 0, CharRef.Root.Position.Z) - Vector3.new(targetPos.X, 0, targetPos.Z)
        if flatDist.Magnitude < 3 then break end
        task.wait(0.1)
        timeOut -= 0.1
    end
end

local function FollowPath(pathArray)
    for _, pos in ipairs(pathArray) do
        if not State.IsFarming then break end
        WalkToPoint(pos)
    end
end

local function FindPrompt(keyword, maxDist, origin)
    if not CharRef.Root then return nil end
    origin = origin or CharRef.Root.Position
    maxDist = maxDist or 20
    
    local found, closest = nil, maxDist
    for _, v in pairs(Services.Workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled and string.find(string.lower(v.ActionText), string.lower(keyword)) then
            local part = v.Parent
            if part and part:IsA("BasePart") then
                local d = (part.Position - origin).Magnitude
                if d < closest then
                    closest = d
                    found = v
                end
            end
        end
    end
    return found
end

local function DoHold(prompt)
    if not prompt then return false end
    pcall(function()
        prompt:InputHoldBegin()
        task.wait((prompt.HoldDuration or 1) + 0.4) -- Buffer biar hold nggak cepet lepas
        prompt:InputHoldEnd()
    end)
    task.wait(0.2)
    return true
end

local function DoTap(prompt)
    if not prompt then return false end
    pcall(function()
        if fireproximityprompt then
            fireproximityprompt(prompt, 1, true)
        else
            prompt:InputHoldBegin()
            task.wait(0.05)
            prompt:InputHoldEnd()
        end
    end)
    task.wait(0.2)
    return true
end

-- ============================================================================
-- // SENSOR DEWA: CEK MESIN RUSAK LANGSUNG DARI TULISAN LAYAR
-- ============================================================================
local function IsMachineBroken()
    for _, gui in pairs(LocalPlayer.PlayerGui:GetChildren()) do
        for _, v in pairs(gui:GetDescendants()) do
            if v:IsA("TextLabel") and v.Visible then
                local txt = string.lower(v.Text)
                if string.find(txt, "machine broke") or string.find(txt, "needs maintenance") or string.find(txt, "fix machine") then
                    return true
                end
            end
        end
    end
    return false
end

local function HasJob()
    local hasJob = true 
    for _, v in pairs(Services.Workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled and string.find(string.lower(v.ActionText), "shift") then
            local part = v.Parent
            if part and part:IsA("BasePart") then
                local dist = (part.Position - Constants.START_SHIFT).Magnitude
                if dist < 40 then
                    if string.find(string.lower(v.ActionText), "end") then
                        hasJob = true
                    elseif string.find(string.lower(v.ActionText), "start") then
                        hasJob = false
                    end
                    break
                end
            end
        end
    end
    return hasJob
end

local function FindByColor(parent, targetColor, tolerance)
    local best, bestDiff = nil, math.huge
    for _, v in pairs(parent:GetDescendants()) do
        if (v:IsA("Frame") or v:IsA("ImageLabel")) and v.Visible then
            local c = v:IsA("ImageLabel") and v.ImageColor3 or v.BackgroundColor3
            if v.BackgroundTransparency < 0.8 then
                local diff = math.abs(c.R - targetColor.R) + math.abs(c.G - targetColor.G) + math.abs(c.B - targetColor.B)
                if diff < bestDiff then
                    bestDiff = diff
                    best = v
                end
            end
        end
    end
    return (bestDiff < (tolerance or 0.6)) and best or nil
end

-- ============================================================================
-- // 7. AI MINIGAME (3-LAYER SMART SENSOR)
-- ============================================================================
local function StartMinigameAI()
    if State.AiThread then task.cancel(State.AiThread) end
    
    State.AiThread = task.spawn(function()
        local cam = Services.Workspace.CurrentCamera

        while State.IsFarming do
            task.wait(0.016)

            local gui = LocalPlayer.PlayerGui:FindFirstChild("BaristaGUI")
            if not gui then task.wait(0.1); continue end

            local mf = gui:FindFirstChild("MinigameFrame", true)
            if not (mf and mf.Visible) then task.wait(0.1); continue end

            local cx = cam.ViewportSize.X / 2
            local cy = cam.ViewportSize.Y / 2
            local pill, bar = nil, nil

            for _, v in pairs(mf:GetDescendants()) do
                if v:IsA("Frame") or v:IsA("ImageLabel") then
                    local nm = string.lower(v.Name)
                    if nm:find("pill") or nm:find("indicator") or nm:find("player") or nm:find("handle") then pill = v end
                    if nm:find("target") or nm:find("zone") or nm:find("goal") or nm:find("safe") then bar = v end
                end
            end

            if not pill then pill = FindByColor(mf, Constants.COLOR_ORANGE, 0.6) end
            if not bar then bar = FindByColor(mf, Constants.COLOR_GREEN, 0.6) end

            if not pill or not bar then
                local allEls = {}
                for _, v in pairs(mf:GetDescendants()) do
                    if (v:IsA("Frame") or v:IsA("ImageLabel")) and v.Visible and v.BackgroundTransparency < 0.9 and v.AbsoluteSize.Y > 10 then
                        table.insert(allEls, v)
                    end
                end
                table.sort(allEls, function(a, b) return a.AbsolutePosition.X < b.AbsolutePosition.X end)
                if #allEls >= 2 then pill = allEls[1]; bar = allEls[#allEls] end
            end

            if pill and bar then
                local pillCY = pill.AbsolutePosition.Y + (pill.AbsoluteSize.Y / 2)
                local barCY  = bar.AbsolutePosition.Y  + (bar.AbsoluteSize.Y  / 2)
                local diff   = pillCY - barCY

                if diff > 6 then
                    Services.VIM:SendMouseButtonEvent(cx, cy, 0, true,  game, 1)
                    task.wait(math.random(55, 90) / 1000)
                    Services.VIM:SendMouseButtonEvent(cx, cy, 0, false, game, 1)
                    task.wait(math.random(30, 60) / 1000)
                elseif diff < -6 then
                    task.wait(0.016)
                else
                    Services.VIM:SendMouseButtonEvent(cx, cy, 0, true,  game, 1)
                    task.wait(math.random(50, 80) / 1000)
                    Services.VIM:SendMouseButtonEvent(cx, cy, 0, false, game, 1)
                    task.wait(math.random(80, 130) / 1000)
                end
            else
                Services.VIM:SendMouseButtonEvent(cx, cy, 0, true,  game, 1)
                task.wait(math.random(55, 90) / 1000)
                Services.VIM:SendMouseButtonEvent(cx, cy, 0, false, game, 1)
                task.wait(math.random(60, 100) / 1000)
            end
        end
    end)
end

-- ============================================================================
-- // 8. MAIN FARMING LOOP (RESTRUCTURED & FIXED SERVE BUG)
-- ============================================================================
local function TakeJob()
    State.StatusText = "🏃 Otw ambil job barusan..."
    WalkToPoint(Constants.START_SHIFT)
    task.wait(0.5)

    local shiftPrompt = FindPrompt("start shift", 30) or FindPrompt("shift", 30)
    if shiftPrompt and string.find(string.lower(shiftPrompt.ActionText), "start") then
        State.StatusText = "💼 Udah dapet job, gass!"
        DoTap(shiftPrompt)
        task.wait(1)
    end
end

local function HasPendingOrder()
    local machinePos = Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]
    local brew = FindPrompt("brewing", 40, machinePos) or FindPrompt("brew", 40, machinePos) or FindPrompt("make", 40, machinePos)
    return brew ~= nil
end

local function FarmLoop()
    local isAtCashier = false 
    
    while State.IsFarming do
        if not CharRef.Character or not CharRef.Character.Parent then
            CharRef.Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            CharRef.Humanoid = CharRef.Character:WaitForChild("Humanoid")
            CharRef.Root = CharRef.Character:WaitForChild("HumanoidRootPart")
        end

        if not HasJob() then
            State.StatusText = "⚠️ Lah di-kick jobnya, putar balik dulu..."
            local distToMachine = (CharRef.Root.Position - Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]).Magnitude
            local distToCashier = (CharRef.Root.Position - Paths.MACHINE_TO_CASHIER[#Paths.MACHINE_TO_CASHIER]).Magnitude
            if distToMachine < distToCashier then
                FollowPath(Paths.MACHINE_TO_START)
            else
                FollowPath(Paths.CASHIER_TO_START)
            end
            TakeJob()
            State.StatusText = "🚶 Balik ke spot kerja ah..."
            FollowPath(Paths.START_TO_MACHINE)
            isAtCashier = false
            continue
        end

        while not HasPendingOrder() and not IsMachineBroken() and State.IsFarming do
            State.StatusText = "⏳ Sepi nih, nunggu pembeli..."
            task.wait(1)
        end
        
        if not State.IsFarming then break end
        if not HasJob() then continue end

        if IsMachineBroken() then
            State.StatusText = "🚨 Waduh mesin meledak!"
            if isAtCashier then
                State.StatusText = "🚶 Jalan dari kasir ke mesin..."
                FollowPath(Paths.CASHIER_TO_MACHINE)
                isAtCashier = false
            end
            State.StatusText = "🚶 Otw ke luar meja (jalur perbaikan)..."
            FollowPath(Paths.MACHINE_TO_FIX)
            task.wait(0.5)
            
            local realFix = FindPrompt("fix", 20) or FindPrompt("repair", 20) or FindPrompt("clean", 20) or FindPrompt("maintain", 20)
            if realFix then
                State.StatusText = "🔧 Benerin mesin dulu..."
                DoHold(realFix)
            else
                for _, v in pairs(Services.Workspace:GetDescendants()) do
                    if v:IsA("ProximityPrompt") and v.Enabled then
                        local part = v.Parent
                        if part and part:IsA("BasePart") then
                            if (part.Position - CharRef.Root.Position).Magnitude < 15 then
                                DoHold(v)
                            end
                        end
                    end
                end
            end
            task.wait(0.5)

            State.StatusText = "🚶 Balik masuk ke dalem meja..."
            FollowPath(Paths.FIX_TO_MACHINE)
            continue 
        end

        if HasPendingOrder() then
            if isAtCashier then
                State.StatusText = "🚶 Otw mesin kopi bray..."
                FollowPath(Paths.CASHIER_TO_MACHINE)
                isAtCashier = false
            else
                WalkToPoint(Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE])
            end

            local brewPrompt = FindPrompt("brewing", 30, Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]) or FindPrompt("brew", 30, Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]) or FindPrompt("make", 30, Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE])
            if brewPrompt then
                State.StatusText = "🔄 Lagi nyeduh kopi..."
                DoTap(brewPrompt)
                task.wait(1) 
                while State.IsFarming do
                    local gui = LocalPlayer.PlayerGui:FindFirstChild("BaristaGUI")
                    local mf = gui and gui:FindFirstChild("MinigameFrame", true)
                    if not mf or not mf.Visible then break end
                    task.wait(0.5)
                end
            end
            task.wait(1)

            State.StatusText = "🥤 Ngambil kopi dari mesin..."
            local drinkPrompt = FindPrompt("take", 25, Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]) or FindPrompt("grab", 25, Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE])
            if drinkPrompt then 
                DoTap(drinkPrompt) 
            end
            task.wait(0.5)
            
            local tool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool") or CharRef.Character:FindFirstChildOfClass("Tool")
            if tool then CharRef.Humanoid:EquipTool(tool) end

            State.StatusText = "🚶 Otw nganter ke kasir..."
            FollowPath(Paths.MACHINE_TO_CASHIER)
            isAtCashier = true 

            -- ==========================================
            -- [FIX BUG SERVE PHP]: Loop sampai Kopi Hilang
            -- ==========================================
            local serveAttempt = 0
            while CharRef.Character:FindFirstChildOfClass("Tool") and State.IsFarming and serveAttempt < 5 do
                State.StatusText = "🥤 Menyerahkan Kopi... (Hold)"
                local servePrompt = FindPrompt("serve", 25) or FindPrompt("deliver", 25)
                
                if servePrompt then
                    DoHold(servePrompt) 
                else
                    break
                end
                
                serveAttempt += 1
                task.wait(0.5) -- Tunggu sebentar ngecek apa kopinya beneran udah lenyap
            end

            -- ==========================================
            -- PASTIKAN KOPI SUDAH HILANG DARI TANGAN SEBELUM NYATET PROFIT
            -- ==========================================
            if not CharRef.Character:FindFirstChildOfClass("Tool") then
                State.OrderCount += 1
                State.StatusText = "✅ Mantap! Kopi laku: " .. State.OrderCount
            else
                State.StatusText = "❌ Gagal Serve, kopi masih nyangkut!"
            end

            task.wait(State.ActionDelay)
        end
    end
end

local function StartScript()
    if State.IsFarming then return end
    State.IsFarming = true
    State.UangAwal = GetPlayerMoney() -- NGUNCI MODAL AWAL DI SINI
    task.spawn(function()
        TakeJob()
        StartMinigameAI()
        FarmLoop()
    end)
end

local function StopScript()
    State.IsFarming = false
    State.StatusText = "⏹️ Istirahat Dulu"
    if CharRef.Humanoid and CharRef.Root then 
        CharRef.Humanoid:MoveTo(CharRef.Root.Position) 
    end
end

-- ============================================================================
-- // 9. UI CONSTRUCTION
-- ============================================================================
local windowSize = IsMobile and UDim2.fromOffset(420, 320) or UDim2.fromOffset(580, 460)
local minSize    = IsMobile and Vector2.new(600, 300) or Vector2.new(600, 350)
local maxSize    = IsMobile and Vector2.new(650, 400) or Vector2.new(850, 560)

local Window = WindUI:CreateWindow({
    Title = "King Akbarion",
    Icon = "crown",
    Author = "Akbarion",
    Folder = "MySuperHub",
    Size = windowSize,
    MinSize = minSize,
    MaxSize = maxSize,
    Transparent = false,
    Background = "rbxassetid://127295801178451",
    BackgroundImageTransparency = 0.5,
    Theme = "Dark",
    Resizable = true,
    SideBarWidth = 200,
    HideSearchBar = false,
    ScrollBarEnabled = true,
})

local TabBarista = Window:Tab({ Title = "Menu Utama", Icon = "coffee", Border = true })
local StatusParagraph = TabBarista:Paragraph({ Title = "Info System", Desc = "Lagi Santai Coy", Transparent = true })
local OrderParagraph  = TabBarista:Paragraph({ Title = "Statistik", Desc = "0", Transparent = true })

task.spawn(function()
    while task.wait(0.5) do
        pcall(function()
            StatusParagraph:Set({ Desc = State.StatusText })
            OrderParagraph:Set({ Desc = "Kopi Kejual: " .. State.OrderCount })
        end)
    end
end)

TabBarista:Toggle({
    Title = "Gas Auto Barista",
    Desc = "Nyalain fitur auto nyeduh & nguli 24/7",
    Icon = "play",
    Value = false,
    Callback = function(state) if state then StartScript() else StopScript() end end
})

-- TAB WEBHOOK
local TabWeb = Window:Tab({ Title = "Webhook", Icon = "bell", Border = true })
TabWeb:Toggle({ Title = "Nyalain Webhook", Callback = function(s) State.WebhookEnabled = s end })
TabWeb:Input({ Title = "Link Webhook Discord", Placeholder = "Paste Link Disini", Callback = function(v) State.WebhookURL = v end })
TabWeb:Slider({ Title = "Jeda Laporan (Menit)", Value = {Min = 1, Max = 60, Default = 10}, Callback = function(v) State.WebhookInterval = v end })
TabWeb:Button({ Title = "Test Kirim Laporan", Callback = function() 
    if State.UangAwal == 0 then State.UangAwal = GetPlayerMoney() end
    State.WebhookEnabled = true; SendDiscordReport()
    WindUI:Notify({Title = "Berhasil!", Content = "Cek laporan di Discord bosku!"}) 
end})

TabBarista:Section({ Title = "🚀 Tombol Darurat" })
TabBarista:Button({ Title = "Ambil Job Manual", Callback = function() task.spawn(TakeJob) end })
TabBarista:Button({ Title = "Jalan ke Mesin Kopi", Callback = function() FollowPath(Paths.START_TO_MACHINE) end })
TabBarista:Button({ Title = "Jalan ke Kasir", Callback = function() FollowPath(Paths.MACHINE_TO_CASHIER) end })

local TabSecurity = Window:Tab({ Title = "Keamanan", Icon = "shield", Border = true })
TabSecurity:Toggle({ Title = "Anti-AFK Bypass", Icon = "clock", Value = true, Callback = function(state) State.AntiAFK = state end })
TabSecurity:Toggle({ Title = "Radar Anti-Admin", Icon = "user-minus", Value = true, Callback = function(state) State.AntiAdmin = state end })

local TabSettings = Window:Tab({ Title = "Pengaturan", Icon = "settings", Border = true })
TabSettings:Slider({ Title = "Jeda Santai (Detik)", Step = 1, Value = { Min = 1, Max = 10, Default = 2 }, Callback = function(v) State.ActionDelay = v end })

Window:EditOpenButton({ Title = "Open King Akbarion", Icon = "crown", CornerRadius = UDim.new(0, 12), StrokeThickness = 2, Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromHex("#ffffff")), ColorSequenceKeypoint.new(1, Color3.fromHex("#0a0a0a"))}), Enabled = true, Draggable = true })

local FpsTag = Window:Tag({ Title = "Fps: Ngitung...", Color = WindUI:Gradient({[0] = {Color = Color3.fromHex("#0a0a0a"), Transparency = 0}, [100] = {Color = Color3.fromHex("#888888"), Transparency = 0}}, {Rotation = 45}) })

task.spawn(function()
    while task.wait(1) do
        pcall(function()
            local fps = math.floor(1 / Services.RunService.RenderStepped:Wait())
            local ping = math.floor(Services.Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            if FpsTag and FpsTag.SetTitle then FpsTag:SetTitle(string.format("Fps: %d | Ping: %d", fps, ping)) end
        end)
    end
end)

Window:SetIconSize(47)
WindUI:SetTheme("dark")
TabBarista:Select()
WindUI:Notify({ Title = "👑 KING AKBARION READY!", Content = "Base V4.1 sukses digabung sama Webhook DDS Free!", Duration = 5 })
