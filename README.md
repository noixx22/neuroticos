--// Universal Roblox Script | Painel: Misc / Aimbot / ESP
--// Coloque num executor compatível (Synapse, KRNL, Fluxus, etc.)
--// Executar em qualquer jogo – não requer hooks adicionais

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Locals
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Interface
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 300, 0, 400)
Main.Position = UDim2.new(0.5, -150, 0.5, -200)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

-- Tabs
local TabHolder = Instance.new("Frame", Main)
TabHolder.Size = UDim2.new(1, 0, 0, 30)
TabHolder.BackgroundTransparency = 1
local MiscBtn = btn("Misc", 0)
local AimbotBtn = btn("Aimbot", 100)
local EspBtn = btn("ESP", 200)

-- Containers
local Misc = container()
local Aimbot = container()
local ESP = container()

-- Estados
local aimbotAtivo = false
local aimKey = Enum.UserInputType.MouseButton2 -- Botão direito
local fov = 80
local showFov = true
local espName, espBox, espSkeleton = true, true, true

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = fov*5
fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
fovCircle.Color = Color3.new(1,1,1)
fovCircle.Thickness = 2
fovCircle.NumSides = 64
fovCircle.Filled = false
fovCircle.Visible = showFov

-- Aimbot Functions
local function getClosest()
    local closest, dist = nil, math.huge
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end
    for _,p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local root = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
        if root then
            local pos, vis = Camera:WorldToViewportPoint(root.Position)
            if vis then
                local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if mag < fov*5 and mag < dist then
                    closest = root; dist = mag
                end
            end
        end
    end
    return closest
end

-- Aimbot Loop
RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    fovCircle.Radius = fov*5
    fovCircle.Visible = showFov and aimbotAtivo
    if aimbotAtivo and UserInputService:IsMouseButtonPressed(aimKey) then
        local target = getClosest()
        if target then
            local pos = Camera:WorldToViewportPoint(target.Position)
            local targetPos = Vector2.new(pos.X, pos.Y)
            local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            local delta = (targetPos - center)
            mousemoverel(delta.X/3, delta.Y/3) -- Smooth pull
        end
    end
end)

-- ESP Storage
local espCache = {}

local function addESPs(p)
    if p == LocalPlayer then return end
    local folder = Instance.new("Folder", ScreenGui); folder.Name = p.Name
    espCache[p] = {folder=folder, objects={}}
    local function build()
        -- Name
        local name = Drawing.new("Text")
        name.Visible = espName; name.Center = true; name.Outline = true
        name.Color = Color3.new(1,1,1); name.Size = 18
        espCache[p].objects.name = name
        -- Box
        local box = Drawing.new("Quad")
        box.Visible = espBox; box.Thickness = 1
        box.Color = Color3.fromRGB(0,255,0)
        espCache[p].objects.box = box
        -- Skeleton
        local lines = {}
        for i=1,6 do
            local l = Drawing.new("Line")
            l.Visible = espSkeleton; l.Thickness = 1; l.Color = Color3.new(1,1,1)
            lines[i] = l
        end
        espCache[p].objects.skeleton = lines
    end
    build()
    local con; con = RunService.RenderStepped:Connect(function()
        local char = p.Character
        if not char then
            for _,v in pairs(espCache[p].objects) do
                if typeof(v) == "table" then for _,l in pairs(v) do l.Visible = false end
                else v.Visible = false end
            end; return
        end
        local root = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        if not root or not head then return end
        local pos, vis = Camera:WorldToViewportPoint(root.Position)
        if not vis then
            for _,v in pairs(espCache[p].objects) do
                if typeof(v) == "table" then for _,l in pairs(v) do l.Visible = false end
                else v.Visible = false end
            end; return
        end
        -- Name
        local name = espCache[p].objects.name
        name.Position = Vector2.new(pos.X, pos.Y - 45)
        name.Text = p.Name
        name.Visible = espName
        -- Box 2D
        local box = espCache[p].objects.box
        local hrp = root
        local cframe = hrp.CFrame
        local size = Vector3.fromNormalId(Enum.NormalId.Right)
        local top = (cframe * CFrame.new(0, 2, 0)).Position
        local bottom = (cframe * CFrame.new(0, -2.5, 0)).Position
        local tl, tr, bl, br = Camera:WorldToViewportPoint((cframe * CFrame.new(-2, 2, 0)).Position),
                               Camera:WorldToViewportPoint((cframe * CFrame.new(2, 2, 0)).Position),
                               Camera:WorldToViewportPoint((cframe * CFrame.new(-2, -2.5, 0)).Position),
                               Camera:WorldToViewportPoint((cframe * CFrame.new(2, -2.5, 0)).Position)
        box.PointA = Vector2.new(tl.X, tl.Y)
        box.PointB = Vector2.new(tr.X, tr.Y)
        box.PointC = Vector2.new(br.X, br.Y)
        box.PointD = Vector2.new(bl.X, bl.Y)
        box.Visible = espBox
        -- Skeleton
        local bones = {
            {"Head", "UpperTorso"},
            {"UpperTorso", "LowerTorso"},
            {"UpperTorso", "LeftUpperArm"},
            {"UpperTorso", "RightUpperArm"},
            {"LowerTorso", "LeftUpperLeg"},
            {"LowerTorso", "RightUpperLeg"}
        }
        local lines = espCache[p].objects.skeleton
        for i=1,6 do
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
        for _,v in pairs(espCache[p].objects) do
            if typeof(v) == "table" then for _,l in pairs(v) do l:Remove() end
            else v:Remove() end
        end
        espCache[p].folder:Destroy()
        espCache[p] = nil
    end
end

Players.PlayerAdded:Connect(addESPs)
Players.PlayerRemoving:Connect(removeESPs)
for _,p in pairs(Players:GetPlayers()) do addESPs(p) end

-- UI Helper
function btn(text, x)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 100, 1, 0)
    b.Position = UDim2.new.new(x, 0, 0, 0)
    b.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    b.Text = text; b.TextColor3 = Color3.new(1,1,1)
    b.BorderSizePixel = 0
    b.Parent = TabHolder
    b.MouseButton1Click:Connect(function()
        for _,v in pairs{Misc, Aimbot, ESP} do v.Visible = false end
        if text == "Misc" then Misc.Visible = true
        elseif text == "Aimbot" then Aimbot.Visible = true
        else ESP.Visible = true end
    end)
    return b
