--[[ SCRIPT UNIVERSAL DE DEBUG PARA ROBLOX
Inclui:
- ESP com distância
- Lista rolável de jogadores para teleporte
- Infinite Jump
- Noclip
- Auto Coletar Moedas
- Speed com controle de velocidade
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- UI Principal
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DebugGui"

local mainFrame = Instance.new("Frame", gui)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.Size = UDim2.new(0, 230, 0, 300)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.Visible = false

local toggleGuiBtn = Instance.new("TextButton", gui)
toggleGuiBtn.Position = UDim2.new(0, 10, 0, 10)
toggleGuiBtn.Size = UDim2.new(0, 120, 0, 30)
toggleGuiBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleGuiBtn.Text = "Abrir Interface"
toggleGuiBtn.TextColor3 = Color3.new(1,1,1)
toggleGuiBtn.Font = Enum.Font.SourceSansBold
toggleGuiBtn.TextSize = 14

toggleGuiBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
	toggleGuiBtn.Text = mainFrame.Visible and "Fechar Interface" or "Abrir Interface"
end)

local function createButton(text)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -10, 0, 30)
	btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextSize = 14
	btn.Text = text
	return btn
end

local scroll = Instance.new("ScrollingFrame", mainFrame)
scroll.Position = UDim2.new(0, 0, 0, 0)
scroll.Size = UDim2.new(1, 0, 1, 0)
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.ScrollBarThickness = 6
scroll.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", scroll)
layout.Padding = UDim.new(0, 4)
layout.SortOrder = Enum.SortOrder.LayoutOrder

-- Funções
local infJump = false
local noclip = false
local espOn = false
local autoCollect = false
local espObjects = {}
local speedEnabled = false
local speedValue = 50

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

-- Botões
local espBtn = createButton("ESP: OFF")
espBtn.MouseButton1Click:Connect(function()
	espOn = not espOn
	espBtn.Text = "ESP: " .. (espOn and "ON" or "OFF")
	clearESP()
	if espOn then
		for _, p in pairs(Players:GetPlayers()) do
			addESP(p)
		end
	end
end)
espBtn.Parent = scroll

local infJumpBtn = createButton("Infinite Jump: OFF")
infJumpBtn.MouseButton1Click:Connect(function()
	infJump = not infJump
	infJumpBtn.Text = "Infinite Jump: " .. (infJump and "ON" or "OFF")
end)
infJumpBtn.Parent = scroll

UIS.JumpRequest:Connect(function()
	if infJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
	end
end)

local noclipBtn = createButton("Noclip: OFF")
noclipBtn.MouseButton1Click:Connect(function()
	noclip = not noclip
	noclipBtn.Text = "Noclip: " .. (noclip and "ON" or "OFF")
end)
noclipBtn.Parent = scroll

RunService.Stepped:Connect(function()
	if noclip and LocalPlayer.Character then
		for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
			end
		end
	end
end)

-- Auto coletar moedas
local collectBtn = createButton("Auto Coletar Moedas: OFF")
collectBtn.MouseButton1Click:Connect(function()
	autoCollect = not autoCollect
	collectBtn.Text = "Auto Coletar Moedas: " .. (autoCollect and "ON" or "OFF")
end)
collectBtn.Parent = scroll

task.spawn(function()
	while true do
		task.wait(0.5)
		if autoCollect and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
			for _, coin in pairs(workspace:GetDescendants()) do
				if coin:IsA("BasePart") and coin.Name:lower():find("coin") then
					LocalPlayer.Character.HumanoidRootPart.CFrame = coin.CFrame + Vector3.new(0, 2, 0)
					task.wait(0.2)
				end
			end
		end
	end
end)

-- Speed
local speedBtn = createButton("Speed: OFF")
speedBtn.MouseButton1Click:Connect(function()
	speedEnabled = not speedEnabled
	speedBtn.Text = "Speed: " .. (speedEnabled and "ON" or "OFF")
end)
speedBtn.Parent = scroll

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(1, -10, 0, 30)
speedBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.Font = Enum.Font.SourceSansBold
speedBox.TextSize = 14
speedBox.Text = tostring(speedValue)
speedBox.ClearTextOnFocus = false
speedBox.PlaceholderText = "Velocidade"
speedBox.Parent = scroll

speedBox.FocusLost:Connect(function()
	local val = tonumber(speedBox.Text)
	if val then
		speedValue = val
	end
end)

RunService.Stepped:Connect(function()
	if speedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
	elseif LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
	end
end)

-- Lista de jogadores para teleporte
local tpBtn = createButton("Mostrar Jogadores")
tpBtn.MouseButton1Click:Connect(function()
	for _, c in pairs(scroll:GetChildren()) do
		if c:IsA("TextButton") and not (c.Text:find(":") or c.Text:find("ON") or c.Text:find("OFF")) then
			c:Destroy()
		end
	end
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local dist = "N/A"
			if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
				local d = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
				dist = tostring(math.floor(d)) .. "m"
			end
			local playerBtn = createButton(p.Name .. " (" .. dist .. ")")
			playerBtn.MouseButton1Click:Connect(function()
				LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame + Vector3.new(2, 0, 0)
			end)
			playerBtn.Parent = scroll
		end
	end
end)
tpBtn.Parent = scroll

-- Ajustar scroll
RunService.RenderStepped:Connect(function()
	scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
end)
