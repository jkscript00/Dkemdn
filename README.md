--==================================================
--                    JK SCRIPTS
--             AIMBOT + ESP | ROBLOX STUDIO
--==================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--==================================================
-- CONFIG
--==================================================

local Config = {
	Aimbot = false,
	ESP = false,

	FOV = 300,
	Smoothness = 0.75,

	BoxColor = Color3.fromRGB(255,255,255),
	HealthColor = Color3.fromRGB(0,255,0)
}

--==================================================
-- GUI PRINCIPAL
--==================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "JK_SCRIPTS"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(390,330)
Main.Position = UDim2.new(0.5,-195,0.5,-165)
Main.BackgroundColor3 = Color3.fromRGB(18,18,23)
Main.BorderSizePixel = 0
Main.Parent = Gui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0,14)
MainCorner.Parent = Main

local Stroke = Instance.new("UIStroke")
Stroke.Color = Color3.fromRGB(95,60,180)
Stroke.Thickness = 1.5
Stroke.Parent = Main

--==================================================
-- TITULO
--==================================================

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,-70,0,48)
Title.Position = UDim2.fromOffset(15,0)
Title.BackgroundTransparency = 1
Title.Text = "JK SCRIPTS"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 21
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Main

local Subtitle = Instance.new("TextLabel")
Subtitle.Size = UDim2.new(1,-70,0,20)
Subtitle.Position = UDim2.fromOffset(16,29)
Subtitle.BackgroundTransparency = 1
Subtitle.Text = "AIM • ESP • STUDIO"
Subtitle.TextColor3 = Color3.fromRGB(145,145,155)
Subtitle.TextSize = 10
Subtitle.Font = Enum.Font.Gotham
Subtitle.TextXAlignment = Enum.TextXAlignment.Left
Subtitle.Parent = Main

--==================================================
-- MINIMIZAR
--==================================================

local Minimize = Instance.new("TextButton")
Minimize.Size = UDim2.fromOffset(40,35)
Minimize.Position = UDim2.new(1,-47,0,7)
Minimize.BackgroundColor3 = Color3.fromRGB(32,32,40)
Minimize.Text = "−"
Minimize.TextColor3 = Color3.new(1,1,1)
Minimize.TextSize = 22
Minimize.Font = Enum.Font.GothamBold
Minimize.Parent = Main

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0,8)
MinCorner.Parent = Minimize

local OpenButton = Instance.new("TextButton")
OpenButton.Size = UDim2.fromOffset(62,62)
OpenButton.Position = UDim2.new(0,20,0.5,-31)
OpenButton.BackgroundColor3 = Color3.fromRGB(25,20,35)
OpenButton.Text = "JK"
OpenButton.TextColor3 = Color3.fromRGB(255,255,255)
OpenButton.TextSize = 19
OpenButton.Font = Enum.Font.GothamBold
OpenButton.Visible = false
OpenButton.Parent = Gui

local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(1,0)
OpenCorner.Parent = OpenButton

local OpenStroke = Instance.new("UIStroke")
OpenStroke.Color = Color3.fromRGB(95,60,180)
OpenStroke.Thickness = 2
OpenStroke.Parent = OpenButton

Minimize.MouseButton1Click:Connect(function()
	Main.Visible = false
	OpenButton.Visible = true
end)

OpenButton.MouseButton1Click:Connect(function()
	Main.Visible = true
	OpenButton.Visible = false
end)

--==================================================
-- ABAS
--==================================================

local AimTab = Instance.new("TextButton")
AimTab.Size = UDim2.fromOffset(175,38)
AimTab.Position = UDim2.fromOffset(15,58)
AimTab.BackgroundColor3 = Color3.fromRGB(80,50,150)
AimTab.Text = "AIMBOT"
AimTab.TextColor3 = Color3.new(1,1,1)
AimTab.TextSize = 14
AimTab.Font = Enum.Font.GothamBold
AimTab.Parent = Main

