-- JK ADMIN AIM - INSTALLER
-- Cole este ÚNICO Script em ServerScriptService.
-- Ele cria automaticamente toda a estrutura do sistema.

local ServerScriptService = game:GetService("ServerScriptService")
local StarterPlayer = game:GetService("StarterPlayer")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

--------------------------------------------------
-- CONFIG
--------------------------------------------------

local WHITELIST = {
	-- [SEU_USER_ID] = true,
	-- [USER_ID_DO_OUTRO_ADMIN] = true,
}

--------------------------------------------------
-- REMOTE
--------------------------------------------------

local folder = ReplicatedStorage:FindFirstChild("JKAdminAim")

if not folder then
	folder = Instance.new("Folder")
	folder.Name = "JKAdminAim"
	folder.Parent = ReplicatedStorage
end

local remote = folder:FindFirstChild("Remote")

if not remote then
	remote = Instance.new("RemoteEvent")
	remote.Name = "Remote"
	remote.Parent = folder
end

--------------------------------------------------
-- SERVER CODE
--------------------------------------------------

local serverSource = [==[
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local PhysicsService = game:GetService("PhysicsService")

local folder = ReplicatedStorage:WaitForChild("JKAdminAim")
local remote = folder:WaitForChild("Remote")

local WHITELIST = {
	-- COLOQUE OS USERIDS AUTORIZADOS AQUI
	-- [123456789] = true,
}

local MAX_FLY_SPEED = 100
local MAX_AIM_DISTANCE = 1000

local state = {}

local function isAdmin(player)
	return WHITELIST[player.UserId] == true
end

local function getCharacter(player)
	local character = player.Character

	if not character then
		return
	end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local root = character:FindFirstChild("HumanoidRootPart")

	if not humanoid or not root then
		return
	end

	return character, humanoid, root
end

pcall(function()
	PhysicsService:RegisterCollisionGroup("JKAdminFly")
end)

pcall(function()
	PhysicsService:CollisionGroupSetCollidable(
		"JKAdminFly",
		"Default",
		false
	)

	PhysicsService:CollisionGroupSetCollidable(
		"JKAdminFly",
		"JKAdminFly",
		false
	)
end)

local function setCollision(player, enabled)
	local character = player.Character

	if not character then
		return
	end

	for _, obj in ipairs(character:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.CollisionGroup =
				enabled and "JKAdminFly" or "Default"
		end
	end
end

local function stopFly(player)
	local s = state[player]

	if s then
		s.fly = false
	end

	local character, humanoid, root =
		getCharacter(player)

	if humanoid then
		humanoid.PlatformStand = false
		humanoid.AutoRotate = true
	end

	if root then
		root.AssemblyLinearVelocity = Vector3.zero
		root.AssemblyAngularVelocity = Vector3.zero
	end

	setCollision(player, false)
end

local function validateTarget(player, target)
	if typeof(target) ~= "Instance" then
		return false
	end

	if not target:IsA("BasePart") then
		return false
	end

	local character =
		target:FindFirstAncestorOfClass("Model")

	if not character then
		return false
	end

	local targetPlayer =
		Players:GetPlayerFromCharacter(character)

	if not targetPlayer or targetPlayer == player then
		return false
	end

	local humanoid =
		character:FindFirstChildOfClass("Humanoid")

	local root =
		character:FindFirstChild("HumanoidRootPart")

	if not humanoid or humanoid.Health <= 0 or not root then
		return false
	end

	if player.Team ~= nil and targetPlayer.Team == player.Team then
		return false
	end

	local _, _, playerRoot = getCharacter(player)

	if not playerRoot then
		return false
	end

	if (root.Position - playerRoot.Position).Magnitude
		> MAX_AIM_DISTANCE then
		return false
	end

	local origin = playerRoot.Position
	local direction = target.Position - origin

	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {
		player.Character
	}

	local result =
		workspace:Raycast(
			origin,
			direction,
			params
		)

	if result and
		not result.Instance:IsDescendantOf(character) then
		return false
	end

	return true
end

Players.PlayerAdded:Connect(function(player)

	state[player] = {
		authorized = isAdmin(player),
		fly = false,
		lastAim = 0
	}

	player.CharacterAdded:Connect(function()

		task.wait(0.5)

		local s = state[player]

		if s then
			s.fly = false
		end

		setCollision(player, false)
	end)
end)

Players.PlayerRemoving:Connect(function(player)
	state[player] = nil
end)

remote.OnServerEvent:Connect(function(player, action, data)

	local s = state[player]

	if not s or not s.authorized then
		return
	end

	--------------------------------------------------
	-- ACESSO
	--------------------------------------------------

	if action == "RequestAccess" then
		remote:FireClient(
			player,
			"AccessGranted"
		)

		return
	end

	--------------------------------------------------
	-- FLY
	--------------------------------------------------

	if action == "Fly" then

		if typeof(data) ~= "boolean" then
			return
		end

		if data then

			local character, humanoid, root =
				getCharacter(player)

			if not character then
				return
			end

			s.fly = true

			humanoid.PlatformStand = true
			humanoid.AutoRotate = false

			setCollision(player, true)

			root.AssemblyLinearVelocity = Vector3.zero

		else
			stopFly(player)
		end

		return
	end

	--------------------------------------------------
	-- MOVIMENTO FLY
	--------------------------------------------------

	if action == "FlyMove" then

		if not s.fly then
			return
		end

		if typeof(data) ~= "table" then
			return
		end

		local direction = data.direction
		local speed = tonumber(data.speed)

		if typeof(direction) ~= "Vector3" then
			return
		end

		if not speed then
			return
		end

		speed = math.clamp(
			speed,
			1,
			MAX_FLY_SPEED
		)

		if direction.Magnitude > 1.05 then
			direction = direction.Unit
		end

		local character, humanoid, root =
			getCharacter(player)

		if not character or not humanoid or not root then
			return
		end

		if humanoid.Health <= 0 then
			stopFly(player)
			return
		end

		root.AssemblyLinearVelocity =
			direction * speed

		return
	end

	--------------------------------------------------
	-- AIM
	--------------------------------------------------

	if action == "AimTarget" then

		local now = os.clock()

		if now - s.lastAim < 1 / 30 then
			return
		end

		s.lastAim = now

		if not validateTarget(player, data) then
			return
		end

		remote:FireClient(
			player,
			"ValidAimTarget",
			data
		)

		return
	end

	--------------------------------------------------
	-- DESATIVAR
	--------------------------------------------------

	if action == "DisableAll" then

		stopFly(player)

		remote:FireClient(
			player,
			"ForceDisable"
		)

	end
end)
]==]

--------------------------------------------------
-- CLIENT CODE
--------------------------------------------------

local clientSource = [==[
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local folder =
	ReplicatedStorage:WaitForChild("JKAdminAim")

local remote =
	folder:WaitForChild("Remote")

--------------------------------------------------
-- STATE
--------------------------------------------------

local aimEnabled = false
local espEnabled = false
local flyEnabled = false

local aimFOV = 180
local aimDistance = 500
local aimSmooth = 0.18
local flySpeed = 70

local espObjects = {}

--------------------------------------------------
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "JK_ADMIN_AIM"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Enabled = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(650, 350)
main.Position = UDim2.new(
	0.5,
	-325,
	0.5,
	-175
)
main.BackgroundColor3 =
	Color3.fromRGB(15,15,20)
main.BorderSizePixel = 0
main.Parent = gui

local mainCorner =
	Instance.new("UICorner")

mainCorner.CornerRadius =
	UDim.new(0,16)

mainCorner.Parent = main

local stroke =
	Instance.new("UIStroke")

stroke.Color =
	Color3.fromRGB(145,70,255)

stroke.Thickness = 1.5
stroke.Transparency = .2
stroke.Parent = main

--------------------------------------------------
-- TITLE
--------------------------------------------------

local title =
	Instance.new("TextLabel")

title.Size =
	UDim2.new(1,-70,0,35)

title.Position =
	UDim2.fromOffset(18,5)

title.BackgroundTransparency = 1
title.Text = "JK ADMIN AIM"
title.TextColor3 =
	Color3.fromRGB(230,220,255)

title.Font =
	Enum.Font.GothamBold

title.TextSize = 21
title.TextXAlignment =
	Enum.TextXAlignment.Left

title.Parent = main

local sub =
	Instance.new("TextLabel")

sub.Size =
	UDim2.new(1,-70,0,18)

sub.Position =
	UDim2.fromOffset(20,35)

sub.BackgroundTransparency = 1
sub.Text = "AUTHORIZED ADMIN TEST SYSTEM"

sub.TextColor3 =
	Color3.fromRGB(130,130,150)

sub.Font =
	Enum.Font.Gotham

sub.TextSize = 9
sub.TextXAlignment =
	Enum.TextXAlignment.Left

sub.Parent = main

--------------------------------------------------
-- MINIMIZE
--------------------------------------------------

local minimize =
	Instance.new("TextButton")

minimize.Size =
	UDim2.fromOffset(35,35)

minimize.Position =
	UDim2.new(1,-45,0,10)

minimize.BackgroundColor3 =
	Color3.fromRGB(45,25,65)

minimize.Text = "—"

minimize.TextColor3 =
	Color3.fromRGB(240,230,255)

minimize.Font =
	Enum.Font.GothamBold

minimize.TextSize = 20
minimize.BorderSizePixel = 0
minimize.Parent = main

Instance.new("UICorner",minimize).CornerRadius =
	UDim.new(0,9)

local mini =
	Instance.new("TextButton")

mini.Size =
	UDim2.fromOffset(52,52)

mini.Position =
	UDim2.new(0,20,.5,-26)

mini.BackgroundColor3 =
	Color3.fromRGB(40,20,65)

mini.Text = "JK"

mini.TextColor3 =
	Color3.fromRGB(230,210,255)

mini.Font =
	Enum.Font.GothamBold

mini.TextSize = 17
mini.Visible = false
mini.BorderSizePixel = 0
mini.Parent = gui

Instance.new("UICorner",mini).CornerRadius =
	UDim.new(1,0)

minimize.MouseButton1Click:Connect(function()
	main.Visible = false
	mini.Visible = true
end)

mini.MouseButton1Click:Connect(function()
	main.Visible = true
	mini.Visible = false
end)

--------------------------------------------------
-- DRAG
--------------------------------------------------

local dragging = false
local dragStart
local startPosition

main.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPosition = main.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if not dragging then
		return
	end

	if input.UserInputType ~=
		Enum.UserInputType.MouseMovement
		and input.UserInputType ~=
		Enum.UserInputType.Touch then
		return
	end

	local delta =
		input.Position - dragStart

	main.Position =
		UDim2.new(
			startPosition.X.Scale,
			startPosition.X.Offset + delta.X,
			startPosition.Y.Scale,
			startPosition.Y.Offset + delta.Y
		)
end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		dragging = false
	end
end)

--------------------------------------------------
-- CARDS
--------------------------------------------------

local content =
	Instance.new("Frame")

content.Size =
	UDim2.new(1,-30,1,-80)

content.Position =
	UDim2.fromOffset(15,65)

content.BackgroundTransparency = 1
content.Parent = main

local grid =
	Instance.new("UIGridLayout")

grid.CellSize =
	UDim2.new(.31,-8,0,115)

grid.CellPadding =
	UDim2.fromOffset(10,10)

grid.Parent = content

local function card(name,description)

	local frame =
		Instance.new("Frame")

	frame.BackgroundColor3 =
		Color3.fromRGB(23,23,30)

	frame.BorderSizePixel = 0
	frame.Parent = content

	Instance.new("UICorner",frame).CornerRadius =
		UDim.new(0,13)

	local label =
		Instance.new("TextLabel")

	label.Size =
		UDim2.new(1,-20,0,25)

	label.Position =
		UDim2.fromOffset(10,7)

	label.BackgroundTransparency = 1
	label.Text = name

	label.TextColor3 =
		Color3.fromRGB(225,215,255)

	label.Font =
		Enum.Font.GothamBold

	label.TextSize = 14
	label.TextXAlignment =
		Enum.TextXAlignment.Left

	label.Parent = frame

	local descriptionLabel =
		Instance.new("TextLabel")

	descriptionLabel.Size =
		UDim2.new(1,-20,0,30)

	descriptionLabel.Position =
		UDim2.fromOffset(10,32)

	descriptionLabel.BackgroundTransparency = 1
	descriptionLabel.Text = description
	descriptionLabel.TextWrapped = true

	descriptionLabel.TextColor3 =
		Color3.fromRGB(135,135,150)

	descriptionLabel.Font =
		Enum.Font.Gotham

	descriptionLabel.TextSize = 9
	descriptionLabel.TextXAlignment =
		Enum.TextXAlignment.Left

	descriptionLabel.Parent = frame

	local button =
		Instance.new("TextButton")

	button.Size =
		UDim2.new(1,-20,0,30)

	button.Position =
		UDim2.new(0,10,1,-38)

	button.BackgroundColor3 =
		Color3.fromRGB(65,35,105)

	button.Text = "OFF"

	button.TextColor3 =
		Color3.fromRGB(240,235,255)

	button.Font =
		Enum.Font.GothamBold

	button.TextSize = 11
	button.BorderSizePixel = 0
	button.Parent = frame

	Instance.new("UICorner",button).CornerRadius =
		UDim.new(0,8)

	return button
end

local aimButton =
	card(
		"AIM ASSIST",
		"Head • FOV • Smooth • Wall Check"
	)

local espButton =
	card(
		"ESP",
		"2D Box • Health • Auto Refresh"
	)

local flyButton =
	card(
		"FLY",
		"WASD / Analog • NoClip"
	)

--------------------------------------------------
-- AIM FOV
--------------------------------------------------

local fovCard =
	Instance.new("Frame")

fovCard.BackgroundColor3 =
	Color3.fromRGB(23,23,30)

fovCard.BorderSizePixel = 0
fovCard.Parent = content

Instance.new("UICorner",fovCard).CornerRadius =
	UDim.new(0,13)

local fovTitle =
	Instance.new("TextLabel")

fovTitle.Size =
	UDim2.new(1,-20,0,25)

fovTitle.Position =
	UDim2.fromOffset(10,8)

fovTitle.BackgroundTransparency = 1
fovTitle.Text = "AIM FOV"

fovTitle.TextColor3 =
	Color3.fromRGB(225,215,255)

fovTitle.Font =
	Enum.Font.GothamBold

fovTitle.TextSize = 14
fovTitle.TextXAlignment =
	Enum.TextXAlignment.Left

fovTitle.Parent = fovCard

local fovValue =
	Instance.new("TextLabel")

fovValue.Size =
	UDim2.new(1,-20,0,30)

fovValue.Position =
	UDim2.fromOffset(10,40)

fovValue.BackgroundTransparency = 1
fovValue.Text = tostring(aimFOV)

fovValue.TextColor3 =
	Color3.fromRGB(180,130,255)

fovValue.Font =
	Enum.Font.GothamBold

fovValue.TextSize = 20
fovValue.Parent = fovCard

local fovMinus =
	Instance.new("TextButton")

fovMinus.Size =
	UDim2.fromOffset(40,28)

fovMinus.Position =
	UDim2.fromOffset(10,78)

fovMinus.Text = "-"
fovMinus.Font =
	Enum.Font.GothamBold

fovMinus.TextSize = 18
fovMinus.TextColor3 = Color3.new(1,1,1)

fovMinus.BackgroundColor3 =
	Color3.fromRGB(45,35,60)

fovMinus.BorderSizePixel = 0
fovMinus.Parent = fovCard

local fovPlus =
	fovMinus:Clone()

fovPlus.Text = "+"
fovPlus.Position =
	UDim2.new(1,-50,0,78)

fovPlus.Parent = fovCard

Instance.new("UICorner",fovMinus).CornerRadius =
	UDim.new(0,8)

Instance.new("UICorner",fovPlus).CornerRadius =
	UDim.new(0,8)

fovMinus.MouseButton1Click:Connect(function()

	aimFOV =
		math.clamp(
			aimFOV - 10,
			30,
			500
		)

	fovValue.Text =
		tostring(aimFOV)
end)

fovPlus.MouseButton1Click:Connect(function()

	aimFOV =
		math.clamp(
			aimFOV + 10,
			30,
			500
		)

	fovValue.Text =
		tostring(aimFOV)
end)

--------------------------------------------------
-- FLY SPEED
--------------------------------------------------

local speedCard =
	Instance.new("Frame")

speedCard.BackgroundColor3 =
	Color3.fromRGB(23,23,30)

speedCard.BorderSizePixel = 0
speedCard.Parent = content

Instance.new("UICorner",speedCard).CornerRadius =
	UDim.new(0,13)

local speedTitle =
	Instance.new("TextLabel")

speedTitle.Size =
	UDim2.new(1,-20,0,25)

speedTitle.Position =
	UDim2.fromOffset(10,8)

speedTitle.BackgroundTransparency = 1
speedTitle.Text = "FLY SPEED"

speedTitle.TextColor3 =
	Color3.fromRGB(225,215,255)

speedTitle.Font =
	Enum.Font.GothamBold

speedTitle.TextSize = 14
speedTitle.TextXAlignment =
	Enum.TextXAlignment.Left

speedTitle.Parent = speedCard

local speedValue =
	Instance.new("TextLabel")

speedValue.Size =
	UDim2.new(1,-20,0,30)

speedValue.Position =
	UDim2.fromOffset(10,40)

speedValue.BackgroundTransparency = 1
speedValue.Text = tostring(flySpeed)

speedValue.TextColor3 =
	Color3.fromRGB(180,130,255)

speedValue.Font =
	Enum.Font.GothamBold

speedValue.TextSize = 20
speedValue.Parent = speedCard

local speedMinus =
	Instance.new("TextButton")

speedMinus.Size =
	UDim2.fromOffset(40,28)

speedMinus.Position =
	UDim2.fromOffset(10,78)

speedMinus.Text = "-"
speedMinus.Font =
	Enum.Font.GothamBold

speedMinus.TextSize = 18
speedMinus.TextColor3 =
	Color3.new(1,1,1)

speedMinus.BackgroundColor3 =
	Color3.fromRGB(45,35,60)

speedMinus.BorderSizePixel = 0
speedMinus.Parent = speedCard

local speedPlus =
	speedMinus:Clone()

speedPlus.Text = "+"
speedPlus.Position =
	UDim2.new(1,-50,0,78)

speedPlus.Parent = speedCard

Instance.new("UICorner",speedMinus).CornerRadius =
	UDim.new(0,8)

Instance.new("UICorner",speedPlus).CornerRadius =
	UDim.new(0,8)

speedMinus.MouseButton1Click:Connect(function()

	flySpeed =
		math.clamp(
			flySpeed - 10,
			10,
			100
		)

	speedValue.Text =
		tostring(flySpeed)
end)

speedPlus.MouseButton1Click:Connect(function()

	flySpeed =
		math.clamp(
			flySpeed + 10,
			10,
			100
		)

	speedValue.Text =
		tostring(flySpeed)
end)

--------------------------------------------------
-- FOV CIRCLE
--------------------------------------------------

local fovCircle =
	Instance.new("Frame")

fovCircle.AnchorPoint =
	Vector2.new(.5,.5)

fovCircle.Position =
	UDim2.fromScale(.5,.5)

fovCircle.Size =
	UDim2.fromOffset(
		aimFOV * 2,
		aimFOV * 2
	)

fovCircle.BackgroundTransparency = 1
fovCircle.Visible = false
fovCircle.Parent = gui

Instance.new("UICorner",fovCircle).CornerRadius =
	UDim.new(1,0)

local fovStroke =
	Instance.new("UIStroke")

fovStroke.Color =
	Color3.fromRGB(175,80,255)

fovStroke.Thickness = 1.5
fovStroke.Transparency = .25
fovStroke.Parent = fovCircle

--------------------------------------------------
-- TARGET
--------------------------------------------------

local function getTarget()

	local closest
	local closestDistance = aimFOV

	local center =
		Vector2.new(
			camera.ViewportSize.X / 2,
			camera.ViewportSize.Y / 2
		)

	for _,targetPlayer in
		ipairs(Players:GetPlayers()) do

		if targetPlayer ~= player then

			local character =
				targetPlayer.Character

			if character then

				local humanoid =
					character:FindFirstChildOfClass(
						"Humanoid"
					)

				local head =
					character:FindFirstChild("Head")

				local root =
					character:FindFirstChild(
						"HumanoidRootPart"
					)

				if humanoid
					and head
					and root
					and humanoid.Health > 0 then

					if player.Team == nil
						or targetPlayer.Team ~= player.Team then

						local distance =
							(root.Position -
								camera.CFrame.Position).Magnitude

						if distance <= aimDistance then

							local screenPos,visible =
								camera:WorldToViewportPoint(
									head.Position
								)

							if visible and screenPos.Z > 0 then

								local point =
									Vector2.new(
										screenPos.X,
										screenPos.Y
									)

								local screenDistance =
									(point-center).Magnitude

								if screenDistance <
									closestDistance then

									local params =
										RaycastParams.new()

									params.FilterType =
										Enum.RaycastFilterType.Exclude

									params.FilterDescendantsInstances =
										{
											player.Character
										}

									local result =
										workspace:Raycast(
											camera.CFrame.Position,
											head.Position -
												camera.CFrame.Position,
											params
										)

									if result and
										result.Instance:
										IsDescendantOf(character) then

										closestDistance =
											screenDistance

										closest = head
									end
								end
							end
						end
					end
				end
			end
		end
	end

	return closest
end

--------------------------------------------------
-- AIM
--------------------------------------------------

local function updateAim()

	if not aimEnabled then
		return
	end

	local target =
		getTarget()

	if not target then
		return
	end

	local desired =
		CFrame.lookAt(
			camera.CFrame.Position,
			target.Position
		)

	camera.CFrame =
		camera.CFrame:Lerp(
			desired,
			aimSmooth
		)

	remote:FireServer(
		"AimTarget",
		target
	)
end

--------------------------------------------------
-- ESP
--------------------------------------------------

local function removeESP(targetPlayer)

	local guiObject =
		espObjects[targetPlayer]

	if guiObject then
		guiObject:Destroy()
		espObjects[targetPlayer] = nil
	end
end

local function createESP(targetPlayer)

	if targetPlayer == player then
		return
	end

	if espObjects[targetPlayer] then
		return
	end

	local billboard =
		Instance.new("BillboardGui")

	billboard.Name = "JKESP"
	billboard.Size =
		UDim2.fromOffset(55,75)

	billboard.StudsOffset =
		Vector3.new(0,1.8,0)

	billboard.AlwaysOnTop = true
	billboard.Parent = gui

	local box =
		Instance.new("Frame")

	box.Size =
		UDim2.fromScale(1,1)

	box.BackgroundTransparency = 1
	box.Parent = billboard

	local outline =
		Instance.new("UIStroke")

	outline.Color =
		Color3.fromRGB(180,80,255)

	outline.Thickness = 1.5
	outline.Parent = box

	local hpBack =
		Instance.new("Frame")

	hpBack.Size =
		UDim2.fromOffset(5,65)

	hpBack.Position =
		UDim2.fromOffset(-9,5)

	hpBack.BackgroundColor3 =
		Color3.fromRGB(35,35,35)

	hpBack.BorderSizePixel = 0
	hpBack.Parent = box

	local hp =
		Instance.new("Frame")

	hp.Name = "Health"

	hp.AnchorPoint =
		Vector2.new(0,1)

	hp.Position =
		UDim2.new(0,0,1,0)

	hp.Size =
		UDim2.fromScale(1,1)

	hp.BackgroundColor3 =
		Color3.fromRGB(80,220,100)

	hp.BorderSizePixel = 0
	hp.Parent = hpBack

	local name =
		Instance.new("TextLabel")

	name.Size =
		UDim2.new(1,0,0,16)

	name.Position =
		UDim2.fromOffset(0,-18)

	name.BackgroundTransparency = 1

	name.Text =
		targetPlayer.DisplayName

	name.TextColor3 =
		Color3.fromRGB(220,180,255)

	name.Font =
		Enum.Font.GothamBold

	name.TextSize = 10
	name.Parent = box

	espObjects[targetPlayer] =
		billboard
end

local function updateESP(targetPlayer)

	local billboard =
		espObjects[targetPlayer]

	if not billboard then
		return
	end

	local character =
		targetPlayer.Character

	if not character then
		billboard.Adornee = nil
		return
	end

	local root =
		character:FindFirstChild(
			"HumanoidRootPart"
		)

	local humanoid =
		character:FindFirstChildOfClass(
			"Humanoid"
		)

	if not root or
		not humanoid or
		humanoid.Health <= 0 then

		billboard.Adornee = nil
		return
	end

	billboard.Adornee = root

	local box =
		billboard:FindFirstChildOfClass(
			"Frame"
		)

	if box then

		local hpBack =
			box:FindFirstChildOfClass(
				"Frame"
			)

		if hpBack then

			local hp =
				hpBack:FindFirstChild(
					"Health"
				)

			if hp then

				local percent =
					math.clamp(
						humanoid.Health /
							humanoid.MaxHealth,
						0,
						1
					)

				hp.Size =
					UDim2.new(
						1,0,
						percent,0
					)
			end
		end
	end
end

local function clearESP()

	for _,object in pairs(espObjects) do
		object:Destroy()
	end

	table.clear(espObjects)
end

Players.PlayerAdded:Connect(function(targetPlayer)

	targetPlayer.CharacterAdded:Connect(function()

		task.wait(.3)

		if espEnabled then
			removeESP(targetPlayer)
			createESP(targetPlayer)
		end
	end)
end)

Players.PlayerRemoving:Connect(function(targetPlayer)
	removeESP(targetPlayer)
end)

--------------------------------------------------
-- FLY
--------------------------------------------------

local keys = {
	W=false,
	A=false,
	S=false,
	D=false,
	Up=false,
	Down=false
}

UserInputService.InputBegan:Connect(function(input,processed)

	if processed then
		return
	end

	if input.KeyCode == Enum.KeyCode.W then
		keys.W=true
	elseif input.KeyCode == Enum.KeyCode.A then
		keys.A=true
	elseif input.KeyCode == Enum.KeyCode.S then
		keys.S=true
	elseif input.KeyCode == Enum.KeyCode.D then
		keys.D=true
	elseif input.KeyCode == Enum.KeyCode.Space then
		keys.Up=true
	elseif input.KeyCode == Enum.KeyCode.LeftShift then
		keys.Down=true
	end
end)

UserInputService.InputEnded:Connect(function(input)

	if input.KeyCode == Enum.KeyCode.W then
		keys.W=false
	elseif input.KeyCode == Enum.KeyCode.A then
		keys.A=false
	elseif input.KeyCode == Enum.KeyCode.S then
		keys.S=false
	elseif input.KeyCode == Enum.KeyCode.D then
		keys.D=false
	elseif input.KeyCode == Enum.KeyCode.Space then
		keys.Up=false
	elseif input.KeyCode == Enum.KeyCode.LeftShift then
		keys.Down=false
	end
end)

local function getFlyDirection()

	local direction =
		Vector3.zero

	local cf =
		camera.CFrame

	if keys.W then
		direction += cf.LookVector
	end

	if keys.S then
		direction -= cf.LookVector
	end

	if keys.D then
		direction += cf.RightVector
	end

	if keys.A then
		direction -= cf.RightVector
	end

	if keys.Up then
		direction += Vector3.yAxis
	end

	if keys.Down then
		direction -= Vector3.yAxis
	end

	if direction.Magnitude > 1 then
		direction = direction.Unit
	end

	return direction
end

local function updateFly()

	if not flyEnabled then
		return
	end

	local direction

	if UserInputService.TouchEnabled
		and not UserInputService.KeyboardEnabled then

		local character =
			player.Character

		local humanoid =
			character and
			character:FindFirstChildOfClass(
				"Humanoid"
			)

		direction =
			humanoid and
			humanoid.MoveDirection
			or Vector3.zero

	else

		direction =
			getFlyDirection()

	end

	remote:FireServer(
		"FlyMove",
		{
			direction = direction,
			speed = flySpeed
		}
	)
end

--------------------------------------------------
-- BUTTON STATE
--------------------------------------------------

local function setButton(button,enabled)

	if enabled then

		button.Text = "ON"

		button.BackgroundColor3 =
			Color3.fromRGB(
				105,50,170
			)

	else

		button.Text = "OFF"

		button.BackgroundColor3 =
			Color3.fromRGB(
				65,35,105
			)
	end
end

--------------------------------------------------
-- BUTTONS
--------------------------------------------------

aimButton.MouseButton1Click:Connect(function()

	aimEnabled =
		not aimEnabled

	setButton(
		aimButton,
		aimEnabled
	)

	fovCircle.Visible =
		aimEnabled
end)

espButton.MouseButton1Click:Connect(function()

	espEnabled =
		not espEnabled

	setButton(
		espButton,
		espEnabled
	)

	if espEnabled then

		for _,targetPlayer in
			ipairs(Players:GetPlayers()) do

			createESP(targetPlayer)
		end

	else

		clearESP()

	end
end)

flyButton.MouseButton1Click:Connect(function()

	flyEnabled =
		not flyEnabled

	setButton(
		flyButton,
		flyEnabled
	)

	remote:FireServer(
		"Fly",
		flyEnabled
	)
end)

--------------------------------------------------
-- SERVER RESPONSE
--------------------------------------------------

remote.OnClientEvent:Connect(function(action)

	if action == "AccessGranted" then

		gui.Enabled = true

	elseif action == "ForceDisable" then

		aimEnabled = false
		espEnabled = false
		flyEnabled = false

		setButton(aimButton,false)
		setButton(espButton,false)
		setButton(flyButton,false)

		fovCircle.Visible = false

		clearESP()
	end
end)

--------------------------------------------------
-- LOOP
--------------------------------------------------

RunService.RenderStepped:Connect(function()

	if aimEnabled then

		fovCircle.Size =
			UDim2.fromOffset(
				aimFOV * 2,
				aimFOV * 2
			)

		updateAim()
	end

	if espEnabled then

		for _,targetPlayer in
			ipairs(Players:GetPlayers()) do

			if targetPlayer ~= player then

				if not espObjects[targetPlayer] then
					createESP(targetPlayer)
				end

				updateESP(targetPlayer)
			end
		end
	end
end)

RunService.Heartbeat:Connect(function()
	updateFly()
end)

--------------------------------------------------
-- REQUEST
--------------------------------------------------

remote:FireServer(
	"RequestAccess"
)
]==]

--------------------------------------------------
-- CREATE SERVER SCRIPT
--------------------------------------------------

local oldServer =
	ServerScriptService:FindFirstChild(
		"JKAdminAimServer"
	)

if oldServer then
	oldServer:Destroy()
end

local serverScript =
	Instance.new("Script")

serverScript.Name =
	"JKAdminAimServer"

serverScript.Source =
	serverSource

serverScript.Parent =
	ServerScriptService

--------------------------------------------------
-- CREATE CLIENT SCRIPT
--------------------------------------------------

local starterScripts =
	StarterPlayer:WaitForChild(
		"StarterPlayerScripts"
	)

local oldClient =
	starterScripts:FindFirstChild(
		"JKAdminAimClient"
	)

if oldClient then
	oldClient:Destroy()
end

local clientScript =
	Instance.new("LocalScript")

clientScript.Name =
	"JKAdminAimClient"

clientScript.Source =
	clientSource

clientScript.Parent =
	starterScripts

print("================================")
print("JK ADMIN AIM INSTALADO")
print("ServerScript criado")
print("LocalScript criado")
print("RemoteEvent criado")
print("================================")
