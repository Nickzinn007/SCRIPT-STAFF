local Players = game:GetService("Players")
local SoundService = game:GetService("SoundService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

-- 👑 quem pode criar key
local keyCreators = {
	["shrouldxolve32"] = true,
	["FeshGrago_9852"] = true
}

-- 🔊 sons
local soundSuccess = Instance.new("Sound", SoundService)
soundSuccess.SoundId = "rbxassetid://911342077"
soundSuccess.Volume = 1

local soundAlert = Instance.new("Sound", SoundService)
soundAlert.SoundId = "rbxassetid://911342077"
soundAlert.Volume = 0.7

-- ⏰ tempo
local EXPIRATION_TIME = 3600

-- 📋 staff (ATUALIZADA)
local staffList = {
	["suahbeybe paxta"] = true,
	["swyssous"] = true,
	["bfdhhhhuh"] = true,
	["NAYRAN224"] = true,
	["suahcu"] = true,
	["botzinho940"] = true,
	["Diego_5415"] = true,
	["lindosvaldo65"] = true,
	["felipedarcel6"] = true,
	["miguelmaisguatavo"] = true,

	-- antigos mantidos
	["yammeleite paxta"] = true,
	["suahleite paxta"] = true,
	["betoogsl"] = true,
	["joaozin823863"] = true,
	["botzin"] = true,
	["soldadocleiton"] = true,
	["careca_raspadaaa"] = true,
	["Leviamol"] = true,
	["arthzstyles"] = true,
	["21peteca"] = true,
	["usxrch"] = true,
	["mano_bute1"] = true,
	["SlowlylNotBack"] = true,
	["sptmatheus"] = true,
	["dglancaperfume"] = true,
	["t3ynzz"] = true,
	["ZVNErqguxMp"] = true
}

-- 📋 keys com tempo de expiração
local validKeys = {
	["ABC123"] = {expires = 0}, -- 0 = nunca expira
	["STAFF2026"] = {expires = 0}
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
validKeys[autoKey] = {expires = 0}
print("🔑 Key gerada:", autoKey)

-- 🧠 estado
local active = false
local startTime = 0
local currentKeyExpiration = 0
local spectating = false
local currentTarget = nil

-- 💬 NOTIFICAÇÃO
local function notify(user, message, color)
	color = color or Color3.fromRGB(0, 170, 255)
	
	local gui = Instance.new("ScreenGui")
	gui.ResetOnSpawn = false
	gui.Parent = playerGui

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 350, 0, 80)
	frame.Position = UDim2.new(1, 370, 0.05, 0)
	frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
	frame.BorderSizePixel = 0
	frame.Parent = gui
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 12)
	corner.Parent = frame
	
	local stroke = Instance.new("UIStroke")
	stroke.Color = color
	stroke.Thickness = 2
	stroke.Transparency = 0.3
	stroke.Parent = frame

	local icon = Instance.new("TextLabel")
	icon.Size = UDim2.new(0, 40, 0, 40)
	icon.Position = UDim2.new(0, 15, 0.5, 0)
	icon.AnchorPoint = Vector2.new(0, 0.5)
	icon.BackgroundTransparency = 1
	icon.Text = "🔔"
	icon.TextSize = 24
	icon.Font = Enum.Font.GothamBold
	icon.Parent = frame

	local name = Instance.new("TextLabel")
	name.Position = UDim2.new(0, 65, 0, 12)
	name.Size = UDim2.new(1, -80, 0, 20)
	name.Text = "@" .. user
	name.TextColor3 = Color3.fromRGB(150, 150, 160)
	name.BackgroundTransparency = 1
	name.Font = Enum.Font.Gotham
	name.TextSize = 12
	name.TextXAlignment = Enum.TextXAlignment.Left
	name.Parent = frame

	local msg = Instance.new("TextLabel")
	msg.Position = UDim2.new(0, 65, 0, 35)
	msg.Size = UDim2.new(1, -80, 0, 30)
	msg.Text = message
	msg.TextColor3 = Color3.fromRGB(255, 255, 255)
	msg.BackgroundTransparency = 1
	msg.Font = Enum.Font.GothamBold
	msg.TextSize = 14
	msg.TextXAlignment = Enum.TextXAlignment.Left
	msg.TextWrapped = true
	msg.Parent = frame

	frame:TweenPosition(UDim2.new(1, -370, 0.05, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quint, 0.5, true)

	task.delay(4, function()
		frame:TweenPosition(UDim2.new(1, 370, 0.05, 0), Enum.EasingDirection.In, Enum.EasingStyle.Quint, 0.4, true)
		task.wait(0.5)
		gui:Destroy()
	end)
end

-- 👁️ PAINEL DE STAFF ONLINE (MOVÍVEL E MINIMIZÁVEL)
local staffPanel
local staffListFrame
local staffCountLabel
local isPanelMinimized = false
local isDragging = false
local dragStart = nil
local startPos = nil

local function createStaffPanel()
	local gui = Instance.new("ScreenGui")
	gui.ResetOnSpawn = false
	gui.Name = "StaffPanel"
	gui.Parent = playerGui

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 240, 0, 320)
	frame.Position = UDim2.new(1, -260, 0, 20)
	frame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
	frame.BorderSizePixel = 0
	frame.Parent = gui
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 15)
	corner.Parent = frame
	
	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(60, 60, 80)
	stroke.Thickness = 1.5
	stroke.Parent = frame

	-- Header (arrastável)
	local header = Instance.new("Frame")
	header.Size = UDim2.new(1, 0, 0, 55)
	header.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
	header.BorderSizePixel = 0
	header.Parent = frame
	
	local headerCorner = Instance.new("UICorner")
	headerCorner.CornerRadius = UDim.new(0, 15)
	headerCorner.Parent = header

	local headerBottom = Instance.new("Frame")
	headerBottom.Size = UDim2.new(1, 0, 0, 15)
	headerBottom.Position = UDim2.new(0, 0, 1, -15)
	headerBottom.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
	headerBottom.BorderSizePixel = 0
	headerBottom.Parent = header

	-- Botão minimizar
	local minimizeBtn = Instance.new("TextButton")
	minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
	minimizeBtn.Position = UDim2.new(1, -35, 0, 5)
	minimizeBtn.Text = "−"
	minimizeBtn.TextSize = 20
	minimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
	minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	minimizeBtn.BorderSizePixel = 0
	minimizeBtn.Font = Enum.Font.GothamBold
	minimizeBtn.Parent = header
	
	local minCorner = Instance.new("UICorner")
	minCorner.CornerRadius = UDim.new(0, 8)
	minCorner.Parent = minimizeBtn

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -50, 0.5, 0)
	title.Position = UDim2.new(0, 10, 0, 5)
	title.Text = "👁️ STAFF ONLINE"
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.GothamBold
	title.TextSize = 14
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = header

	local countLabel = Instance.new("TextLabel")
	countLabel.Size = UDim2.new(1, -50, 0.4, 0)
	countLabel.Position = UDim2.new(0, 10, 0.55, 0)
	countLabel.Text = "0 staff detectado(s)"
	countLabel.TextColor3 = Color3.fromRGB(150, 150, 160)
	countLabel.BackgroundTransparency = 1
	countLabel.Font = Enum.Font.Gotham
	countLabel.TextSize = 10
	countLabel.TextXAlignment = Enum.TextXAlignment.Left
	countLabel.Parent = header

	-- Lista de staff
	local scrollFrame = Instance.new("ScrollingFrame")
	scrollFrame.Size = UDim2.new(1, -20, 1, -70)
	scrollFrame.Position = UDim2.new(0, 10, 0, 65)
	scrollFrame.BackgroundTransparency = 1
	scrollFrame.BorderSizePixel = 0
	scrollFrame.ScrollBarThickness = 4
	scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(60, 60, 80)
	scrollFrame.Parent = frame

	local listLayout = Instance.new("UIListLayout")
	listLayout.Padding = UDim.new(0, 6)
	listLayout.SortOrder = Enum.SortOrder.LayoutOrder
	listLayout.Parent = scrollFrame

	staffPanel = frame
	staffListFrame = scrollFrame
	staffCountLabel = countLabel
	
	-- 🎯 SISTEMA DE ARRASTE
	header.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			isDragging = true
			dragStart = input.Position
			startPos = frame.Position
			
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					isDragging = false
				end
			end)
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if isDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end)

	-- 🔽 SISTEMA DE MINIMIZAR
	minimizeBtn.MouseButton1Click:Connect(function()
		isPanelMinimized = not isPanelMinimized
		
		if isPanelMinimized then
			minimizeBtn.Text = "+"
			frame:TweenSize(
				UDim2.new(0, 240, 0, 55),
				Enum.EasingDirection.Out,
				Enum.EasingStyle.Quint,
				0.3,
				true
			)
			scrollFrame.Visible = false
		else
			minimizeBtn.Text = "−"
			frame:TweenSize(
				UDim2.new(0, 240, 0, 320),
				Enum.EasingDirection.Out,
				Enum.EasingStyle.Quint,
				0.3,
				true
			)
			scrollFrame.Visible = true
		end
	end)

	minimizeBtn.MouseEnter:Connect(function()
		minimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 190, 50)
	end)

	minimizeBtn.MouseLeave:Connect(function()
		minimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
	end)
	
	return gui