end

function container()
    local f = Instance.new("Frame", Main)
    f.Size = UDim2.new(1, 0, 1, -30)
    f.Position = UDim2.new(0, 0, 0, 30)
    f.BackgroundTransparency = 1
    f.Visible = false
    return f
end

-- Aimbot UI
do
    Aimbot.Visible = true
    local function toggle(name, default, callback)
        local b = Instance.new("TextButton", Aimbot)
        b.Size = UDim2.new(1, -10, 0, 30)
        b.Position = UDim2.new(0, 5, 0, #Aimbot:GetChildren()*35)
        b.Text = name .. ": "..tostring(default)
        b.BackgroundColor3 = default and Color3.green or Color3.red
        b.TextColor3 = Color3.new(1,1,1)
        b.MouseButton1Click:Connect(function()
            local v = not callback()
            b.Text = name .. ": "..tostring(v)
            b.BackgroundColor3 = v and Color3.green or Color3.red
        end)
    end
    toggle("Ativo", aimbotAtivo, function() aimbotAtivo = not aimbotAtivo; return aimbotAtivo end)
    toggle("Mostrar FOV", showFov, function() showFov = not showFov; return showFov end)

    local slider = Instance.new("TextLabel", Aimbot)
    slider.Size = UDim2.new(1, -10, 0, 30)
    slider.Position = UDim2.new(0, 5, 0, 70)
    slider.Text = "FOV: "..fov
    slider.TextColor3 = Color3.new(1,1,1)
    local drag = Instance.new("TextButton",, slider)
    drag.Size = UDim2.new(0, 200, 0, 20)
    drag.Position = UDim2.new(0, 50, 0, 5)
    drag.Text = ""
    drag.Background = Color3.fromRGB(60,60,60)
    local fill = Instance.new("Frame", drag)
    fill.Size = UDim2.new.new(fov/100, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0,120,255)
    local function update(x)
        local percent = math.clamp((x - drag.AbsolutePosition.X)/drag.AbsoluteSize.X, 0, 1)
        fov = math.floor(percent*100)
        slider.Text = "FOV: "..fov
        fill.Size = UDim2.new(percent, 0, 1, 0)
    end
    drag.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 then
            local moved
            moved = drag.MouseMoved:Connect(function(x)
                update(x)
            end)
            drag.InputEnded:Connect(function(e)
                if e.UserInputType == Enum.UserInputType.MouseButton1 then moved:Disconnect() end
            end)
        end
    end)
end

-- ESP UI
do
    local function espToggle(name, var)
        local b = Instance.new("TextButton", ESP)
        b.Size = UDim2.new(1, -10, 0, 30)
        b.Position = UDim2.new(0, 5, 0, #ESP:GetChildren()*35)
        b.Text = name .. ": "..tostring(var)
        b.BackgroundColor3 = var and Color3.green or Color3.red
        b.TextColor3 = Color3.new(1,1,1)
        b.MouseButton1Click:Connect(function()
            var = not var
            b.Text = name .. ": "..tostring(var)
            b.BackgroundColor3 = var and Color3.green or Color3.red
            if name == "NAME" then espName = var
            elseif name == "BOX" then espBox = var
            elseif name == "SKELETON" then espSkeleton = var end
        end)
    end
    espToggle("NAME", espName)
    espToggle("BOX", espBox)
    espToggle("SKELETON", espSkeleton)
end

-- Misc UI (placeholder)
do
    local lbl = Instance.new("TextLabel", Misc)
    lbl.Size = UDim2.new(1, -10, 0, 30)
    lbl.Position = UDim2.new(0, 5, 0, 5)
    lbl.Text = "Misc vazio – adicione o que quiser aqui"
    lbl.TextColor3 = Color3.new(1,1,1)
end

-- Abre na aba Aimbot por padrão
Misc.Visible = false; ESP.Visible = false; Aimbot.Visible = true
