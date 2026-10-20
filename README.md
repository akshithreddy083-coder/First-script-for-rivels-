
-- DON scripts | Movable Red & Black GUI
-- Roblox Studio | LocalScript
-- Open/close + draggable main panel and floating button

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local oldGui = playerGui:FindFirstChild("DONscripts")
if oldGui then
    oldGui:Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = "DONscripts"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local RED = Color3.fromRGB(255, 20, 55)
local DARK = Color3.fromRGB(15, 13, 17)
local ROW = Color3.fromRGB(29, 25, 31)
local WHITE = Color3.fromRGB(245, 245, 250)
local GRAY = Color3.fromRGB(165, 160, 175)
local OFF = Color3.fromRGB(65, 55, 62)

local function create(class, props, parent)
    local obj = Instance.new(class)
    for key, value in pairs(props) do
        obj[key] = value
    end
    obj.Parent = parent
    return obj
end

-- Main panel
local main = create("Frame", {
    Name = "MainPanel",
    Size = UDim2.fromOffset(330, 390),
    Position = UDim2.new(0.5, -165, 0.5, -195),
    BackgroundColor3 = DARK,
    BorderSizePixel = 0,
    Active = true,
}, gui)

create("UICorner", {
    CornerRadius = UDim.new(0, 12)
}, main)

create("UIStroke", {
    Color = RED,
    Thickness = 1.5
}, main)

-- Floating reopen button
local openButton = create("TextButton", {
    Name = "OpenButton",
    Size = UDim2.fromOffset(120, 42),
    Position = UDim2.new(0, 15, 0.5, -21),
    BackgroundColor3 = DARK,
    Text = "DON scripts",
    TextColor3 = RED,
    TextSize = 15,
    Font = Enum.Font.GothamBold,
    Visible = false,
    Active = true,
    AutoButtonColor = true,
}, gui)

create("UICorner", {
    CornerRadius = UDim.new(0, 9)
}, openButton)

create("UIStroke", {
    Color = RED,
    Thickness = 1.5
}, openButton)

-- Drag either panel with mouse or touch.
-- The title/header is the drag handle for the main panel,
-- so buttons and sliders remain usable.
local function makeDraggable(frame, handle)
    local dragging = false
    local dragStart
    local startPosition
    local activeInput

    handle.Active = true

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            dragStart = input.Position
            startPosition = frame.Position
            activeInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end

        local isMouseMove =
            activeInput.UserInputType == Enum.UserInputType.MouseButton1
            and input.UserInputType == Enum.UserInputType.MouseMovement

        local isTouchMove =
            activeInput.UserInputType == Enum.UserInputType.Touch
            and input == activeInput

        if isMouseMove or isTouchMove then
            local delta = input.Position - dragStart

            frame.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input == activeInput then
            dragging = false
            activeInput = nil
        end
    end)
end

-- Header is a dedicated drag handle.
local header = create("TextLabel", {
    Name = "DragHeader",
    Size = UDim2.new(1, -55, 0, 42),
    Position = UDim2.fromOffset(15, 8),
    BackgroundTransparency = 1,
    Text = "DON scripts",
    TextColor3 = RED,
    TextSize = 25,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    Active = true,
}, main)

makeDraggable(main, header)
makeDraggable(openButton, openButton)

create("TextLabel", {
    Size = UDim2.new(1, -30, 0, 20),
    Position = UDim2.fromOffset(15, 45),
    BackgroundTransparency = 1,
    Text = "GAME CONTROL PANEL",
    TextColor3 = GRAY,
    TextSize = 11,
    Font = Enum.Font.Gotham,
}, main)

-- Feature toggles
local features = {
    "Fly",
    "ESP / Player Highlighting",
    "Nearest-Player Aim Assist",
    "Speed Multiplier",
}

local states = {}
local startY = 76