end

-- 🎯 SISTEMA DE SPECTATE
local spectateConnection

local function stopSpectate()
	if spectateConnection then
		spectateConnection:Disconnect()
		spectateConnection = nil
	end
	
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		camera.CameraSubject = player.Character.Humanoid
	end
	
	spectating = false
	currentTarget = nil
end

local function spectatePlayer(targetPlayer)
	if spectating and currentTarget == targetPlayer then
		stopSpectate()
		notify(player.Name, "Spectate desativado", Color3.fromRGB(255, 100, 100))
		return
	end
	
	stopSpectate()
	
	if targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") then
		spectating = true
		currentTarget = targetPlayer
		camera.CameraSubject = targetPlayer.Character.Humanoid
		
		notify(player.Name, "Spectando: " .. targetPlayer.Name, Color3.fromRGB(100, 255, 100))
		
		spectateConnection = targetPlayer.CharacterAdded:Connect(function(char)
			char:WaitForChild("Humanoid")
			camera.CameraSubject = char.Humanoid
		end)
	end
end

-- 🔄 ATUALIZAR LISTA DE STAFF
local function updateStaffList()
	for _, child in pairs(staffListFrame:GetChildren()) do
		if child:IsA("Frame") then
			child:Destroy()
		end
	end
	
	local count = 0
	
	for _, p in pairs(Players:GetPlayers()) do
		if staffList[p.DisplayName] or staffList[p.Name] then
			count = count + 1
			
			local entry = Instance.new("Frame")
			entry.Size = UDim2.new(1, 0, 0, 48)
			entry.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
			entry.BorderSizePixel = 0
			entry.Parent = staffListFrame
			
			local entryCorner = Instance.new("UICorner")
			entryCorner.CornerRadius = UDim.new(0, 10)
			entryCorner.Parent = entry
			
			local entryStroke = Instance.new("UIStroke")
			entryStroke.Color = Color3.fromRGB(40, 40, 60)
			entryStroke.Thickness = 1
			entryStroke.Parent = entry
			
			local statusDot = Instance.new("Frame")
			statusDot.Size = UDim2.new(0, 6, 0, 6)
			statusDot.Position = UDim2.new(0, 10, 0.5, 0)
			statusDot.AnchorPoint = Vector2.new(0, 0.5)
			statusDot.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
			statusDot.BorderSizePixel = 0
			statusDot.Parent = entry
			
			local dotCorner = Instance.new("UICorner")
			dotCorner.CornerRadius = UDim.new(1, 0)
			dotCorner.Parent = statusDot
			
			local nameLabel = Instance.new("TextLabel")
			nameLabel.Size = UDim2.new(1, -65, 0, 18)
			nameLabel.Position = UDim2.new(0, 24, 0, 6)
			nameLabel.Text = p.Name
			nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			nameLabel.BackgroundTransparency = 1
			nameLabel.Font = Enum.Font.GothamBold
			nameLabel.TextSize = 11
			nameLabel.TextXAlignment = Enum.TextXAlignment.Left
			nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
			nameLabel.Parent = entry
			
			local displayLabel = Instance.new("TextLabel")
			displayLabel.Size = UDim2.new(1, -65, 0, 14)
			displayLabel.Position = UDim2.new(0, 24, 0, 26)
			displayLabel.Text = "@" .. p.DisplayName
			displayLabel.TextColor3 = Color3.fromRGB(120, 120, 140)
			displayLabel.BackgroundTransparency = 1
			displayLabel.Font = Enum.Font.Gotham
			displayLabel.TextSize = 9
			displayLabel.TextXAlignment = Enum.TextXAlignment.Left
			displayLabel.TextTruncate = Enum.TextTruncate.AtEnd
			displayLabel.Parent = entry
			
			local spectateBtn = Instance.new("TextButton")
			spectateBtn.Size = UDim2.new(0, 38, 0, 32)
			spectateBtn.Position = UDim2.new(1, -45, 0.5, 0)
			spectateBtn.AnchorPoint = Vector2.new(0, 0.5)
			spectateBtn.Text = "👁️"
			spectateBtn.TextSize = 16
			spectateBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
			spectateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
			spectateBtn.BorderSizePixel = 0
			spectateBtn.Font = Enum.Font.GothamBold
			spectateBtn.Parent = entry
			
			local btnCorner = Instance.new("UICorner")
			btnCorner.CornerRadius = UDim.new(0, 8)
			btnCorner.Parent = spectateBtn
			
			spectateBtn.MouseButton1Click:Connect(function()
				spectatePlayer(p)
				
				for _, e in pairs(staffListFrame:GetChildren()) do
					if e:IsA("Frame") then
						local btn = e:FindFirstChildOfClass("TextButton")
						if btn then
							btn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
						end
					end
				end
				
				if currentTarget == p and spectating then
					spectateBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
				end
			end)
			
			spectateBtn.MouseEnter:Connect(function()
				if currentTarget ~= p or not spectating then
					spectateBtn.BackgroundColor3 = Color3.fromRGB(20, 190, 255)
				end
			end)
			
			spectateBtn.MouseLeave:Connect(function()
				if currentTarget == p and spectating then
					spectateBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
				else
					spectateBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
				end
			end)
		end
	end
	
	staffCountLabel.Text = count .. " staff detectado(s)"
	staffListFrame.CanvasSize = UDim2.new(0, 0, 0, staffListFrame.UIListLayout.AbsoluteContentSize.Y)
