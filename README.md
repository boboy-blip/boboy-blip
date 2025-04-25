local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- UI Setup
local gui = LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("ScreenGui")
local circle = gui:WaitForChild("TargetCircle")

-- Settings
local circleRadius = 25 -- pixels
local cooldownTime = 3 -- seconds
local lastLockTime = 0
local currentLockedPlayer = nil

-- Function to check if a point is inside the circle
local function isInCircle(screenPos)
    local centerX = circle.AbsolutePosition.X + circle.AbsoluteSize.X / 2
    local centerY = circle.AbsolutePosition.Y + circle.AbsoluteSize.Y / 2
    local dx = screenPos.X - centerX
    local dy = screenPos.Y - centerY
    return (dx * dx + dy * dy) <= (circleRadius * circleRadius)
end

-- Lock camera to a player's head
local function lockToPlayer(player)
    if player and player.Character then
        local head = player.Character:FindFirstChild("Head")
        if head then
            Camera.CameraSubject = head
            Camera.CameraType = Enum.CameraType.Custom
            currentLockedPlayer = player
            lastLockTime = tick()
        end
    end
end

-- Main loop
RunService.RenderStepped:Connect(function()
    local currentTime = tick()

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)

            if onScreen and isInCircle(Vector2.new(screenPos.X, screenPos.Y)) then
                -- Only lock if cooldown has passed and it's a new player
                if player ~= currentLockedPlayer and (currentTime - lastLockTime) >= cooldownTime then
                    lockToPlayer(player)
                end
                break
            end
        end
    end
end)
