local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "Sword*"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 310)
frame.Position = UDim2.new(0, 100, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "Sword"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextScaled = true

local minimizeBtn = Instance.new("TextButton", frame)
minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
minimizeBtn.Position = UDim2.new(1, -35, 0, 0)
minimizeBtn.Text = "-"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
minimizeBtn.TextColor3 = Color3.new(1,1,1)
minimizeBtn.Font = Enum.Font.SourceSansBold
minimizeBtn.TextScaled = true

local toggle = Instance.new("TextButton", frame)
toggle.Position = UDim2.new(0, 10, 0, 40)
toggle.Size = UDim2.new(0, 120, 0, 30)
toggle.Text = "Start Sword"
toggle.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
toggle.TextColor3 = Color3.new(1,1,1)
toggle.Font = Enum.Font.SourceSans
toggle.TextScaled = true

local delayBox = Instance.new("TextBox", frame)
delayBox.Position = UDim2.new(0, 140, 0, 40)
delayBox.Size = UDim2.new(0, 120, 0, 30)
delayBox.PlaceholderText = "Delay (s)"
delayBox.Text = "1"
delayBox.Font = Enum.Font.SourceSans
delayBox.TextScaled = true

local friendToggle = Instance.new("TextButton", frame)
friendToggle.Position = UDim2.new(0, 10, 0, 80)
friendToggle.Size = UDim2.new(1, -20, 0, 30)
friendToggle.Text = "Friend Check: OFF"
friendToggle.BackgroundColor3 = Color3.fromRGB(150, 150, 0)
friendToggle.TextColor3 = Color3.new(1,1,1)
friendToggle.Font = Enum.Font.SourceSansBold
friendToggle.TextScaled = true

local listFrame = Instance.new("ScrollingFrame", frame)
listFrame.Position = UDim2.new(0, 10, 0, 120)
listFrame.Size = UDim2.new(1, -20, 0, 180)
listFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.ScrollBarThickness = 6

local whitelist = {}
local autoKillEnabled = false
local killing = false
local isMinimized = false
local friendCheck = false

friendToggle.MouseButton1Click:Connect(function()
	friendCheck = not friendCheck
	friendToggle.Text = friendCheck and "Friend Check: ON" or "Friend Check: OFF"
	friendToggle.BackgroundColor3 = friendCheck and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(150, 150, 0)
end)

minimizeBtn.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	minimizeBtn.Text = isMinimized and "+" or "-"
	for _, child in ipairs(frame:GetChildren()) do
		if child ~= title and child ~= minimizeBtn then
			child.Visible = not isMinimized
		end
	end
	frame.Size = isMinimized and UDim2.new(0, 300, 0, 30) or UDim2.new(0, 300, 0, 310)
end)

local function updatePlayerList()
	listFrame:ClearAllChildren()
	local y = 0
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local btn = Instance.new("TextButton", listFrame)
			btn.Size = UDim2.new(1, -10, 0, 30)
			btn.Position = UDim2.new(0, 5, 0, y)
			btn.Text = plr.DisplayName .. " (@" .. plr.Name .. ")"
			btn.Font = Enum.Font.SourceSans
			btn.TextScaled = true
			btn.TextColor3 = Color3.new(1, 1, 1)
			btn.BackgroundColor3 = whitelist[plr.Name] and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(80, 80, 80)
			btn.MouseButton1Click:Connect(function()
				if whitelist[plr.Name] then
					whitelist[plr.Name] = nil
					whitelist[plr.DisplayName] = nil
					btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
				else
					whitelist[plr.Name] = true
					whitelist[plr.DisplayName] = true
					btn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
				end
			end)
			y += 35
		end
	end
	listFrame.CanvasSize = UDim2.new(0, 0, 0, y)
end

task.spawn(function()
	while true do
		if not isMinimized then
			updatePlayerList()
		end
		task.wait(2)
	end
end)

toggle.MouseButton1Click:Connect(function()
	autoKillEnabled = not autoKillEnabled
	toggle.Text = autoKillEnabled and "Stop kill all" or "Start kill all"
	toggle.BackgroundColor3 = autoKillEnabled and Color3.fromRGB(170, 0, 0) or Color3.fromRGB(0, 170, 0)
end)

local function killTarget(target)
	if not target.Character or not target.Character:FindFirstChild("Humanoid") then return end
	if whitelist[target.Name] or whitelist[target.DisplayName] then return end
	if target == LocalPlayer then return end
	if target.Character:FindFirstChildOfClass("ForceField") then return end
	if target.Character.Humanoid.Health <= 0 then return end
	if friendCheck and LocalPlayer:IsFriendsWith(target.UserId) then return end
	local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
	if tool then
		target.Character:PivotTo(LocalPlayer.Character:GetPivot() * CFrame.new(2, 0, -2))
		tool:Activate()
	end
end

task.spawn(function()
	while true do
		if autoKillEnabled then
			local char = LocalPlayer.Character
			if char then
				local tool = char:FindFirstChildOfClass("Tool")
				if tool then
					tool:Activate()
				end
			end
		end
		RunService.Heartbeat:Wait()
	end
end)

task.spawn(function()
	while true do
		if autoKillEnabled and not killing then
			killing = true
			local aliveTargets = {}
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer and not whitelist[plr.Name] and not whitelist[plr.DisplayName] then
					if friendCheck and LocalPlayer:IsFriendsWith(plr.UserId) then continue end
					if plr.Character and plr.Character:FindFirstChild("Humanoid") then
						if plr.Character.Humanoid.Health > 0 and not plr.Character:FindFirstChildOfClass("ForceField") then
							table.insert(aliveTargets, plr)
						end
					end
				end
			end
			local waveCount = 0
			for _, target in ipairs(aliveTargets) do
				if waveCount >= 3 then break end
				killTarget(target)
				waveCount += 1
				task.wait(0.15)
			end
			local delayTime = tonumber(delayBox.Text) or 1
			task.wait(delayTime)
			killing = false
		end
		RunService.Heartbeat:Wait()
	end
end)
