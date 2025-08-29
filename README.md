--[[ PAINEL DEBUG UNIVERSAL DE ROBLOX (Hoss Hub)
Todas funções originais, Fly e Headsit fiéis ao seu sistema, Headsit apenas botão, TP lista dinâmica, sem frase Rayfield, título Hoss Hub
by.icarodesenna
]]

local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local Window = Rayfield:CreateWindow({
	Name = "Hoss Hub",
	LoadingTitle = "Hoss Hub",
	LoadingSubtitle = "by.icarodesenna",
	ConfigurationSaving = {Enabled = true, FolderName = "HossHubUniversal"},
	KeySystem = false,
})
local MainTab = Window:CreateTab("Funções", 4483362458)

MainTab:CreateParagraph({
	Title = "Perfil",
	Content = "Jogador: " .. LocalPlayer.Name .. " "
})

-- ESP
local espOn = false
local espObjects = {}
local function clearESP()
    for _, v in pairs(espObjects) do
        if v and v.Parent then v:Destroy() end
    end
    espObjects = {}
end
local function addESP(p)
    if p == LocalPlayer then return end
    local bb = Instance.new("BillboardGui")
    bb.Size = UDim2.new(0, 200, 0, 50)
    bb.AlwaysOnTop = true
    local lbl = Instance.new("TextLabel", bb)
    lbl.Size = UDim2.new(1,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.TextStrokeTransparency = 0
    lbl.Font = Enum.Font.SourceSansBold
    lbl.TextScaled = true
    lbl.Text = p.Name
    local function update()
        if p.Character and p.Character:FindFirstChild("Head") and p.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude) or 0
            lbl.Text = string.format("%s (%.0fm)", p.Name, dist)
            bb.Adornee = p.Character.Head
            bb.Parent = p.Character.Head
        end
    end
    update()
    RunService.RenderStepped:Connect(update)
    table.insert(espObjects, bb)
end
MainTab:CreateToggle({
	Name = "ESP",
	CurrentValue = false,
	Flag = "EspToggle",
	Callback = function(state)
		espOn = state
		clearESP()
		if espOn then
			for _, p in pairs(Players:GetPlayers()) do
				addESP(p)
			end
		end
	end
})

-- TP Point alternando loop (agora com delay de 6 segundos!!)
local tpPointAtivo = false
local tpPointCoords = {Vector3.new(13.955657, 64.454094, -405.709747), Vector3.new(37.9, 67.5, -526.5)}
local tpPointLoop = nil
MainTab:CreateToggle({
	Name = "TP Point alternando",
	CurrentValue = false,
	Flag = "TpPointToggle",
	Callback = function(state)
		tpPointAtivo = state
		if tpPointLoop then
			tpPointLoop = nil
		end
		if tpPointAtivo then
			tpPointLoop = true
			coroutine.wrap(function()
				local idx = 1
				while tpPointAtivo and tpPointLoop do
					if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
						LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(tpPointCoords[idx])
					end
					idx = idx == 1 and 2 or 1
					wait(6) -- delay de 6 segundos
				end
			end)()
		end
	end
})

-- Infinite Jump
local infJump = false
MainTab:CreateToggle({
	Name = "Infinite Jump",
	CurrentValue = false,
	Flag = "InfJumpToggle",
	Callback = function(state)
		infJump = state
	end
})
UIS.JumpRequest:Connect(function()
	if infJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
	end
end)