end

-- 🔍 detector
local function startDetector()
	createStaffPanel()
	updateStaffList()
	
	for _, p in pairs(Players:GetPlayers()) do
		if staffList[p.DisplayName] or staffList[p.Name] then
			notify(p.Name, "staff entrou 👀", Color3.fromRGB(255, 100, 100))
			soundAlert:Play()
		end
	end

	Players.PlayerAdded:Connect(function(p)
		task.wait(0.5)
		if staffList[p.DisplayName] or staffList[p.Name] then
			notify(p.Name, "staff entrou 👀", Color3.fromRGB(255, 100, 100))
			soundAlert:Play()
			updateStaffList()
		end
	end)
	
	Players.PlayerRemoving:Connect(function(p)
		if staffList[p.DisplayName] or staffList[p.Name] then
			notify(p.Name, "staff saiu 👋", Color3.fromRGB(100, 255, 100))
			if currentTarget == p then
				stopSpectate()
			end
			updateStaffList()
		end
	end)
end

-- 🎨 GUI DE KEY - VERSÃO COM ABA DE CRIAÇÃO
print("Criando GUI de KEY...")

local gui = Instance.new("ScreenGui")
gui.Name = "KeySystem"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

print("ScreenGui criado:", gui.Name)

-- Fundo escuro
local bg = Instance.new("Frame")
bg.Name = "Background"
bg.Size = UDim2.new(1, 0, 1, 0)
bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
bg.BackgroundTransparency = 0.5
bg.BorderSizePixel = 0
bg.Parent = gui

