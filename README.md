# uel
fly
-- Roblox手机端飞行与忍者能力脚本
-- 将此代码放入LocalScript中，放置在StarterPlayer/StarterPlayerScripts或StarterGui中

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- 等待角色完全加载
if not rootPart then
	repeat wait() until character:FindFirstChild("HumanoidRootPart")
	rootPart = character.HumanoidRootPart
end

-- 创建手机端控制界面
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileControls"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- 飞行状态变量
local flying = false
local flySpeed = 50
local flyKeys = {
	Forward = false,
	Backward = false,
	Left = false,
	Right = false,
	Up = false,
	Down = false
}

-- 忍者能力状态变量
local ninjaMode = false
local originalWalkSpeed = humanoid.WalkSpeed
local originalJumpPower = humanoid.JumpPower
local originalGravity = workspace.Gravity

-- 创建控制按钮
local function createButton(name, position, size, text)
	local button = Instance.new("TextButton")
	button.Name = name
	button.Size = UDim2.new(0, size.X, 0, size.Y)
	button.Position = UDim2.new(position.X.Scale, position.X.Offset, position.Y.Scale, position.Y.Offset)
	button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	button.BackgroundTransparency = 0.3
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Text = text
	button.Font = Enum.Font.SourceSansBold
	button.TextSize = 18
	button.BorderSizePixel = 0
	button.ZIndex = 10
	button.Parent = screenGui
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = button
	
	return button
end

-- 手机端控制按钮
local flyButton = createButton("FlyToggle", 
	{Scale = 0.05, Offset = 0}, {Scale = 0.1, Offset = 0},
	{50, 50}, "飞行\n(关闭)")
	
local ninjaButton = createButton("NinjaToggle", 
	{Scale = 0.05, Offset = 60}, {Scale = 0.1, Offset = 0},
	{50, 50}, "忍者\n(关闭)")
	
local upButton = createButton("UpButton", 
	{Scale = 0.8, Offset = -100}, {Scale = 0.9, Offset = 0},
	{60, 60}, "↑")
	
local downButton = createButton("DownButton", 
	{Scale = 0.8, Offset = -100}, {Scale = 1.0, Offset = 20},
	{60, 60}, "↓")

-- 方向控制按钮
local forwardButton = createButton("ForwardButton", 
	{Scale = 0.7, Offset = 0}, {Scale = 0.7, Offset = -60},
	{80, 60}, "前进")
	
local backButton = createButton("BackButton", 
	{Scale = 0.7, Offset = 0}, {Scale = 0.8, Offset = 60},
	{80, 60}, "后退")
	
local leftButton = createButton("LeftButton", 
	{Scale = 0.6, Offset = -80}, {Scale = 0.75, Offset = 0},
	{60, 80}, "左")
	
local rightButton = createButton("RightButton", 
	{Scale = 0.8, Offset = 80}, {Scale = 0.75, Offset = 0},
	{60, 80}, "右")

-- 速度控制按钮
local speedUpButton = createButton("SpeedUp", 
	{Scale = 0.05, Offset = 120}, {Scale = 0.1, Offset = 0},
	{50, 25}, "加速")
	
local speedDownButton = createButton("SpeedDown", 
	{Scale = 0.05, Offset = 120}, {Scale = 0.125, Offset = 30},
	{50, 25}, "减速")

-- 按钮触摸事件处理
local function setupButtonTouch(button, action)
	button.TouchTap:Connect(function()
		action()
	end)
	
	-- 用于方向控制的持续按压
	local touching = false
	
	button.TouchStarted:Connect(function()
		touching = true
		if action then action() end
	end)
	
	button.TouchEnded:Connect(function()
		touching = false
	end)
	
	return touching
end

