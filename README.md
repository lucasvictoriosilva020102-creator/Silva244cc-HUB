
-- =============================================
--  Silva244cc HUB | Tecla P = abrir/fechar
--  🎨 Lucky Style  → +3 a cada 3 segundos
--  ⚡ Ability Spin → +3 a cada 4 segundos
--  💎 Gemas        → +50 a cada 5 segundos
--  🔍 Remote Spy com seleção manual por botão
-- =============================================

local Players            = game:GetService("Players")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ─────────────────────────────────────────────
-- PALAVRAS-CHAVE E BLACKLIST
-- ─────────────────────────────────────────────
local STYLE_KEYWORDS   = {"style","luckystyle","styleSpin","giveStyle","addStyle"}
local ABILITY_KEYWORDS = {"ability","abilityS","giveAbility","addAbility"}
local GEM_KEYWORDS     = {"gem","gema","addGem","giveGem","gemAdd"}

-- Remotes com essas palavras NUNCA serão disparados automaticamente
local BLACKLIST = {
    "rebirth","buy","purchase","shop","sell","trade",
    "yen","money","pay","cost","price","spend","use",
    "equip","unequip","vote","kick","ban","report",
    "chat","message","save","load","data","config",
    "teleport","spawn","respawn","damage","kill","die",
    "rank","level","xp","exp","quest","daily","tutorial"
}

local function isBlacklisted(name)
    local low = name:lower()
    for _, bw in ipairs(BLACKLIST) do
        if low:find(bw, 1, true) then return true end
    end
    return false
end

local function nameMatchesAny(name, keywords)
    local low = name:lower()
    for _, kw in ipairs(keywords) do
        if low:find(kw:lower(), 1, true) then return true end
    end
    return false
end

-- ─────────────────────────────────────────────
-- REMOTE SPY
-- ─────────────────────────────────────────────
local detectedRemotes = {}  -- { name = RemoteEvent }
local spyLog          = {}  -- lista ordenada de nomes
local MAX_LOG         = 60

local function registerRemote(obj)
    if not detectedRemotes[obj.Name] then
        detectedRemotes[obj.Name] = obj
        table.insert(spyLog, 1, obj.Name)
        if #spyLog > MAX_LOG then table.remove(spyLog) end
    end
end

local function scanInstance(root)
    for _, obj in ipairs(root:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
            registerRemote(obj)
        end
    end
end

task.spawn(function()
    scanInstance(ReplicatedStorage)
    pcall(function() scanInstance(game:GetService("Workspace")) end)
    ReplicatedStorage.DescendantAdded:Connect(function(obj)
        if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
            registerRemote(obj)
        end
    end)
end)

-- ─────────────────────────────────────────────
-- REMOTES SELECIONADOS MANUALMENTE PELO USUÁRIO
-- ─────────────────────────────────────────────
local selectedRemotes = {
    style   = nil,  -- Remote escolhido para Style Spin
    ability = nil,  -- Remote escolhido para Ability Spin
    gem     = nil,  -- Remote escolhido para Gemas
}

-- ─────────────────────────────────────────────
-- DISPARO SEGURO
-- ─────────────────────────────────────────────
local function findBestRemote(keywords)
    local bestObj, bestScore = nil, 0
    for name, obj in pairs(detectedRemotes) do
        if not isBlacklisted(name) then
            local low   = name:lower()
            local score = 0
            for _, kw in ipairs(keywords) do
                if low:find(kw:lower(), 1, true) then score = score + 1 end
            end
            if score > bestScore then bestScore = score; bestObj = obj end
        end
    end
    return bestObj
end

local function fireRemote(obj, amount)
    if not obj then
        warn("[VBL Menu] Nenhum remote selecionado/encontrado.")
        return
    end
    if obj:IsA("RemoteEvent") then
        obj:FireServer(amount)
    elseif obj:IsA("RemoteFunction") then
        pcall(function() obj:InvokeServer(amount) end)
    end
end

local function getRemoteFor(slot, keywords)
    return selectedRemotes[slot] or findBestRemote(keywords)
end

-- ─────────────────────────────────────────────
-- ESTADO DOS LOOPS
-- ─────────────────────────────────────────────
local styleActive   = false
local abilityActive = false
local gemActive     = false

-- ─────────────────────────────────────────────
-- GUI PRINCIPAL
-- ─────────────────────────────────────────────
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name           = "VBLMenuV4"
ScreenGui.ResetOnSpawn   = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent         = PlayerGui

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name              = "MainFrame"
MainFrame.Size              = UDim2.new(0, 340, 0, 580)
MainFrame.Position          = UDim2.new(0.5, -170, 0.5, -290)
MainFrame.BackgroundColor3  = Color3.fromRGB(11, 10, 20)
MainFrame.BorderSizePixel   = 0
MainFrame.Visible           = false
MainFrame.Active            = true
MainFrame.Draggable         = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)
local MS = Instance.new("UIStroke", MainFrame)
MS.Color = Color3.fromRGB(110, 65, 230); MS.Thickness = 2