print("Background criado")

-- Frame principal (maior para acomodar as abas)
local frame = Instance.new("Frame")
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 500, 0, 400)
frame.Position = UDim2.new(0.5, -250, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(18, 18, 25)
frame.BorderSizePixel = 0
frame.Parent = gui

print("MainFrame criado")

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 20)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(60, 60, 80)
stroke.Thickness = 2
stroke.Parent = frame

-- Header
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 80)
header.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
header.BorderSizePixel = 0
header.Parent = frame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 20)
headerCorner.Parent = header

local headerBottom = Instance.new("Frame")
headerBottom.Size = UDim2.new(1, 0, 0, 20)
headerBottom.Position = UDim2.new(0, 0, 1, -20)
headerBottom.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
headerBottom.BorderSizePixel = 0
headerBottom.Parent = header

local icon = Instance.new("TextLabel")
icon.Size = UDim2.new(0, 50, 0, 50)
icon.Position = UDim2.new(0, 20, 0.5, -25)
icon.BackgroundTransparency = 1
icon.Text = "🔐"
icon.TextSize = 32
icon.Font = Enum.Font.GothamBold
icon.Parent = header

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -80, 0, 25)
title.Position = UDim2.new(0, 80, 0, 15)
title.Text = "SISTEMA DE AUTENTICAÇÃO"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -80, 0, 20)
subtitle.Position = UDim2.new(0, 80, 0, 45)
subtitle.Text = "Insira sua key de acesso"
subtitle.BackgroundTransparency = 1
subtitle.TextColor3 = Color3.fromRGB(200, 230, 255)
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = header

