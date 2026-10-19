--[================================================================]--
--  DON scripts - Enhanced Client Dashboard
--[================================================================]--

local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- UI Root Container Selection
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function()
	if CoreGui:IsLoaded() then
		guiParent = CoreGui
	end
end)

if guiParent:FindFirstChild("DON_Scripts_UI") then
	guiParent.DON_Scripts_UI:Destroy()
end

----------------------------------------------------
-- 1. THEMES CONFIGURATION
----------------------------------------------------
local Themes = {
	Dark = {
		MainBackground = Color3.fromRGB(24, 24, 28),
		HeaderBackground = Color3.fromRGB(35, 35, 42),
		ContainerBackground = Color3.fromRGB(32, 32, 38),
		TextColor = Color3.fromRGB(240, 240, 240),
		AccentColor = Color3.fromRGB(41, 128, 185)
	},
	Light = {
		MainBackground = Color3.fromRGB(240, 240, 245),
		HeaderBackground = Color3.fromRGB(220, 220, 230),
		ContainerBackground = Color3.fromRGB(225, 225, 235),
		TextColor = Color3.fromRGB(30, 30, 30),
		AccentColor = Color3.fromRGB(52, 152, 219)
	}
}

local currentTheme = Themes.Dark

----------------------------------------------------
-- 2. BUILD GUI INTERFACE
----------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DON_Scripts_UI"
screenGui.ResetOnSpawn = false
screenGui.Parent = guiParent

-- Toggle Open/Close Button (Bottom-Left)
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0, 120, 0, 40)
toggleButton.Position = UDim2.new(0, 15, 0.5, -20)
toggleButton.BackgroundColor3 = currentTheme.HeaderBackground
toggleButton.TextColor3 = currentTheme.TextColor
toggleButton.TextSize = 14
toggleButton.Font = Enum.Font.GothamBold
toggleButton.Text = "Toggle Menu"
toggleButton.Parent = screenGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggleButton

-- Main Dashboard Window
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 480, 0, 360)
mainFrame.Position = UDim2.new(0.5, -240, 0.5, -180)
mainFrame.BackgroundColor3 = currentTheme.MainBackground
mainFrame.Visible = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 10)
mainCorner.Parent = mainFrame

-- Title Bar
local titleBar = Instance.new("TextLabel")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = currentTheme.HeaderBackground
titleBar.BorderSizePixel = 0
titleBar.TextColor3 = currentTheme.TextColor
titleBar.TextSize = 16
titleBar.Font = Enum.Font.GothamBold
titleBar.Text = "DON Scripts - Client Dashboard"
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 10)
titleCorner.Parent = titleBar

-- Draggable Logic
local dragging, dragInput, dragStart, startPos

titleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
		
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

titleBar.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(
			startPos.X.Scale, 
			startPos.X.Offset + delta.X, 
			startPos.Y.Scale, 
			startPos.Y.Offset + delta.Y
		)
	end
end)

-- Navigation Tab Bar
local tabBar = Instance.new("Frame")
tabBar.Name = "TabBar"
tabBar.Size = UDim2.new(1, -20, 0, 35)
tabBar.Position = UDim2.new(0, 10, 0, 48)
tabBar.BackgroundTransparency = 1
tabBar.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.Padding = UDim.new(0, 8)
tabLayout.Parent = tabBar

local function createTabButton(name, order)
	local btn = Instance.new("TextButton")
	btn.Name = name .. "TabButton"
	btn.Size = UDim2.new(0, 100, 1, 0)
	btn.BackgroundColor3 = currentTheme.HeaderBackground
	btn.BorderSizePixel = 0
	btn.TextColor3 = currentTheme.TextColor
	btn.TextSize = 13
	btn.Font = Enum.Font.GothamSemibold
	btn.Text = name
	btn.LayoutOrder = order
	btn.Parent = tabBar
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = btn
	return btn
end

local movementTabBtn = createTabButton("Movement", 1)
local fovTabBtn = createTabButton("FOV Overlay", 2)
local settingsTabBtn = createTabButton("Settings", 3)

-- Content Container
local contentContainer = Instance.new("Frame")
contentContainer.Name = "ContentContainer"
contentContainer.Size = UDim2.new(1, -20, 1, -95)
contentContainer.Position = UDim2.new(0, 10, 0, 90)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = mainFrame

local function createContentFrame(name)
	local frm = Instance.new("ScrollingFrame")
	frm.Name = name .. "TabFrame"
	frm.Size = UDim2.new(1, 0, 1, 0)
	frm.BackgroundTransparency = 1
	frm.BorderSizePixel = 0
	frm.CanvasSize = UDim2.new(0, 0, 0, 0)
	frm.AutomaticCanvasSize = Enum.AutomaticSize.Y
	frm.Visible = false
	frm.Parent = contentContainer
	
	local layout = Instance.new("UIListLayout")
	layout.FillDirection = Enum.FillDirection.Vertical
	layout.SortOrder = Enum.SortOrder.LayoutOrder
	layout.Padding = UDim.new(0, 10)
	layout.Parent = frm
	
	return frm
end

local movementFrame = createContentFrame("Movement")
local fovFrame = createContentFrame("FOV Overlay")
local settingsFrame = createContentFrame("Settings")

-- Button Helper
local function createFeatureButton(parent, text, order)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, 38)
	btn.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
	btn.BorderSizePixel = 0
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.TextSize = 13
	btn.Font = Enum.Font.GothamSemibold
	btn.Text = text
	btn.LayoutOrder = order
	btn.Parent = parent
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = btn
	return btn
