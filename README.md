
-- =============================================
--  Silva244cc HUB | Tecla P = abrir/fechar
--  🎨 Lucky Style  → +3 a cada 3 segundos
--  ⚡ Ability Spin → +3 a cada 4 segundos
--  💎 Gemas        → +50 a cada 5 segundos
--  🔍 Remote Spy integrado (auto-detecção)
-- =============================================

local Players            = game:GetService("Players")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ─────────────────────────────────────────────
-- REMOTE SPY
-- ─────────────────────────────────────────────
local detectedRemotes = {}
local spyLog          = {}
local MAX_LOG         = 40

local STYLE_KEYWORDS   = {"style","lucky","spin","spins","giveStyle","addStyle","styleGive"}
local ABILITY_KEYWORDS = {"ability","abilit","power","addAbility","giveAbility","abilityGive"}
local GEM_KEYWORDS     = {"gem","gems","gema","gemas","crystal","jewel","diamond","addGem","giveGem","currency","coin","coins","money"}

local function nameMatchesAny(name, keywords)
    local low = name:lower()
    for _, kw in ipairs(keywords) do
        if low:find(kw:lower(), 1, true) then return true end
    end
    return false
end

local function scanInstance(root)
    for _, obj in ipairs(root:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
            if not detectedRemotes[obj.Name] then
                detectedRemotes[obj.Name] = obj
                local tag = obj:IsA("RemoteEvent") and "[RE]" or "[RF]"
                table.insert(spyLog, 1, tag .. " " .. obj.Name)
                if #spyLog > MAX_LOG then table.remove(spyLog) end
            end
        end
    end
end

task.spawn(function()
    scanInstance(ReplicatedStorage)
    pcall(function() scanInstance(game:GetService("Workspace")) end)
    ReplicatedStorage.DescendantAdded:Connect(function(obj)
        if (obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction")) and not detectedRemotes[obj.Name] then
            detectedRemotes[obj.Name] = obj
            local tag = obj:IsA("RemoteEvent") and "[RE]" or "[RF]"
            table.insert(spyLog, 1, tag .. " " .. obj.Name)
            if #spyLog > MAX_LOG then table.remove(spyLog) end
        end
    end)
end)

-- ─────────────────────────────────────────────
-- LÓGICA DE DISPARO
-- ─────────────────────────────────────────────
local function findBestRemote(keywords)
    local bestObj, bestScore = nil, 0
    for name, obj in pairs(detectedRemotes) do
        local low = name:lower()
        local score = 0
        for _, kw in ipairs(keywords) do
            if low:find(kw:lower(), 1, true) then score = score + 1 end
        end
        if score > bestScore then bestScore = score; bestObj = obj end
    end
    return bestObj
end

local function fireRemote(obj, amount)
    if not obj then return end
    if obj:IsA("RemoteEvent") then
        for i = 1, amount do obj:FireServer(amount) end
    elseif obj:IsA("RemoteFunction") then
        for i = 1, amount do pcall(function() obj:InvokeServer(amount) end) end
    end
end

local styleActive   = false
local abilityActive = false
local gemActive     = false

-- ─────────────────────────────────────────────
-- GUI
-- ─────────────────────────────────────────────
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name           = "VBLMenuV3"
ScreenGui.ResetOnSpawn   = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent         = PlayerGui

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name              = "MainFrame"
MainFrame.Size              = UDim2.new(0, 330, 0, 530)
MainFrame.Position          = UDim2.new(0.5, -165, 0.5, -265)
MainFrame.BackgroundColor3  = Color3.fromRGB(11, 10, 20)
MainFrame.BorderSizePixel   = 0
MainFrame.Visible           = false
MainFrame.Active            = true
MainFrame.Draggable         = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color     = Color3.fromRGB(110, 65, 230)
MainStroke.Thickness = 2

-- Título
local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Size             = UDim2.new(1, 0, 0, 46)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 16, 38)
TitleBar.BorderSizePixel  = 0
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 14)
local TFix = Instance.new("Frame", TitleBar)
TFix.Size             = UDim2.new(1, 0, 0.5, 0)
TFix.Position         = UDim2.new(0, 0, 0.5, 0)
TFix.BackgroundColor3 = Color3.fromRGB(20, 16, 38)
TFix.BorderSizePixel  = 0

