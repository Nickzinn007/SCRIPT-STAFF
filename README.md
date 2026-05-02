local Players = game:GetService("Players")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer

-- 👑 quem pode criar key
local keyCreators = {
	["mirezolve"] = true
}

-- 🔊 som
local sound = Instance.new("Sound", SoundService)
sound.SoundId = "rbxassetid://911342077"
sound.Volume = 1

-- ⏰ tempo
local EXPIRATION_TIME = 3600

-- 📋 staff
local staffList = {
	["yammeleite paxta"] = true,
	["suahleite paxta"] = true,
	["swyssous"] = true,
	["betoogsl"] = true,
	["joaozin823863"] = true,
	["botzin"] = true,
	["soldadocleiton"] = true,
	["careca_raspadaaa"] = true,
	["Leviamol"] = true,
	["arthzstyles"] = true,
	["21peteca"] = true,
	["usxrch"] = true,
	["mano_bute1"] = true
	["Leviamol"] = true
}

-- 📋 keys
local validKeys = {
	["ABC123"] = true,
	["STAFF2026"] = true
}

-- 🔑 gerar key
math.randomseed(os.time())

local function generateKey()
	local chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

	local function part()
		local p = ""
		for i = 1, 4 do
			local index = math.random(1, #chars)
			p = p .. chars:sub(index, index)
		end
		return p
	end

	return "STAFF-" .. part() .. "-" .. part()
end

local autoKey = generateKey()
validKeys[autoKey] = true
print("🔑 Key gerada:", autoKey)

-- 🧠 estado
local active = false
local startTime = 0

-- 💬 NOTIFICAÇÃO
local function notify(user, message)
	local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

	local frame = Instance.new("Frame", gui)
	frame.Size = UDim2.new(0, 300, 0, 100)
	frame.Position = UDim2.new(1, 320, 0.1, 0)
	frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
	Instance.new("UICorner", frame)

	local bar = Instance.new("Frame", frame)
	bar.Size = UDim2.new(1,0,0,4)
	bar.BackgroundColor3 = Color3.fromRGB(0,170,255)

	local name = Instance.new("TextLabel", frame)
	name.Position = UDim2.new(0.1,0,0.15,0)
	name.Size = UDim2.new(0.8,0,0.2,0)
	name.Text = "@" .. user
	name.TextColor3 = Color3.fromRGB(200,200,200)
	name.BackgroundTransparency = 1
	name.Font = Enum.Font.Gotham
	name.TextScaled = true

	local msg = Instance.new("TextLabel", frame)
	msg.Position = UDim2.new(0.1,0,0.4,0)
	msg.Size = UDim2.new(0.8,0,0.5,0)
	msg.Text = message
	msg.TextColor3 = Color3.fromRGB(255,255,255)
	msg.BackgroundTransparency = 1
	msg.Font = Enum.Font.GothamBold
	msg.TextScaled = true

	frame:TweenPosition(UDim2.new(1,-320,0.1,0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.4, true)

	task.delay(4, function()
		frame:TweenPosition(UDim2.new(1,320,0.1,0), Enum.EasingDirection.In, Enum.EasingStyle.Quad, 0.4, true)
		task.wait(0.4)
		gui:Destroy()
	end)
end

-- 🔍 detector
local function startDetector()
	for _, p in pairs(Players:GetPlayers()) do
		if staffList[p.DisplayName] or staffList[p.Name] then
			notify(p.Name, "staff entrou 👀")
			sound:Play()
		end
	end

	Players.PlayerAdded:Connect(function(p)
		if staffList[p.DisplayName] or staffList[p.Name] then
			notify(p.Name, "staff entrou 👀")
			sound:Play()
		end
	end)
end

-- 🎨 GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

local bg = Instance.new("Frame", gui)
bg.Size = UDim2.new(1,0,1,0)
bg.BackgroundTransparency = 0.4
bg.BackgroundColor3 = Color3.fromRGB(0,0,0)

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,0,0,0)
frame.Position = UDim2.new(0.5,-160,0.5,-100)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Instance.new("UICorner", frame)
Instance.new("UIStroke", frame)

frame:TweenSize(UDim2.new(0,320,0,200), Enum.EasingDirection.Out, Enum.EasingStyle.Back, 0.5, true)

-- título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0.25,0)
title.Text = "🔐 SISTEMA DE KEY"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextScaled = true

