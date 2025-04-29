local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local teleportEvent = ReplicatedStorage:WaitForChild("TeleportToPlayer")

teleportEvent.OnServerEvent:Connect(function(requestingPlayer, targetName, mode)
	local targetPlayer = Players:FindFirstChild(targetName)
	if not (targetPlayer and targetPlayer.Character and requestingPlayer.Character) then return end

	local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
	local requesterHRP = requestingPlayer.Character:FindFirstChild("HumanoidRootPart")
	if not (targetHRP and requesterHRP) then return end

	if mode == "Goto" then
		requesterHRP.CFrame = targetHRP.CFrame + Vector3.new(3, 0, 0) -- Ir até jogador
	elseif mode == "Bring" then
		targetHRP.CFrame = requesterHRP.CFrame + Vector3.new(3, 0, 0) -- Puxar jogador até você
	end
end)