local TitleLabel = Instance.new("TextLabel", TitleBar)
TitleLabel.Size               = UDim2.new(1, -50, 1, 0)
TitleLabel.Position           = UDim2.new(0, 14, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text               = "🏐  VBL Spin Menu  v3"
TitleLabel.TextColor3         = Color3.fromRGB(215, 175, 255)
TitleLabel.TextSize           = 16
TitleLabel.Font               = Enum.Font.GothamBold
TitleLabel.TextXAlignment     = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TitleBar)
CloseBtn.Size             = UDim2.new(0, 30, 0, 30)
CloseBtn.Position         = UDim2.new(1, -40, 0.5, -15)
CloseBtn.BackgroundColor3 = Color3.fromRGB(190, 45, 65)
CloseBtn.Text             = "✕"
CloseBtn.TextColor3       = Color3.fromRGB(255,255,255)
CloseBtn.TextSize         = 14
CloseBtn.Font             = Enum.Font.GothamBold
CloseBtn.BorderSizePixel  = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

local HintLabel = Instance.new("TextLabel", MainFrame)
HintLabel.Size               = UDim2.new(1, -24, 0, 18)
HintLabel.Position           = UDim2.new(0, 12, 0, 52)
HintLabel.BackgroundTransparency = 1
HintLabel.Text               = "Pressione  P  para abrir / fechar  •  Arraste para mover"
HintLabel.TextColor3         = Color3.fromRGB(90, 85, 130)
HintLabel.TextSize           = 10
HintLabel.Font               = Enum.Font.Gotham
HintLabel.TextXAlignment     = Enum.TextXAlignment.Left

local function makeDivider(posY)
    local d = Instance.new("Frame", MainFrame)
    d.Size             = UDim2.new(1, -24, 0, 1)
    d.Position         = UDim2.new(0, 12, 0, posY)
    d.BackgroundColor3 = Color3.fromRGB(45, 35, 85)
    d.BorderSizePixel  = 0
end

local function makeSectionLabel(txt, posY)
    local l = Instance.new("TextLabel", MainFrame)
    l.Size               = UDim2.new(1, -24, 0, 20)
    l.Position           = UDim2.new(0, 12, 0, posY)
    l.BackgroundTransparency = 1
    l.Text               = txt
    l.TextColor3         = Color3.fromRGB(170, 130, 255)
    l.TextSize           = 12
    l.Font               = Enum.Font.GothamBold
    l.TextXAlignment     = Enum.TextXAlignment.Left
end

makeDivider(76)
makeSectionLabel("⚙️  Automação", 82)