local AimCorner = Instance.new("UICorner")
AimCorner.CornerRadius = UDim.new(0,8)
AimCorner.Parent = AimTab

local EspTab = Instance.new("TextButton")
EspTab.Size = UDim2.fromOffset(175,38)
EspTab.Position = UDim2.fromOffset(200,58)
EspTab.BackgroundColor3 = Color3.fromRGB(35,35,43)
EspTab.Text = "ESP"
EspTab.TextColor3 = Color3.new(1,1,1)
EspTab.TextSize = 14
EspTab.Font = Enum.Font.GothamBold
EspTab.Parent = Main

local EspCorner = Instance.new("UICorner")
EspCorner.CornerRadius = UDim.new(0,8)
EspCorner.Parent = EspTab

--==================================================
-- CONTAINERS
--==================================================

local AimPage = Instance.new("Frame")
AimPage.Size = UDim2.new(1,-30,1,-115)
AimPage.Position = UDim2.fromOffset(15,105)
AimPage.BackgroundTransparency = 1
AimPage.Parent = Main

local EspPage = Instance.new("Frame")
EspPage.Size = UDim2.new(1,-30,1,-115)
EspPage.Position = UDim2.fromOffset(15,105)
EspPage.BackgroundTransparency = 1
EspPage.Visible = false
EspPage.Parent = Main

AimTab.MouseButton1Click:Connect(function()
	AimPage.Visible = true
	EspPage.Visible = false

	AimTab.BackgroundColor3 = Color3.fromRGB(80,50,150)
	EspTab.BackgroundColor3 = Color3.fromRGB(35,35,43)
end)

EspTab.MouseButton1Click:Connect(function()
	AimPage.Visible = false
	EspPage.Visible = true

	EspTab.BackgroundColor3 = Color3.fromRGB(80,50,150)
	AimTab.BackgroundColor3 = Color3.fromRGB(35,35,43)
end)

--==================================================
-- FUNÇÃO BOTÃO
--==================================================

local function CreateButton(parent,text,y)
	local Button = Instance.new("TextButton")
	Button.Size = UDim2.new(1,0,0,43)
	Button.Position = UDim2.fromOffset(0,y)
	Button.BackgroundColor3 = Color3.fromRGB(34,34,42)
	Button.Text = text
	Button.TextColor3 = Color3.new(1,1,1)
	Button.TextSize = 14
	Button.Font = Enum.Font.Gotham
	Button.Parent = parent

	local Corner = Instance.new("UICorner")
	Corner.CornerRadius = UDim.new(0,8)
	Corner.Parent = Button

	return Button
end

--==================================================
-- AIMBOT PAGE
--==================================================

local AimbotButton = CreateButton(
	AimPage,
	"Aimbot: OFF",
	5
)

local FOVLabel = Instance.new("TextLabel")
FOVLabel.Size = UDim2.new(1,0,0,25)
FOVLabel.Position = UDim2.fromOffset(0,60)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "FOV: 300"
FOVLabel.TextColor3 = Color3.new(1,1,1)
FOVLabel.TextSize = 14
FOVLabel.Font = Enum.Font.Gotham
FOVLabel.TextXAlignment = Enum.TextXAlignment.Left
FOVLabel.Parent = AimPage

local FOVBox = Instance.new("TextBox")
FOVBox.Size = UDim2.new(1,0,0,40)
FOVBox.Position = UDim2.fromOffset(0,88)
FOVBox.BackgroundColor3 = Color3.fromRGB(34,34,42)
FOVBox.Text = "300"
FOVBox.TextColor3 = Color3.new(1,1,1)
FOVBox.TextSize = 14
FOVBox.Font = Enum.Font.Gotham
FOVBox.ClearTextOnFocus = false
FOVBox.Parent = AimPage

local FOVCorner = Instance.new("UICorner")
FOVCorner.CornerRadius = UDim.new(0,8)
FOVCorner.Parent = FOVBox