-- 切换飞行模式
local function toggleFlight()
	flying = not flying
	
	if flying then
		flyButton.Text = "飞行\n(开启)"
		flyButton.BackgroundColor3 = Color3.fromRGB(30, 100, 200)
		
		-- 启用飞行
		humanoid.PlatformStand = true
		
		-- 创建飞行动效
		local bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.Name = "FlyVelocity"
		bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
		bodyVelocity.Velocity = Vector3.new(0, 0, 0)
		bodyVelocity.P = 10000
		bodyVelocity.Parent = rootPart
		
		local bodyGyro = Instance.new("BodyGyro")
		bodyGyro.Name = "FlyGyro"
		bodyGyro.MaxTorque = Vector3.new(40000, 40000, 40000)
		bodyGyro.P = 10000
		bodyGyro.D = 500
		bodyGyro.Parent = rootPart
		
		-- 飞行特效
		local flyEffect = Instance.new("ParticleEmitter")
		flyEffect.Name = "FlyEffect"
		flyEffect.Parent = rootPart
		flyEffect.Color = ColorSequence.new(Color3.fromRGB(100, 150, 255))
		flyEffect.Size = NumberSequence.new(0.5)
		flyEffect.Transparency = NumberSequence.new(0.5)
		flyEffect.Lifetime = NumberRange.new(0.5)
		flyEffect.Rate = 20
		flyEffect.Speed = NumberRange.new(2)
		
	else
		flyButton.Text = "飞行\n(关闭)"
		flyButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
		
		-- 禁用飞行
		humanoid.PlatformStand = false
		
		-- 移除飞行组件
		if rootPart:FindFirstChild("FlyVelocity") then
			rootPart.FlyVelocity:Destroy()
		end
		if rootPart:FindFirstChild("FlyGyro") then
			rootPart.FlyGyro:Destroy()
		end
		if rootPart:FindFirstChild("FlyEffect") then
			rootPart.FlyEffect:Destroy()
		end
	end
end

-- 切换忍者模式
local function toggleNinja()
	ninjaMode = not ninjaMode
	
	if ninjaMode then
		ninjaButton.Text = "忍者\n(开启)"
		ninjaButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
		
		-- 增强能力
		humanoid.WalkSpeed = 40
		humanoid.JumpPower = 80
		
		-- 忍者视觉效果
		local trail = Instance.new("Trail")
		trail.Name = "NinjaTrail"
		trail.Parent = rootPart
		trail.Color = ColorSequence.new(Color3.fromRGB(255, 50, 50))
		trail.Transparency = NumberSequence.new(0.5)
		trail.Lifetime = 0.5
		
		-- 跳跃增强效果
		humanoid.StateChanged:Connect(function(oldState, newState)
			if newState == Enum.HumanoidStateType.Jumping then
				-- 跳跃时产生冲击波
				local shockwave = Instance.new("Part")
				shockwave.Name = "Shockwave"
				shockwave.Size = Vector3.new(1, 0.2, 1)
				shockwave.Position = rootPart.Position - Vector3.new(0, 3, 0)
				shockwave.Anchored = true
				shockwave.CanCollide = false
				shockwave.Transparency = 0.5
				shockwave.BrickColor = BrickColor.new("Really red")
				shockwave.Material = EnumMaterial.Neon
				shockwave.Parent = workspace
				
				local mesh = Instance.new("CylinderMesh")
				mesh.Parent = shockwave
				
				-- 冲击波动画
				local tween = TweenService:Create(shockwave, TweenInfo.new(0.5), {
					Size = Vector3.new(20, 0.2, 20),
					Transparency = 1
				})
				tween:Play()
				
				game:GetService("Debris"):AddItem(shockwave, 0.6)
			end
		end)
		
	else
		ninjaButton.Text = "忍者\n(关闭)"
		ninjaButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
		
		-- 恢复原始能力
		humanoid.WalkSpeed = originalWalkSpeed
		humanoid.JumpPower = originalJumpPower
		
		-- 移除视觉效果
		if rootPart:FindFirstChild("NinjaTrail") then
			rootPart.NinjaTrail:Destroy()
		end
	end
end

-- 飞行速度控制
local function increaseSpeed()
	flySpeed = math.min(flySpeed + 10, 100)
end

local function decreaseSpeed()
	flySpeed = math.max(flySpeed - 10, 20)
end

-- 设置按钮事件
setupButtonTouch(flyButton, toggleFlight)
setupButtonTouch(ninjaButton, toggleNinja)
setupButtonTouch(speedUpButton, increaseSpeed)
setupButtonTouch(speedDownButton, decreaseSpeed)

