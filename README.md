# Stone-Lib

--[[
    STONE ENGINE UI 🗿 (VERSÃO COMPLETA)
    - PC & Mobile Friendly
    - Sistema de Keybind para Minimizar
    - Redimensionamento Manual
    - Componentes Modernos e Modulares
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- CONFIGURAÇÕES DE TEMA
local UI_THEME = {
    Main = Color3.fromRGB(20, 20, 20),
    Secondary = Color3.fromRGB(30, 30, 30),
    Accent = Color3.fromRGB(80, 150, 255),
    Text = Color3.fromRGB(255, 255, 255),
    TextDark = Color3.fromRGB(160, 160, 160),
    Rounding = UDim.new(0, 10)
}

local CurrentBind = Enum.KeyCode.RightControl

-- BASE DA UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StoneEngine_Complete"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

-- FUNÇÃO DRAGGABLE (ARRASTAR)
local function MakeDraggable(frame, handle)
    local dragging, dragInput, dragStart, startPos
    handle = handle or frame
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    handle.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- JANELA PRINCIPAL
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 350)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -175)
MainFrame.BackgroundColor3 = UI_THEME.Main
MainFrame.ClipsDescendants = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UI_THEME.Rounding

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundTransparency = 1
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -20, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "STONE ENGINE 🗿"
TitleLabel.TextColor3 = UI_THEME.Text
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 16
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

MakeDraggable(MainFrame, TitleBar)

-- BOTÃO FLUTUANTE
local FloatingBtn = Instance.new("TextButton")
FloatingBtn.Size = UDim2.new(0, 50, 0, 50)
FloatingBtn.Position = UDim2.new(0, 20, 0.2, 0)
FloatingBtn.BackgroundColor3 = UI_THEME.Main
FloatingBtn.Text = "🗿"
FloatingBtn.TextSize = 25
FloatingBtn.Parent = ScreenGui
Instance.new("UICorner", FloatingBtn).CornerRadius = UDim.new(0, 12)
MakeDraggable(FloatingBtn)

-- LOGICA ABRIR/FECHAR
local function ToggleUI()
    MainFrame.Visible = not MainFrame.Visible
    if MainFrame.Visible then
        MainFrame.Size = UDim2.new(0, 0, 0, 0)
        TweenService:Create(MainFrame, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Size = UDim2.new(0, 500, 0, 350)}):Play()
    end
end

FloatingBtn.MouseButton1Click:Connect(ToggleUI)
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == CurrentBind then ToggleUI() end
end)

-- SIDEBAR E CONTEÚDO
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -60)
Sidebar.Position = UDim2.new(0, 10, 0, 50)
Sidebar.BackgroundTransparency = 1
Sidebar.Parent = MainFrame

Instance.new("UIListLayout", Sidebar).Padding = UDim.new(0, 5)

local ContentArea = Instance.new("Frame")
ContentArea.Size = UDim2.new(1, -160, 1, -60)
ContentArea.Position = UDim2.new(0, 150, 0, 50)
ContentArea.BackgroundColor3 = UI_THEME.Secondary
ContentArea.Parent = MainFrame
Instance.new("UICorner", ContentArea).CornerRadius = UI_THEME.Rounding

-- MOTOR DE COMPONENTES
local UI = {}
local Tabs = {}

function UI:CreateTab(name)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, 0, 0, 35)
    TabBtn.BackgroundColor3 = UI_THEME.Secondary
    TabBtn.Text = name
    TabBtn.TextColor3 = UI_THEME.TextDark
    TabBtn.Font = Enum.Font.Gotham
    TabBtn.Parent = Sidebar
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)

    local Page = Instance.new("ScrollingFrame")
    Page.Size = UDim2.new(1, -10, 1, -10)
    Page.Position = UDim2.new(0, 5, 0, 5)
    Page.BackgroundTransparency = 1
    Page.ScrollBarThickness = 2
    Page.Visible = false
    Page.Parent = ContentArea

    local PageLayout = Instance.new("UIListLayout")
    PageLayout.Padding = UDim.new(0, 8)
    PageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    PageLayout.Parent = Page
    
    PageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        Page.CanvasSize = UDim2.new(0, 0, 0, PageLayout.AbsoluteContentSize.Y + 10)
    end)

    TabBtn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do
            t.Page.Visible = false
            t.Btn.TextColor3 = UI_THEME.TextDark
        end
        Page.Visible = true
        TabBtn.TextColor3 = UI_THEME.Accent
    end)

    Tabs[name] = {Btn = TabBtn, Page = Page}
    if #Sidebar:GetChildren() <= 2 then Page.Visible = true TabBtn.TextColor3 = UI_THEME.Accent end
    return Page
end