AimbotButton.MouseButton1Click:Connect(function()

	Config.Aimbot = not Config.Aimbot

	if Config.Aimbot then
		AimbotButton.Text = "Aimbot: ON"
		AimbotButton.BackgroundColor3 = Color3.fromRGB(80,50,150)
	else
		AimbotButton.Text = "Aimbot: OFF"
		AimbotButton.BackgroundColor3 = Color3.fromRGB(34,34,42)
	end
end)

FOVBox.FocusLost:Connect(function()

	local Value = tonumber(FOVBox.Text)

	if Value then
		Config.FOV = math.clamp(Value,50,1000)
		FOVBox.Text = tostring(Config.FOV)
		FOVLabel.Text = "FOV: "..Config.FOV
	else
		FOVBox.Text = tostring(Config.FOV)
	end
end)

--==================================================
-- FOV CIRCLE
--==================================================

local FOVCircle = Instance.new("Frame")
FOVCircle.Name = "JK_FOV"
FOVCircle.AnchorPoint = Vector2.new(0.5,0.5)
FOVCircle.Position = UDim2.fromScale(0.5,0.5)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Visible = false
FOVCircle.Parent = Gui

local FOVCorner2 = Instance.new("UICorner")
FOVCorner2.CornerRadius = UDim.new(1,0)
FOVCorner2.Parent = FOVCircle

local FOVStroke = Instance.new("UIStroke")
FOVStroke.Color = Color3.fromRGB(170,120,255)
FOVStroke.Thickness = 2
FOVStroke.Parent = FOVCircle

--==================================================
-- ESP PAGE
--==================================================

local ESPButton = CreateButton(
	EspPage,
	"ESP: OFF",
	5
)

local Info = Instance.new("TextLabel")
Info.Size = UDim2.new(1,0,0,80)
Info.Position = UDim2.fromOffset(0,60)
Info.BackgroundTransparency = 1
Info.Text = "ESP inclui:\n• Caixa ao redor do personagem\n• Barra de vida\n• Vida numérica\n• Atualização automática"
Info.TextColor3 = Color3.fromRGB(180,180,190)
Info.TextSize = 13
Info.Font = Enum.Font.Gotham
Info.TextXAlignment = Enum.TextXAlignment.Left
Info.TextYAlignment = Enum.TextYAlignment.Top
Info.Parent = EspPage

ESPButton.MouseButton1Click:Connect(function()

	Config.ESP = not Config.ESP

	if Config.ESP then
		ESPButton.Text = "ESP: ON"
		ESPButton.BackgroundColor3 = Color3.fromRGB(80,50,150)
	else
		ESPButton.Text = "ESP: OFF"
		ESPButton.BackgroundColor3 = Color3.fromRGB(34,34,42)
	end
end)

--==================================================
-- AIMBOT
--==================================================

local function IsValidTarget(Character)

	if not Character then
		return false
	end

	if Character == LocalPlayer.Character then
		return false
	end

	local Humanoid = Character:FindFirstChildOfClass("Humanoid")
	local Head = Character:FindFirstChild("Head")

	if not Humanoid or not Head then
		return false
	end

	return Humanoid.Health > 0
end

local function CanSeeTarget(Target)

	if not Target then
		return false
	end

	local Origin = Camera.CFrame.Position
	local Direction = Target.Position - Origin

	local Params = RaycastParams.new()
	Params.FilterType = Enum.RaycastFilterType.Exclude
	Params.FilterDescendantsInstances = {
		LocalPlayer.Character
	}
	Params.IgnoreWater = true

	local Result = workspace:Raycast(
		Origin,
		Direction,
		Params
	)

	if not Result then
		return true
	end

	return Result.Instance:IsDescendantOf(
		Target.Parent
	)
end

