
-- =============================================
--  VBL Spin Menu v5 | Tecla P = abrir/fechar
--  Hook real em FireServer — captura e repete
--  os remotes EXATOS que o jogo já usa
-- =============================================

local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ─────────────────────────────────────────────
-- HOOK ENGINE
-- Intercepta TODOS os FireServer do jogo
-- ─────────────────────────────────────────────
local capturedRemotes = {}   -- { [remoteName] = { remote=obj, args={...} } }
local hookLog         = {}   -- lista de nomes para exibir
local MAX_LOG         = 60

local styleRemote   = nil
local abilityRemote = nil
local gemRemote     = nil

local STYLE_KW   = {"style","lucky","spin","spins"}
local ABILITY_KW = {"ability","abilit","power"}
local GEM_KW     = {"gem","gema","crystal","jewel","shard"}

local function matchesAny(name, kws)
    local low = name:lower()
    for _, k in ipairs(kws) do
        if low:find(k, 1, true) then return true end
    end
    return false
end

-- hookmetamethod disponível em Synapse X, KRNL, Fluxus, Celery etc.
local hooked = false
local function tryHook()
    if not hookmetamethod then return false end
    local oldNC
    oldNC = hookmetamethod(game, "__namecall", function(self, ...)
        local method = getnamecallmethod()
        if method == "FireServer" and self:IsA("RemoteEvent") then
            local name = self.Name
            local args  = { ... }
            if not capturedRemotes[name] then
                table.insert(hookLog, 1, name)
                if #hookLog > MAX_LOG then table.remove(hookLog) end
            end
            capturedRemotes[name] = { remote = self, args = args }
        end
        return oldNC(self, ...)
    end)
    return true
end
hooked = tryHook()

-- Fallback passivo: escaneia ReplicatedStorage e escuta OnClientEvent
if not hooked then
    local RS = game:GetService("ReplicatedStorage")
    local function reg(obj)
        if obj:IsA("RemoteEvent") and not capturedRemotes[obj.Name] then
            capturedRemotes[obj.Name] = { remote = obj, args = {} }
            table.insert(hookLog, 1, obj.Name)
            if #hookLog > MAX_LOG then table.remove(hookLog) end
            -- captura args que o SERVER envia de volta ao client
            obj.OnClientEvent:Connect(function(...)
                capturedRemotes[obj.Name].args = { ... }
            end)
        end
    end
    for _, o in ipairs(RS:GetDescendants()) do reg(o) end
    RS.DescendantAdded:Connect(reg)
end

-- ─────────────────────────────────────────────
-- DISPARO — repete exatamente como o jogo fez
-- ─────────────────────────────────────────────
local styleActive   = false
local abilityActive = false
local gemActive     = false

local function fireCapture(entry, times)
    if not entry or not entry.remote then return end
    local r, args = entry.remote, entry.args or {}
    for i = 1, (times or 1) do
        pcall(function() r:FireServer(table.unpack(args)) end)
    end
end

local function bestFor(kws)
    for name, entry in pairs(capturedRemotes) do
        if matchesAny(name, kws) then return entry end
    end
end

local function getEntry(slot, kws)
    if slot == "style"   and styleRemote   then return capturedRemotes[styleRemote]   end
    if slot == "ability" and abilityRemote then return capturedRemotes[abilityRemote] end
    if slot == "gem"     and gemRemote     then return capturedRemotes[gemRemote]     end
    return bestFor(kws)
end

-- ─────────────────────────────────────────────
-- GUI
-- ─────────────────────────────────────────────
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VBLv5"; ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

local W, H = 340, 590
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name = "Main"; MainFrame.Size = UDim2.new(0,W,0,H)
MainFrame.Position = UDim2.new(0.5,-W/2,0.5,-H/2)
MainFrame.BackgroundColor3 = Color3.fromRGB(10,9,18)
MainFrame.BorderSizePixel = 0; MainFrame.Visible = false
MainFrame.Active = true; MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,14)
local MS = Instance.new("UIStroke", MainFrame)
MS.Color = Color3.fromRGB(110,60,230); MS.Thickness = 2

