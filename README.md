--// SKYNET Universal Roblox Script – Painel completo (toggle P)
--// Executar em qualquer executor compatível (Synapse, KRNL, Fluxus...)

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Locals
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Interface raiz
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "SKYNET_Panel"

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 300, 0, 400)
Main.Position = UDim2.new(0.5, -150, 0.5, -200)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

-- Toggle visibilidade (P)
local isOpen = true
UserInputService.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == Enum.KeyCode.P then
        isOpen = not isOpen
        Main.Visible = isOpen
    end
end)

-- Abas
local TabHolder = Instance.new("Frame", Main)
TabHolder.Size = UDim2.new(1, 0, 0, 30)
TabHolder.BackgroundTransparency = 1

function newTab(name, x)
    local btn = Instance.new("TextButton", TabHolder)
    btn.Size = UDim2.new(0, 100, 1, 0)
    btn.Position = UDim2.new(x, 0, 0, 0)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Text = name
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.SourceBold
    return btn
end

local MiscBtn = newTab("Misc", 0)
local AimbotBtn = newTab("Aimbot", 100)
local EspBtn = newTab("ESP", 200)

-- Containers
function newContainer()
    local f = Instance.new("Frame", Main)
    f.Size = UDim2.new(1, 0, 1, -30)
    f.Position = UDim2.new(0, 0, 0, 30)
    f.BackgroundTransparency = 1
    f.Visible = false
    return f
end

local Misc = newContainer()
local Aimbot = newContainer()
local ESP = newContainer()

-- Estados
local aimbotAtivo = false
local aimKey = Enum.UserInputType.MouseButton2 -- Botão direito
local fov = 80
local showFov = true

local espName, espBox, espSkeleton = true, true, true

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = fov * 5
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
fovCircle.Color = Color3.new(1, 1, 1)
fovCircle.Thickness = 2
fovCircle.NumSides = 64
fovCircle.Filled = false
fovCircle.Visible = showFov

-- Aimbot lógica
local function getClosest()
    local closest, dist = nil, math.huge
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end
    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local root = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
        if root then
            local pos, vis = Camera:WorldToViewportPoint(root.Position)
            if vis then
                local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
                if mag < fov * 5 and mag < dist then
                    closest = root
                    dist = mag
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    fovCircle.Radius = fov * 5
    fovCircle.Visible = showFov and aimbotAtivo
    if aimbotAtivo and UserInputService:IsMouseButtonPressed(aimKey) then
        local target = getClosest()
        if target then
            local pos = Camera:WorldToViewportPoint(target.Position)
            local targetPos = Vector2.new(pos.X, pos.Y)
            local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
            local delta = (targetPos - center)
            mousemoverel(delta.X / 3, delta.Y / 3)
        end
    end
end)

-- ESP
local espCache = {}

local function addESPs(p)
    if p == LocalPlayer then return end
    local folder = Instance.new("Folder", ScreenGui)
    folder.Name = p.Name
    espCache[p] = { folder = folder, objects = {} }

    local name = Drawing.new("Text")
    name.Visible = espName
    name.Center = true
    name.Outline = true
    name.Color = Color3.new(1, 1, 1)
    name.Size = 18
    espCache[p].objects.name = name

    local box = Drawing.new("Quad")
    box.Visible = espBox
    box.Thickness = 1
    box.Color = Color3.fromRGB(0, 255, 0)
    espCache[p].objects.box = box

    local lines = {}
    for i = 1, 6 do
        local l = Drawing.new("Line")
        l.Visible = espSkeleton
        l.Thickness = 1
        l.Color = Color3.new(1, 1, 1)
        lines[i] = l
    end
    espCache[p].objects.skeleton = lines

    local con = RunService.RenderStepped:Connect(function()
        local char = p.Character
        if not char then
            for _, v in pairs(espCache[p].objects) do
                if typeof(v) == "table" then
                    for _, l in pairs(v) do l.Visible = false end
                else
                    v.Visible = false
                end
            end
            return
        end
        local root = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        if not root or not head then return end
        local pos, vis = Camera:WorldToViewportPoint(root.Position)
        if not vis then
            for _, v in pairs(espCache[p].objects) do
                if typeof(v) == "table" then
                    for _, l in pairs(v) do l.Visible = false end
                else
                    v.Visible = false
                end
            end
            return
        end

        -- Name
        local name = espCache[p].objects.name
        name.Position = Vector2.new(pos.X, pos.Y - 45)
        name.Text = p.Name
        name.Visible = espName

        -- Box
        local box = espCache[p].objects.box
        local cframe = root.CFrame
        local tl = Camera:WorldToViewportPoint((cframe * CFrame.new(-2, 2, 0)).Position)
        local tr = Camera:WorldToViewportPoint((cframe * CFrame.new(2, 2, 0)).Position)
        local bl = Camera:WorldToViewportPoint((cframe * CFrame.new(-2, -2.5, 0)).Position)
        local br = Camera:WorldToViewportPoint((cframe * CFrame.new(2, -2.5, 0)).Position)
        box.PointA = Vector2.new(tl.X, tl.Y)
        box.PointB = Vector2.new(tr.X, tr.Y)
        box.PointC = Vector2.new(br.X, br.Y)
        box.PointD = Vector2.new(bl.X, bl.Y)
        box.Visible = espBox

        -- Skeleton
        local bones = {
            { "Head", "UpperTorso" },
            { "UpperTorso", "LowerTorso" },
            { "UpperTorso", "LeftUpperArm" },
            { "UpperTorso", "RightUpperArm" },
            { "LowerTorso", "LeftUpperLeg" },
            { "LowerTorso", "RightUpperLeg" }
        }
        local lines = espCache[p].objects.skeleton
        for i = 1, 6 do
            local a, b = bones[i][1], bones[i][2]
            local partA = char:FindFirstChild(a)
            local partB = char:FindFirstChild(b)
            if partA and partB then
                local p1 = Camera:WorldToViewportPoint(partA.Position)
                local p2 = Camera:WorldToViewportPoint(partB.Position)
                lines[i].From = Vector2.new(p1.X, p1.Y)
                lines[i].To = Vector2.new(p2.X, p2.Y)
                lines[i].Visible = espSkeleton
            end
        end
    end)
    espCache[p].con = con