-- COMPONENTES
function UI:Button(parent, text, callback)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, -10, 0, 35)
    b.BackgroundColor3 = UI_THEME.Main
    b.Text = text
    b.TextColor3 = UI_THEME.Text
    b.Font = Enum.Font.Gotham
    b.Parent = parent
    Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(callback)
end

function UI:Toggle(parent, text, callback)
    local state = false
    local tgl = Instance.new("TextButton")
    tgl.Size = UDim2.new(1, -10, 0, 35)
    tgl.BackgroundColor3 = UI_THEME.Main
    tgl.Text = "  " .. text
    tgl.TextColor3 = UI_THEME.Text
    tgl.TextXAlignment = Enum.TextXAlignment.Left
    tgl.Font = Enum.Font.Gotham
    tgl.Parent = parent
    Instance.new("UICorner", tgl)

    local ind = Instance.new("Frame")
    ind.Size = UDim2.new(0, 20, 0, 20)
    ind.Position = UDim2.new(1, -30, 0.5, -10)
    ind.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    ind.Parent = tgl
    Instance.new("UICorner", ind).CornerRadius = UDim.new(1, 0)

    tgl.MouseButton1Click:Connect(function()
        state = not state
        TweenService:Create(ind, TweenInfo.new(0.2), {BackgroundColor3 = state and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)}):Play()
        callback(state)
    end)
end

function UI:Slider(parent, text, min, max, callback)
    local sFrame = Instance.new("Frame")
    sFrame.Size = UDim2.new(1, -10, 0, 50)
    sFrame.BackgroundColor3 = UI_THEME.Main
    sFrame.Parent = parent
    Instance.new("UICorner", sFrame)

    local lab = Instance.new("TextLabel")
    lab.Size = UDim2.new(1, -20, 0, 25)
    lab.Position = UDim2.new(0, 10, 0, 0)
    lab.BackgroundTransparency = 1
    lab.Text = text .. ": " .. min
    lab.TextColor3 = UI_THEME.Text
    lab.Font = Enum.Font.Gotham
    lab.TextXAlignment = Enum.TextXAlignment.Left
    lab.Parent = sFrame

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, -20, 0, 6)
    bg.Position = UDim2.new(0, 10, 0, 35)
    bg.BackgroundColor3 = UI_THEME.Secondary
    bg.Parent = sFrame
    Instance.new("UICorner", bg)

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = UI_THEME.Accent
    fill.Parent = bg
    Instance.new("UICorner", fill)

    local dragging = false
    local function move(input)
        local pos = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local val = math.floor(min + (max - min) * pos)
        lab.Text = text .. ": " .. val
        callback(val)
    end

    bg.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = true move(i) end end)
    UserInputService.InputChanged:Connect(function(i) if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then move(i) end end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = false end end)
end

function UI:TextBox(parent, placeholder, callback)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, -10, 0, 35)
    f.BackgroundColor3 = UI_THEME.Main
    f.Parent = parent
    Instance.new("UICorner", f)

    local i = Instance.new("TextBox")
    i.Size = UDim2.new(1, -20, 1, 0)
    i.Position = UDim2.new(0, 10, 0, 0)
    i.BackgroundTransparency = 1
    i.PlaceholderText = placeholder
    i.Text = ""
    i.TextColor3 = UI_THEME.Text
    i.Font = Enum.Font.Gotham
    i.TextXAlignment = Enum.TextXAlignment.Left
    i.Parent = f
    i.FocusLost:Connect(function(enter) callback(i.Text, enter) end)
end