-- 🔥 SISTEMA DE ABAS
local tabsContainer = Instance.new("Frame")
tabsContainer.Size = UDim2.new(1, -40, 0, 40)
tabsContainer.Position = UDim2.new(0, 20, 0, 90)
tabsContainer.BackgroundTransparency = 1
tabsContainer.Parent = frame

local loginTab = Instance.new("TextButton")
loginTab.Size = UDim2.new(0.48, 0, 1, 0)
loginTab.Position = UDim2.new(0, 0, 0, 0)
loginTab.Text = "🔓 ENTRAR"
loginTab.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
loginTab.TextColor3 = Color3.fromRGB(255, 255, 255)
loginTab.Font = Enum.Font.GothamBold
loginTab.TextSize = 14
loginTab.BorderSizePixel = 0
loginTab.Parent = tabsContainer

local loginTabCorner = Instance.new("UICorner")
loginTabCorner.CornerRadius = UDim.new(0, 10)
loginTabCorner.Parent = loginTab

local createTab = Instance.new("TextButton")
createTab.Size = UDim2.new(0.48, 0, 1, 0)
createTab.Position = UDim2.new(0.52, 0, 0, 0)
createTab.Text = "🔑 CRIAR KEY"
createTab.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
createTab.TextColor3 = Color3.fromRGB(150, 150, 170)
createTab.Font = Enum.Font.GothamBold
createTab.TextSize = 14
createTab.BorderSizePixel = 0
createTab.Parent = tabsContainer

local createTabCorner = Instance.new("UICorner")
createTabCorner.CornerRadius = UDim.new(0, 10)
createTabCorner.Parent = createTab

-- 📄 ABA DE LOGIN
local loginPage = Instance.new("Frame")
loginPage.Size = UDim2.new(1, 0, 1, -140)
loginPage.Position = UDim2.new(0, 0, 0, 140)
loginPage.BackgroundTransparency = 1
loginPage.Visible = true
loginPage.Parent = frame

-- Container do input
local inputContainer = Instance.new("Frame")
inputContainer.Size = UDim2.new(0, 440, 0, 50)
inputContainer.Position = UDim2.new(0.5, -220, 0, 20)
inputContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
inputContainer.BorderSizePixel = 0
inputContainer.Parent = loginPage

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 12)
inputCorner.Parent = inputContainer

local inputStroke = Instance.new("UIStroke")
inputStroke.Color = Color3.fromRGB(60, 60, 80)
inputStroke.Thickness = 1.5
inputStroke.Parent = inputContainer

-- TextBox
local box = Instance.new("TextBox")
box.Name = "KeyInput"
box.Size = UDim2.new(1, -20, 1, 0)
box.Position = UDim2.new(0, 10, 0, 0)
box.PlaceholderText = "STAFF-XXXX-XXXX"
box.PlaceholderColor3 = Color3.fromRGB(100, 100, 120)
box.Text = ""
box.BackgroundTransparency = 1
box.TextColor3 = Color3.fromRGB(255, 255, 255)
box.Font = Enum.Font.GothamMedium
box.TextSize = 16
box.ClearTextOnFocus = false
box.TextXAlignment = Enum.TextXAlignment.Left
box.Parent = inputContainer