-- Título
local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Size             = UDim2.new(1, 0, 0, 46)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 16, 38)
TitleBar.BorderSizePixel  = 0
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 14)
local TFix = Instance.new("Frame", TitleBar)
TFix.Size = UDim2.new(1,0,0.5,0); TFix.Position = UDim2.new(0,0,0.5,0)
TFix.BackgroundColor3 = Color3.fromRGB(20,16,38); TFix.BorderSizePixel = 0

local TLabel = Instance.new("TextLabel", TitleBar)
TLabel.Size = UDim2.new(1,-50,1,0); TLabel.Position = UDim2.new(0,14,0,0)
TLabel.BackgroundTransparency = 1; TLabel.Text = "🏐  VBL Spin Menu  v4"
TLabel.TextColor3 = Color3.fromRGB(215,175,255); TLabel.TextSize = 16
TLabel.Font = Enum.Font.GothamBold; TLabel.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TitleBar)
CloseBtn.Size = UDim2.new(0,30,0,30); CloseBtn.Position = UDim2.new(1,-40,0.5,-15)
CloseBtn.BackgroundColor3 = Color3.fromRGB(190,45,65); CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255,255,255); CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

local Hint = Instance.new("TextLabel", MainFrame)
Hint.Size = UDim2.new(1,-24,0,16); Hint.Position = UDim2.new(0,12,0,52)
Hint.BackgroundTransparency = 1; Hint.Text = "Pressione P para abrir/fechar  •  Arraste para mover"
Hint.TextColor3 = Color3.fromRGB(80,75,120); Hint.TextSize = 10
Hint.Font = Enum.Font.Gotham; Hint.TextXAlignment = Enum.TextXAlignment.Left

local function makeDivider(posY)
    local d = Instance.new("Frame", MainFrame)
    d.Size = UDim2.new(1,-24,0,1); d.Position = UDim2.new(0,12,0,posY)
    d.BackgroundColor3 = Color3.fromRGB(40,32,80); d.BorderSizePixel = 0
end
local function makeSectionLabel(txt, posY, color)
    local l = Instance.new("TextLabel", MainFrame)
    l.Size = UDim2.new(1,-24,0,20); l.Position = UDim2.new(0,12,0,posY)
    l.BackgroundTransparency = 1; l.Text = txt
    l.TextColor3 = color or Color3.fromRGB(170,130,255)
    l.TextSize = 12; l.Font = Enum.Font.GothamBold
    l.TextXAlignment = Enum.TextXAlignment.Left
end

makeDivider(74)
makeSectionLabel("⚙️  Automação", 80)