for i, name in ipairs(features) do
    states[name] = false

    local row = create("Frame", {
        Size = UDim2.new(1, -24, 0, 43),
        Position = UDim2.fromOffset(12, startY + (i - 1) * 49),
        BackgroundColor3 = ROW,
        BorderSizePixel = 0,
    }, main)

    create("UICorner", {
        CornerRadius = UDim.new(0, 7)
    }, row)

    create("TextLabel", {
        Size = UDim2.new(1, -95, 1, 0),
        Position = UDim2.fromOffset(10, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        TextXAlignment = Enum.TextXAlignment.Left,
    }, row)

    local button = create("TextButton", {
        Size = UDim2.fromOffset(65, 29),
        Position = UDim2.new(1, -75, 0.5, -14),
        BackgroundColor3 = OFF,
        Text = "OFF",
        TextColor3 = WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamBold,
    }, row)

    create("UICorner", {
        CornerRadius = UDim.new(0, 6)
    }, button)

    button.Activated:Connect(function()
        states[name] = not states[name]
        button.Text = states[name] and "ON" or "OFF"
        button.BackgroundColor3 = states[name] and RED or OFF

        -- Connect to your own game's authorized
        -- feature implementation here.
        print(name, states[name] and "enabled" or "disabled")
    end)
end

-- Speed slider
create("TextLabel", {
    Size = UDim2.new(1, -24, 0, 25),
    Position = UDim2.fromOffset(12, 280),
    BackgroundTransparency = 1,
    Text = "SPEED MULTIPLIER",
    TextColor3 = WHITE,
    TextSize = 13,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
}, main)

local speed = 2

local speedValue = create("TextLabel", {
    Size = UDim2.fromOffset(55, 25),
    Position = UDim2.new(1, -67, 0, 280),
    BackgroundTransparency = 1,
    Text = "2.0x",
    TextColor3 = RED,
    TextSize = 14,
    Font = Enum.Font.GothamBold,
}, main)

local slider = create("TextButton", {
    Size = UDim2.new(1, -30, 0, 12),
    Position = UDim2.fromOffset(15, 315),
    BackgroundColor3 = OFF,
    Text = "",
    AutoButtonColor = false,
}, main)

create("UICorner", {
    CornerRadius = UDim.new(1, 0)
}, slider)

local fill = create("Frame", {
    Size = UDim2.new(0.25, 0, 1, 0),
    BackgroundColor3 = RED,
    BorderSizePixel = 0,
}, slider)

create("UICorner", {
    CornerRadius = UDim.new(1, 0)
}, fill)

local function updateSpeed(x)
    local width = slider.AbsoluteSize.X
    if width <= 0 then return end

    local alpha = math.clamp(
        (x - slider.AbsolutePosition.X) / width,
        0, 1
    )

    speed = math.floor((1 + alpha * 4) * 2 + 0.5) / 2
    fill.Size = UDim2.new((speed - 1) / 4, 0, 1, 0)
    speedValue.Text = string.format("%.1fx", speed)

    -- Apply speed through your own game's
    -- movement system when the feature is enabled.
end

local sliderDragging = false

slider.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = true
        updateSpeed(input.Position.X)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if sliderDragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        updateSpeed(input.Position.X)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = false
    end
end)

create("TextLabel", {
    Size = UDim2.new(1, -30, 0, 22),
    Position = UDim2.fromOffset(15, 335),
    BackgroundTransparency = 1,
    Text = "DON scripts | Ready",
    TextColor3 = GRAY,
    TextSize = 11,
    Font = Enum.Font.Gotham,
}, main)

-- Close button
local close = create("TextButton", {
    Size = UDim2.fromOffset(30, 30),
    Position = UDim2.new(1, -38, 0, 10),
    BackgroundColor3 = ROW,
    Text = "X",
    TextColor3 = RED,
    TextSize = 15,
    Font = Enum.Font.GothamBold,
}, main)

create("UICorner", {
    CornerRadius = UDim.new(0, 6)
}, close)

-- Hide and reopen without destroying the GUI.
close.Activated:Connect(function()
    main.Visible = false
    openButton.Visible = true
end)

openButton.Activated:Connect(function()
    main.Visible = true
    openButton.Visible = false
end)

print("DON scripts GUI loaded")