-- Título
local TBar = Instance.new("Frame", MainFrame)
TBar.Size = UDim2.new(1,0,0,46); TBar.BorderSizePixel = 0
TBar.BackgroundColor3 = Color3.fromRGB(18,14,36)
Instance.new("UICorner", TBar).CornerRadius = UDim.new(0,14)
local TFix = Instance.new("Frame", TBar)
TFix.Size = UDim2.new(1,0,0.5,0); TFix.Position = UDim2.new(0,0,0.5,0)
TFix.BackgroundColor3 = Color3.fromRGB(18,14,36); TFix.BorderSizePixel = 0

local TLbl = Instance.new("TextLabel", TBar)
TLbl.Size = UDim2.new(1,-50,1,0); TLbl.Position = UDim2.new(0,14,0,0)
TLbl.BackgroundTransparency = 1; TLbl.Text = "🏐  VBL Spin Menu  v5"
TLbl.TextColor3 = Color3.fromRGB(215,175,255); TLbl.TextSize = 16
TLbl.Font = Enum.Font.GothamBold; TLbl.TextXAlignment = Enum.TextXAlignment.Left

local HookBadge = Instance.new("TextLabel", TBar)
HookBadge.Size = UDim2.new(0,88,0,20); HookBadge.Position = UDim2.new(1,-130,0.5,-10)
HookBadge.BackgroundColor3 = hooked and Color3.fromRGB(28,130,75) or Color3.fromRGB(130,75,28)
HookBadge.BorderSizePixel = 0
HookBadge.Text = hooked and "✔ HOOK ON" or "⚠ PASSIVO"
HookBadge.TextColor3 = Color3.fromRGB(255,255,255); HookBadge.TextSize = 10
HookBadge.Font = Enum.Font.GothamBold
Instance.new("UICorner", HookBadge).CornerRadius = UDim.new(0,6)

local CloseBtn = Instance.new("TextButton", TBar)
CloseBtn.Size = UDim2.new(0,30,0,30); CloseBtn.Position = UDim2.new(1,-40,0.5,-15)
CloseBtn.BackgroundColor3 = Color3.fromRGB(190,45,65); CloseBtn.BorderSizePixel = 0
CloseBtn.Text = "✕"; CloseBtn.TextColor3 = Color3.fromRGB(255,255,255)
CloseBtn.TextSize = 14; CloseBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0,8)

-- Hint
local Hint = Instance.new("TextLabel", MainFrame)
Hint.Size = UDim2.new(1,-24,0,30); Hint.Position = UDim2.new(0,12,0,52)
Hint.BackgroundTransparency = 1; Hint.TextSize = 10; Hint.Font = Enum.Font.Gotham
Hint.TextXAlignment = Enum.TextXAlignment.Left; Hint.TextWrapped = true
Hint.TextColor3 = Color3.fromRGB(80,75,120)
Hint.Text = hooked
    and "✅ Hook ativo! Jogue normalmente — cada ação captura o remote automaticamente."
    or  "⚠️ Modo passivo. Execute ações no jogo (ganhe spins/gemas) para os remotes aparecerem."

local function makeDivider(posY)
    local d = Instance.new("Frame", MainFrame)
    d.Size = UDim2.new(1,-24,0,1); d.Position = UDim2.new(0,12,0,posY)
    d.BackgroundColor3 = Color3.fromRGB(40,30,80); d.BorderSizePixel = 0
end
local function makeSec(txt, posY, col)
    local l = Instance.new("TextLabel", MainFrame)
    l.Size = UDim2.new(1,-24,0,20); l.Position = UDim2.new(0,12,0,posY)
    l.BackgroundTransparency = 1; l.Text = txt
    l.TextColor3 = col or Color3.fromRGB(170,130,255)
    l.TextSize = 12; l.Font = Enum.Font.GothamBold
    l.TextXAlignment = Enum.TextXAlignment.Left