-- ─────────────────────────────────────────────
-- CARD COM BADGE DO REMOTE SELECIONADO
-- ─────────────────────────────────────────────
local function createCard(posY, icon, label, subtext, accentColor, slot, keywords, onToggle)
    local Card = Instance.new("TextButton", MainFrame)
    Card.Size = UDim2.new(1,-24,0,72); Card.Position = UDim2.new(0,12,0,posY)
    Card.BackgroundColor3 = Color3.fromRGB(17,15,30); Card.BorderSizePixel = 0
    Card.AutoButtonColor = false; Card.Text = ""
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0,10)
    local CS = Instance.new("UIStroke", Card)
    CS.Color = Color3.fromRGB(50,40,90); CS.Thickness = 1

    -- ícone
    local IBG = Instance.new("Frame", Card)
    IBG.Size = UDim2.new(0,40,0,40); IBG.Position = UDim2.new(0,10,0.5,-20)
    IBG.BackgroundColor3 = Color3.fromRGB(
        math.floor(accentColor.R*255*0.22),
        math.floor(accentColor.G*255*0.22),
        math.floor(accentColor.B*255*0.22))
    IBG.BorderSizePixel = 0
    Instance.new("UICorner", IBG).CornerRadius = UDim.new(0,10)
    local Ico = Instance.new("TextLabel", IBG)
    Ico.Size = UDim2.new(1,0,1,0); Ico.BackgroundTransparency = 1
    Ico.Text = icon; Ico.TextSize = 22; Ico.Font = Enum.Font.Gotham

    local Lbl = Instance.new("TextLabel", Card)
    Lbl.Size = UDim2.new(1,-130,0,20); Lbl.Position = UDim2.new(0,60,0,8)
    Lbl.BackgroundTransparency = 1; Lbl.Text = label
    Lbl.TextColor3 = Color3.fromRGB(235,235,255); Lbl.TextSize = 13
    Lbl.Font = Enum.Font.GothamBold; Lbl.TextXAlignment = Enum.TextXAlignment.Left

    local Sub = Instance.new("TextLabel", Card)
    Sub.Size = UDim2.new(1,-130,0,14); Sub.Position = UDim2.new(0,60,0,30)
    Sub.BackgroundTransparency = 1; Sub.Text = subtext
    Sub.TextColor3 = Color3.fromRGB(120,115,165); Sub.TextSize = 10
    Sub.Font = Enum.Font.Gotham; Sub.TextXAlignment = Enum.TextXAlignment.Left

    -- badge do remote em uso
    local Badge = Instance.new("TextLabel", Card)
    Badge.Size = UDim2.new(1,-70,0,14); Badge.Position = UDim2.new(0,60,0,48)
    Badge.BackgroundTransparency = 1
    Badge.Text = "🔗 remote: nenhum selecionado"
    Badge.TextColor3 = Color3.fromRGB(90,90,130); Badge.TextSize = 9
    Badge.Font = Enum.Font.Code; Badge.TextXAlignment = Enum.TextXAlignment.Left
    Badge.TextTruncate = Enum.TextTruncate.AtEnd

    local function refreshBadge()
        local r = selectedRemotes[slot]
        if r then
            Badge.Text = "🔗 " .. r.Name
            Badge.TextColor3 = accentColor
        else
            local auto = findBestRemote(keywords)
            if auto then
                Badge.Text = "🤖 auto: " .. auto.Name
                Badge.TextColor3 = Color3.fromRGB(140,140,180)
            else
                Badge.Text = "⚠️  nenhum remote encontrado"
                Badge.TextColor3 = Color3.fromRGB(220,80,80)
            end
        end
    end

    -- pill ON/OFF
    local Pill = Instance.new("Frame", Card)
    Pill.Size = UDim2.new(0,52,0,24); Pill.Position = UDim2.new(1,-62,0.5,-12)
    Pill.BackgroundColor3 = Color3.fromRGB(35,30,55); Pill.BorderSizePixel = 0
    Instance.new("UICorner", Pill).CornerRadius = UDim.new(1,0)
    local PT = Instance.new("TextLabel", Pill)
    PT.Size = UDim2.new(1,0,1,0); PT.BackgroundTransparency = 1
    PT.Text = "OFF"; PT.TextColor3 = Color3.fromRGB(120,120,155)
    PT.TextSize = 12; PT.Font = Enum.Font.GothamBold

    local isOn = false
    local function updateCard()
        TweenService:Create(CS, TweenInfo.new(0.15), {
            Color = isOn and accentColor or Color3.fromRGB(50,40,90)
        }):Play()
        TweenService:Create(Pill, TweenInfo.new(0.15), {
            BackgroundColor3 = isOn and accentColor or Color3.fromRGB(35,30,55)
        }):Play()
        PT.Text = isOn and "ON" or "OFF"
        PT.TextColor3 = isOn and Color3.fromRGB(255,255,255) or Color3.fromRGB(120,120,155)
    end

    Card.MouseButton1Click:Connect(function()
        isOn = not isOn
        updateCard()
        refreshBadge()
        onToggle(isOn)
    end)
    Card.MouseEnter:Connect(function()
        TweenService:Create(Card,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(22,19,40)}):Play()
    end)
    Card.MouseLeave:Connect(function()
        TweenService:Create(Card,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(17,15,30)}):Play()
    end)

    -- atualiza badge periodicamente
    task.spawn(function()
        while true do refreshBadge(); task.wait(3) end
    end)

    return refreshBadge
end

-- Cards
createCard(104, "🎨","Lucky Style Spins","+3 spins • a cada 3 seg",
    Color3.fromRGB(140,65,240), "style", STYLE_KEYWORDS,
    function(on)
        styleActive = on
        if on then task.spawn(function()
            while styleActive do
                fireRemote(getRemoteFor("style", STYLE_KEYWORDS), 3)
                task.wait(3)
            end
        end) end
    end)

createCard(184, "⚡","Ability Spins","+3 spins • a cada 4 seg",
    Color3.fromRGB(40,165,235), "ability", ABILITY_KEYWORDS,
    function(on)
        abilityActive = on
        if on then task.spawn(function()
            while abilityActive do
                fireRemote(getRemoteFor("ability", ABILITY_KEYWORDS), 3)
                task.wait(4)
            end
        end) end
    end)

