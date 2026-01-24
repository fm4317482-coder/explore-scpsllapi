local tool = Instance.new("Tool")
tool.Name = "GrabTools"
tool.RequiresHandle = false
tool.Parent = game.Players.LocalPlayer:WaitForChild("Backpack")

local grabbed = nil
local weld = nil

tool.Activated:Connect(function()
    local player = game.Players.LocalPlayer
    local mouse = player:GetMouse()
    local target = mouse.Target

    if target and target.Parent:FindFirstChild("Humanoid") then
        local char = target.Parent
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local myhrp = player.Character:FindFirstChild("HumanoidRootPart")

        if hrp and myhrp then
            weld = Instance.new("WeldConstraint")
            weld.Part0 = hrp
            weld.Part1 = myhrp
            weld.Parent = hrp
            grabbed = char
        end
    end
end)

tool.Deactivated:Connect(function()
    if weld then
        weld:Destroy()
        grabbed = nil
    end
end)