end

makeDivider(86); makeSec("⚙️  Automação", 92)

-- ─────────────────────────────────────────────
-- CARD
-- ─────────────────────────────────────────────
local function createCard(posY, icon, mainLbl, subLbl, accent, slot, kws, interval, times, onToggle)
    local Card = Instance.new("TextButton", MainFrame)
    Card.Size = UDim2.new(1,-24,0,72); Card.Position = UDim2.new(0,12,0,posY)
    Card.BackgroundColor3 = Color3.fromRGB(16,14,28); Card.BorderSizePixel = 0
    Card.AutoButtonColor = false; Card.Text = ""
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0,10)
    local CS = Instance.new("UIStroke", Card)
    CS.Color = Color3.fromRGB(45,36,85); CS.Thickness = 1

    local IBG = Instance.new("Frame", Card)
    IBG.Size = UDim2.new(0,40,0,40); IBG.Position = UDim2.new(0,10,0.5,-20); IBG.BorderSizePixel = 0
    IBG.BackgroundColor3 = Color3.fromRGB(
        math.floor(accent.R*255*0.2),math.floor(accent.G*255*0.2),math.floor(accent.B*255*0.2))
    Instance.new("UICorner", IBG).CornerRadius = UDim.new(0,10)
    local Ico = Instance.new("TextLabel", IBG)
    Ico.Size = UDim2.new(1,0,1,0); Ico.BackgroundTransparency = 1
    Ico.Text = icon; Ico.TextSize = 22; Ico.Font = Enum.Font.Gotham

    local L1 = Instance.new("TextLabel", Card)
    L1.Size = UDim2.new(1,-130,0,20); L1.Position = UDim2.new(0,60,0,8)
    L1.BackgroundTransparency = 1; L1.Text = mainLbl
    L1.TextColor3 = Color3.fromRGB(235,235,255); L1.TextSize = 13
    L1.Font = Enum.Font.GothamBold; L1.TextXAlignment = Enum.TextXAlignment.Left

    local L2 = Instance.new("TextLabel", Card)
    L2.Size = UDim2.new(1,-130,0,14); L2.Position = UDim2.new(0,60,0,30)
    L2.BackgroundTransparency = 1; L2.Text = subLbl
    L2.TextColor3 = Color3.fromRGB(115,110,160); L2.TextSize = 10
    L2.Font = Enum.Font.Gotham; L2.TextXAlignment = Enum.TextXAlignment.Left

    local Badge = Instance.new("TextLabel", Card)
    Badge.Size = UDim2.new(1,-70,0,13); Badge.Position = UDim2.new(0,60,0,51)
    Badge.BackgroundTransparency = 1; Badge.TextSize = 9; Badge.Font = Enum.Font.Code
    Badge.TextXAlignment = Enum.TextXAlignment.Left
    Badge.TextTruncate = Enum.TextTruncate.AtEnd
    Badge.TextColor3 = Color3.fromRGB(80,80,120)
    Badge.Text = "⏳ aguardando captura de remote..."

    local Pill = Instance.new("Frame", Card)
    Pill.Size = UDim2.new(0,52,0,24); Pill.Position = UDim2.new(1,-62,0.5,-12)
    Pill.BackgroundColor3 = Color3.fromRGB(32,28,52); Pill.BorderSizePixel = 0
    Instance.new("UICorner", Pill).CornerRadius = UDim.new(1,0)
    local PT = Instance.new("TextLabel", Pill)
    PT.Size = UDim2.new(1,0,1,0); PT.BackgroundTransparency = 1
    PT.Text = "OFF"; PT.TextColor3 = Color3.fromRGB(115,115,150)
    PT.TextSize = 12; PT.Font = Enum.Font.GothamBold

    -- atualiza badge com nome do remote
    task.spawn(function()
        while true do
            local entry = getEntry(slot, kws)
            if entry and entry.remote then
                Badge.Text = "✅ " .. entry.remote.Name
                Badge.TextColor3 = accent
            else
                Badge.Text = "⏳ aguardando captura de remote..."
                Badge.TextColor3 = Color3.fromRGB(80,80,120)
            end
            task.wait(1)
        end
    end)

    local isOn = false
    local function updateCard()
        TweenService:Create(CS,TweenInfo.new(0.15),{Color=isOn and accent or Color3.fromRGB(45,36,85)}):Play()
        TweenService:Create(Pill,TweenInfo.new(0.15),{BackgroundColor3=isOn and accent or Color3.fromRGB(32,28,52)}):Play()
        PT.Text = isOn and "ON" or "OFF"
        PT.TextColor3 = isOn and Color3.fromRGB(255,255,255) or Color3.fromRGB(115,115,150)
    end
    Card.MouseButton1Click:Connect(function()
        isOn = not isOn; updateCard(); onToggle(isOn)
    end)
    Card.MouseEnter:Connect(function()
        TweenService:Create(Card,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(21,18,36)}):Play()
    end)
    Card.MouseLeave:Connect(function()
        TweenService:Create(Card,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(16,14,28)}):Play()
    end)