print("TextBox criado:", box.Name)

-- Label de feedback
local keyLabel = Instance.new("TextLabel")
keyLabel.Size = UDim2.new(0, 440, 0, 30)
keyLabel.Position = UDim2.new(0.5, -220, 0, 80)
keyLabel.Text = ""
keyLabel.BackgroundTransparency = 1
keyLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
keyLabel.Font = Enum.Font.GothamBold
keyLabel.TextSize = 12
keyLabel.Parent = loginPage

-- Botão confirmar
local btn = Instance.new("TextButton")
btn.Name = "ConfirmButton"
btn.Size = UDim2.new(0, 200, 0, 50)
btn.Position = UDim2.new(0.5, -100, 1, -65)
btn.Text = "CONFIRMAR"
btn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 14
btn.BorderSizePixel = 0
btn.Parent = loginPage

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 12)
btnCorner.Parent = btn

print("Botão confirmação criado")

btn.MouseEnter:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(20, 190, 255)
end)

btn.MouseLeave:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
end)

-- 📄 ABA DE CRIAR KEY (APENAS PARA KEY CREATORS)
local createPage = Instance.new("Frame")
createPage.Size = UDim2.new(1, 0, 1, -140)
createPage.Position = UDim2.new(0, 0, 0, 140)
createPage.BackgroundTransparency = 1
createPage.Visible = false
createPage.Parent = frame

