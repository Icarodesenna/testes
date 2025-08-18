--[[ PAINEL DEBUG UNIVERSAL DE ROBLOX
Inclui:
- Painel móvel para PC e celular
- Botão para abrir/fechar painel
- TP Point alternado: teleporta entre duas coordenadas a cada 5 segundos, ativado/desativado no botão TP Point
- ESP com distância
- Infinite Jump
- Noclip
- Auto Coletar Moedas
- Speed com controle
- WalkFling
- Fly
- Lista de jogadores para teleporte (botão "Mostrar Jogadores" fixo, abre/fecha a lista)
by.icarodesenna
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- UI
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

-- Mover painel (PC e celular)
local dragging = false
local dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    local newPos = startPos + UDim2.new(0, delta.X, 0, delta.Y)
    mainFrame.Position = newPos
    toggleGuiBtn.Position = UDim2.new(newPos.X.Scale, newPos.X.Offset, newPos.Y.Scale, newPos.Y.Offset)
end

local function dragBegin(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end

toggleGuiBtn.InputBegan:Connect(dragBegin)
toggleGuiBtn.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        update(input)
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        update(input)
    end
end)

-- Criador de botões
local function createButton(text)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.Text = text
    btn.TextWrapped = true
    btn.TextXAlignment = Enum.TextXAlignment.Center
    btn.Position = UDim2.new(0, 5, 0, 0)
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

-- ESP
local espOn = false
local espObjects = {}
local espBtn = createButton("ESP: OFF")
espBtn.Parent = scroll
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

-- TP Point alternando loop
local tpPointAtivo = false
local tpPointBtn = createButton("TP Point: OFF")
tpPointBtn.Parent = scroll

local coord1 = Vector3.new(13.955657, 64.454094, -405.709747)
local coord2 = Vector3.new(37.9, 67.5, -526.5)
task.spawn(function()
    local atual = 1
    while true do
        if tpPointAtivo and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if atual == 1 then
                LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(coord1)
                atual = 2
            else
                LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(coord2)
                atual = 1
            end
        end
        task.wait(5)
    end
end)
tpPointBtn.MouseButton1Click:Connect(function()
    tpPointAtivo = not tpPointAtivo
    tpPointBtn.Text = "TP Point: " .. (tpPointAtivo and "ON" or "OFF")
end)

-- Infinite Jump
local infJump = false
local infJumpBtn = createButton("Infinite Jump: OFF")
infJumpBtn.Parent = scroll
infJumpBtn.MouseButton1Click:Connect(function()
    infJump = not infJump
    infJumpBtn.Text = "Infinite Jump: " .. (infJump and "ON" or "OFF")
end)
UIS.JumpRequest:Connect(function()
    if infJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- Noclip
local noclip = false
local noclipBtn = createButton("Noclip: OFF")
noclipBtn.Parent = scroll
noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipBtn.Text = "Noclip: " .. (noclip and "ON" or "OFF")
end)
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
local collectBtn = createButton("Auto Coletar Moedas: OFF")
collectBtn.Parent = scroll
collectBtn.MouseButton1Click:Connect(function()
    autoCollect = not autoCollect
    collectBtn.Text = "Auto Coletar Moedas: " .. (autoCollect and "ON" or "OFF")
end)
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
local speedEnabled = false
local speedValue = 50
local speedBtn = createButton("Speed: OFF")
speedBtn.Parent = scroll
speedBtn.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    speedBtn.Text = "Speed: " .. (speedEnabled and "ON" or "OFF")
end)
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

-- WalkFling
local walkflingAtivo = false
local walkflingBtn = createButton("WalkFling: OFF")
walkflingBtn.Parent = scroll
walkflingBtn.MouseButton1Click:Connect(function()
    walkflingAtivo = not walkflingAtivo
    walkflingBtn.Text = "WalkFling: " .. (walkflingAtivo and "ON" or "OFF")
    if walkflingAtivo then
        loadstring(game:HttpGet("https://raw.githubusercontent.com/0Ben1/fe/main/obf_rf6iQURzu1fqrytcnLBAvW34C9N55kS9g9G3CKz086rC47M6632sEd4ZZYB0AYgV.lua.txt"))()
    end
end)

-- WalkFling
local walkflingAtivo = false
local walkflingBtn = createButton("Fly gui v3: OFF")
walkflingBtn.MouseButton1Click:Connect(function()
	walkflingAtivo = not walkflingAtivo
	walkflingBtn.Text = "WalkFling: " .. (walkflingAtivo and "ON" or "OFF")
	if walkflingAtivo then
		loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Fly-Gui-45909"))()
	end
end)
walkflingBtn.Parent = scroll

-- Lista de Jogadores para TP (toggle)
local tpBtn = createButton("Mostrar Jogadores")
tpBtn.Name = "MostrarJogadoresBtn"
tpBtn.Parent = scroll
local function limparBotoesJogadores()
    for _, c in pairs(scroll:GetChildren()) do
        if c:IsA("TextButton") and c.Name ~= "MostrarJogadoresBtn"
        and c ~= tpPointBtn and c ~= espBtn and c ~= infJumpBtn
        and c ~= noclipBtn and c ~= collectBtn and c ~= speedBtn and c ~= walkflingBtn and c ~= flyBtn then
            c:Destroy()
        end
    end
end
local listaAberta = false
tpBtn.MouseButton1Click:Connect(function()
    if listaAberta then
        limparBotoesJogadores()
        listaAberta = false
        tpBtn.Text = "Mostrar Jogadores"
    else
        limparBotoesJogadores()
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
        listaAberta = true
        tpBtn.Text = "Fechar Jogadores"
    end
end)

-- Assinatura
local assinatura = Instance.new("TextLabel")
assinatura.Size = UDim2.new(1, -10, 0, 20)
assinatura.BackgroundTransparency = 1
assinatura.TextColor3 = Color3.fromRGB(200, 200, 200)
assinatura.Font = Enum.Font.SourceSansItalic
assinatura.TextSize = 14
assinatura.Text = "by.icarodesenna"
assinatura.TextXAlignment = Enum.TextXAlignment.Center
assinatura.Parent = scroll

RunService.RenderStepped:Connect(function()
    scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
end)