-- ─────────────────────────────────────────────
-- CARD DE TOGGLE
-- ─────────────────────────────────────────────
local function createCard(posY, icon, label, subtext, accentColor, onToggle)
    local Card = Instance.new("TextButton", MainFrame)
    Card.Size             = UDim2.new(1, -24, 0, 66)
    Card.Position         = UDim2.new(0, 12, 0, posY)
    Card.BackgroundColor3 = Color3.fromRGB(18, 16, 32)
    Card.BorderSizePixel  = 0
    Card.AutoButtonColor  = false
    Card.Text             = ""
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0, 10)

    local CS = Instance.new("UIStroke", Card)
    CS.Color     = Color3.fromRGB(55, 45, 95)
    CS.Thickness = 1

    -- ícone com fundo colorido
    local IconBG = Instance.new("Frame", Card)
    IconBG.Size             = UDim2.new(0, 42, 0, 42)
    IconBG.Position         = UDim2.new(0, 10, 0.5, -21)
    IconBG.BackgroundColor3 = Color3.fromRGB(
        math.floor(accentColor.R * 255 * 0.25),
        math.floor(accentColor.G * 255 * 0.25),
        math.floor(accentColor.B * 255 * 0.25)
    )
    IconBG.BorderSizePixel  = 0
    Instance.new("UICorner", IconBG).CornerRadius = UDim.new(0, 10)

    local Ico = Instance.new("TextLabel", IconBG)
    Ico.Size               = UDim2.new(1, 0, 1, 0)
    Ico.BackgroundTransparency = 1
    Ico.Text               = icon
    Ico.TextSize           = 24
    Ico.Font               = Enum.Font.Gotham

    local Lbl = Instance.new("TextLabel", Card)
    Lbl.Size               = UDim2.new(1, -130, 0, 22)
    Lbl.Position           = UDim2.new(0, 62, 0, 10)
    Lbl.BackgroundTransparency = 1
    Lbl.Text               = label
    Lbl.TextColor3         = Color3.fromRGB(235, 235, 255)
    Lbl.TextSize           = 14
    Lbl.Font               = Enum.Font.GothamBold
    Lbl.TextXAlignment     = Enum.TextXAlignment.Left

    local Sub = Instance.new("TextLabel", Card)
    Sub.Size               = UDim2.new(1, -130, 0, 16)
    Sub.Position           = UDim2.new(0, 62, 0, 34)
    Sub.BackgroundTransparency = 1
    Sub.Text               = subtext
    Sub.TextColor3         = Color3.fromRGB(130, 125, 170)
    Sub.TextSize           = 11
    Sub.Font               = Enum.Font.Gotham
    Sub.TextXAlignment     = Enum.TextXAlignment.Left

    -- pill ON/OFF
    local Pill = Instance.new("Frame", Card)
    Pill.Size             = UDim2.new(0, 56, 0, 26)
    Pill.Position         = UDim2.new(1, -66, 0.5, -13)
    Pill.BackgroundColor3 = Color3.fromRGB(35, 32, 55)
    Pill.BorderSizePixel  = 0
    Instance.new("UICorner", Pill).CornerRadius = UDim.new(1, 0)

    local PillTxt = Instance.new("TextLabel", Pill)
    PillTxt.Size               = UDim2.new(1, 0, 1, 0)
    PillTxt.BackgroundTransparency = 1
    PillTxt.Text               = "OFF"
    PillTxt.TextColor3         = Color3.fromRGB(130, 130, 160)
    PillTxt.TextSize           = 12
    PillTxt.Font               = Enum.Font.GothamBold

    local isOn = false
    local function updateCard()
        TweenService:Create(CS, TweenInfo.new(0.15), {
            Color = isOn and accentColor or Color3.fromRGB(55, 45, 95)
        }):Play()
        TweenService:Create(Pill, TweenInfo.new(0.15), {
            BackgroundColor3 = isOn and accentColor or Color3.fromRGB(35, 32, 55)
        }):Play()
        PillTxt.Text      = isOn and "ON" or "OFF"
        PillTxt.TextColor3 = isOn and Color3.fromRGB(255,255,255) or Color3.fromRGB(130,130,160)
    end

    Card.MouseButton1Click:Connect(function()
        isOn = not isOn
        updateCard()
        onToggle(isOn)
    end)
    Card.MouseEnter:Connect(function()
        TweenService:Create(Card, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(24,20,44)}):Play()
    end)
    Card.MouseLeave:Connect(function()
        TweenService:Create(Card, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(18,16,32)}):Play()
    end)
end

-- ── Card 1 – Lucky Style Spins ──
createCard(108,
    "🎨", "Lucky Style Spins", "+3 spins  •  a cada 3 segundos",
    Color3.fromRGB(140, 65, 240),
    function(on)
        styleActive = on
        if on then task.spawn(function()
            while styleActive do
                fireRemote(findBestRemote(STYLE_KEYWORDS), 3)
                task.wait(3)
            end
        end) end
    end
)

-- ── Card 2 – Ability Spins ──
createCard(184,
    "⚡", "Ability Spins", "+3 spins  •  a cada 4 segundos",
    Color3.fromRGB(40, 165, 235),
    function(on)
        abilityActive = on
        if on then task.spawn(function()
            while abilityActive do
                fireRemote(findBestRemote(ABILITY_KEYWORDS), 3)
                task.wait(4)
            end
        end) end
    end
)

-- ── Card 3 – Gemas 💎 ──
createCard(260,
    "💎", "Gemas Automáticas", "+50 gemas  •  a cada 5 segundos",
    Color3.fromRGB(50, 220, 170),
    function(on)
        gemActive = on
        if on then task.spawn(function()
            while gemActive do
                fireRemote(findBestRemote(GEM_KEYWORDS), 50)
                task.wait(5)
            end
        end) end
    end
)

-- ─────────────────────────────────────────────
-- REMOTE SPY
-- ─────────────────────────────────────────────
makeDivider(338)
makeSectionLabel("🔍  Remote Spy  (auto-detecção)", 344)