-- Verificar se o usuário pode criar keys
if keyCreators[player.Name] then
	-- Label de instruções
	local instructionLabel = Instance.new("TextLabel")
	instructionLabel.Size = UDim2.new(1, -60, 0, 30)
	instructionLabel.Position = UDim2.new(0, 30, 0, 10)
	instructionLabel.Text = "Defina o tempo de expiração em minutos:"
	instructionLabel.BackgroundTransparency = 1
	instructionLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
	instructionLabel.Font = Enum.Font.GothamBold
	instructionLabel.TextSize = 13
	instructionLabel.TextXAlignment = Enum.TextXAlignment.Left
	instructionLabel.Parent = createPage

	-- Container do input de tempo
	local timeInputContainer = Instance.new("Frame")
	timeInputContainer.Size = UDim2.new(0, 440, 0, 50)
	timeInputContainer.Position = UDim2.new(0.5, -220, 0, 50)
	timeInputContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
	timeInputContainer.BorderSizePixel = 0
	timeInputContainer.Parent = createPage

	local timeInputCorner = Instance.new("UICorner")
	timeInputCorner.CornerRadius = UDim.new(0, 12)
	timeInputCorner.Parent = timeInputContainer

	local timeInputStroke = Instance.new("UIStroke")
	timeInputStroke.Color = Color3.fromRGB(60, 60, 80)
	timeInputStroke.Thickness = 1.5
	timeInputStroke.Parent = timeInputContainer

	-- TextBox para minutos
	local timeBox = Instance.new("TextBox")
	timeBox.Name = "TimeInput"
	timeBox.Size = UDim2.new(1, -20, 1, 0)
	timeBox.Position = UDim2.new(0, 10, 0, 0)
	timeBox.PlaceholderText = "60 (minutos) - 0 = nunca expira"
	timeBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 120)
	timeBox.Text = "60"
	timeBox.BackgroundTransparency = 1
	timeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	timeBox.Font = Enum.Font.GothamMedium
	timeBox.TextSize = 16
	timeBox.ClearTextOnFocus = false
	timeBox.TextXAlignment = Enum.TextXAlignment.Left
	timeBox.Parent = timeInputContainer

	-- Container da key gerada
	local generatedKeyContainer = Instance.new("Frame")
	generatedKeyContainer.Size = UDim2.new(0, 440, 0, 50)
	generatedKeyContainer.Position = UDim2.new(0.5, -220, 0, 120)
	generatedKeyContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
	generatedKeyContainer.BorderSizePixel = 0
	generatedKeyContainer.Parent = createPage

	local generatedKeyCorner = Instance.new("UICorner")
	generatedKeyCorner.CornerRadius = UDim.new(0, 12)
	generatedKeyCorner.Parent = generatedKeyContainer

	local generatedKeyStroke = Instance.new("UIStroke")
	generatedKeyStroke.Color = Color3.fromRGB(0, 255, 100)
	generatedKeyStroke.Thickness = 2
	generatedKeyStroke.Parent = generatedKeyContainer

	-- TextBox para mostrar a key gerada
	local generatedKeyBox = Instance.new("TextBox")
	generatedKeyBox.Name = "GeneratedKey"
	generatedKeyBox.Size = UDim2.new(1, -20, 1, 0)
	generatedKeyBox.Position = UDim2.new(0, 10, 0, 0)
	generatedKeyBox.PlaceholderText = "A key aparecerá aqui..."
	generatedKeyBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 120)
	generatedKeyBox.Text = ""
	generatedKeyBox.BackgroundTransparency = 1
	generatedKeyBox.TextColor3 = Color3.fromRGB(0, 255, 150)
	generatedKeyBox.Font = Enum.Font.GothamBold
	generatedKeyBox.TextSize = 16
	generatedKeyBox.ClearTextOnFocus = false
	generatedKeyBox.TextXAlignment = Enum.TextXAlignment.Center
	generatedKeyBox.TextEditable = false
	generatedKeyBox.Parent = generatedKeyContainer

	-- Label de info
	local infoLabel = Instance.new("TextLabel")
	infoLabel.Size = UDim2.new(0, 440, 0, 40)
	infoLabel.Position = UDim2.new(0.5, -220, 0, 180)
	infoLabel.Text = ""
	infoLabel.BackgroundTransparency = 1
	infoLabel.TextColor3 = Color3.fromRGB(150, 150, 170)
	infoLabel.Font = Enum.Font.Gotham
	infoLabel.TextSize = 11
	infoLabel.TextWrapped = true
	infoLabel.Parent = createPage

	-- Botão gerar key
	local generateBtn = Instance.new("TextButton")
	generateBtn.Size = UDim2.new(0, 200, 0, 50)
	generateBtn.Position = UDim2.new(0.5, -100, 1, -65)
	generateBtn.Text = "🔑 GERAR KEY"
	generateBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
	generateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	generateBtn.Font = Enum.Font.GothamBold
	generateBtn.TextSize = 14
	generateBtn.BorderSizePixel = 0
	generateBtn.Parent = createPage

	local generateCorner = Instance.new("UICorner")
	generateCorner.CornerRadius = UDim.new(0, 12)
	generateCorner.Parent = generateBtn

	generateBtn.MouseEnter:Connect(function()
		generateBtn.BackgroundColor3 = Color3.fromRGB(20, 220, 120)
	end)

	generateBtn.MouseLeave:Connect(function()
		generateBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
	end)

	-- Ação de gerar key
	generateBtn.MouseButton1Click:Connect(function()
		local timeText = timeBox.Text
		local minutes = tonumber(timeText) or 60
		
		if minutes < 0 then
			minutes = 0
		end
		
		local newKey = generateKey()
		local expirationTime = 0
		
		if minutes > 0 then
			expirationTime = os.time() + (minutes * 60)
			infoLabel.Text = "⏰ Esta key expira em " .. minutes .. " minuto(s)"
		else
			expirationTime = 0
			infoLabel.Text = "♾️ Esta key nunca expira"
		end
		
		validKeys[newKey] = {expires = expirationTime}
		
		generatedKeyBox.Text = newKey
		notify(player.Name, "Key criada com sucesso! 🔥", Color3.fromRGB(0, 255, 100))
		soundSuccess:Play()
		
		-- Copiar automaticamente se possível
		if setclipboard then
			setclipboard(newKey)
			notify(player.Name, "Key copiada! 📋", Color3.fromRGB(100, 200, 255))
		end
	end)