function UI:Dropdown(parent, text, options, callback)
    local open = false
    local d = Instance.new("Frame")
    d.Size = UDim2.new(1, -10, 0, 35)
    d.BackgroundColor3 = UI_THEME.Main
    d.ClipsDescendants = true
    d.Parent = parent
    Instance.new("UICorner", d)

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.BackgroundTransparency = 1
    btn.Text = "  " .. text
    btn.TextColor3 = UI_THEME.Text
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Font = Enum.Font.Gotham
    btn.Parent = d

    for i, opt in pairs(options) do
        local o = Instance.new("TextButton")
        o.Size = UDim2.new(1, 0, 0, 30)
        o.Position = UDim2.new(0, 0, 0, 35 + (i-1)*30)
        o.BackgroundColor3 = UI_THEME.Secondary
        o.Text = opt
        o.TextColor3 = UI_THEME.TextDark
        o.Parent = d
        o.MouseButton1Click:Connect(function()
            btn.Text = "  " .. text .. ": " .. opt
            open = false
            TweenService:Create(d, TweenInfo.new(0.3), {Size = UDim2.new(1, -10, 0, 35)}):Play()
            callback(opt)
        end)
    end

    btn.MouseButton1Click:Connect(function()
        open = not open
        TweenService:Create(d, TweenInfo.new(0.3), {Size = open and UDim2.new(1, -10, 0, 35 + (#options * 30)) or UDim2.new(1, -10, 0, 35)}):Play()
    end)
end

function UI:MultiDropdown(parent, text, options, callback)
    local open = false
    local selected = {}
    local d = Instance.new("Frame")
    d.Size = UDim2.new(1, -10, 0, 35)
    d.BackgroundColor3 = UI_THEME.Main
    d.ClipsDescendants = true
    d.Parent = parent
    Instance.new("UICorner", d)

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.BackgroundTransparency = 1
    btn.Text = "  " .. text .. " (Multi)"
    btn.TextColor3 = UI_THEME.Text
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Font = Enum.Font.Gotham
    btn.Parent = d

    for i, opt in pairs(options) do
        local o = Instance.new("TextButton")
        o.Size = UDim2.new(1, 0, 0, 30)
        o.Position = UDim2.new(0, 0, 0, 35 + (i-1)*30)
        o.BackgroundColor3 = UI_THEME.Secondary
        o.Text = "[ ] " .. opt
        o.TextColor3 = UI_THEME.TextDark
        o.Parent = d
        o.MouseButton1Click:Connect(function()
            if table.find(selected, opt) then
                table.remove(selected, table.find(selected, opt))
                o.Text = "[ ] " .. opt
                o.TextColor3 = UI_THEME.TextDark
            else
                table.insert(selected, opt)
                o.Text = "[X] " .. opt
                o.TextColor3 = UI_THEME.Accent
            end
            callback(selected)
        end)
    end

    btn.MouseButton1Click:Connect(function()
        open = not open
        TweenService:Create(d, TweenInfo.new(0.3), {Size = open and UDim2.new(1, -10, 0, 35 + (#options * 30)) or UDim2.new(1, -10, 0, 35)}):Play()
    end)
end

function UI:Keybind(parent, text, callback)
    local b = Instance.new("Frame")
    b.Size = UDim2.new(1, -10, 0, 35)
    b.BackgroundColor3 = UI_THEME.Main
    b.Parent = parent
    Instance.new("UICorner", b)

    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -10, 1, 0)
    l.Position = UDim2.new(0, 10, 0, 0)
    l.BackgroundTransparency = 1
    l.Text = text .. ": " .. CurrentBind.Name
    l.TextColor3 = UI_THEME.Text
    l.Font = Enum.Font.Gotham
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = b

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 80, 0, 25)
    btn.Position = UDim2.new(1, -90, 0.5, -12)
    btn.BackgroundColor3 = UI_THEME.Secondary
    btn.Text = "Bind"
    btn.TextColor3 = UI_THEME.Accent
    btn.Parent = b
    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        btn.Text = "..."
        local conn
        conn = UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                CurrentBind = input.KeyCode
                l.Text = text .. ": " .. input.KeyCode.Name
                btn.Text = "Bind"
                callback(input.KeyCode)
                conn:Disconnect()
            end
        end)
    end)
end

-- RESIZE HANDLE
local Resizer = Instance.new("TextButton")
Resizer.Size = UDim2.new(0, 20, 0, 20)
Resizer.Position = UDim2.new(1, -20, 1, -20)
Resizer.BackgroundTransparency = 1
Resizer.Text = "◢"
Resizer.TextColor3 = UI_THEME.TextDark
Resizer.Parent = MainFrame

local resizing = false
Resizer.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then resizing = true end end)
UserInputService.InputChanged:Connect(function(i)
    if resizing and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local m = UserInputService:GetMouseLocation()
        MainFrame.Size = UDim2.new(0, math.max(400, m.X - MainFrame.AbsolutePosition.X), 0, math.max(250, m.Y - MainFrame.AbsolutePosition.Y + 36))
    end
end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then resizing = false end end)

-- EXEMPLO DE USO
local Tab1 = UI:CreateTab("Principal")
local Tab2 = UI:CreateTab("Config")

UI:Button(Tab1, "Notificação Teste", function() print("Olá Mundo!") end)
UI:Toggle(Tab1, "Ativar Aura", function(v) print("Aura:", v) end)
UI:Slider(Tab1, "Velocidade", 16, 300, function(v) 
    if Player.Character and Player.Character:FindFirstChild("Humanoid") then Player.Character.Humanoid.WalkSpeed = v end
end)
UI:TextBox(Tab1, "Nome do Alvo...", function(t) print("Alvo definido:", t) end)

UI:Dropdown(Tab2, "Teleporte", {"Spawn", "Loja", "Vip"}, function(v) print("TP para", v) end)
UI:MultiDropdown(Tab2, "Filtros ESP", {"Players", "Npcs", "Items"}, function(list) print("Ativos:", #list) end)
UI:Keybind(Tab2, "Atalho Menu", function(k) print("Novo bind:", k.Name) end)

print("Stone Engine Carregada com Sucesso! 🗿")