end

createCard(116,"🎨","Lucky Style Spins","+3 spins • a cada 3 seg",
    Color3.fromRGB(140,65,240),"style",STYLE_KW,3,3,
    function(on)
        styleActive = on
        if on then task.spawn(function()
            while styleActive do
                fireCapture(getEntry("style",STYLE_KW),3)
                task.wait(3)
            end
        end) end
    end)

createCard(196,"⚡","Ability Spins","+3 spins • a cada 4 seg",
    Color3.fromRGB(40,165,235),"ability",ABILITY_KW,4,3,
    function(on)
        abilityActive = on
        if on then task.spawn(function()
            while abilityActive do
                fireCapture(getEntry("ability",ABILITY_KW),3)
                task.wait(4)
            end
        end) end
    end)

createCard(276,"💎","Gemas Automáticas","+x gemas • a cada 5 seg",
    Color3.fromRGB(50,220,170),"gem",GEM_KW,5,1,
    function(on)
        gemActive = on
        if on then task.spawn(function()
            while gemActive do
                fireCapture(getEntry("gem",GEM_KW),1)
                task.wait(5)
            end
        end) end
    end)

-- ─────────────────────────────────────────────
-- REMOTE SPY + SELEÇÃO MANUAL
-- ─────────────────────────────────────────────
makeDivider(358)
makeSec("🔍  Remotes Capturados — clique num para fixar ao slot", 364)

local SpyBox = Instance.new("ScrollingFrame", MainFrame)
SpyBox.Size = UDim2.new(1,-24,0,148); SpyBox.Position = UDim2.new(0,12,0,388)
SpyBox.BackgroundColor3 = Color3.fromRGB(6,5,12); SpyBox.BorderSizePixel = 0
SpyBox.ScrollBarThickness = 4
SpyBox.ScrollBarImageColor3 = Color3.fromRGB(90,55,200)
SpyBox.CanvasSize = UDim2.new(0,0,0,0)
SpyBox.AutomaticCanvasSize = Enum.AutomaticSize.Y
Instance.new("UICorner", SpyBox).CornerRadius = UDim.new(0,8)
local SP = Instance.new("UIPadding", SpyBox)
SP.PaddingLeft = UDim.new(0,6); SP.PaddingTop = UDim.new(0,4)
Instance.new("UIListLayout", SpyBox).Padding = UDim.new(0,3)

-- Popup de slot
local Popup = Instance.new("Frame", ScreenGui)
Popup.Size = UDim2.new(0,230,0,136)
Popup.Position = UDim2.new(0.5,-115,0.5,-68)
Popup.BackgroundColor3 = Color3.fromRGB(15,11,28)
Popup.BorderSizePixel = 0; Popup.Visible = false; Popup.ZIndex = 20
Instance.new("UICorner", Popup).CornerRadius = UDim.new(0,12)
local PST = Instance.new("UIStroke", Popup)
PST.Color = Color3.fromRGB(110,60,230); PST.Thickness = 2

