--// Roblox Performance Optimizer
--// LocalScript

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local Player = Players.LocalPlayer

--// GUI
local Gui = Instance.new("ScreenGui")
Gui.Name = "PerformanceOptimizer"
Gui.ResetOnSpawn = false
Gui.Parent = Player:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(260, 300)
Main.Position = UDim2.new(0.5, -130, 0.5, -150)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.BorderSizePixel = 0
Main.Parent = Gui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = Main

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "⚡ Performance Optimizer"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 8)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Main

Title.LayoutOrder = 0

--// เก็บค่าก่อนปรับ เพื่อ Restore ได้
local Saved = {
    GlobalShadows = Lighting.GlobalShadows,
    Effects = {},
    Materials = {}
}

local function SaveEffects()
    table.clear(Saved.Effects)
    table.clear(Saved.Materials)

    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter")
        or obj:IsA("Trail")
        or obj:IsA("Beam")
        or obj:IsA("Smoke")
        or obj:IsA("Fire")
        or obj:IsA("Sparkles") then

            Saved.Effects[obj] = obj.Enabled
        end

        if obj:IsA("BasePart") then
            Saved.Materials[obj] = obj.Material
        end
    end
end

SaveEffects()

--// ปิดเงา
local function LowShadows(enabled)
    if enabled then
        Lighting.GlobalShadows = false
    else
        Lighting.GlobalShadows = Saved.GlobalShadows
    end
end

--// ลด Effect
local function LowEffects(enabled)
    if enabled then
        for obj in pairs(Saved.Effects) do
            if obj and obj.Parent then
                pcall(function()
                    obj.Enabled = false
                end)
            end
        end
    else
        for obj, value in pairs(Saved.Effects) do
            if obj and obj.Parent then
                pcall(function()
                    obj.Enabled = value
                end)
            end
        end
    end
end

--// ลด Material
local function LowMaterials(enabled)
    if enabled then
        for obj in pairs(Saved.Materials) do
            if obj and obj.Parent then
                pcall(function()
                    obj.Material = Enum.Material.SmoothPlastic
                end)
            end
        end
    else
        for obj, value in pairs(Saved.Materials) do
            if obj and obj.Parent then
                pcall(function()
                    obj.Material = value
                end)
            end
        end
    end
end

--// ปุ่ม
local function CreateButton(text, callback)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.fromOffset(220, 42)
    Button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    Button.TextColor3 = Color3.new(1, 1, 1)
    Button.TextSize = 14
    Button.Font = Enum.Font.Gotham
    Button.Text = text
    Button.AutoButtonColor = true
    Button.LayoutOrder = #Main:GetChildren()
    Button.Parent = Main

    local C = Instance.new("UICorner")
    C.CornerRadius = UDim.new(0, 7)
    C.Parent = Button

    Button.MouseButton1Click:Connect(function()
        callback(Button)
    end)

    return Button
end

local shadows = false
local effects = false
local materials = false

CreateButton("🌑 Low Shadows : OFF", function(btn)
    shadows = not shadows

    LowShadows(shadows)

    btn.Text = "🌑 Low Shadows : " .. (shadows and "ON" or "OFF")
end)

CreateButton("✨ Disable Effects : OFF", function(btn)
    effects = not effects

    LowEffects(effects)

    btn.Text = "✨ Disable Effects : " .. (effects and "ON" or "OFF")
end)

CreateButton("🧱 Low Materials : OFF", function(btn)
    materials = not materials

    LowMaterials(materials)

    btn.Text = "🧱 Low Materials : " .. (materials and "ON" or "OFF")
end)

CreateButton("🚀 APPLY ALL", function()
    shadows = true
    effects = true
    materials = true

    LowShadows(true)
    LowEffects(true)
    LowMaterials(true)
end)

CreateButton("♻️ RESTORE", function()
    shadows = false
    effects = false
    materials = false

    LowShadows(false)
    LowEffects(false)
    LowMaterials(false)
end)

--// ปุ่มลากเมนู
local dragging = false
local dragStart
local startPos

Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPos = Main.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

Main.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then

        dragStart = input
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and dragStart then
        local delta = input.Position - dragStart.Position

        Main.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)# ABC