else
	-- Mensagem para quem não pode criar keys
	local noAccessLabel = Instance.new("TextLabel")
	noAccessLabel.Size = UDim2.new(1, -60, 1, -60)
	noAccessLabel.Position = UDim2.new(0, 30, 0, 30)
	noAccessLabel.Text = "🔒 Você não tem permissão para criar keys.\n\nApenas usuários autorizados podem acessar esta função."
	noAccessLabel.BackgroundTransparency = 1
	noAccessLabel.TextColor3 = Color3.fromRGB(200, 100, 100)
	noAccessLabel.Font = Enum.Font.GothamBold
	noAccessLabel.TextSize = 14
	noAccessLabel.TextWrapped = true
	noAccessLabel.Parent = createPage
end

-- Sistema de trocar abas
loginTab.MouseButton1Click:Connect(function()
	loginPage.Visible = true
	createPage.Visible = false
	
	loginTab.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
	loginTab.TextColor3 = Color3.fromRGB(255, 255, 255)
	
	createTab.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
	createTab.TextColor3 = Color3.fromRGB(150, 150, 170)
	
	subtitle.Text = "Insira sua key de acesso"
end)

createTab.MouseButton1Click:Connect(function()
	loginPage.Visible = false
	createPage.Visible = true
	
	createTab.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
	createTab.TextColor3 = Color3.fromRGB(255, 255, 255)
	
	loginTab.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
	loginTab.TextColor3 = Color3.fromRGB(150, 150, 170)
	
	subtitle.Text = "Gere keys personalizadas"
end)

-- Botão fechar
local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 35, 0, 35)
close.Position = UDim2.new(1, -40, 0, 5)
close.Text = "✕"
close.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
close.TextColor3 = Color3.fromRGB(255, 255, 255)
close.Font = Enum.Font.GothamBold
close.TextSize = 18
close.BorderSizePixel = 0
close.Parent = frame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 10)
closeCorner.Parent = close

close.MouseEnter:Connect(function()
	close.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
end)

close.MouseLeave:Connect(function()
	close.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
end)

close.MouseButton1Click:Connect(function()
	gui:Destroy()
end)

-- VALIDAÇÃO DE KEY COM VERIFICAÇÃO DE EXPIRAÇÃO
btn.MouseButton1Click:Connect(function()
	local inputText = box.Text
	print("Tentando validar key:", inputText)
	
	if validKeys[inputText] then
		local keyData = validKeys[inputText]
		local currentTime = os.time()
		
		-- Verificar se a key expirou
		if keyData.expires > 0 and currentTime > keyData.expires then
			print("Key expirada!")
			notify(player.Name, "Esta key já expirou! ⏰", Color3.fromRGB(255, 150, 0))
			return
		end
		
		print("Key válida!")
		active = true
		startTime = currentTime
		currentKeyExpiration = keyData.expires
		
		-- Calcular tempo restante
		if keyData.expires > 0 then
			local timeLeft = keyData.expires - currentTime
			local minutesLeft = math.floor(timeLeft / 60)
			notify(player.Name, "Acesso liberado por " .. minutesLeft .. " minuto(s)! 🚀", Color3.fromRGB(0, 255, 100))
		else
			notify(player.Name, "Acesso liberado permanentemente! 🚀", Color3.fromRGB(0, 255, 100))
		end
		
		soundSuccess:Play()
		
		gui:Destroy()
		startDetector()
	else
		print("Key inválida!")
		notify(player.Name, "Key inválida! ❌", Color3.fromRGB(255, 100, 100))
		
		-- Shake effect
		local original = frame.Position
		for i = 1, 6 do
			frame.Position = UDim2.new(0.5, -250 + math.random(-10, 10), 0.5, -200)
			task.wait(0.05)
		end
		frame.Position = original
	end
end)

-- ENTER para confirmar
box.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		btn.MouseButton1Click:Fire()
	end
end)

print("GUI totalmente criado! Deveria estar visível agora.")

-- Expiração baseada no tempo da key
task.spawn(function()
	while true do
		task.wait(10)
		if active then
			local currentTime = os.time()
			
			-- Se a key tem expiração (não é 0)
			if currentKeyExpiration > 0 then
				if currentTime > currentKeyExpiration then
					active = false
					notify(player.Name, "Sua key expirou! ⏳", Color3.fromRGB(255, 150, 0))
					stopSpectate()
					
					-- Fechar painel de staff
					if playerGui:FindFirstChild("StaffPanel") then
						playerGui.StaffPanel:Destroy()
					end
				end
			end
		end
	end
end)
