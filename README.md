-- CONFIGURAÇÕES DE SEGURANÇA
local allowedUserId = 6026315852 -- Substitua com seu UserId real
local senhaCorreta = "12345" -- Senha que você definirá
local dataExpiracao = "2025-05-02 23:59:00" -- Formato: AAAA-MM-DD HH:MM:SS

local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Verifica UserId
if player.UserId ~= allowedUserId then
	player:Kick("Acesso negado. Você não está autorizado.")
	return
end

-- Interface de Login
local guiLogin = Instance.new("ScreenGui", game.CoreGui)
guiLogin.Name = "LoginDebug"

local frame = Instance.new("Frame", guiLogin)
frame.Size = UDim2.new(0, 300, 0, 150)
frame.Position = UDim2.new(0.5, -150, 0.5, -75)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)

local senhaBox = Instance.new("TextBox", frame)
senhaBox.Size = UDim2.new(0.8, 0, 0, 40)
senhaBox.Position = UDim2.new(0.1, 0, 0.2, 0)
senhaBox.PlaceholderText = "Digite a senha"
senhaBox.Text = ""
senhaBox.TextSize = 18
senhaBox.Font = Enum.Font.SourceSansBold
senhaBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
senhaBox.TextColor3 = Color3.new(1,1,1)

local aviso = Instance.new("TextLabel", frame)
aviso.Size = UDim2.new(1, 0, 0, 20)
aviso.Position = UDim2.new(0, 0, 0.6, 0)
aviso.BackgroundTransparency = 1
aviso.TextColor3 = Color3.new(1,0,0)
aviso.Text = ""
aviso.TextSize = 14
aviso.Font = Enum.Font.SourceSansBold

local botao = Instance.new("TextButton", frame)
botao.Size = UDim2.new(0.5, 0, 0, 30)
botao.Position = UDim2.new(0.25, 0, 0.8, 0)
botao.Text = "Entrar"
botao.TextSize = 16
botao.Font = Enum.Font.SourceSansBold
botao.BackgroundColor3 = Color3.fromRGB(60,60,60)
botao.TextColor3 = Color3.new(1,1,1)

-- Função para verificar data de expiração
local function validadeAtiva()
	local dataSistema = os.date("!*t")
	local sistemaTime = os.time(dataSistema)

	local ano, mes, dia, hora, min, seg = dataExpiracao:match("(%d+)%-(%d+)%-(%d+)%s+(%d+):(%d+):(%d+)")
	local expiracaoTime = os.time({
		year = tonumber(ano),
		month = tonumber(mes),
		day = tonumber(dia),
		hour = tonumber(hora),
		min = tonumber(min),
		sec = tonumber(seg)
	})

	return sistemaTime <= expiracaoTime
end

-- Ação do botão
botao.MouseButton1Click:Connect(function()
	if senhaBox.Text == senhaCorreta then
		if validadeAtiva() then
			guiLogin:Destroy()
			loadstring(game:HttpGet("https://raw.githubusercontent.com/Icarodesenna/testes/refs/heads/main/README.md"))() -- Você pode carregar o script principal aqui
		else
			aviso.Text = "Senha expirada."
		end
	else
		aviso.Text = "Senha incorreta."
	end
end)