local SpyBox = Instance.new("ScrollingFrame", MainFrame)
SpyBox.Size                  = UDim2.new(1, -24, 0, 120)
SpyBox.Position              = UDim2.new(0, 12, 0, 368)
SpyBox.BackgroundColor3      = Color3.fromRGB(7, 6, 14)
SpyBox.BorderSizePixel       = 0
SpyBox.ScrollBarThickness    = 4
SpyBox.ScrollBarImageColor3  = Color3.fromRGB(90, 60, 200)
SpyBox.CanvasSize            = UDim2.new(0, 0, 0, 0)
SpyBox.AutomaticCanvasSize   = Enum.AutomaticSize.Y
Instance.new("UICorner", SpyBox).CornerRadius = UDim.new(0, 8)
Instance.new("UIPadding", SpyBox).PaddingLeft = UDim.new(0, 6)
Instance.new("UIListLayout", SpyBox).Padding  = UDim.new(0, 2)

local spyLines = {}
local function refreshSpyUI()
    for _, l in ipairs(spyLines) do l:Destroy() end
    spyLines = {}
    for i, entry in ipairs(spyLog) do
        local color
        if nameMatchesAny(entry, GEM_KEYWORDS) then
            color = Color3.fromRGB(60, 230, 180)
        elseif nameMatchesAny(entry, STYLE_KEYWORDS) then
            color = Color3.fromRGB(180, 110, 255)
        elseif nameMatchesAny(entry, ABILITY_KEYWORDS) then
            color = Color3.fromRGB(80, 200, 255)
        elseif entry:find("[RE]", 1, true) then
            color = Color3.fromRGB(160, 210, 150)
        else
            color = Color3.fromRGB(225, 165, 75)
        end
        local line = Instance.new("TextLabel", SpyBox)
        line.Size               = UDim2.new(1, -4, 0, 17)
        line.BackgroundTransparency = 1
        line.Text               = entry
        line.TextColor3         = color
        line.TextSize           = 11
        line.Font               = Enum.Font.Code
        line.TextXAlignment     = Enum.TextXAlignment.Left
        line.TextTruncate       = Enum.TextTruncate.AtEnd
        line.LayoutOrder        = i
        table.insert(spyLines, line)
    end
end

-- Botão escanear
local ScanBtn = Instance.new("TextButton", MainFrame)
ScanBtn.Size             = UDim2.new(1, -24, 0, 30)
ScanBtn.Position         = UDim2.new(0, 12, 0, 494)
ScanBtn.BackgroundColor3 = Color3.fromRGB(24, 18, 52)
ScanBtn.BorderSizePixel  = 0
ScanBtn.Text             = "🔄  Escanear Remotes Agora"
ScanBtn.TextColor3       = Color3.fromRGB(175, 145, 255)
ScanBtn.TextSize         = 12
ScanBtn.Font             = Enum.Font.GothamBold
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 8)

ScanBtn.MouseButton1Click:Connect(function()
    detectedRemotes = {}; spyLog = {}
    scanInstance(ReplicatedStorage)
    pcall(function() scanInstance(game:GetService("Workspace")) end)
    refreshSpyUI()
    ScanBtn.Text = "✅  " .. #spyLog .. " remotes encontrados!"
    task.delay(2.5, function() ScanBtn.Text = "🔄  Escanear Remotes Agora" end)
end)

task.spawn(function()
    while true do refreshSpyUI(); task.wait(2) end
end)

-- ─────────────────────────────────────────────
-- Tecla P – abrir / fechar
-- ─────────────────────────────────────────────
local menuOpen = false
local function toggleMenu()
    menuOpen = not menuOpen
    if menuOpen then
        MainFrame.Size     = UDim2.new(0, 0, 0, 0)
        MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
        MainFrame.Visible  = true
        TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Size     = UDim2.new(0, 330, 0, 530),
            Position = UDim2.new(0.5, -165, 0.5, -265),
        }):Play()
    else
        TweenService:Create(MainFrame, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Size     = UDim2.new(0, 0, 0, 0),
            Position = UDim2.new(0.5, 0, 0.5, 0),
        }):Play()
        task.delay(0.2, function() MainFrame.Visible = false end)
    end
end

CloseBtn.MouseButton1Click:Connect(toggleMenu)
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.P then toggleMenu() end
end)

print("✅ Silva244cc HUB | [P] abre/fecha | Style • Ability • Gemas | Remote Spy ativo")
