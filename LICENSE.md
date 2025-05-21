local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "FFH4X_GUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 9999

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 180)
frame.Position = UDim2.new(0.5, -125, 0.5, -90)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Active = true
frame.Draggable = true -- Arrastável

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "FFH4X Panel"
title.TextColor3 = Color3.new(1, 1, 0)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.Font = Enum.Font.GothamBold
title.TextSize = 16

-- Função para botão
local function createButton(name, posY)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	btn.TextColor3 = Color3.new(0, 1, 0)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.Text = name
	return btn
end

local aimbotOn, espBoxOn, espLineOn, speedOn = false, false, false, false

local btnAimbot = createButton("Aimbot (100% HS)", 40)
local btnESPBox = createButton("ESP Box", 75)
local btnESPLine = createButton("ESP Line", 110)
local btnSpeed = createButton("Speed 90%", 145)

-- Alternar visibilidade com Q
UIS.InputBegan:Connect(function(input, gpe)
	if not gpe and input.KeyCode == Enum.KeyCode.Q then
		frame.Visible = not frame.Visible
	end
end)

-- Aimbot
RunService.RenderStepped:Connect(function()
	if not aimbotOn then return end

	local closest, distance = nil, math.huge
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
			local head = p.Character.Head
			local dist = (head.Position - Camera.CFrame.Position).Magnitude
			if dist < distance then
				distance = dist
				closest = head
			end
		end
	end
	if closest then
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Position)
	end
end)

-- ESP Box
local function updateESPBoxes()
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and not p.Character:FindFirstChild("ESPBox") then
			local adorn = Instance.new("BoxHandleAdornment")
			adorn.Name = "ESPBox"
			adorn.Adornee = p.Character
			adorn.Size = Vector3.new(4, 6, 2)
			adorn.AlwaysOnTop = true
			adorn.ZIndex = 10
			adorn.Color3 = Color3.new(1, 0, 0)
			adorn.Transparency = 0.5
			adorn.Parent = p.Character
		end
	end
end

-- ESP Line
local lines = {}

local function clearLines()
	for _, line in ipairs(lines) do
		line:Destroy()
	end
	lines = {}
end

RunService.RenderStepped:Connect(function()
	if not espLineOn then return end
	clearLines()
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local line = Drawing.new("Line")
			line.Color = Color3.fromRGB(255, 0, 0)
			line.Thickness = 2
			line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
			local pos, onScreen = Camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
			if onScreen then
				line.To = Vector2.new(pos.X, pos.Y)
				line.Visible = true
				table.insert(lines, line)
			else
				line:Remove()
			end
		end
	end
end)

-- Speed
RunService.RenderStepped:Connect(function()
	if speedOn and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 90
	end
end)

-- Botão funções
btnAimbot.MouseButton1Click:Connect(function()
	aimbotOn = not aimbotOn
	btnAimbot.TextColor3 = aimbotOn and Color3.new(1, 0, 0) or Color3.new(0, 1, 0)
end)

btnESPBox.MouseButton1Click:Connect(function()
	espBoxOn = not espBoxOn
	btnESPBox.TextColor3 = espBoxOn and Color3.new(1, 0, 0) or Color3.new(0, 1, 0)
	if espBoxOn then
		updateESPBoxes()
	else
		for _, p in pairs(Players:GetPlayers()) do
			if p.Character and p.Character:FindFirstChild("ESPBox") then
				p.Character.ESPBox:Destroy()
			end
		end
	end
end)

btnESPLine.MouseButton1Click:Connect(function()
	espLineOn = not espLineOn
	btnESPLine.TextColor3 = espLineOn and Color3.new(1, 0, 0) or Color3.new(0, 1, 0)
	if not espLineOn then clearLines() end
end)

btnSpeed.MouseButton1Click:Connect(function()
	speedOn = not speedOn
	btnSpeed.TextColor3 = speedOn and Color3.new(1, 0, 0) or Color3.new(0, 1, 0)
	if not speedOn and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
		LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
	end
end)
