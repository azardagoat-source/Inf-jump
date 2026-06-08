-- // TITLE: Azar Expanded Game Settings Panel
-- // CONFIG: Configured for Developer: Azar
-- // DESCRIPTION: Clean multi-line layout with integrated raycast wall checking.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- // 1. SYSTEM CONFIGURATION MATRIX
local Config = {
	Master = false,
	Lock = false,
	Aimbot = false,
	AutoPlay = false,
	Range = 200,
	Smooth = 1.0, -- 1.0 = Instant hardware snap
	Part = "Head"
}

-- // 2. BUILD INTERFACE
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AzarEngine"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 240, 0, 290)
Frame.Position = UDim2.new(0.05, 0, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Frame.Active = true
Frame.Draggable = true
Frame.Parent = Frame -- Allows manual dragging across the screen
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
Title.Text = "  AZAR DEV ENGINE"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Frame

-- Function to generate UI buttons cleanly
local function AddUiBtn(text, yPos)
	local Btn = Instance.new("TextButton")
	Btn.Size = UDim2.new(0, 200, 0, 30)
	Btn.Position = UDim2.new(0, 20, 0, yPos)
	Btn.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
	Btn.Text = text
	Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	Btn.Font = Enum.Font.SourceSansSemibold
	Btn.TextSize = 13
	Btn.Parent = Frame
	return Btn
end

local Btn1 = AddUiBtn("MASTER CORE: OFF", 50)
local Btn2 = AddUiBtn("TARGET LOCK: OFF", 90)
local Btn3 = AddUiBtn("AIMBOT MODE: OFF", 130)
local Btn4 = AddUiBtn("RANGE: 200 STUDS", 170)
local Btn5 = AddUiBtn("SNAP SPEED: 1.0", 210)
local Btn6 = AddUiBtn("AUTOPLAY: OFF", 250)

-- // 3. INTERACTION WIREUP
local function Toggle(btn, state, tText, fText)
	if state then 
		btn.Text = tText 
		btn.BackgroundColor3 = Color3.fromRGB(0, 150, 90)
	else 
		btn.Text = fText 
		btn.BackgroundColor3 = Color3.fromRGB(45, 45, 50) 
	end
end

Btn1.MouseButton1Click:Connect(function() Config.Master = not Config.Master Toggle(Btn1, Config.Master, "MASTER CORE: ON", "MASTER CORE: OFF") end)
Btn2.MouseButton1Click:Connect(function() Config.Lock = not Config.Lock Toggle(Btn2, Config.Lock, "TARGET LOCK: ON", "TARGET LOCK: OFF") end)
Btn3.MouseButton1Click:Connect(function() Config.Aimbot = not Config.Aimbot Toggle(Btn3, Config.Aimbot, "AIMBOT MODE: ON", "AIMBOT MODE: OFF") end)
Btn6.MouseButton1Click:Connect(function() Config.AutoPlay = not Config.AutoPlay Toggle(Btn6, Config.AutoPlay, "AUTOPLAY: ON", "AUTOPLAY: OFF") end)

-- // 4. RAYCAST WALL CHECK FUNCTION
local function IsVisible(origin, targetPosition, character)
	local raycastParams = RaycastParams.new()
	raycastParams.FilterType = Enum.RaycastFilterType.Exclude
	raycastParams.FilterDescendantsInstances = {character} -- Ignore your own body parts
	
	local raycastResult = Workspace:Raycast(origin, targetPosition - origin, raycastParams)
	
	-- If the ray hit nothing, the path is clear. If it hit something, verify if it belongs to the target character.
	if not raycastResult then
		return true
	elseif raycastResult.Instance:IsDescendantOf(targetPosition.Parent) or raycastResult.Instance.CanCollide == false then
		return true
	end
	return false -- A wall or solid barrier is blocking the target
end

-- // 5. TARGET SCANNING MATH
local function GetTarget()
	local char = LocalPlayer.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
	local closest, shortest = nil, Config.Range
	
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character then
			local tChar = p.Character
			local hum = tChar:FindFirstChildOfClass("Humanoid")
			local part = tChar:FindFirstChild(Config.Part) or tChar:FindFirstChild("HumanoidRootPart")
			
			if part and hum and hum.Health > 0 then
				local dist = (part.Position - char.HumanoidRootPart.Position).Magnitude
				if dist < shortest then
					-- Perform the wall check before assigning the target
					if IsVisible(Camera.CFrame.Position, part.Position, char) then
						shortest = dist
						closest = part
					end
				end
			end
		end
	end
	return closest
end

-- // 6. RENDER FRAME CYCLE LOOP
RunService.RenderStepped:Connect(function()
	Camera = Workspace.CurrentCamera
	if not Config.Master then return end
	
	local target = GetTarget()
	if target and Camera then
		if Config.Lock or Config.Aimbot then
			local targetLookCFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position)
			Camera.CFrame = Camera.CFrame:Lerp(targetLookCFrame, Config.Smooth)
		end
	end
end)

print("[SYSTEM] Azar Engine fully loaded with wall verification loops.")