end

----------------------------------------------------
-- 3. FOV OVERLAY IMPLEMENTATION
----------------------------------------------------
local fovCircle = Instance.new("Frame")
fovCircle.Name = "FOVCircle"
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircle.BackgroundTransparency = 1
fovCircle.Size = UDim2.new(0, 200, 0, 200)
fovCircle.Visible = false
fovCircle.Parent = screenGui

local fovStroke = Instance.new("UIStroke")
fovStroke.Color = Color3.fromRGB(255, 255, 255)
fovStroke.Thickness = 2
fovStroke.Transparency = 0.2
fovStroke.Parent = fovCircle

local fovCorner = Instance.new("UICorner")
fovCorner.CornerRadius = UDim.new(1, 0)
fovCorner.Parent = fovCircle

----------------------------------------------------
-- 4. CONTROLLERS & LOGIC
----------------------------------------------------
local isInfiniteJumpActive = false
local isSpeedBoostActive = false
local isFovVisible = false

local fovColors = {
	Color3.fromRGB(255, 255, 255),
	Color3.fromRGB(255, 50, 50),
	Color3.fromRGB(50, 255, 50),
	Color3.fromRGB(50, 150, 255)
}
local fovColorNames = {"White", "Red", "Green", "Blue"}
local currentFovColorIndex = 1

local function updateButtonState(button, state, textPrefix)
	if state then
		button.Text = textPrefix .. ": ON"
		button.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
	else
		button.Text = textPrefix .. ": OFF"
		button.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
	end
end

-- Movement Buttons
local speedBtn = createFeatureButton(movementFrame, "Speed Modifier: OFF", 1)
local jumpBtn = createFeatureButton(movementFrame, "Infinite Jump: OFF", 2)

-- FOV Buttons
local fovToggleBtn = createFeatureButton(fovFrame, "FOV Overlay: OFF", 1)
local fovSizeBtn = createFeatureButton(fovFrame, "FOV Radius: 200px", 2)
local fovColorBtn = createFeatureButton(fovFrame, "FOV Color: White", 3)

-- Settings Buttons
local themeToggleBtn = createFeatureButton(settingsFrame, "Theme: Dark Mode", 1)

-- Tab Switcher
local function showTab(targetFrame)
	for _, child in ipairs(contentContainer:GetChildren()) do
		if child:IsA("ScrollingFrame") then
			child.Visible = false
		end
	end
	targetFrame.Visible = true
end

movementTabBtn.MouseButton1Click:Connect(function() showTab(movementFrame) end)
fovTabBtn.MouseButton1Click:Connect(function() showTab(fovFrame) end)
settingsTabBtn.MouseButton1Click:Connect(function() showTab(settingsFrame) end)

toggleButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)

-- Speed Modifier
speedBtn.MouseButton1Click:Connect(function()
	isSpeedBoostActive = not isSpeedBoostActive
	updateButtonState(speedBtn, isSpeedBoostActive, "Speed Modifier")
end)

RunService.Heartbeat:Connect(function()
	local character = LocalPlayer.Character
	if character and character:FindFirstChildOfClass("Humanoid") then
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if isSpeedBoostActive then
			humanoid.WalkSpeed = 32
		else
			humanoid.WalkSpeed = 16
		end
	end
end)

-- Infinite Jump
jumpBtn.MouseButton1Click:Connect(function()
	isInfiniteJumpActive = not isInfiniteJumpActive
	updateButtonState(jumpBtn, isInfiniteJumpActive, "Infinite Jump")
end)

UserInputService.JumpRequest:Connect(function()
	if isInfiniteJumpActive then
		local character = LocalPlayer.Character
		if character and character:FindFirstChildOfClass("Humanoid") then
			character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- FOV Circle Controls
fovToggleBtn.MouseButton1Click:Connect(function()
	isFovVisible = not isFovVisible
	fovCircle.Visible = isFovVisible
	updateButtonState(fovToggleBtn, isFovVisible, "FOV Overlay")
end)

fovSizeBtn.MouseButton1Click:Connect(function()
	local currentSize = fovCircle.Size.X.Offset
	local newSize = currentSize >= 400 and 100 or currentSize + 100
	fovCircle.Size = UDim2.new(0, newSize, 0, newSize)
	fovSizeBtn.Text = "FOV Radius: " .. tostring(newSize) .. "px"
end)

fovColorBtn.MouseButton1Click:Connect(function()
	currentFovColorIndex = (currentFovColorIndex % #fovColors) + 1
	fovStroke.Color = fovColors[currentFovColorIndex]
	fovColorBtn.Text = "FOV Color: " .. fovColorNames[currentFovColorIndex]
end)

-- Theme Switcher
themeToggleBtn.MouseButton1Click:Connect(function()
	if currentTheme == Themes.Dark then
		currentTheme = Themes.Light
		themeToggleBtn.Text = "Theme: Light Mode"
	else
		currentTheme = Themes.Dark
		themeToggleBtn.Text = "Theme: Dark Mode"
	end
	
	mainFrame.BackgroundColor3 = currentTheme.MainBackground
	titleBar.BackgroundColor3 = currentTheme.HeaderBackground
	titleBar.TextColor3 = currentTheme.TextColor
	toggleButton.BackgroundColor3 = currentTheme.HeaderBackground
	toggleButton.TextColor3 = currentTheme.TextColor
end)

-- Initialize Default Tab
showTab(movementFrame)