-- input
local box = Instance.new("TextBox", frame)
box.Size = UDim2.new(0.8,0,0.2,0)
box.Position = UDim2.new(0.1,0,0.3,0)
box.PlaceholderText = "Digite sua key..."
box.BackgroundColor3 = Color3.fromRGB(35,35,35)
box.TextColor3 = Color3.fromRGB(255,255,255)
Instance.new("UICorner", box)

-- label da key criada
local keyLabel = Instance.new("TextLabel", frame)
keyLabel.Size = UDim2.new(0.9,0,0.15,0)
keyLabel.Position = UDim2.new(0.05,0,0.52,0)
keyLabel.Text = ""
keyLabel.BackgroundTransparency = 1
keyLabel.TextColor3 = Color3.fromRGB(200,200,200)
keyLabel.Font = Enum.Font.Gotham
keyLabel.TextScaled = true

-- botão confirmar
local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(0.35,0,0.2,0)
btn.Position = UDim2.new(0.6,0,0.72,0)
btn.Text = "CONFIRMAR"
btn.BackgroundColor3 = Color3.fromRGB(0,170,255)
btn.TextColor3 = Color3.fromRGB(255,255,255)
Instance.new("UICorner", btn)

-- botão criar key (SÓ PRA VOCÊ)
if keyCreators[player.Name] then
	local createBtn = Instance.new("TextButton", frame)
	createBtn.Size = UDim2.new(0.35,0,0.2,0)
	createBtn.Position = UDim2.new(0.05,0,0.72,0)
	createBtn.Text = "CRIAR KEY"
	createBtn.BackgroundColor3 = Color3.fromRGB(0,200,100)
	createBtn.TextColor3 = Color3.fromRGB(255,255,255)
	Instance.new("UICorner", createBtn)

	createBtn.MouseButton1Click:Connect(function()
		local newKey = generateKey()
		validKeys[newKey] = true

		keyLabel.Text = "🔑 " .. newKey
		notify(player.Name, "key criada 🔥")

		-- 📋 BOTÃO COPIAR KEY
		local copyBtn = Instance.new("TextButton", frame)
		copyBtn.Size = UDim2.new(0.9,0,0.1,0)
		copyBtn.Position = UDim2.new(0.05,0,0.85,0)
		copyBtn.Text = "📋 COPIAR KEY"
		copyBtn.BackgroundColor3 = Color3.fromRGB(80,80,80)
		copyBtn.TextColor3 = Color3.fromRGB(255,255,255)
		Instance.new("UICorner", copyBtn)

		copyBtn.MouseButton1Click:Connect(function()
			if setclipboard then
				setclipboard(newKey)
				notify(player.Name, "key copiada 📋")
			else
				notify(player.Name, "copie manualmente: " .. newKey)
			end

			box.Text = newKey
		end)
	end)
end

-- ❌ fechar
local close = Instance.new("TextButton", frame)
close.Size = UDim2.new(0,30,0,30)
close.Position = UDim2.new(1,-35,0,5)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(200,60,60)
close.TextColor3 = Color3.fromRGB(255,255,255)
Instance.new("UICorner", close)

close.MouseButton1Click:Connect(function()
	frame:TweenSize(UDim2.new(0,0,0,0), Enum.EasingDirection.In, Enum.EasingStyle.Back, 0.3, true)
	task.wait(0.3)
	gui:Destroy()
end)

-- validação
btn.MouseButton1Click:Connect(function()
	if validKeys[box.Text] then
		active = true
		startTime = os.time()

		notify(player.Name, "acesso liberado 🚀")
		sound:Play()

		gui:Destroy()
		startDetector()
	else
		notify(player.Name, "key inválida ❌")
	end
end)

-- expiração
task.spawn(function()
	while true do
		task.wait(10)
		if active and (os.time() - startTime > EXPIRATION_TIME) then
			active = false
			notify(player.Name, "key expirou ⏳")
		end
	end
end)
