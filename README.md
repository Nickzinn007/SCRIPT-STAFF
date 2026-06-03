local Players = game:GetService("Players")
local SoundService = game:GetService("SoundService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

-- 🔊 sons
local soundAlert = Instance.new("Sound", SoundService)
soundAlert.SoundId = "rbxassetid://911342077"
soundAlert.Volume = 0.7

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

	-- novos
	["junin_brpro"] = true,
	["S7nndzz"] = true,
	["7deskk"] = true,

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

-- 🧠 estado
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

-- 🔍 INICIAR DETECTOR DIRETAMENTE
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