local function GetTarget()

	local Closest = nil
	local Shortest = Config.FOV

	local Center = Vector2.new(
		Camera.ViewportSize.X / 2,
		Camera.ViewportSize.Y / 2
	)

	for _,Player in ipairs(Players:GetPlayers()) do

		if Player ~= LocalPlayer then

			local Character = Player.Character

			if IsValidTarget(Character) then

				local Head = Character:FindFirstChild("Head")

				local Screen,Visible =
					Camera:WorldToViewportPoint(
						Head.Position
					)

				if Visible and Screen.Z > 0 then

					local Distance = (
						Vector2.new(
							Screen.X,
							Screen.Y
						) - Center
					).Magnitude

					if Distance < Shortest
						and CanSeeTarget(Head) then

						Shortest = Distance
						Closest = Head
					end
				end
			end
		end
	end

	return Closest
end

--==================================================
-- ESP NATIVO DO ROBLOX
--==================================================

local ESPObjects = {}

local function RemoveESP(Character)

	local Data = ESPObjects[Character]

	if Data then

		if Data.Gui then
			Data.Gui:Destroy()
		end

		ESPObjects[Character] = nil
	end
end

local function CreateESP(Character)

	if not Character then
		return
	end

	if Character == LocalPlayer.Character then
		return
	end

	if ESPObjects[Character] then
		return
	end

	local Humanoid =
		Character:FindFirstChildOfClass("Humanoid")

	local Head =
		Character:FindFirstChild("Head")

	if not Humanoid or not Head then
		return
	end

	-- Billboard
	local Billboard = Instance.new("BillboardGui")
	Billboard.Name = "JK_ESP"
	Billboard.Adornee = Character:FindFirstChild("HumanoidRootPart") or Head
	Billboard.Size = UDim2.fromOffset(90,120)
	Billboard.StudsOffset = Vector3.new(0,0,0)
	Billboard.AlwaysOnTop = true
	Billboard.Enabled = false
	Billboard.Parent = Gui

	-- Caixa
	local Box = Instance.new("Frame")
	Box.Size = UDim2.fromScale(0.75,0.9)
	Box.Position = UDim2.fromScale(0.125,0.05)
	Box.BackgroundTransparency = 1
	Box.BorderSizePixel = 0
	Box.Parent = Billboard

	local BoxStroke = Instance.new("UIStroke")
	BoxStroke.Color = Config.BoxColor
	BoxStroke.Thickness = 2
	BoxStroke.Parent = Box

	-- Barra de vida
	local HealthBackground = Instance.new("Frame")
	HealthBackground.Size = UDim2.fromOffset(5,90)
	HealthBackground.Position = UDim2.fromOffset(4,8)
	HealthBackground.BackgroundColor3 = Color3.fromRGB(35,35,35)
	HealthBackground.BorderSizePixel = 0
	HealthBackground.Parent = Billboard

	local HealthFill = Instance.new("Frame")
	HealthFill.AnchorPoint = Vector2.new(0,1)
	HealthFill.Position = UDim2.fromScale(0,1)
	HealthFill.Size = UDim2.fromScale(1,1)
	HealthFill.BackgroundColor3 = Config.HealthColor
	HealthFill.BorderSizePixel = 0
	HealthFill.Parent = HealthBackground

	-- Texto de vida
	local HealthText = Instance.new("TextLabel")
	HealthText.Size = UDim2.fromOffset(80,20)
	HealthText.Position = UDim2.fromOffset(5,-14)
	HealthText.BackgroundTransparency = 1
	HealthText.Text = "100"
	HealthText.TextColor3 = Color3.new(1,1,1)
	HealthText.TextStrokeTransparency = 0
	HealthText.TextSize = 13
	HealthText.Font = Enum.Font.GothamBold
	HealthText.Parent = Billboard

	ESPObjects[Character] = {
		Gui = Billboard,
		HealthFill = HealthFill,
		HealthText = HealthText
	}

	Humanoid.Died:Connect(function()
		RemoveESP(Character)
	end)
end