local PopLbl = Instance.new("TextLabel", Popup)
PopLbl.Size = UDim2.new(1,-10,0,28); PopLbl.Position = UDim2.new(0,8,0,6)
PopLbl.BackgroundTransparency = 1; PopLbl.TextColor3 = Color3.fromRGB(210,175,255)
PopLbl.TextSize = 12; PopLbl.Font = Enum.Font.GothamBold
PopLbl.TextXAlignment = Enum.TextXAlignment.Left; PopLbl.ZIndex = 21
PopLbl.TextTruncate = Enum.TextTruncate.AtEnd

local chosenName = ""
local function popBtn(lbl, posY, col, action)
    local B = Instance.new("TextButton", Popup)
    B.Size = UDim2.new(1,-16,0,26); B.Position = UDim2.new(0,8,0,posY)
    B.BackgroundColor3 = col; B.Text = lbl
    B.TextColor3 = Color3.fromRGB(255,255,255); B.TextSize = 12
    B.Font = Enum.Font.GothamBold; B.BorderSizePixel = 0; B.ZIndex = 21
    Instance.new("UICorner", B).CornerRadius = UDim.new(0,7)
    B.MouseButton1Click:Connect(function() action(); Popup.Visible = false end)
end
popBtn("🎨 Lucky Style Spins", 38, Color3.fromRGB(100,42,200), function() styleRemote   = chosenName end)
popBtn("⚡ Ability Spins",     68, Color3.fromRGB(28,120,210), function() abilityRemote  = chosenName end)
popBtn("💎 Gemas",             98, Color3.fromRGB(32,170,120), function() gemRemote      = chosenName end)

local PC = Instance.new("TextButton", Popup)
PC.Size = UDim2.new(0,24,0,24); PC.Position = UDim2.new(1,-30,0,6)
PC.BackgroundColor3 = Color3.fromRGB(180,38,58); PC.Text = "✕"
PC.TextColor3 = Color3.fromRGB(255,255,255); PC.TextSize = 12
PC.Font = Enum.Font.GothamBold; PC.BorderSizePixel = 0; PC.ZIndex = 21
Instance.new("UICorner", PC).CornerRadius = UDim.new(0,6)
PC.MouseButton1Click:Connect(function() Popup.Visible = false end)

-- Renderiza spy
local spyLines = {}
local function refreshSpy()
    for _, l in ipairs(spyLines) do l:Destroy() end; spyLines = {}
    for i, name in ipairs(hookLog) do
        local isSel = (styleRemote==name and "🎨") or (abilityRemote==name and "⚡") or (gemRemote==name and "💎")
        local isS = matchesAny(name,STYLE_KW)
        local isA = matchesAny(name,ABILITY_KW)
        local isG = matchesAny(name,GEM_KW)
        local dot = isSel or (isG and "🟢") or (isS and "🟣") or (isA and "🔵") or "⬜"
        local col = isSel and Color3.fromRGB(255,255,255)
            or (isG and Color3.fromRGB(55,215,165))
            or (isS and Color3.fromRGB(160,100,240))
            or (isA and Color3.fromRGB(70,180,240))
            or Color3.fromRGB(150,200,140)

        local Row = Instance.new("TextButton", SpyBox)
        Row.Size = UDim2.new(1,-4,0,18); Row.BackgroundTransparency = 1
        Row.Text = dot.." "..name; Row.TextColor3 = col
        Row.TextSize = 11; Row.Font = Enum.Font.Code
        Row.TextXAlignment = Enum.TextXAlignment.Left
        Row.TextTruncate = Enum.TextTruncate.AtEnd
        Row.LayoutOrder = i; Row.ZIndex = 5
        Row.MouseButton1Click:Connect(function()
            chosenName = name
            PopLbl.Text = "Fixar \""..name.."\" em:"
            Popup.Visible = true
        end)
        Row.MouseEnter:Connect(function()
            TweenService:Create(Row,TweenInfo.new(0.08),{BackgroundTransparency=0.65,BackgroundColor3=Color3.fromRGB(35,26,65)}):Play()
        end)
        Row.MouseLeave:Connect(function()
            TweenService:Create(Row,TweenInfo.new(0.08),{BackgroundTransparency=1}):Play()
        end)
        table.insert(spyLines, Row)
    end