-- 飞行方向控制
local forwardTouching = false
local backTouching = false
local leftTouching = false
local rightTouching = false
local upTouching = false
local downTouching = false

forwardButton.TouchStarted:Connect(function()
	forwardTouching = true
end)
forwardButton.TouchEnded:Connect(function()
	forwardTouching = false
end)

backButton.TouchStarted:Connect(function()
	backTouching = true
end)
backButton.TouchEnded:Connect(function()
	backTouching = false
end)

leftButton.TouchStarted:Connect(function()
	leftTouching = true
end)
leftButton.TouchEnded:Connect(function()
	leftTouching = false
end)

rightButton.TouchStarted:Connect(function()
	rightTouching = true
end)
rightButton.TouchEnded:Connect(function()
	rightTouching = false
end)

upButton.TouchStarted:Connect(function()
	upTouching = true
end)
upButton.TouchEnded:Connect(function()
	upTouching = false
end)

downButton.TouchStarted:Connect(function()
	downTouching = true
end)
downButton.TouchEnded:Connect(function()
	downTouching = false
end)

-- 飞行控制逻辑
RunService.RenderStepped:Connect(function(deltaTime)
	if flying and rootPart and rootPart:FindFirstChild("FlyVelocity") then
		local flyVelocity = rootPart.FlyVelocity
		local flyGyro = rootPart.FlyGyro
		
		-- 设置飞行方向
		local direction = Vector3.new(0, 0, 0)
		
		if forwardTouching then
			direction = direction + (rootPart.CFrame.LookVector * flySpeed)
		end
		if backTouching then
			direction = direction - (rootPart.CFrame.LookVector * flySpeed)
		end
		if leftTouching then
			direction = direction - (rootPart.CFrame.RightVector * flySpeed)
		end
		if rightTouching then
			direction = direction + (rootPart.CFrame.RightVector * flySpeed)
		end
		if upTouching then
			direction = direction + Vector3.new(0, flySpeed, 0)
		end
		if downTouching then
			direction = direction - Vector3.new(0, flySpeed, 0)
		end
		
		-- 更新速度
		flyVelocity.Velocity = direction
		
		-- 更新陀螺仪保持方向
		if flyGyro then
			flyGyro.CFrame = rootPart.CFrame
		end
	end
end)

-- 键盘控制支持（PC端备用）
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	
	if input.KeyCode == Enum.KeyCode.F then
		toggleFlight()
	elseif input.KeyCode == Enum.KeyCode.N then
		toggleNinja()
	elseif input.KeyCode == Enum.KeyCode.Equals then
		increaseSpeed()
	elseif input.KeyCode == Enum.KeyCode.Minus then
		decreaseSpeed()
	end
end)

-- 角色变化时重新初始化
player.CharacterAdded:Connect(function(newCharacter)
	character = newCharacter
	humanoid = character:WaitForChild("Humanoid")
	rootPart = character:WaitForChild("HumanoidRootPart")
	originalWalkSpeed = humanoid.WalkSpeed
	originalJumpPower = humanoid.JumpPower
	
	-- 重置状态
	flying = false
	flyButton.Text = "飞行\n(关闭)"
	flyButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	
	ninjaMode = false
	ninjaButton.Text = "忍者\n(关闭)"
	ninjaButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)

-- 提示信息
local hint = Instance.new("TextLabel")
hint.Name = "Hint"
hint.Size = UDim2.new(0, 300, 0, 60)
hint.Position = UDim2.new(0.5, -150, 0.05, 0)
hint.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
hint.BackgroundTransparency = 0.5
hint.TextColor3 = Color3.fromRGB(255, 255, 255)
hint.Text = "飞行与忍者能力已加载！\n点击按钮激活功能，使用方向键控制飞行"
hint.Font = Enum.Font.SourceSans
hint.TextSize = 16
hint.TextWrapped = true
hint.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = hint

-- 5秒后隐藏提示
wait(5)
hint:Destroy()

print("飞行与忍者能力脚本已加载！")
print("飞行速度: " .. flySpeed)
print("手机端: 使用屏幕按钮控制")
print("PC端: F键切换飞行, N键切换忍者模式")