-- Noclip
local noclip = false
MainTab:CreateToggle({
	Name = "Noclip",
	CurrentValue = false,
	Flag = "NoclipToggle",
	Callback = function(state)
		noclip = state
	end
})
RunService.Stepped:Connect(function()
    if noclip and LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Auto Coletar Moedas
local autoCollect = false
local coinCon = nil
MainTab:CreateToggle({
	Name = "Auto Coletar Moedas",
	CurrentValue = false,
	Flag = "AutoCoinsToggle",
	Callback = function(state)
		autoCollect = state
		if coinCon then coinCon:Disconnect() coinCon = nil end
		if autoCollect then
			coinCon = RunService.RenderStepped:Connect(function()
				for _, coin in pairs(workspace:GetDescendants()) do
					if coin:IsA("BasePart") and coin.Name:lower():find("coin") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
						LocalPlayer.Character.HumanoidRootPart.CFrame = coin.CFrame + Vector3.new(0, 2, 0)
						wait(0.2)
					end
				end
			end)
		end
	end
})

-- Speed
local speedEnabled = false
local speedValue = 50
local speedCon = nil
local speedInput = MainTab:CreateInput({
	Name = "Velocidade",
	PlaceholderText = tostring(speedValue),
	RemoveTextAfterFocusLost = false,
	Callback = function(value)
		local val = tonumber(value)
		if val then speedValue = val end
		if speedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
			LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
		end
	end
})
MainTab:CreateToggle({
	Name = "Speed",
	CurrentValue = false,
	Flag = "SpeedToggle",
	Callback = function(state)
		speedEnabled = state
		if speedCon then speedCon:Disconnect() speedCon = nil end
		if speedEnabled then
			speedCon = RunService.Stepped:Connect(function()
				if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
					LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
				end
			end)
		else
			if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
				LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
			end
		end
	end
})

-- WalkFling customizado (PERSISTENTE APÓS MORRER)
local walkflinging = false
local walkflingCon
local function stopWalkFling()
    walkflinging = false
    if walkflingCon then walkflingCon:Disconnect() walkflingCon = nil end
end
local function startWalkFling(char)
    local Root = char:FindFirstChild("HumanoidRootPart")
    local Humanoid = char:FindFirstChildOfClass("Humanoid")
    if not Root or not Humanoid then return end

    Root.CanCollide = false
    Humanoid:ChangeState(11)
    walkflingCon = RunService.Heartbeat:Connect(function()
        if walkflinging and Root and Root.Parent then
            local vel = Root.Velocity
            Root.Velocity = vel * 99999999 + Vector3.new(0, 99999999, 0)
            RunService.RenderStepped:Wait()
            Root.Velocity = vel
            RunService.Stepped:Wait()
            Root.Velocity = vel + Vector3.new(0, 0.1, 0)
        end
    end)
end
local function handleCharacter(char)
    if walkflinging then
        startWalkFling(char)
    end
end
LocalPlayer.CharacterAdded:Connect(handleCharacter)
MainTab:CreateToggle({
	Name = "WalkFling (Persistente)",
	CurrentValue = false,
	Flag = "WalkFlingToggle",
	Callback = function(state)
		walkflinging = state
		if walkflinging then
			infJump = true
			if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
				LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
			end
			task.wait(0.3)
			infJump = false
			if LocalPlayer.Character then
				startWalkFling(LocalPlayer.Character)
			end
		else
			stopWalkFling()
		end
	end
})

-- FLY igual ao seu sistema (Toggle com pulso Infinite Jump)
local FLYING = false
local velocityHandlerName = "IyVelocityHandler"
local gyroHandlerName = "IyGyroHandler"
local iyflyspeed, vehicleflyspeed = 1, 1
local IsOnMobile = UIS.TouchEnabled
local QEfly = false
local mfly1, mfly2
local function getRoot(char)
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
end
local function unmobilefly(speaker)
    FLYING = false
    if mfly1 then mfly1:Disconnect() end
    if mfly2 then mfly2:Disconnect() end
    local root = getRoot(speaker.Character)
    if root then
        local vel = root:FindFirstChild(velocityHandlerName)
        local gyro = root:FindFirstChild(gyroHandlerName)
        if vel then vel:Destroy() end
        if gyro then gyro:Destroy() end
    end
end
local function mobilefly(speaker, vfly)
	unmobilefly(speaker)
	FLYING = true
	local root = getRoot(speaker.Character)
	local camera = workspace.CurrentCamera
	local v3none = Vector3.new()
	local v3zero = Vector3.new(0, 0, 0)
	local v3inf = Vector3.new(9e9, 9e9, 9e9)
	local controlModule = require(speaker.PlayerScripts:WaitForChild("PlayerModule"):WaitForChild("ControlModule"))
	local bv = Instance.new("BodyVelocity")
	bv.Name = velocityHandlerName
	bv.Parent = root
	bv.MaxForce = v3zero
	bv.Velocity = v3zero
	local bg = Instance.new("BodyGyro")
	bg.Name = gyroHandlerName
	bg.Parent = root
	bg.MaxTorque = v3inf
	bg.P = 1000
	bg.D = 50
	mfly1 = speaker.CharacterAdded:Connect(function()
		local bv = Instance.new("BodyVelocity")
		bv.Name = velocityHandlerName
		bv.Parent = root
		bv.MaxForce = v3zero
		bv.Velocity = v3zero
		local bg = Instance.new("BodyGyro")
		bg.Name = gyroHandlerName
		bg.Parent = root
		bg.MaxTorque = v3inf
		bg.P = 1000
		bg.D = 50
	end)
	mfly2 = RunService.RenderStepped:Connect(function()
		root = getRoot(speaker.Character)
		camera = workspace.CurrentCamera
		if speaker.Character:FindFirstChildWhichIsA("Humanoid") and root and root:FindFirstChild(velocityHandlerName) and root:FindFirstChild(gyroHandlerName) then
			local humanoid = speaker.Character:FindFirstChildWhichIsA("Humanoid")
			local VelocityHandler = root:FindFirstChild(velocityHandlerName)
			local GyroHandler = root:FindFirstChild(gyroHandlerName)
			VelocityHandler.MaxForce = v3inf
			GyroHandler.MaxTorque = v3inf
			if not vfly then humanoid.PlatformStand = true end
			GyroHandler.CFrame = camera.CoordinateFrame
			VelocityHandler.Velocity = v3none
			local direction = controlModule:GetMoveVector()
			if direction.X > 0 then
				VelocityHandler.Velocity = VelocityHandler.Velocity + camera.CFrame.RightVector * (direction.X * ((vfly and vehicleflyspeed or iyflyspeed) * 50))
			end
			if direction.X < 0 then
				VelocityHandler.Velocity = VelocityHandler.Velocity + camera.CFrame.RightVector * (direction.X * ((vfly and vehicleflyspeed or iyflyspeed) * 50))
			end
			if direction.Z > 0 then
				VelocityHandler.Velocity = VelocityHandler.Velocity - camera.CFrame.LookVector * (direction.Z * ((vfly and vehicleflyspeed or iyflyspeed) * 50))
			end
			if direction.Z < 0 then
				VelocityHandler.Velocity = VelocityHandler.Velocity - camera.CFrame.LookVector * (direction.Z * ((vfly and vehicleflyspeed or iyflyspeed) * 50))
			end
		end
	end)
end
local function NOFLY()
    FLYING = false
    local root = getRoot(LocalPlayer.Character)
    if root then
        local vel = root:FindFirstChild(velocityHandlerName)
        local gyro = root:FindFirstChild(gyroHandlerName)
        if vel then vel:Destroy() end
        if gyro then gyro:Destroy() end
    end
end
local function sFLY(vfly)
    NOFLY()
    wait()
    FLYING = true
    -- Pulse Infinite Jump + pulo antes de ativar
    infJump = true
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
    task.wait(0.3)
    infJump = false
    mobilefly(LocalPlayer, vfly)
end

MainTab:CreateToggle({
	Name = "Fly",
	CurrentValue = false,
	Flag = "FlyToggle",
	Callback = function(state)
		if state then
			if not IsOnMobile then sFLY() else mobilefly(LocalPlayer) end
		else
			if not IsOnMobile then NOFLY() else unmobilefly(LocalPlayer) end
			-- Pulse Infinite Jump + pulo ao sair do fly
			infJump = true
			if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
				LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
			end
			task.wait(0.3)
			infJump = false
		end
	end
})

-- HEADSIT - Botão único, lista fiel ao seu código
local headsitBtns = {}
local headSitConn = nil
local function clearHeadsit()
	for _, btn in pairs(headsitBtns) do if btn.Destroy then btn:Destroy() end end
	headsitBtns = {}
	if headSitConn then headSitConn:Disconnect() headSitConn = nil end
end
local function getRootSit(char)
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
end
local function headsitOn(targetPlayer)
	if headSitConn then headSitConn:Disconnect() end
	local speaker = LocalPlayer
	local humanoid = speaker.Character and speaker.Character:FindFirstChildOfClass("Humanoid")
	if not humanoid or not targetPlayer.Character or not getRootSit(targetPlayer.Character) then return end
	humanoid.Sit = true
	headSitConn = RunService.Heartbeat:Connect(function()
		if Players:FindFirstChild(targetPlayer.Name)
		and targetPlayer.Character and getRootSit(targetPlayer.Character)
		and getRootSit(speaker.Character) and humanoid.Sit == true then
			getRootSit(speaker.Character).CFrame = getRootSit(targetPlayer.Character).CFrame * CFrame.Angles(0, math.rad(0), 0) * CFrame.new(0, 1.6, 0.4)
		else
			if headSitConn then headSitConn:Disconnect() headSitConn = nil end
		end
	end)
end

MainTab:CreateButton({
	Name = "Headsit (Escolher jogador)",
	Callback = function()
		clearHeadsit()
		for _, p in pairs(Players:GetPlayers()) do
			if p ~= LocalPlayer and p.Character and getRootSit(p.Character) then
				local btn = MainTab:CreateButton({
					Name = "Headsit: " .. p.Name,
					Callback = function()
						clearHeadsit()
						headsitOn(p)
					end
				})
				table.insert(headsitBtns, btn)
			end
		end
	end
})

-- TP para pessoas (Toggle) - lista dinâmica COM DELAY DE 6 SEGUNDOS!!
local tpActive = false
local tpBtns = {}
local function clearTPBtns()
	for _, btn in pairs(tpBtns) do if btn.Destroy then btn:Destroy() end end
	tpBtns = {}
end
MainTab:CreateToggle({
	Name = "TP para Jogadores",
	CurrentValue = false,
	Flag = "TpPlayersToggle",
	Callback = function(state)
		tpActive = state
		clearTPBtns()
		if tpActive then
			for _, p in pairs(Players:GetPlayers()) do
				if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
					local dist = "N/A"
					if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
						local d = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
						dist = tostring(math.floor(d)) .. "m"
					end
					local btn = MainTab:CreateButton({
						Name = p.Name .. " (" .. dist .. ")",
						Callback = function()
							clearTPBtns()
							tpActive = false
							wait(6) -- delay de 6 segundos antes de teleportar!
							if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
								LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame + Vector3.new(2, 0, 0)
							end
						end
					})
					table.insert(tpBtns, btn)
				end
			end
		end
	end
})

Players.PlayerAdded:Connect(function()
	if tpActive then
		clearTPBtns()
		for _, p in pairs(Players:GetPlayers()) do
			if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
				local dist = "N/A"
				if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
					local d = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
					dist = tostring(math.floor(d)) .. "m"
				end
				local btn = MainTab:CreateButton({
					Name = p.Name .. " (" .. dist .. ")",
					Callback = function()
						clearTPBtns()
						tpActive = false
						wait(6) -- delay de 6 segundos antes de teleportar!
						if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
							LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame + Vector3.new(2, 0, 0)
						end
					end
				})
				table.insert(tpBtns, btn)
			end
		end
	end
end)
Players.PlayerRemoving:Connect(function()
	if tpActive then
		clearTPBtns()
		for _, p in pairs(Players:GetPlayers()) do
			if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
				local dist = "N/A"
				if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
					local d = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
					dist = tostring(math.floor(d)) .. "m"
				end
				local btn = MainTab:CreateButton({
					Name = p.Name .. " (" .. dist .. ")",
					Callback = function()
						clearTPBtns()
						tpActive = false
						wait(6) -- delay de 6 segundos antes de teleportar!
						if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
							LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame + Vector3.new(2, 0, 0)
						end
					end
				})
				table.insert(tpBtns, btn)
			end
		end
	end
end)

-- ProximityPrompt sem tempo
for _, p in pairs(game:GetDescendants()) do
	if p:IsA("ProximityPrompt") then
		p.HoldDuration = 0
	end
end
game.DescendantAdded:Connect(function(d)
	if d:IsA("ProximityPrompt") then
		d.HoldDuration = 0
	end
end)

MainTab:CreateParagraph({
	Title = "by.Hoss Hub",
	Content = "Painel The Flow Is Lave Hoss Hub Rayfield"
})

-- FIM