end
task.spawn(function() while true do refreshSpy(); task.wait(1.5) end end)

-- Botões inferiores
local function makeBtn(lbl, posX, posY, w, col, fn)
    local B = Instance.new("TextButton", MainFrame)
    B.Size = UDim2.new(0,w,0,28); B.Position = UDim2.new(0,posX,0,posY)
    B.BackgroundColor3 = col; B.BorderSizePixel = 0
    B.Text = lbl; B.TextColor3 = Color3.fromRGB(200,180,255)
    B.TextSize = 11; B.Font = Enum.Font.GothamBold
    Instance.new("UICorner", B).CornerRadius = UDim.new(0,8)
    B.MouseButton1Click:Connect(fn)
    return B
end

local clearSelBtn = makeBtn("🗑 Limpar Slots",12,542,152,Color3.fromRGB(20,14,44),function()
    styleRemote=nil; abilityRemote=nil; gemRemote=nil
end)

local clearLogBtn = makeBtn("🔄 Limpar Log",176,542,152,Color3.fromRGB(20,14,44),function()
    hookLog={}; capturedRemotes={}
    styleRemote=nil; abilityRemote=nil; gemRemote=nil
    refreshSpy()
end)

-- Contador
local CountLbl = Instance.new("TextLabel", MainFrame)
CountLbl.Size = UDim2.new(1,-24,0,14); CountLbl.Position = UDim2.new(0,12,0,574)
CountLbl.BackgroundTransparency = 1; CountLbl.TextSize = 10
CountLbl.Font = Enum.Font.Code; CountLbl.TextXAlignment = Enum.TextXAlignment.Left
CountLbl.TextColor3 = Color3.fromRGB(65,60,105)
task.spawn(function()
    while true do
        local n = 0; for _ in pairs(capturedRemotes) do n=n+1 end
        CountLbl.Text = "📡 "..n.." remotes capturados  •  hook: "..(hooked and "ativo ✔" or "passivo ⚠")
        task.wait(1)
    end
end)

-- ─────────────────────────────────────────────
-- Toggle [P]
-- ─────────────────────────────────────────────
local menuOpen = false
local function toggleMenu()
    menuOpen = not menuOpen
    if menuOpen then
        MainFrame.Size = UDim2.new(0,0,0,0)
        MainFrame.Position = UDim2.new(0.5,0,0.5,0)
        MainFrame.Visible = true
        TweenService:Create(MainFrame,TweenInfo.new(0.25,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{
            Size=UDim2.new(0,W,0,H), Position=UDim2.new(0.5,-W/2,0.5,-H/2)
        }):Play()
    else
        TweenService:Create(MainFrame,TweenInfo.new(0.18,Enum.EasingStyle.Quad,Enum.EasingDirection.In),{
            Size=UDim2.new(0,0,0,0), Position=UDim2.new(0.5,0,0.5,0)
        }):Play()
        Popup.Visible = false
        task.delay(0.2,function() MainFrame.Visible=false end)
    end
end

CloseBtn.MouseButton1Click:Connect(toggleMenu)
UserInputService.InputBegan:Connect(function(i,gp)
    if not gp and i.KeyCode==Enum.KeyCode.P then toggleMenu() end
end)

print("✅ VBL Menu v5 carregado! [P] = abrir | Hook: "..(hooked and "ATIVO" or "PASSIVO"))