end

local function removeESPs(p)
    if espCache[p] then
        espCache[p].con:Disconnect()
        for _, v in pairs(espCache[p].objects) do
            if typeof(v) == "table" then
                for _, l in pairs(v) do l:Remove() end
            else
                v:Remove()
            end
        end
        espCache[p].folder:Destroy()
        espCache[p] = nil
    end
end

Players.PlayerAdded:Connect(addESPs)
Players.PlayerRemoving:Connect(removeESPs)
for _, p in pairs(Players:GetPlayers()) do addESPs(p) end

-- UI Helpers
function newToggle(parent, name, default, callback)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -10, 0, 30)
    b.Position = UDim2.new(0, 5, 0, #parent:GetChildren() * 35)
    b.BackgroundColor3 = default and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    b.Text = name .. ": " .. tostring(default)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.BorderSizePixel = 0
    b.Font = Enum.Font.SourceBold
    b.MouseButton1Click:Connect(function()
        local v = not default
        default = v
        b.Text = name .. ": " .. tostring(v)
        b.BackgroundColor3 = v and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        callback(v)
    end)
    return b
end

-- Aimbot UI
do
    newToggle(Aimbot, "Ativo", aimbotAtivo, function(v)
        aimbotAtivo = v
    end)
    newToggle(Aimbot, "Mostrar FOV", showFov, function(v)
        showFov = v
    end)

    local slider = Instance.new("TextLabel", Aimbot)
    slider.Size = UDim2.new(1, -10, 0, 30)
    slider.Position = UDim2new(0, 5, 0, 70)
    slider.Text = "FOV: " .. fov
    slider.TextColor3 = Color3.new(1, 1, 1)
    slider.Font = Enum.Font.SourceBold

    local bar = Instance.new("Frame", slider)
    bar.Size = UDim2.new(0, 200, 0, 20)
    bar.Position = UDim2.new(0, 80, 0, 5)
    bar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    bar.BorderSizePixel = 0

    local fill = Instance.new("Frame", bar)
    fill.Size = UDim2.new(fov / 100, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
    fill.BorderSizePixel = 0

    local dragging = false
    bar.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    bar.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    bar.InputChanged:Connect(function(inp)
        if dragging and inp.UserInputType == Enum.UserInputType.MouseMovement then
            local percent = math.clamp((inp.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
            fov = math.floor(percent * 100)
            slider.Text = "FOV: " .. fov
            fill.Size = UDim2.new(percent, 0, 1, 0)
        end
    end)
end

-- ESP UI
do
    newToggle(ESP, "NAME", espName, function(v)
        espName = v
        for _, p in pairs(espCache) do
            if p.objects.name then p.objects.name.Visible = v end
        end
    end)
    newToggle(ESP, "BOX", espBox, function(v)
        espBox = v
        for _, p in pairs(espCache) do
            if p.objects.box then p.objects.box.Visible = v end
        end
    end)
    newToggle(ESP, "SKELETON", espSkeleton, function(v)
        espSkeleton = v
        for _, p in pairs(espCache) do
            if p.objects.skeleton then
                for _, l in pairs(p.objects.skeleton) do
                    l.Visible = v
                end
            end
        end
    end)
end

-- Misc UI (placeholder)
do
    local lbl = Instance.new("TextLabel", Misc)
    lbl.Size = UDim2.new(1, -10, 0, 30)
    lbl.Position = UDim2.new(0, 5, 0, 5)
    lbl.Text = "Misc vazio – adicione o que quiser aqui"
    lbl.TextColor3 = Color3.new(1, 1, 1)
    lbl.Font = Enum.Font.SourceBold
end

-- Tab switching
local function switchTo(tab)
    Misc.Visible = tab == Misc
    Aimbot.Visible = tab == Aimbot
    ESP.Visible = tab == ESP
end

MiscBtn.MouseButton1Click:Connect(function() switchTo(Misc) end)
AimbotBtn.MouseButton1Click:Connect(function() switchTo(Aimbot) end)
EspBtn.MouseButton1Click:Connect(function() switchTo(ESP) end)

-- Abre na aba Aimbot
switchTo(Aimbot)