--==================================================
-- ATUALIZA ESP
--==================================================

local function UpdateESP()

	for Character,Data in pairs(ESPObjects) do

		if not Character.Parent then
			RemoveESP(Character)
			continue
		end

		local Humanoid =
			Character:FindFirstChildOfClass("Humanoid")

		if not Humanoid or Humanoid.Health <= 0 then
			Data.Gui.Enabled = false
			continue
		end

		Data.Gui.Enabled = Config.ESP

		local Percent =
			math.clamp(
				Humanoid.Health /
				math.max(Humanoid.MaxHealth,1),
				0,
				1
			)

		Data.HealthFill.Size =
			UDim2.fromScale(1,Percent)

		Data.HealthText.Text =
			tostring(math.floor(Humanoid.Health))
	end
end

--==================================================
-- PLAYERS
--==================================================

local function SetupPlayer(Player)

	if Player == LocalPlayer then
		return
	end

	Player.CharacterAdded:Connect(function(Character)

		task.wait(0.5)

		CreateESP(Character)
	end)

	if Player.Character then
		CreateESP(Player.Character)
	end
end

for _,Player in ipairs(Players:GetPlayers()) do
	SetupPlayer(Player)
end

Players.PlayerAdded:Connect(SetupPlayer)

Players.PlayerRemoving:Connect(function(Player)

	if Player.Character then
		RemoveESP(Player.Character)
	end
end)

--==================================================
-- NPCs / BOTS
--==================================================

local function TryCreateNPC(Model)

	if not Model:IsA("Model") then
		return
	end

	if Model == LocalPlayer.Character then
		return
	end

	if Players:GetPlayerFromCharacter(Model) then
		return
	end

	local Humanoid =
		Model:FindFirstChildOfClass("Humanoid")

	if Humanoid then
		CreateESP(Model)
	end
end

for _,Obj in ipairs(workspace:GetDescendants()) do
	TryCreateNPC(Obj)
end

workspace.DescendantAdded:Connect(function(Obj)

	task.wait(0.15)

	TryCreateNPC(Obj)
end)

--==================================================
-- LOOP PRINCIPAL
--==================================================

RunService.RenderStepped:Connect(function()

	-- FOV
	FOVCircle.Visible = Config.Aimbot

	FOVCircle.Size =
		UDim2.fromOffset(
			Config.FOV * 2,
			Config.FOV * 2
		)

	-- AIMBOT
	if Config.Aimbot then

		local Target = GetTarget()

		if Target then

			local AimCFrame =
				CFrame.lookAt(
					Camera.CFrame.Position,
					Target.Position
				)

			Camera.CFrame =
				Camera.CFrame:Lerp(
					AimCFrame,
					Config.Smoothness
				)
		end
	end

	-- ESP
	UpdateESP()
end)

--==================================================
-- ARRASTAR MENU
--==================================================

local Dragging = false
local DragStart
local StartPosition

Title.InputBegan:Connect(function(Input)

	if Input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or Input.UserInputType ==
		Enum.UserInputType.Touch then

		Dragging = true
		DragStart = Input.Position
		StartPosition = Main.Position
	end
end)

UserInputService.InputChanged:Connect(function(Input)

	if not Dragging then
		return
	end

	if Input.UserInputType ==
		Enum.UserInputType.MouseMovement
		or Input.UserInputType ==
		Enum.UserInputType.Touch then

		local Delta =
			Input.Position - DragStart

		Main.Position =
			UDim2.new(
				StartPosition.X.Scale,
				StartPosition.X.Offset + Delta.X,
				StartPosition.Y.Scale,
				StartPosition.Y.Offset + Delta.Y
			)
	end
end)

UserInputService.InputEnded:Connect(function(Input)

	if Input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or Input.UserInputType ==
		Enum.UserInputType.Touch then

		Dragging = false
	end
end)

print("JK SCRIPTS | AIMBOT + ESP carregado.")