createCard(264, "💎","Gemas Automáticas","+50 gemas • a cada 5 seg",
    Color3.fromRGB(50,220,170), "gem", GEM_KEYWORDS,
    function(on)
        gemActive = on
        if on then task.spawn(function()
            while gemActive do
                fireRemote(getRemoteFor("gem", GEM_KEYWORDS), 50)
                task.wait(5)
            end
        end) end
    end)

-- ─────────────────────────────────────────────
-- REMOTE SPY COM SELEÇÃO MANUAL
-- ─────────────────────────────────────────────
makeDivider(346)
makeSectionLabel("🔍  Remote Spy  —  clique para atribuir ao slot", 352)

-- Legenda de slots
local SlotHint = Instance.new("TextLabel", MainFrame)
SlotHint.Size = UDim2.new(1,-24,0,14); SlotHint.Position = UDim2.new(0,12,0,370)
SlotHint.BackgroundTransparency = 1
SlotHint.Text = "🟣=Style   🔵=Ability   🟢=Gem   ⬜=Outro   🚫=Bloqueado"
SlotHint.TextColor3 = Color3.fromRGB(100,95,145); SlotHint.TextSize = 9
SlotHint.Font = Enum.Font.Code; SlotHint.TextXAlignment = Enum.TextXAlignment.Left

local SpyBox = Instance.new("ScrollingFrame", MainFrame)
SpyBox.Size = UDim2.new(1,-24,0,130); SpyBox.Position = UDim2.new(0,12,0,388)
SpyBox.BackgroundColor3 = Color3.fromRGB(7,6,14); SpyBox.BorderSizePixel = 0
SpyBox.ScrollBarThickness = 4
SpyBox.ScrollBarImageColor3 = Color3.fromRGB(90,60,200)
SpyBox.CanvasSize = UDim2.new(0,0,0,0)
SpyBox.AutomaticCanvasSize = Enum.AutomaticSize.Y
Instance.new("UICorner", SpyBox).CornerRadius = UDim.new(0,8)
local SpyPad = Instance.new("UIPadding", SpyBox)
SpyPad.PaddingLeft = UDim.new(0,6); SpyPad.PaddingTop = UDim.new(0,4)
Instance.new("UIListLayout", SpyBox).Padding = UDim.new(0,3)

-- Popup de seleção de slot
local Popup = Instance.new("Frame", ScreenGui)
Popup.Size = UDim2.new(0,220,0,120); Popup.Position = UDim2.new(0.5,-110,0.5,-60)
Popup.BackgroundColor3 = Color3.fromRGB(18,14,34); Popup.BorderSizePixel = 0
Popup.Visible = false; Popup.ZIndex = 10
Instance.new("UICorner", Popup).CornerRadius = UDim.new(0,12)
local PopStroke = Instance.new("UIStroke", Popup)
PopStroke.Color = Color3.fromRGB(110,65,230); PopStroke.Thickness = 2

local PopTitle = Instance.new("TextLabel", Popup)
PopTitle.Size = UDim2.new(1,-10,0,30); PopTitle.Position = UDim2.new(0,8,0,6)
PopTitle.BackgroundTransparency = 1; PopTitle.Text = "Atribuir a qual slot?"
PopTitle.TextColor3 = Color3.fromRGB(210,175,255); PopTitle.TextSize = 13
PopTitle.Font = Enum.Font.GothamBold; PopTitle.TextXAlignment = Enum.TextXAlignment.Left

local selectedRemoteName = ""
local function makePopBtn(txt, yPos, color, slot)
    local B = Instance.new("TextButton", Popup)
    B.Size = UDim2.new(1,-16,0,24); B.Position = UDim2.new(0,8,0,yPos)
    B.BackgroundColor3 = color; B.Text = txt
    B.TextColor3 = Color3.fromRGB(255,255,255); B.TextSize = 12
    B.Font = Enum.Font.GothamBold; B.BorderSizePixel = 0
    Instance.new("UICorner", B).CornerRadius = UDim.new(0,7)
    B.ZIndex = 11
    B.MouseButton1Click:Connect(function()
        if detectedRemotes[selectedRemoteName] then
            selectedRemotes[slot] = detectedRemotes[selectedRemoteName]
        end
        Popup.Visible = false
    end)
end
makePopBtn("🎨 Style Spins",  36, Color3.fromRGB(100,45,200), "style")
makePopBtn("⚡ Ability Spins", 64, Color3.fromRGB(30,130,210), "ability")
makePopBtn("💎 Gemas",         92, Color3.fromRGB(35,175,130), "gem")

