-- Parent this LocalScript to StarterPlayerScripts or StarterGui

-- Create a ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlayerListGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Create a main frame to hold the player list
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0.3, 0, 0.5, 0) -- Adjusts to 30% width and 50% height of screen
mainFrame.Position = UDim2.new(0.35, 0, 0.25, 0) -- Centers the frame on screen
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20) -- Semi Black
mainFrame.BorderSizePixel = 0
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.Parent = screenGui

-- Add a corner radius to the frame
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10) -- Adjust for more or less curve
corner.Parent = mainFrame

-- Create a container for the player list and teleport button
local contentFrame = Instance.new("Frame")
contentFrame.Name = "ContentFrame"
contentFrame.Size = UDim2.new(1, 0, 1, -90)
contentFrame.Position = UDim2.new(0, 0, 0, 50)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Make the frame draggable
local dragging = false
local dragInput, dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
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

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

local UserInputService = game:GetService("UserInputService")
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

-- Add a title to the frame
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -80, 0, 40) -- Leave space for buttons
title.Position = UDim2.new(0.5, 0, 0, 0)
title.AnchorPoint = Vector2.new(0.5, 0) -- Center horizontally
title.BackgroundTransparency = 1
title.Text = "AyaXLeo Hack"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 40
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Parent = mainFrame

-- Create a container for buttons
local buttonContainer = Instance.new("Frame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(0, 80, 0, 40)
buttonContainer.Position = UDim2.new(1, -80, 0, 0)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Parent = mainFrame

-- Minimize button
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0.5, 0, 1, 0)
minimizeButton.Position = UDim2.new(0, 0, 0, 0)
minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- White
minimizeButton.TextSize = 25
minimizeButton.Text = "-"
minimizeButton.TextColor3 = Color3.fromRGB(0, 0, 0) -- Black text
minimizeButton.Parent = buttonContainer

local cornerMinimize = Instance.new("UICorner")
cornerMinimize.CornerRadius = UDim.new(0, 10)
cornerMinimize.Parent = minimizeButton

-- Exit button
local exitButton = Instance.new("TextButton")
exitButton.Size = UDim2.new(0.5, 0, 1, 0)
exitButton.Position = UDim2.new(0.5, 0, 0, 0)
exitButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Red
exitButton.TextSize = 25
exitButton.Text = "X"
exitButton.TextColor3 = Color3.fromRGB(0, 0, 0) -- Black text
exitButton.Parent = buttonContainer

local cornerExit = Instance.new("UICorner")
cornerExit.CornerRadius = UDim.new(0, 10)
cornerExit.Parent = exitButton

-- Ensure these variables are initialized before use
local minimized = false -- To track whether the frame is minimized
local originalSize = mainFrame.Size -- Store the original size of the frame

-- Create a teleport button (outside updatePlayerList())
local teleportButton = Instance.new("TextButton")
teleportButton.Size = UDim2.new(1, -20, 0, 40)
teleportButton.Position = UDim2.new(0, 10, 1, -50) -- Adjusted to the bottom of the main frame
teleportButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
teleportButton.TextColor3 = Color3.fromRGB(0, 255, 0)
teleportButton.Font = Enum.Font.SourceSansBold
teleportButton.TextSize = 20
teleportButton.Text = "Teleport to Selected Player"
teleportButton.Parent = mainFrame

-- Add a corner radius to the teleport button
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 5)
buttonCorner.Parent = teleportButton

-- Initially hide teleport button if minimized
teleportButton.Visible = true

-- Toggle the minimization on button click
minimizeButton.MouseButton1Click:Connect(function()
    if minimized then
        mainFrame.Size = originalSize
        contentFrame.Visible = true
        teleportButton.Visible = true -- Show teleport button
        minimized = false
    else
        mainFrame.Size = UDim2.new(originalSize.X.Scale, originalSize.X.Offset, 0, 40)
        contentFrame.Visible = false
        teleportButton.Visible = false -- Hide teleport button
        minimized = true
    end
end)

-- Exit button functionality
exitButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Add a UIListLayout to the content frame to organize player names
local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 10)
layout.FillDirection = Enum.FillDirection.Vertical
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = contentFrame

-- Selected player tracking
local selectedPlayer = nil

-- Function to create a label for each player
local function createPlayerLabel(player)
    local playerLabel = Instance.new("TextButton")
    playerLabel.Size = UDim2.new(1, 0, 0, 30) -- Adjust height as needed
    playerLabel.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    playerLabel.BorderSizePixel = 0
    playerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    playerLabel.Font = Enum.Font.SourceSans
    playerLabel.TextSize = 20
    playerLabel.Text = player.Name
    playerLabel.Parent = contentFrame

    -- Add a corner radius to the player label
    local labelCorner = Instance.new("UICorner")
    labelCorner.CornerRadius = UDim.new(0, 5)
    labelCorner.Parent = playerLabel

    -- Toggle selection state on click
    playerLabel.MouseButton1Click:Connect(function()
        if selectedPlayer == player then
            playerLabel.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
            selectedPlayer = nil
        else
            -- Reset all other labels' colors
            for _, child in ipairs(contentFrame:GetChildren()) do
                if child:IsA("TextButton") then
                    child.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
                end
            end
            playerLabel.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            selectedPlayer = player
        end
    end)

    return playerLabel
end

-- Function to update the player list
local function updatePlayerList()
    -- Clear existing labels
    for _, child in ipairs(contentFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end

    -- Add a label for each player
    for _, player in ipairs(game.Players:GetPlayers()) do
        createPlayerLabel(player)
    end

    -- Adjust contentFrame size to leave space for the teleport button
    contentFrame.Size = UDim2.new(1, 0, 1, -80)
end

-- Initial update of the player list
updatePlayerList()

-- Update the list when a player joins or leaves
game.Players.PlayerAdded:Connect(function(player)
    updatePlayerList()
end)

game.Players.PlayerRemoving:Connect(function(player)
    updatePlayerList()
end)

-- Teleport button functionality
teleportButton.MouseButton1Click:Connect(function()
    if selectedPlayer then
        local character = game.Players.LocalPlayer.Character
        local targetCharacter = selectedPlayer.Character

        if character and targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position
            character:MoveTo(targetPosition)
            print("Teleporting to " .. selectedPlayer.Name)
        else
            print("Unable to teleport, target not found.")
        end
    else
        print("No player selected.")
    end
end)
