local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local camera = workspace.CurrentCamera

-- Criar GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "FFH4XPanel"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 1000 -- sobrepõe tudo

-- Frame principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 280, 0, 400)
frame.Position = UDim2.new(0.5, -140, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.BorderSizePixel = 0
frame.Visible = false
frame.ZIndex = 10

-- Cabeçalho arrastável
local header = Instance.new("TextLabel", frame)
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
header.Text = "FFH4X Dev Panel"
header.TextColor3 = Color3.fromRGB(255, 255, 0)
header.Font = Enum.Font.GothamBold
header.TextSize = 16
header.ZIndex = 11

-- Criar botão
local function createButton(name, y)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, y)
	btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	btn.TextColor3 = Color3.fromRGB(0, 255, 0)
	btn.Text = name
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.ZIndex = 12
	return btn
end

-- Botões
local funcs = {"Aimbot", "ESP", "Teleport", "Speed", "Jump", "Fly", "Gravity", "Heal"}
local buttons = {}

for i, name in ipairs(funcs) do
	buttons[name] = createButton(name, 35 + (i - 1) * 35)
end

-- Toggle com Q
UIS.InputBegan:Connect(function(input, gpe)
	if not gpe and input.KeyCode == Enum.KeyCode.Q then
		frame.Visible = not frame.Visible
	end
end)

-- Arrastar
local dragging, dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

header.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and input == dragInput then
		update(input)
	end
end)

-- Funções
local function getRoot()
	return player.Character and player.Character:FindFirstChild("HumanoidRootPart")
end

local function getHum()
	return player.Character and player.Character:FindFirstChildOfClass("Humanoid")
end

-- Aimbot
local function aimbot()
	local closest, dist = nil, math.huge
	for _, plr in pairs(game.Players:GetPlayers()) do
		if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
			local head = plr.Character.Head
			local d = (head.Position - camera.CFrame.Position).Magnitude
			if d < dist then
				dist = d
				closest = head
			end
		end
	end
	if closest then
		camera.CFrame = CFrame.new(camera.CFrame.Position, closest.Position)
	end
end

-- ESP
local function esp()
	for _, plr in pairs(game.Players:GetPlayers()) do
		if plr ~= player and plr.Character and not plr.Character:FindFirstChild("ESPHighlight") then
			local h = Instance.new("Highlight")
			h.Name = "ESPHighlight"
			h.FillColor = Color3.fromRGB(255, 0, 0)
			h.OutlineColor = Color3.fromRGB(255, 255, 255)
			h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
			h.Parent = plr.Character
		end
	end
end

-- Teleport
local function teleport()
	local root = getRoot()
	if root then
		root.CFrame = CFrame.new(150, 15, 150)
	end
end

-- Speed
local function speed()
	local hum = getHum()
	if hum then
		hum.WalkSpeed = 100
	end
end

-- Jump
local function jump()
	local hum = getHum()
	if hum then
		hum.JumpPower = 150
	end
end

-- Fly
local flying = false
local bv

local function fly()
	local root = getRoot()
	if not root then return end
	if not flying then
		bv = Instance.new("BodyVelocity")
		bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		bv.Velocity = Vector3.zero
		bv.Parent = root
		flying = true
	else
		if bv then bv:Destroy() end
		flying = false
	end
end

RS.RenderStepped:Connect(function()
	if flying and bv then
		local dir = Vector3.zero
		if UIS:IsKeyDown(Enum.KeyCode.W) then dir += camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0, 1, 0) end
		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0, 1, 0) end
		bv.Velocity = dir * 50
	end
end)

-- Gravity
local function gravity()
	workspace.Gravity = 40
end

-- Heal
local function heal()
	local hum = getHum()
	if hum then
		hum.Health = hum.MaxHealth
	end
end

-- Conectar botões
buttons.Aimbot.MouseButton1Click:Connect(aimbot)
buttons.ESP.MouseButton1Click:Connect(esp)
buttons.Teleport.MouseButton1Click:Connect(teleport)
buttons.Speed.MouseButton1Click:Connect(speed)
buttons.Jump.MouseButton1Click:Connect(jump)
buttons.Fly.MouseButton1Click:Connect(fly)
buttons.Gravity.MouseButton1Click:Connect(gravity)
buttons.Heal.MouseButton1Click:Connect(heal)