local PopClose = Instance.new("TextButton", Popup)
PopClose.Size = UDim2.new(0,22,0,22); PopClose.Position = UDim2.new(1,-28,0,6)
PopClose.BackgroundColor3 = Color3.fromRGB(180,40,60); PopClose.Text = "✕"
PopClose.TextColor3 = Color3.fromRGB(255,255,255); PopClose.TextSize = 12
PopClose.Font = Enum.Font.GothamBold; PopClose.BorderSizePixel = 0
PopClose.ZIndex = 11
Instance.new("UICorner", PopClose).CornerRadius = UDim.new(0,6)
PopClose.MouseButton1Click:Connect(function() Popup.Visible = false end)

-- Renderiza linhas do Spy
local spyLines = {}
local function refreshSpyUI()
    for _, l in ipairs(spyLines) do l:Destroy() end
    spyLines = {}
    for i, name in ipairs(spyLog) do
        local blocked = isBlacklisted(name)
        local isStyle   = nameMatchesAny(name, STYLE_KEYWORDS)
        local isAbility = nameMatchesAny(name, ABILITY_KEYWORDS)
        local isGem     = nameMatchesAny(name, GEM_KEYWORDS)

        local dot = blocked and "🚫" or (isStyle and "🟣" or (isAbility and "🔵" or (isGem and "🟢" or "⬜")))
        local color = blocked and Color3.fromRGB(160,60,60)
            or (isGem and Color3.fromRGB(60,230,180))
            or (isStyle and Color3.fromRGB(180,110,255))
            or (isAbility and Color3.fromRGB(80,200,255))
            or Color3.fromRGB(160,210,150)

        local Row = Instance.new("TextButton", SpyBox)
        Row.Size = UDim2.new(1,-4,0,18); Row.BackgroundTransparency = 1
        Row.Text = dot .. " " .. name; Row.TextColor3 = color
        Row.TextSize = 11; Row.Font = Enum.Font.Code
        Row.TextXAlignment = Enum.TextXAlignment.Left
        Row.TextTruncate = Enum.TextTruncate.AtEnd
        Row.LayoutOrder = i; Row.ZIndex = 5
        Row.MouseButton1Click:Connect(function()
            if not blocked then
                selectedRemoteName = name
                Popup.Visible = true
            end
        end)
        Row.MouseEnter:Connect(function()
            if not blocked then
                TweenService:Create(Row,TweenInfo.new(0.08),{
                    BackgroundTransparency = 0.7,
                    BackgroundColor3 = Color3.fromRGB(40,30,70)
                }):Play()
            end
        end)
        Row.MouseLeave:Connect(function()
            TweenService:Create(Row,TweenInfo.new(0.08),{
                BackgroundTransparency = 1
            }):Play()
        end)
        table.insert(spyLines, Row)
    end
end

-- Botão escanear
local ScanBtn = Instance.new("TextButton", MainFrame)
ScanBtn.Size = UDim2.new(1,-24,0,28); ScanBtn.Position = UDim2.new(0,12,0,526)
ScanBtn.BackgroundColor3 = Color3.fromRGB(22,16,48); ScanBtn.BorderSizePixel = 0
ScanBtn.Text = "🔄  Escanear Remotes Agora"
ScanBtn.TextColor3 = Color3.fromRGB(170,140,255); ScanBtn.TextSize = 12
ScanBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0,8)
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
-- Tecla P
-- ─────────────────────────────────────────────
local menuOpen = false
local function toggleMenu()
    menuOpen = not menuOpen
    if menuOpen then
        MainFrame.Size = UDim2.new(0,0,0,0)
        MainFrame.Position = UDim2.new(0.5,0,0.5,0)
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.25,Enum.EasingStyle.Back,Enum.EasingDirection.Out), {
            Size     = UDim2.new(0,340,0,580),
            Position = UDim2.new(0.5,-170,0.5,-290),
        }):Play()
    else
        TweenService:Create(MainFrame, TweenInfo.new(0.18,Enum.EasingStyle.Quad,Enum.EasingDirection.In), {
            Size     = UDim2.new(0,0,0,0),
            Position = UDim2.new(0.5,0,0.5,0),
        }):Play()
        Popup.Visible = false
        task.delay(0.2, function() MainFrame.Visible = false end)
    end
end

CloseBtn.MouseButton1Click:Connect(toggleMenu)
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.P then toggleMenu() end
end)

print("✅ Silva244cc HUB | [P] abre • Spy ativo • Clique no remote para atribuir ao slot")
