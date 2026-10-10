--[[
	nnnn.lua | Cerberus Style Modern UI + Config Manager System (Updated & Fixed with Notifications + Bottom Line)
	loadstring(readfile("nnnn.lua"))()
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
	Players:GetPropertyChangedSignal("LocalPlayer"):Wait()
	LocalPlayer = Players.LocalPlayer
end
if not LocalPlayer then
	warn("[nnnn] LocalPlayer 없음 - 게임 들어간 뒤 다시 실행")
	return
end
if not table.clear then function table.clear(t) for k in pairs(t) do t[k] = nil end end end
local Camera = Workspace.CurrentCamera
local env = (getgenv and getgenv()) or _G

local nnnnConns = {}
local function nnnnTrack(conn)
	if conn then table.insert(nnnnConns, conn) end
	return conn
end

print("[Ronaldo Hook] build: v7.7-boot")
print("[Ronaldo Hook] boot ok, player=", LocalPlayer and LocalPlayer.Name)
print("[Ronaldo Hook] placeId=", game.PlaceId, "gameId=", game.GameId)
warn("[Ronaldo Hook] SCRIPT STARTED")

-- 즉시 부트 배너 (뒤에 뭐가 터져도 일단 보이게)
pcall(function()
	local function parentGui(g)
		pcall(function() if gethui then g.Parent = gethui() end end)
		if not g.Parent then pcall(function() g.Parent = game:GetService("CoreGui") end) end
		if not g.Parent then
			local pg = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer:WaitForChild("PlayerGui", 5)
			if pg then g.Parent = pg end
		end
	end
	local old = game:GetService("CoreGui"):FindFirstChild("RonaldoHookBoot")
	if old then old:Destroy() end
	local g = Instance.new("ScreenGui")
	g.Name = "RonaldoHookBoot"
	g.ResetOnSpawn = false
	g.DisplayOrder = 10000
	g.IgnoreGuiInset = true
	g.Enabled = true
	parentGui(g)
	local f = Instance.new("Frame")
	f.Size = UDim2.new(0, 280, 0, 70)
	f.Position = UDim2.new(0.5, -140, 0, 40)
	f.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
	f.BorderSizePixel = 0
	f.Parent = g
	Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
	local s = Instance.new("UIStroke")
	s.Color = Color3.fromRGB(220, 40, 40)
	s.Thickness = 2
	s.Parent = f
	local l = Instance.new("TextLabel")
	l.Size = UDim2.new(1, -12, 1, -12)
	l.Position = UDim2.new(0, 6, 0, 6)
	l.BackgroundTransparency = 1
	l.Font = Enum.Font.GothamBold
	l.TextSize = 14
	l.TextColor3 = Color3.fromRGB(255, 255, 255)
	l.TextWrapped = true
	l.Text = "Ronaldo Hook\nloading..."
	l.Parent = f
	env.__RonaldoBootGui = g
	env.__RonaldoBootLabel = l
	warn("[Ronaldo Hook] boot banner parent=", g.Parent and g.Parent:GetFullName())

	-- MINI MENU (항상 뜨는 최소 UI)
	local mini = Instance.new("Frame")
	mini.Name = "MiniMenu"
	mini.Size = UDim2.new(0, 320, 0, 180)
	mini.Position = UDim2.new(0.5, -160, 0.5, -90)
	mini.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
	mini.BorderSizePixel = 0
	mini.Parent = g
	Instance.new("UICorner", mini).CornerRadius = UDim.new(0, 10)
	local ms = Instance.new("UIStroke")
	ms.Color = Color3.fromRGB(212, 175, 55)
	ms.Thickness = 2
	ms.Parent = mini
	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, 0, 0, 36)
	title.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
	title.BackgroundTransparency = 0.2
	title.BorderSizePixel = 0
	title.Font = Enum.Font.GothamBold
	title.TextSize = 16
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.Text = "Ronaldo Hook"
	title.ZIndex = 2
	title.Parent = mini
	local info = Instance.new("TextLabel")
	info.Size = UDim2.new(1, -20, 0, 60)
	info.Position = UDim2.new(0, 10, 0, 50)
	info.BackgroundTransparency = 1
	info.Font = Enum.Font.Gotham
	info.TextSize = 13
	info.TextColor3 = Color3.fromRGB(220, 220, 230)
	info.TextWrapped = true
	info.Text = "UI loading...\nFull menu opens when ready.\nRightShift / Insert"
	info.ZIndex = 2
	info.Parent = mini
	local close = Instance.new("TextButton")
	close.Size = UDim2.new(0, 100, 0, 30)
	close.Position = UDim2.new(0.5, -50, 1, -45)
	close.BackgroundColor3 = Color3.fromRGB(70, 140, 220)
	close.Text = "OK"
	close.Font = Enum.Font.GothamBold
	close.TextSize = 14
	close.TextColor3 = Color3.fromRGB(255, 255, 255)
	close.ZIndex = 2
	close.Parent = mini
	Instance.new("UICorner", close).CornerRadius = UDim.new(0, 6)
	close.MouseButton1Click:Connect(function()
		mini.Visible = false
	end)
	env.__RonaldoMini = mini
end)


-- 이전 인스턴스 정리 (실패해도 계속)
pcall(function()
	if env.__NnnnUnload and type(env.__NnnnUnload) == "function" then
		env.__NnnnUnload()
	end
end)

-- 다른 스크립트 UI는 건드리지 않음 (nnnn 전용만 정리)
pcall(function()
	local only = {
		"NnnnHubUI", "NnnnToggleGui", "CerberusNotifyGui",
		"RageHUD", "ActiveFeaturesHUD", "NnnnSkinPicker",
		"NnnnTargetHUD", "NnnnHitMarker",
	}
	local parents = { CoreGui }
	local pg = LocalPlayer:FindFirstChild("PlayerGui")
	if pg then table.insert(parents, pg) end
	pcall(function()
		if gethui then table.insert(parents, gethui()) end
	end)
	for _, parent in ipairs(parents) do
		for _, name in ipairs(only) do
			local old = parent:FindFirstChild(name)
			if old then old:Destroy() end
		end
	end
end)

-- GUI 부모 (CoreGui 막힌 실행기 대비)
local function nnnnGuiParent()
	local parent
	pcall(function()
		if gethui then parent = gethui() end
	end)
	if parent then return parent end
	local ok = pcall(function()
		local t = Instance.new("ScreenGui")
		t.Parent = CoreGui
		t:Destroy()
	end)
	if ok then return CoreGui end
	return LocalPlayer:WaitForChild("PlayerGui")
end
env.__NnnnGuiParent = nnnnGuiParent

-- 우측 상단 알림 시스템 함수 (하단 밑줄 추가 완료)
local function showNotification(messageText)
	task.spawn(function()
		pcall(function()
			local old = CoreGui:FindFirstChild("CerberusNotifyGui")
			if old then old:Destroy() end

			local notifGui = Instance.new("ScreenGui")
			notifGui.Name = "CerberusNotifyGui"
			notifGui.ResetOnSpawn = false
			local gp = (env.__NnnnGuiParent and env.__NnnnGuiParent()) or CoreGui
			pcall(function() notifGui.Parent = gp end)
			if not notifGui.Parent then notifGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

			local notifFrame = Instance.new("Frame")
			notifFrame.Size = UDim2.new(0, 240, 0, 40)
			notifFrame.Position = UDim2.new(1, -260, 0, 20)
			notifFrame.BackgroundColor3 = Color3.fromRGB(20, 23, 30)
			notifFrame.BorderSizePixel = 0
			notifFrame.Parent = notifGui

			Instance.new("UICorner", notifFrame).CornerRadius = UDim.new(0, 6)
			
			local stroke = Instance.new("UIStroke")
			stroke.Color = Color3.fromRGB(220, 40, 40)
			stroke.Thickness = 1
			stroke.Parent = notifFrame

			-- 알림창 하단에 들어가는 밑줄 바
			local bottomBar = Instance.new("Frame")
			bottomBar.Size = UDim2.new(1, 0, 0, 2)
			bottomBar.Position = UDim2.new(0, 0, 1, -2)
			bottomBar.BackgroundColor3 = Settings and Settings.ThemeColor or Color3.fromRGB(220, 40, 40)
			bottomBar.BorderSizePixel = 0
			bottomBar.Parent = notifFrame

			local label = Instance.new("TextLabel")
			label.Size = UDim2.new(1, -16, 1, 0)
			label.Position = UDim2.new(0, 8, 0, 0)
			label.BackgroundTransparency = 1
			label.Text = messageText
			label.TextColor3 = Color3.fromRGB(240, 240, 250)
			label.Font = Enum.Font.GothamBold
			label.TextSize = 12
			label.TextXAlignment = Enum.TextXAlignment.Left
			label.Parent = notifFrame

			-- 애니메이션 (등장 후 사라짐)
			notifFrame.Position = UDim2.new(1, 20, 0, 20)
			TweenService:Create(notifFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(1, -260, 0, 20)}):Play()
			
			task.wait(2.5)
			
			TweenService:Create(notifFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 20, 0, 20)}):Play()
			task.wait(0.3)
			notifGui:Destroy()
		end)
	end)
end

local Settings = {
	AimbotEnabled = false,
	ShowFOV = false,
	FOVRadius = 220,
	HitChance = 100,
	TargetPart = "Head",
	RageEnabled = false,
	AutoWall = false,
	RapidFire = false,
	Spinbot = false,
	SpinSpeed = 120,
	TargetSelection = "Distance",
	RageTP = false,
	TPDistance = 0.25,
	Desync = false,
	DesyncAmount = 35,
	DesyncMode = "AdvancedJitter",
	TeamCheck = true,
	WalkSpeed = 16,
	UnlockAllSkins = false,
	FlyEnabled = false,
	FlySpeed = 50,
	WaitForShield = false,
	MenuKey = Enum.KeyCode.RightShift,
	IntenseViewShake = true,
	-- Extra features
	Triggerbot = false,
	TriggerDelay = 0.05,
	TriggerFOV = 40,
	Resolver = false,
	TargetHUD = false,
	HitMarker = false,
	HitEffect = false,
	JumpPower = 50,
	CustomJump = false,
	InfJump = false,
	NoFall = false,
	ViewmodelChams = false,
	ViewmodelOffset = false,
	VM_X = 0,
	VM_Y = 0,
	VM_Z = 0,

	ThemeColor = Color3.fromRGB(220, 40, 40),
	-- Bullet Tracer (총알 궤적)
	BulletTracer = false,
	TracerMode = "Solid", -- Rainbow | Solid
	TracerThickness = 0.08,
	TracerDuration = 0.28,
	TracerSolidColor = Color3.fromRGB(100, 255, 220), -- mint
	-- Hit Sound
	HitSoundEnabled = false,
	HitSoundName = "Skeet",
	HitSoundVolume = 2,
	-- Third Person
	ThirdPersonEnabled = false,
	ThirdPersonMode = "ThirdPerson",
	UnlockMouse = false,
	-- Spoofers
	NameSpoofEnabled = false,
	NameSpoofText = "nnnn",
	DeviceSpoofEnabled = false,
	DeviceSpoofMode = "MouseKeyboard",
	EmoteHop = false,
	EmoteSpeed = 40,
	-- Stat Spoof (클라 표시만)
	SpoofNameEnabled = false,
	SpoofName = "",
	SpoofHideRealName = true,
	-- 기능 보기 UI 표시 여부
	ShowActiveFeatures = false,
	-- Anti-Aim
	AntiAimEnabled = false,
	AntiAimPitch = "disabled",
	AntiAimYaw = "disabled",
	AntiAimUnderground = false,
	AntiAimSpeed = 180,
	AntiAimBodySpin = false, -- 몸/팔다리 미친 회전
	AntiAimEmote = false,
	AntiAimEmoteId = "",
	AntiAimStretch = true, -- 몸 늘어나며 회전
}

local OrbitEnabled = false
local orbitConnection

local RageConfig = {
	on = false,
	desync = true,
	hide = 0.04,
	attack = 1.2,
	height = 0.08,
	fireRate = 0.0001,
	prediction = true,
	predStrength = 0.12,
}

local _rb = {
	active = false,
	gen = 0,
	readyAt = 0,
	target = nil,
	lastTargetCheck = 0,
	lastHide = 0,
	isHiding = false,
	lastShot = 0,
}

local ESPConfig = {
	enabled = false,
	box = false,
	distance = false,
	health = false,
	healthBar = true,
	skeleton = false,
	name = false,
}

local CurrentTarget = nil
local spinAngle = 0
local _aaUnderY = nil
local desyncOffset = Vector3.zero
local lastDesyncTick = 0
local continuousDesyncConn = nil

local RageHUD, rageStatusLabel
local function createRageHUD()
	if RageHUD then pcall(function() RageHUD:Destroy() end) end
	local gui = Instance.new("ScreenGui")
	gui.Name = "RageHUD"
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	pcall(function() gui.Parent = CoreGui end)
	if not gui.Parent then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 280, 0, 30)
	label.Position = UDim2.new(0.5, 40, 0.5, -80)
	label.BackgroundTransparency = 1
	label.Text = "Ragebot: -"
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.TextSize = 15
	label.Font = Enum.Font.Code
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.TextStrokeTransparency = 0
	label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
	label.Parent = gui

	RageHUD = gui
	rageStatusLabel = label
	return label
end

local function updateRageHUD(text)
	if not rageStatusLabel or not rageStatusLabel.Parent then
		createRageHUD()
	end
	if rageStatusLabel then
		local name = tostring(text or "")
		if name == "" then
			rageStatusLabel.Text = "Ragebot: -"
		else
			rageStatusLabel.Text = "Ragebot: " .. name
		end
	end
end

local function destroyRageHUD()
	if RageHUD then
		pcall(function() RageHUD:Destroy() end)
		RageHUD = nil
		rageStatusLabel = nil
	end
end

-- [수정됨] 맨 왼쪽 중간에 켜져 있는 기능 보여주는 HUD 생성 (크기 및 글자 확대 통합)
local ActiveFeaturesHUD, activeFeaturesLabel
local function createActiveFeaturesHUD()
	if ActiveFeaturesHUD then pcall(function() ActiveFeaturesHUD:Destroy() end) end
	local gui = Instance.new("ScreenGui")
	gui.Name = "ActiveFeaturesHUD"
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	pcall(function() gui.Parent = CoreGui end)
	if not gui.Parent then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 260, 0, 240) -- 크기 확대
	frame.Position = UDim2.new(0, 10, 0.5, -120) -- 맨 왼쪽 중간 정렬 조정
	frame.BackgroundColor3 = Color3.fromRGB(15, 17, 23)
	frame.BackgroundTransparency = 0.25
	frame.BorderSizePixel = 0
	frame.Parent = gui
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(35, 38, 48)
	stroke.Thickness = 1
	stroke.Parent = frame

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -16, 0, 28)
	title.Position = UDim2.new(0, 8, 0, 6)
	title.BackgroundTransparency = 1
	title.Text = "⚡ Active Features"
	title.TextColor3 = Color3.fromRGB(220, 220, 230)
	title.Font = Enum.Font.GothamBold
	title.TextSize = 13 -- 타이틀 글자 크기 확대
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = frame

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -16, 1, -38)
	label.Position = UDim2.new(0, 8, 0, 34)
	label.BackgroundTransparency = 1
	label.Text = "None"
	label.TextColor3 = Color3.fromRGB(200, 205, 215)
	label.Font = Enum.Font.Code
	label.TextSize = 13 -- 텍스트 크기 대폭 키움 (기존 10 -> 13)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.TextYAlignment = Enum.TextYAlignment.Top
	label.TextWrapped = true
	label.Parent = frame

	ActiveFeaturesHUD = gui
	activeFeaturesLabel = label
	return label
end

local function destroyActiveFeaturesHUD()
	pcall(function() if WeatherAPI and WeatherAPI.clearAura then WeatherAPI.clearAura() end end)
	if ActiveFeaturesHUD then
		pcall(function() ActiveFeaturesHUD:Destroy() end)
		ActiveFeaturesHUD = nil
		activeFeaturesLabel = nil
	end
end

-- 실시간으로 켜진 기능 수집하여 업데이트
local _lastActiveFeat = 0
RunService.Heartbeat:Connect(function()
	if not Settings.ShowActiveFeatures then
		return
	end
	if tick() - _lastActiveFeat < 0.5 then return end
	_lastActiveFeat = tick()

	if not activeFeaturesLabel or not activeFeaturesLabel.Parent then
		createActiveFeaturesHUD()
	end

	local activeList = {}
	if Settings.AimbotEnabled then table.insert(activeList, "• Silent Aim") end
	if Settings.RageEnabled then table.insert(activeList, "• Ragebot Mode") end
	if RageConfig.on then table.insert(activeList, "• Ultra Rage Core") end
	if OrbitEnabled then table.insert(activeList, "• Orbit Mode") end
	if Settings.Spinbot then table.insert(activeList, "• Spinbot") end
	if Settings.Desync then table.insert(activeList, "• Desync Active") end
	if Settings.FlyEnabled then table.insert(activeList, "• Fly Mode") end
	if Settings.RapidFire then table.insert(activeList, "• Rapid Fire") end
	if Settings.UnlockAllSkins then table.insert(activeList, "• unlock all cosmetics") end
	if ESPConfig.enabled then table.insert(activeList, "• ESP Enabled") end
	if Settings.BulletTracer then table.insert(activeList, "• Bullet Tracer") end
	if Settings.HitSoundEnabled then table.insert(activeList, "• Hit Sound: " .. tostring(Settings.HitSoundName)) end
	if Settings.AntiAimEnabled then table.insert(activeList, "• Anti-Aim") end
	if Settings.NoclipEnabled then table.insert(activeList, "• Noclip") end

	if #activeList > 0 then
		activeFeaturesLabel.Text = table.concat(activeList, "\n")
	else
		activeFeaturesLabel.Text = "None (No active features)"
	end
end)

local Skyboxes = {
	{ Name = "Default", SkyboxBk = "", SkyboxDn = "", SkyboxFt = "", SkyboxLf = "", SkyboxRt = "", SkyboxUp = "" },
	{ Name = "Night Galaxy",
		SkyboxBk = "rbxassetid://159454299", SkyboxDn = "rbxassetid://159454301", SkyboxFt = "rbxassetid://159454296",
		SkyboxLf = "rbxassetid://159454293", SkyboxRt = "rbxassetid://159454300", SkyboxUp = "rbxassetid://159454288" },
	{ Name = "Clouds",
		SkyboxBk = "rbxassetid://591058823", SkyboxDn = "rbxassetid://591059876", SkyboxFt = "rbxassetid://591058104",
		SkyboxLf = "rbxassetid://591057861", SkyboxRt = "rbxassetid://591057625", SkyboxUp = "rbxassetid://591059642" },
	{ Name = "Space Dust",
		SkyboxBk = "rbxassetid://159454286", SkyboxDn = "rbxassetid://159454286", SkyboxFt = "rbxassetid://159454286",
		SkyboxLf = "rbxassetid://159454286", SkyboxRt = "rbxassetid://159454286", SkyboxUp = "rbxassetid://159454286" },
	{ Name = "Pink Nebula",
		SkyboxBk = "rbxassetid://149600354", SkyboxDn = "rbxassetid://149600354", SkyboxFt = "rbxassetid://149600354",
		SkyboxLf = "rbxassetid://149600354", SkyboxRt = "rbxassetid://149600354", SkyboxUp = "rbxassetid://149600354" },
	{ Name = "Amber Sky",
		SkyboxBk = "rbxassetid://150939022", SkyboxDn = "rbxassetid://150939027", SkyboxFt = "rbxassetid://150939029",
		SkyboxLf = "rbxassetid://150939025", SkyboxRt = "rbxassetid://150939024", SkyboxUp = "rbxassetid://150939020" },
	{ Name = "Deep Purple",
		SkyboxBk = "rbxassetid://570557408", SkyboxDn = "rbxassetid://570557408", SkyboxFt = "rbxassetid://570557408",
		SkyboxLf = "rbxassetid://570557408", SkyboxRt = "rbxassetid://570557408", SkyboxUp = "rbxassetid://570557408" },
	{ Name = "Storm Red",
		SkyboxBk = "rbxassetid://401664839", SkyboxDn = "rbxassetid://401664862", SkyboxFt = "rbxassetid://401664960",
		SkyboxLf = "rbxassetid://401664881", SkyboxRt = "rbxassetid://401664901", SkyboxUp = "rbxassetid://401664926" },
	{ Name = "Void Black",
		SkyboxBk = "rbxassetid://201228632", SkyboxDn = "rbxassetid://201228632", SkyboxFt = "rbxassetid://201228632",
		SkyboxLf = "rbxassetid://201228632", SkyboxRt = "rbxassetid://201228632", SkyboxUp = "rbxassetid://201228632" },
	{ Name = "Ice",
		SkyboxBk = "rbxassetid://225284652", SkyboxDn = "rbxassetid://225284652", SkyboxFt = "rbxassetid://225284652",
		SkyboxLf = "rbxassetid://225284652", SkyboxRt = "rbxassetid://225284652", SkyboxUp = "rbxassetid://225284652" },
	{ Name = "Aurora",
		SkyboxBk = "rbxassetid://162479122", SkyboxDn = "rbxassetid://162479122", SkyboxFt = "rbxassetid://162479122",
		SkyboxLf = "rbxassetid://162479122", SkyboxRt = "rbxassetid://162479122", SkyboxUp = "rbxassetid://162479122" },
	{ Name = "Hellfire",
		SkyboxBk = "rbxassetid://194241254", SkyboxDn = "rbxassetid://194241254", SkyboxFt = "rbxassetid://194241254",
		SkyboxLf = "rbxassetid://194241254", SkyboxRt = "rbxassetid://194241254", SkyboxUp = "rbxassetid://194241254" },
	{ Name = "Ocean",
		SkyboxBk = "rbxassetid://113529954", SkyboxDn = "rbxassetid://113529954", SkyboxFt = "rbxassetid://113529954",
		SkyboxLf = "rbxassetid://113529954", SkyboxRt = "rbxassetid://113529954", SkyboxUp = "rbxassetid://113529954" },
	{ Name = "White Out",
		SkyboxBk = "rbxassetid://6444884337", SkyboxDn = "rbxassetid://6444884780", SkyboxFt = "rbxassetid://6444884337",
		SkyboxLf = "rbxassetid://6444884337", SkyboxRt = "rbxassetid://6444884337", SkyboxUp = "rbxassetid://6412503611" },
}

local currentSkyIndex = 1
local lastSkyCheck = 0
local skyForceToken = 0

local function applySkybox(index, force)
	local sky = Skyboxes[index]
	if not sky then return end
	currentSkyIndex = index
	skyForceToken += 1
	pcall(function()
		-- 맵이 넣는 Sky 전부 제거
		for _, child in ipairs(Lighting:GetChildren()) do
			if child:IsA("Sky") then
				child:Destroy()
			end
		end
		if sky.Name == "Default" or sky.SkyboxBk == "" then
			return
		end
		local newSky = Instance.new("Sky")
		newSky.Name = "CustomSky"
		newSky.SkyboxBk = sky.SkyboxBk
		newSky.SkyboxDn = sky.SkyboxDn
		newSky.SkyboxFt = sky.SkyboxFt
		newSky.SkyboxLf = sky.SkyboxLf
		newSky.SkyboxRt = sky.SkyboxRt
		newSky.SkyboxUp = sky.SkyboxUp
		newSky.StarCount = 3000
		newSky.CelestialBodiesShown = false
		newSky.Parent = Lighting
	end)
end

-- 사격장/맵 전환해도 계속 유지: 자주 검사 + 속성 강제
nnnnTrack(RunService.Heartbeat:Connect(function()
	if LionWorld and LionWorld.SkyEnabled then return end
	if currentSkyIndex <= 1 then return end
	if tick() - lastSkyCheck < 2 then return end
	lastSkyCheck = tick()
	local sky = Skyboxes[currentSkyIndex]
	if not sky or sky.SkyboxBk == "" then return end
	pcall(function()
		local existing = Lighting:FindFirstChild("CustomSky")
		if not existing or not existing:IsA("Sky") then
			applySkybox(currentSkyIndex, true)
			return
		end
		-- 맵이 덮어썼으면 다시 씌움
		if existing.SkyboxBk ~= sky.SkyboxBk
			or existing.SkyboxFt ~= sky.SkyboxFt
			or existing.SkyboxUp ~= sky.SkyboxUp then
			existing.SkyboxBk = sky.SkyboxBk
			existing.SkyboxDn = sky.SkyboxDn
			existing.SkyboxFt = sky.SkyboxFt
			existing.SkyboxLf = sky.SkyboxLf
			existing.SkyboxRt = sky.SkyboxRt
			existing.SkyboxUp = sky.SkyboxUp
		end
		-- 다른 Sky가 생기면 제거
		for _, child in ipairs(Lighting:GetChildren()) do
			if child:IsA("Sky") and child ~= existing then
				child:Destroy()
			end
		end
	end)
end))

-- Lighting에 Sky 추가되는 즉시 차단
nnnnTrack(Lighting.ChildAdded:Connect(function(child)
	if currentSkyIndex <= 1 then return end
	if child:IsA("Sky") and child.Name ~= "CustomSky" then
		task.defer(function()
			pcall(function() child:Destroy() end)
			applySkybox(currentSkyIndex, true)
		end)
	end
end))

local Utility, EnumLib, Fighter, ItemLibrary
pcall(function() Utility = require(ReplicatedStorage.Modules.Utility) end)
pcall(function() EnumLib = require(ReplicatedStorage.Modules.EnumLibrary) end)
pcall(function() Fighter = require(LocalPlayer.PlayerScripts.Controllers.FighterController) end)
pcall(function() ItemLibrary = require(ReplicatedStorage.Modules.ItemLibrary) end)

local _itemCooldownBackup = {}
local _fastFireKeys = {
	"ShootSpread", "ShootAccuracy", "ShootRecoil",
	"ShootCooldown", "ShootBurstCooldown", "AttackCooldown",
	"SwingCooldown", "MeleeCooldown", "Cooldown",
	"RecoveryTime", "ResetTime",
}

local function applyFastFireAndMelee(enable)
	task.spawn(function()
		pcall(function()
			local Items = ItemLibrary and ItemLibrary.Items
			if not Items then return end

			if enable then
				for name, data in pairs(Items) do
					if typeof(data) == "table" then
						if not _itemCooldownBackup[name] then
							local bak = {}
							for _, k in ipairs(_fastFireKeys) do
								bak[k] = data[k]
							end
							_itemCooldownBackup[name] = bak
						end
						data.ShootSpread = 0
						data.ShootAccuracy = 0
						data.ShootRecoil = 0
						data.ShootCooldown = 0.0001
						data.ShootBurstCooldown = 0.0001
						data.AttackCooldown = 0.0001
						data.SwingCooldown = 0.0001
						data.MeleeCooldown = 0.0001
						data.Cooldown = 0.0001
						data.RecoveryTime = 0.0001
						data.ResetTime = 0.0001
					end
				end
			else
				for name, bak in pairs(_itemCooldownBackup) do
					local data = Items[name]
					if typeof(data) == "table" and typeof(bak) == "table" then
						for _, k in ipairs(_fastFireKeys) do
							if bak[k] ~= nil then
								data[k] = bak[k]
							end
						end
					end
				end
			end
		end)
	end)
end

local allSkinsSetup = false
local function enableAllSkins()
	if allSkinsSetup then
		-- 이미 훅 설치됨 → 인벤만 갱신
		pcall(function()
			if env.__NnnnSkinAPI and env.__NnnnSkinAPI.rebuildinv then
				env.__NnnnSkinAPI.rebuildinv()
			end
		end)
		return
	end
	allSkinsSetup = true

	local lps = LocalPlayer.PlayerScripts
	local ctrls = lps.Controllers
	local rmods = ReplicatedStorage.Modules

	local elib = require(rmods:WaitForChild("EnumLibrary", 10))
	if elib then elib:WaitForEnumBuilder() end
	local clib = require(rmods:WaitForChild("CosmeticLibrary", 10))
	local ilib = require(rmods:WaitForChild("ItemLibrary", 10))
	local dctrl = require(ctrls:WaitForChild("PlayerDataController", 10))
	local coss = clib.Cosmetics

	local cdata = dctrl.CurrentData
	if not cdata then
		task.spawn(function()
			repeat task.wait() until dctrl.CurrentData
			cdata = dctrl.CurrentData
		end)
	end

	local equip, favs = {}, {}
	local cwep, vprof, lwep
	local fcache = {}

	local function banned(n)
		if type(n) ~= "string" then return true end
		return n:find("MISSING_") or n:find("Bubblegum") or n:find("Ragdoll") or n:find("Fall Apart") or n:find("Every Finisher Ever")
	end

	local function toenum(n)
		if not elib then return nil end
		local ok, id = pcall(elib.ToEnum, elib, n)
		return ok and id or nil
	end

	local function clonecos(name, ctype, inv, favonly)
		if banned(name) then return nil end
		local base = coss[name]
		if not base then return nil end
		local d = table.clone(base)
		d.Name = name
		d.Type = d.Type or ctype
		d.Seed = d.Seed or math.random(1, 1000000)
		d.Owned = true
		d.Unlocked = true
		d.Locked = false
		d.Amount = math.max(1, tonumber(d.Amount) or 1)
		local eid = toenum(name)
		if eid then d.Enum = eid d.ObjectID = d.ObjectID or eid end
		if inv ~= nil then d.Inverted = inv end
		if favonly ~= nil then d.OnlyUseFavorites = favonly end
		return d
	end

	-- 올스킨 전용 콘픽 (nnnnLinoria/skins.json)
	local savef = "nnnnLinoria/skins.json"

	local function savecfg()
		if not writefile then return end
		pcall(function()
			local cfg = {
				unlock = Settings.UnlockAllSkins and true or false,
				equipped = {},
				favorites = favs,
			}
			for wep, cos in pairs(equip) do
				local slot = {}
				cfg.equipped[wep] = slot
				for ct, cd in pairs(cos) do
					if cd and cd.Name and not banned(cd.Name) then
						slot[ct] = {
							name = cd.Name,
							seed = cd.Seed,
							inverted = cd.Inverted,
						}
					end
				end
			end
			if makefolder then
				pcall(makefolder, "nnnnLinoria")
			end
			writefile(savef, HttpService:JSONEncode(cfg))
			print("[nnnn] skins config saved:", savef)
		end)
	end

	local function loadcfg()
		if not readfile or not isfile then return end
		pcall(function()
			if not isfile(savef) then return end
			local cfg = HttpService:JSONDecode(readfile(savef))
			if type(cfg) ~= "table" then return end
			if cfg.unlock then
				Settings.UnlockAllSkins = true
			end
			if type(cfg.equipped) == "table" then
				for wep, cos in pairs(cfg.equipped) do
					equip[wep] = equip[wep] or {}
					if type(cos) == "table" then
						for ct, cd in pairs(cos) do
							if type(cd) == "table" and cd.name and not banned(cd.name) then
								local cl = clonecos(cd.name, ct, cd.inverted)
								if cl then
									cl.Seed = cd.seed
									equip[wep][ct] = cl
								end
							end
						end
					end
				end
			end
			if type(cfg.favorites) == "table" then
				favs = cfg.favorites
			end
			print("[nnnn] skins config loaded")
		end)
	end

	local finv = {}
	local function rebuildinv()
		table.clear(finv)
		-- Unlock All ON → 라이브러리 전체 소유로 표시
		if Settings.UnlockAllSkins and coss then
			for name, base in pairs(coss) do
				if type(name) == "string" and not banned(name) then
					finv[name] = true
					pcall(function()
						base.Owned = true
						base.Unlocked = true
						base.Locked = false
					end)
				end
			end
		end
		for _, cos in pairs(equip) do
			for _, cd in pairs(cos) do
				if cd and cd.Name and not banned(cd.Name) then
					finv[cd.Name] = true
					cd.Owned = true
					cd.Unlocked = true
					cd.Locked = false
				end
			end
		end
		for name, fv in pairs(favs) do
			if type(fv) == "table" then
				for n in pairs(fv) do
					if not banned(n) then finv[n] = true end
				end
			elseif type(name) == "string" and not banned(name) then
				finv[name] = true
			end
		end
	end

	rebuildinv()
	-- 영구 자동 로드 안 함 (콘픽 저장 시에만 unlock 플래그 저장)

	local oget = dctrl.Get
	dctrl.Get = function(self, key)
		local data = oget(self, key)
		if key == "CosmeticInventory" then
			local proxy = {}
			if data then for k, v in pairs(data) do if not banned(k) then proxy[k] = v end end end
			for name in pairs(finv) do proxy[name] = true end
			return proxy
		end
		if key == "FavoritedCosmetics" then
			local res = data and table.clone(data) or {}
			for wep, fv in pairs(favs) do
				local slot = res[wep] or {}
				res[wep] = slot
				for name, isfav in pairs(fv) do
					if not banned(name) then slot[name] = isfav end
				end
			end
			return res
		end
		return data
	end

	local ogetwep = dctrl.GetWeaponData
	dctrl.GetWeaponData = function(self, wname)
		local data = ogetwep(self, wname)
		if not data then return nil end
		local merged = table.clone(data)
		merged.Name = wname
		local weq = equip[wname]
		if weq then for ct, cd in pairs(weq) do merged[ct] = cd end end
		return merged
	end

	local function saferep(key)
		pcall(function()
			if not cdata and dctrl.CurrentData then cdata = dctrl.CurrentData end
			if cdata then cdata:Replicate(key) end
		end)
	end

	local fctrl
	pcall(function() fctrl = require(ctrls:WaitForChild("FighterController", 10)) end)

	local function getewep()
		if not fctrl then return nil end
		local fighter = fctrl:GetFighter(LocalPlayer)
		if not fighter or not fighter.Items then return nil end
		for _, item in pairs(fighter.Items) do
			if item.IsEquipped then return item.Name end
		end
		return nil
	end

	if hookmetamethod then
		local rems = ReplicatedStorage:FindFirstChild("Remotes")
		local drems = rems and rems:FindFirstChild("Data")
		local eqrem = drems and drems:FindFirstChild("EquipCosmetic")
		local favrem = drems and drems:FindFirstChild("FavoriteCosmetic")
		local rrems = rems and rems:FindFirstChild("Replication")
		local frems = rrems and rrems:FindFirstChild("Fighter")
		local uirem = frems and frems:FindFirstChild("UseItem")

		if eqrem then
			local oncall
			oncall = hookmetamethod(game, "__namecall", function(self, ...)
				if getnamecallmethod() ~= "FireServer" then return oncall(self, ...) end
				local args = { ... }

				if uirem and self == uirem and fctrl then
					pcall(function()
						local fighter = fctrl:GetFighter(LocalPlayer)
						if fighter and fighter.Items then
							local oid = args[1]
							for _, item in pairs(fighter.Items) do
								if item:Get("ObjectID") == oid then
									lwep = item.Name
									break
								end
							end
						end
					end)
				end

				if self == eqrem then
					local wname, ctype, cname, opts = args[1], args[2], args[3], args[4] or {}
					if not cname or cname == "None" or cname == "" then
						equip[wname] = equip[wname] or {}
						equip[wname][ctype] = nil
						if not next(equip[wname]) then equip[wname] = nil end
						rebuildinv()
						task.defer(function()
							saferep("WeaponInventory")
						end)
						return oncall(self, ...)
					end
					if banned(cname) then return oncall(self, ...) end
					local rdata = oget(dctrl, "CosmeticInventory")
					if rdata and type(rdata[cname]) ~= "boolean" and rdata[cname] ~= nil then
						return oncall(self, ...)
					end
					equip[wname] = equip[wname] or {}
					local cloned = clonecos(cname, ctype, opts.IsInverted, opts.OnlyUseFavorites)
					if cloned then equip[wname][ctype] = cloned end
					if ctype == "Finisher" then fcache[wname] = cname end
					rebuildinv()
					task.defer(function()
						saferep("WeaponInventory")
					end)
					return
				end

				if self == favrem then
					local fwep, fname, fstate = args[1], args[2], args[3]
					if not fname or fname == "None" or fname == "" then return oncall(self, ...) end
					if banned(fname) then return oncall(self, ...) end
					favs[fwep] = favs[fwep] or {}
					favs[fwep][fname] = fstate or nil
					task.spawn(saferep, "FavoritedCosmetics")
					return
				end

				return oncall(self, ...)
			end)
		end
	end

	local citem
	pcall(function() citem = require(lps.Modules.ClientReplicatedClasses.ClientFighter.ClientItem) end)

	if citem and citem._CreateViewModel then
		local ocvm = citem._CreateViewModel
		citem._CreateViewModel = function(self, vmref)
			local wname = self.Name
			local wplr = self.ClientFighter and self.ClientFighter.Player
			cwep = (wplr == LocalPlayer) and wname or nil
			if wplr == LocalPlayer and equip[wname] and equip[wname].Skin and vmref then
				local skin = equip[wname].Skin
				local dk = self:ToEnum("Data")
				if vmref[dk] then
					vmref[dk][self:ToEnum("Skin")] = skin
					vmref[dk][self:ToEnum("Name")] = skin.Name
				elseif vmref.Data then
					vmref.Data.Skin = skin
					vmref.Data.Name = skin.Name
				end
			end
			local res = ocvm(self, vmref)
			cwep = nil
			return res
		end
	end

	local vmmod = lps.Modules.ClientReplicatedClasses.ClientFighter.ClientItem:FindFirstChild("ClientViewModel")
	if vmmod then
		local cvm = require(vmmod)
		if cvm.GetWrap then
			local ogw = cvm.GetWrap
			cvm.GetWrap = function(self)
				local ci = self.ClientItem
				local wname = ci and ci.Name
				local wplr = ci and ci.ClientFighter and ci.ClientFighter.Player
				local weq = wname and wplr == LocalPlayer and equip[wname]
				return (weq and weq.Wrap) or ogw(self)
			end
		end
		local onew = cvm.new
		cvm.new = function(rdata, cliitm)
			local wplr = cliitm.ClientFighter and cliitm.ClientFighter.Player
			local wname = cwep or cliitm.Name
			if wplr == LocalPlayer and equip[wname] then
				local rcls = require(ReplicatedStorage.Modules.ReplicatedClass)
				local dk = rcls:ToEnum("Data")
				rdata[dk] = rdata[dk] or {}
				local cos = equip[wname]
				local slot = rdata[dk]
				if cos.Skin then slot[rcls:ToEnum("Skin")] = cos.Skin end
				if cos.Wrap then slot[rcls:ToEnum("Wrap")] = cos.Wrap end
				if cos.Charm then slot[rcls:ToEnum("Charm")] = cos.Charm end
			end
			local res = onew(rdata, cliitm)
			if wplr == LocalPlayer and equip[wname] and equip[wname].Wrap and res._UpdateWrap then
				res:_UpdateWrap()
				task.delay(0.1, function() if not res._destroyed then res:_UpdateWrap() end end)
			end
			return res
		end
	end

	local ogvi = ilib.GetViewModelImageFromWeaponData
	ilib.GetViewModelImageFromWeaponData = function(self, wdata, hires)
		if not wdata then return ogvi(self, wdata, hires) end
		local wname = wdata.Name
		local weq = equip[wname]
		if weq and weq.Skin and (wdata.Skin == weq.Skin or vprof == LocalPlayer) then
			local sinfo = self.ViewModels[weq.Skin.Name]
			if sinfo then return sinfo[hires and "ImageHighResolution" or "Image"] or sinfo.Image end
		end
		return ogvi(self, wdata, hires)
	end

	pcall(function()
		local vpmod = require(lps.Modules.Pages.ViewProfile)
		if vpmod and vpmod.Fetch then
			local ofetch = vpmod.Fetch
			vpmod.Fetch = function(self, tplr)
				vprof = tplr
				return ofetch(self, tplr)
			end
		end
	end)

	local cent
	pcall(function() cent = require(lps.Modules.ClientReplicatedClasses.ClientEntity) end)

	if cent and cent._PlayFinisher then
		local ofin = cent._PlayFinisher
		cent._PlayFinisher = function(self, fname, ...)
			local ewep = getewep()
			local tfin = ewep and (fcache[ewep] or (equip[ewep] and equip[ewep].Finisher and equip[ewep].Finisher.Name))
			return ofin(self, tfin or fname, ...)
		end
	end

	rebuildinv()
	for wname, wdata in pairs(equip) do
		if wdata.Finisher and wdata.Finisher.Name then
			fcache[wname] = wdata.Finisher.Name
		end
	end

	-- 스킨 피커용 API
	env.__NnnnSkinAPI = {
		equip = equip,
		favs = favs,
		coss = coss,
		clonecos = clonecos,
		rebuildinv = rebuildinv,
		saferep = saferep,
		savecfg = savecfg,
		loadcfg = loadcfg,
		banned = banned,
		getewep = getewep,
		fcache = fcache,
	}
	print("[nnnn] all skins + picker API ready")
end

local function openSkinPicker()
	Settings.UnlockAllSkins = true
	pcall(enableAllSkins)
	pcall(enableAllSkins)
	local api = env.__NnnnSkinAPI
	if not api or not api.coss then
		showNotification("스킨 데이터 로드 실패")
		return
	end

	local old = CoreGui:FindFirstChild("NnnnSkinPicker")
	if old then old:Destroy() end

	local gui = Instance.new("ScreenGui")
	gui.Name = "NnnnSkinPicker"
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	pcall(function() gui.Parent = CoreGui end)
	if not gui.Parent then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 620, 0, 560)
	frame.Position = UDim2.new(0.5, -310, 0.5, -280)
	frame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
	frame.BorderSizePixel = 0
	frame.Parent = gui
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(220, 40, 40)
	stroke.Thickness = 1.5
	stroke.Parent = frame

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -100, 0, 36)
	title.Position = UDim2.new(0, 14, 0, 8)
	title.BackgroundTransparency = 1
	title.Text = "Cosmetic Changer"
	title.TextColor3 = Color3.fromRGB(245, 245, 250)
	title.Font = Enum.Font.GothamBold
	title.TextSize = 16
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = frame

	local closeBtn = Instance.new("TextButton")
	closeBtn.Size = UDim2.new(0, 34, 0, 30)
	closeBtn.Position = UDim2.new(1, -46, 0, 10)
	closeBtn.BackgroundColor3 = Color3.fromRGB(42, 44, 54)
	closeBtn.Text = "X"
	closeBtn.TextColor3 = Color3.fromRGB(255, 210, 210)
	closeBtn.Font = Enum.Font.GothamBold
	closeBtn.TextSize = 14
	closeBtn.Parent = frame
	Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
	closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

	-- 페이지: weapons | skins
	local page = "weapons"
	local selectedWeapon = api.getewep() or nil
	local selectedType = "Skin"
	local selectedCos = nil

	local types = {}
	local typeSet = {}
	for name, def in pairs(api.coss) do
		if not api.banned(name) and type(def) == "table" then
			local tp = tostring(def.Type or "Skin")
			if not typeSet[tp] then
				typeSet[tp] = true
				table.insert(types, tp)
			end
		end
	end
	table.sort(types)
	if #types == 0 then types = {"Skin", "Wrap", "Charm", "Finisher"} end
	if not typeSet[selectedType] then selectedType = types[1] end

	-- 무기 목록 + 아이콘 시도
	local weapons = {}
	pcall(function()
		local ilib = require(ReplicatedStorage.Modules.ItemLibrary)
		if ilib and ilib.Items then
			for wname, wdef in pairs(ilib.Items) do
				local icon = nil
				if type(wdef) == "table" then
					icon = wdef.Icon or wdef.Image or wdef.ImageId or wdef.Thumbnail
					if type(icon) == "number" then icon = "rbxassetid://" .. tostring(icon) end
					if type(icon) ~= "string" then icon = nil end
				end
				table.insert(weapons, { Name = wname, Icon = icon })
			end
		end
	end)
	table.sort(weapons, function(a, b) return a.Name < b.Name end)
	if #weapons == 0 then
		table.insert(weapons, { Name = selectedWeapon or "Weapon", Icon = nil })
	end
	if not selectedWeapon and weapons[1] then selectedWeapon = weapons[1].Name end

	local subtitle = Instance.new("TextLabel")
	subtitle.Size = UDim2.new(1, -28, 0, 20)
	subtitle.Position = UDim2.new(0, 14, 0, 42)
	subtitle.BackgroundTransparency = 1
	subtitle.Text = "무기를 눌러 스킨 선택"
	subtitle.TextColor3 = Color3.fromRGB(160, 165, 180)
	subtitle.Font = Enum.Font.Gotham
	subtitle.TextSize = 12
	subtitle.TextXAlignment = Enum.TextXAlignment.Left
	subtitle.Parent = frame

	local content = Instance.new("Frame")
	content.Size = UDim2.new(1, -28, 1, -110)
	content.Position = UDim2.new(0, 14, 0, 68)
	content.BackgroundTransparency = 1
	content.Parent = frame

	local bottomBar = Instance.new("Frame")
	bottomBar.Size = UDim2.new(1, -28, 0, 36)
	bottomBar.Position = UDim2.new(0, 14, 1, -48)
	bottomBar.BackgroundTransparency = 1
	bottomBar.Parent = frame

	local backBtn = Instance.new("TextButton")
	backBtn.Size = UDim2.new(0, 90, 1, 0)
	backBtn.Position = UDim2.new(0, 0, 0, 0)
	backBtn.BackgroundColor3 = Color3.fromRGB(40, 42, 52)
	backBtn.Text = "← 무기"
	backBtn.TextColor3 = Color3.fromRGB(230, 230, 240)
	backBtn.Font = Enum.Font.Gotham
	backBtn.TextSize = 12
	backBtn.Visible = false
	backBtn.Parent = bottomBar
	Instance.new("UICorner", backBtn).CornerRadius = UDim.new(0, 5)

	local applyBtn = Instance.new("TextButton")
	applyBtn.Size = UDim2.new(0, 120, 1, 0)
	applyBtn.Position = UDim2.new(1, -120, 0, 0)
	applyBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 50)
	applyBtn.Text = "Apply"
	applyBtn.Visible = false -- 클릭 즉시 적용이라 불필요
	applyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	applyBtn.Font = Enum.Font.GothamBold
	applyBtn.TextSize = 13
	applyBtn.Visible = false
	applyBtn.Parent = bottomBar
	Instance.new("UICorner", applyBtn).CornerRadius = UDim.new(0, 5)

	local function clearContent()
		for _, c in ipairs(content:GetChildren()) do
			c:Destroy()
		end
	end

	local showSkins
	local function showWeapons()
		page = "weapons"
		selectedCos = nil
		clearContent()
		subtitle.Text = "무기를 한 개씩 눌러 변경"
		backBtn.Visible = false
		applyBtn.Visible = false

		local scroll = Instance.new("ScrollingFrame")
		scroll.Size = UDim2.new(1, 0, 1, 0)
		scroll.BackgroundTransparency = 1
		scroll.BorderSizePixel = 0
		scroll.ScrollBarThickness = 5
		scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
		scroll.Parent = content

		local grid = Instance.new("UIGridLayout")
		grid.CellSize = UDim2.new(0, 118, 0, 140)
		grid.CellPadding = UDim2.new(0, 10, 0, 10)
		grid.SortOrder = Enum.SortOrder.LayoutOrder
		grid.Parent = scroll

		for i, w in ipairs(weapons) do
			local card = Instance.new("TextButton")
			card.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
			card.Text = ""
			card.AutoButtonColor = true
			card.LayoutOrder = i
			card.Parent = scroll
			Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)
			local cs = Instance.new("UIStroke")
			cs.Color = Color3.fromRGB(50, 52, 64)
			cs.Thickness = 1
			cs.Parent = card

			-- 원형 아이콘 영역
			local circle = Instance.new("Frame")
			circle.Size = UDim2.new(0, 72, 0, 72)
			circle.Position = UDim2.new(0.5, -36, 0, 12)
			circle.BackgroundColor3 = Color3.fromRGB(40, 42, 52)
			circle.BorderSizePixel = 0
			circle.Parent = card
			Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

			local img = Instance.new("ImageLabel")
			img.Size = UDim2.new(1, -8, 1, -8)
			img.Position = UDim2.new(0, 4, 0, 4)
			img.BackgroundTransparency = 1
			img.ScaleType = Enum.ScaleType.Fit
			img.Parent = circle
			if w.Icon then
				img.Image = w.Icon
			else
				-- 무기 이니셜
				local init = Instance.new("TextLabel")
				init.Size = UDim2.new(1, 0, 1, 0)
				init.BackgroundTransparency = 1
				init.Text = string.sub(w.Name, 1, 1)
				init.TextColor3 = Color3.fromRGB(200, 200, 210)
				init.Font = Enum.Font.GothamBold
				init.TextSize = 28
				init.Parent = circle
			end

			local nameL = Instance.new("TextLabel")
			nameL.Size = UDim2.new(1, -8, 0, 36)
			nameL.Position = UDim2.new(0, 4, 1, -42)
			nameL.BackgroundTransparency = 1
			nameL.Text = w.Name
			nameL.TextColor3 = Color3.fromRGB(220, 222, 235)
			nameL.Font = Enum.Font.Gotham
			nameL.TextSize = 11
			nameL.TextWrapped = true
			nameL.TextYAlignment = Enum.TextYAlignment.Top
			nameL.Parent = card

			card.MouseButton1Click:Connect(function()
				selectedWeapon = w.Name
				showSkins()
			end)
		end

		task.defer(function()
			scroll.CanvasSize = UDim2.new(0, 0, 0, grid.AbsoluteContentSize.Y + 12)
		end)
	end

	showSkins = function()
		page = "skins"
		selectedCos = nil
		clearContent()
		subtitle.Text = tostring(selectedWeapon) .. "  ·  타입/스킨 선택"
		backBtn.Visible = true
		applyBtn.Visible = true

		local searchQuery = ""

		-- 검색창
		local searchBox = Instance.new("TextBox")
		searchBox.Size = UDim2.new(1, 0, 0, 30)
		searchBox.Position = UDim2.new(0, 0, 0, 0)
		searchBox.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
		searchBox.Text = ""
		searchBox.PlaceholderText = "검색... (이름 입력)"
		searchBox.PlaceholderColor3 = Color3.fromRGB(120, 125, 140)
		searchBox.TextColor3 = Color3.fromRGB(240, 240, 250)
		searchBox.Font = Enum.Font.Gotham
		searchBox.TextSize = 13
		searchBox.ClearTextOnFocus = false
		searchBox.TextXAlignment = Enum.TextXAlignment.Left
		searchBox.Parent = content
		Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 6)
		local sp = Instance.new("UIPadding")
		sp.PaddingLeft = UDim.new(0, 10)
		sp.Parent = searchBox

		-- 타입 탭
		local typeBar = Instance.new("ScrollingFrame")
		typeBar.Size = UDim2.new(1, 0, 0, 34)
		typeBar.Position = UDim2.new(0, 0, 0, 36)
		typeBar.BackgroundTransparency = 1
		typeBar.BorderSizePixel = 0
		typeBar.ScrollBarThickness = 0
		typeBar.CanvasSize = UDim2.new(0, 0, 0, 0)
		typeBar.Parent = content
		local typeLayout = Instance.new("UIListLayout")
		typeLayout.FillDirection = Enum.FillDirection.Horizontal
		typeLayout.Padding = UDim.new(0, 6)
		typeLayout.Parent = typeBar

		local skinScroll = Instance.new("ScrollingFrame")
		skinScroll.Size = UDim2.new(1, 0, 1, -78)
		skinScroll.Position = UDim2.new(0, 0, 0, 74)
		skinScroll.BackgroundTransparency = 1
		skinScroll.BorderSizePixel = 0
		skinScroll.ScrollBarThickness = 5
		skinScroll.Parent = content

		local skinGrid = Instance.new("UIGridLayout")
		skinGrid.CellSize = UDim2.new(0, 110, 0, 120)
		skinGrid.CellPadding = UDim2.new(0, 8, 0, 8)
		skinGrid.Parent = skinScroll

		-- 우측 큰 미리보기
		local previewFrame = Instance.new("Frame")
		previewFrame.Size = UDim2.new(0, 0, 0, 0) -- 목록 안에 통합 (카드로 표시)
		previewFrame.Visible = false
		previewFrame.Parent = content
		local previewImg = Instance.new("ImageLabel")
		previewImg.Size = UDim2.new(0, 1, 0, 1)
		previewImg.BackgroundTransparency = 1
		previewImg.Parent = previewFrame
		local previewName = Instance.new("TextLabel")
		previewName.Size = UDim2.new(0, 1, 0, 1)
		previewName.BackgroundTransparency = 1
		previewName.Text = ""
		previewName.Parent = previewFrame

		local function matchesWeapon(name, def, weapon)
			if not weapon or weapon == "" then return true end
			local w = string.lower(tostring(weapon)):gsub("%s+", " "):match("^%s*(.-)%s*$") or ""
			local n = string.lower(tostring(name))

			-- 1) 데이터 필드에 무기명 정확히 연결된 경우만 우선
			if type(def) == "table" then
				local keys = {"Weapon", "WeaponName", "Item", "ItemName", "Gun", "BaseItem", "SourceWeapon"}
				for _, k in ipairs(keys) do
					local v = def[k]
					if type(v) == "string" then
						local lv = string.lower(v):gsub("%s+", " "):match("^%s*(.-)%s*$") or ""
						if lv == w then return true end
					end
				end
				for _, k in ipairs({"Weapons", "Items", "CompatibleWeapons"}) do
					local v = def[k]
					if type(v) == "table" then
						for _, vv in pairs(v) do
							if type(vv) == "string" and string.lower(vv) == w then
								return true
							end
						end
					end
				end
			end

			-- 2) 스킨 이름이 "WeaponName ..." 또는 "... WeaponName" 형태
			--    공백/언더스코어 기준 전체 무기명 포함만 허용 (짧은 토큰 매칭 금지)
			if n == w then return true end
			if n:find(w, 1, true) then
				-- "assault rifle" 가 "burst assault" 같은 다른 총에 부분만 겹치면 제외
				-- 무기 풀네임이 스킨명에 연속으로 들어있을 때만
				return true
			end

			-- 3) 약어 매핑 (라이벌 영문/표기 차이)
			local aliases = {
				["assault rifle"] = {"assault rifle", "ar ", " ar", "rifle_assault"},
				["burst rifle"] = {"burst rifle", "burst"},
				["sniper"] = {"sniper"},
				["shotgun"] = {"shotgun"},
				["smg"] = {"smg", "submachine"},
				["lmg"] = {"lmg", "light machine"},
				["pistol"] = {"pistol"},
				["revolver"] = {"revolver"},
				["knife"] = {"knife", "melee"},
			}
			-- 선택 무기가 burst 계열이면 assault 스킨 제외, 반대도 제외
			local isBurst = w:find("burst", 1, true) ~= nil
			local isAssault = (w:find("assault", 1, true) ~= nil) and not isBurst
			if isAssault and n:find("burst", 1, true) then return false end
			if isBurst and n:find("assault", 1, true) and not n:find("burst", 1, true) then return false end

			-- 4) 필드/이름 둘 다 실패하면 비매칭 (전체 목록 폴백은 fillSkins에서)
			return false
		end

		local function fillSkins()
			for _, c in ipairs(skinScroll:GetChildren()) do
				if c:IsA("TextButton") then c:Destroy() end
			end
			local names = {}
			local weapon = selectedWeapon
			-- Skin만 무기 필터 / Wrap·Charm·Emote·Finisher 등은 공용 목록
			local filterByWeapon = (selectedType == "Skin")
			for name, def in pairs(api.coss) do
				if not api.banned(name) and type(def) == "table" then
					local tp = tostring(def.Type or "Skin")
					if string.lower(tp) == string.lower(tostring(selectedType)) then
						if (not filterByWeapon) or matchesWeapon(name, def, weapon) then
							if searchQuery == "" or string.lower(name):find(searchQuery, 1, true) then
								table.insert(names, name)
							end
						end
					end
				end
			end
			-- Skin 필터 결과 0이면 Type 문자열 대소문자/별칭 재시도
			if #names == 0 and selectedType == "Wrap" then
				for name, def in pairs(api.coss) do
					if not api.banned(name) and type(def) == "table" then
						local tp = string.lower(tostring(def.Type or ""))
						if tp:find("wrap") or tp:find("camo") or tp:find("skinwrap") then
							if searchQuery == "" or string.lower(name):find(searchQuery, 1, true) then
								table.insert(names, name)
							end
						end
					end
				end
			end
			if #names == 0 then
				subtitle.Text = tostring(selectedWeapon) .. "  ·  " .. tostring(selectedType) .. " 없음"
			else
				local tag = filterByWeapon and "무기 전용" or "공용"
				subtitle.Text = tostring(selectedWeapon) .. "  ·  " .. tostring(selectedType) .. " " .. tostring(#names) .. "개 (" .. tag .. ")"
			end
			table.sort(names)
			local function cosIcon(name, def)
				if type(def) ~= "table" then def = (api.coss and api.coss[name]) or {} end
				local found = {}

				local function consider(v, depth)
					if depth > 4 then return end
					if type(v) == "number" and v > 1000 then
						table.insert(found, v)
					elseif type(v) == "string" and #v > 0 then
						if v:find("rbxassetid://") or v:find("rbxthumb://") then
							table.insert(found, v)
							return
						end
						local id = v:match("(%d%d%d%d%d%d+)")
						if id then table.insert(found, tonumber(id) or id) end
						local n = tonumber(v)
						if n and n > 1000 then table.insert(found, n) end
					elseif type(v) == "table" then
						for _, sv in pairs(v) do
							consider(sv, depth + 1)
						end
					end
				end

				-- 우선 키
				local priority = {
					"Icon", "Image", "ImageId", "Thumbnail", "ThumbnailId", "DisplayIcon",
					"Preview", "TextureId", "Texture", "WrapTexture", "CamoTexture",
					"Diffuse", "Albedo", "Decal", "AssetId", "Id",
				}
				for _, k in ipairs(priority) do
					if def[k] ~= nil then consider(def[k], 0) end
				end
				-- 전체 스캔 (랩 데이터에 Texture 등이 깊게 박혀 있는 경우)
				if #found == 0 then
					consider(def, 0)
				end

				local id = found[1]
				if not id then return nil end
				if type(id) == "string" and (id:find("rbxasset") or id:find("rbxthumb")) then
					return id
				end
				local num = tonumber(id)
				if not num then return nil end
				-- rbxthumb 이 랩/텍스처 표시에 더 잘 됨
				return string.format("rbxthumb://type=Asset&id=%d&w=150&h=150", num)
			end

			local function applySelected(name)
				if not selectedWeapon or not name then return end
				local cloned = api.clonecos(name, selectedType)
				if not cloned then
					showNotification("적용 실패")
					return
				end
				api.equip[selectedWeapon] = api.equip[selectedWeapon] or {}
				api.equip[selectedWeapon][selectedType] = cloned
				if selectedType == "Finisher" then
					api.fcache[selectedWeapon] = name
				end
				api.rebuildinv()
				pcall(api.saferep, "WeaponInventory")
				pcall(api.saferep, "CosmeticInventory")
				selectedCos = name
				showNotification(selectedWeapon .. " ← " .. name)
				subtitle.Text = selectedWeapon .. "  ·  적용: " .. name
			end

			for _, name in ipairs(names) do
				local def = api.coss[name]
				local card = Instance.new("TextButton")
				card.BackgroundColor3 = Color3.fromRGB(28, 30, 40)
				card.Text = ""
				card.AutoButtonColor = true
				card.Parent = skinScroll
				Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)
				local stroke = Instance.new("UIStroke")
				stroke.Color = Color3.fromRGB(50, 52, 64)
				stroke.Thickness = 1
				stroke.Parent = card

				-- 미리보기 이미지
				local img = Instance.new("ImageLabel")
				img.Size = UDim2.new(1, -10, 1, -28)
				img.Position = UDim2.new(0, 5, 0, 4)
				img.BackgroundColor3 = Color3.fromRGB(22, 24, 32)
				img.BorderSizePixel = 0
				img.ScaleType = Enum.ScaleType.Fit
				img.Parent = card
				Instance.new("UICorner", img).CornerRadius = UDim.new(0, 6)
				local icon = cosIcon(name, def)
				-- 타입별 배경색 (Skin/Wrap/Charm/Emote/Finisher)
				local typeColors = {
					Skin = Color3.fromRGB(40, 55, 80),
					Wrap = Color3.fromRGB(45, 70, 50),
					Charm = Color3.fromRGB(70, 50, 75),
					Emote = Color3.fromRGB(75, 55, 40),
					Finisher = Color3.fromRGB(80, 40, 45),
				}
				local tp = tostring(selectedType or "Skin")
				img.BackgroundColor3 = typeColors[tp] or Color3.fromRGB(22, 24, 32)
				if icon then
					img.Image = icon
					img.ImageTransparency = 0
					-- 실패 시 rbxassetid 폴백
					if icon:find("rbxthumb") then
						local id = icon:match("id=(%d+)")
						if id then
							task.defer(function()
								if img and img.Parent and (img.Image == "" or img.IsLoaded == false) then
									-- 일부 실행기에선 thumb 실패 → assetid 재시도
									pcall(function()
										img.Image = "rbxassetid://" .. id
									end)
								end
							end)
						end
					end
				else
					img.Image = ""
					local init = Instance.new("TextLabel")
					init.Size = UDim2.new(1, 0, 1, -4)
					init.Position = UDim2.new(0, 0, 0, 0)
					init.BackgroundTransparency = 1
					init.Text = string.sub(tostring(name), 1, 3)
					init.TextColor3 = Color3.fromRGB(230, 230, 240)
					init.Font = Enum.Font.GothamBold
					init.TextSize = 20
					init.Parent = img
					local tag = Instance.new("TextLabel")
					tag.Size = UDim2.new(1, 0, 0, 14)
					tag.Position = UDim2.new(0, 0, 1, -16)
					tag.BackgroundTransparency = 1
					tag.Text = tp
					tag.TextColor3 = Color3.fromRGB(160, 165, 180)
					tag.Font = Enum.Font.Gotham
					tag.TextSize = 10
					tag.Parent = img
				end

				local nameL = Instance.new("TextLabel")
				nameL.Size = UDim2.new(1, -8, 0, 20)
				nameL.Position = UDim2.new(0, 4, 1, -22)
				nameL.BackgroundTransparency = 1
				nameL.Text = name
				nameL.TextColor3 = Color3.fromRGB(220, 222, 235)
				nameL.Font = Enum.Font.Gotham
				nameL.TextSize = 11
				nameL.TextTruncate = Enum.TextTruncate.AtEnd
				nameL.Parent = card

				card.MouseButton1Click:Connect(function()
					for _, o in ipairs(skinScroll:GetChildren()) do
						if o:IsA("TextButton") then
							local s = o:FindFirstChildOfClass("UIStroke")
							if s then s.Color = Color3.fromRGB(50, 52, 64) end
							o.BackgroundColor3 = Color3.fromRGB(28, 30, 40)
						end
					end
					card.BackgroundColor3 = Color3.fromRGB(55, 28, 35)
					stroke.Color = Color3.fromRGB(220, 50, 60)
					-- 큰 미리보기 갱신
					if previewImg then
						local ic = cosIcon(name, def)
						previewImg.Image = ic or ""
						previewName.Text = name
						if not ic then
							previewImg.Image = ""
						end
					end
					applySelected(name)
				end)
			end
			task.defer(function()
				skinScroll.CanvasSize = UDim2.new(0, 0, 0, skinGrid.AbsoluteContentSize.Y + 8)
			end)
		end


		for _, tp in ipairs(types) do
			local tb = Instance.new("TextButton")
			tb.Size = UDim2.new(0, 88, 0, 32)
			tb.BackgroundColor3 = (tp == selectedType) and Color3.fromRGB(180, 40, 50) or Color3.fromRGB(36, 38, 48)
			tb.Text = tp
			tb.TextColor3 = Color3.fromRGB(240, 240, 245)
			tb.Font = Enum.Font.Gotham
			tb.TextSize = 11
			tb.Parent = typeBar
			Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 5)
			tb.MouseButton1Click:Connect(function()
				selectedType = tp
				for _, o in ipairs(typeBar:GetChildren()) do
					if o:IsA("TextButton") then
						o.BackgroundColor3 = Color3.fromRGB(36, 38, 48)
					end
				end
				tb.BackgroundColor3 = Color3.fromRGB(180, 40, 50)
				fillSkins()
			end)
		end
		task.defer(function()
			typeBar.CanvasSize = UDim2.new(0, typeLayout.AbsoluteContentSize.X + 8, 0, 0)
		end)

		local searchDebounce = 0
		searchBox:GetPropertyChangedSignal("Text"):Connect(function()
			searchQuery = string.lower((searchBox.Text or ""):match("^%s*(.-)%s*$") or "")
			local id = tick()
			searchDebounce = id
			task.delay(0.15, function()
				if searchDebounce == id then
					fillSkins()
				end
			end)
		end)

		fillSkins()
	end

	backBtn.MouseButton1Click:Connect(function()
		showWeapons()
	end)

	applyBtn.MouseButton1Click:Connect(function()
		if not selectedWeapon then
			showNotification("무기 선택 필요")
			return
		end
		if not selectedCos then
			showNotification("스킨 선택 필요")
			return
		end
		local cloned = api.clonecos(selectedCos, selectedType)
		if not cloned then
			showNotification("적용 실패")
			return
		end
		api.equip[selectedWeapon] = api.equip[selectedWeapon] or {}
		api.equip[selectedWeapon][selectedType] = cloned
		if selectedType == "Finisher" then
			api.fcache[selectedWeapon] = selectedCos
		end
		api.rebuildinv()
		pcall(api.saferep, "WeaponInventory")
		pcall(api.saferep, "CosmeticInventory")
		showNotification(selectedWeapon .. " ← " .. selectedCos)
	end)

	-- 드래그
	local dragging, dragStart, startPos
	title.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
		end
	end)
	title.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)

	showWeapons()
	showNotification("Skin Picker 열림")
end

local function hasSpawnShield(player)
	if not player or not player.Character then return false end
	local char = player.Character
	if char:FindFirstChildOfClass("ForceField") then return true end
	for _, v in pairs(char:GetDescendants()) do
		if v:IsA("ParticleEmitter") or v:IsA("Beam") or v:IsA("Highlight") or v:IsA("Part") or v:IsA("MeshPart") then
			local n = tostring(v.Name):lower()
			if n:find("shield") or n:find("barrier") or n:find("force") or n:find("spawn") or n:find("protect") or n:find("invuln") then return true end
		end
	end
	return false
end

local _desyncHead = nil
local _desyncRealCF = nil
local _desyncRealVel = nil
local _lastHideTP = 0
local _hideRoamCF = nil -- 서버에 보낼 40000 위치 캐시

local function stopContinuousDesync()
	if continuousDesyncConn then continuousDesyncConn:Disconnect() continuousDesyncConn = nil end
	_desyncHead = nil
	_desyncRealCF = nil
	_desyncRealVel = nil
	_hideRoamCF = nil
	pcall(function() RunService:UnbindFromRenderStep("__nnnn_cont_restore") end)
end

local function setDesyncHead(targetHead)
	_desyncHead = targetHead
end

local function ensureDesyncRestore()
	pcall(function() RunService:UnbindFromRenderStep("__nnnn_cont_restore") end)
	-- 클라 화면: Heartbeat에서 저장한 실제 위치로 복구 (이동 가능)
	RunService:BindToRenderStep("__nnnn_cont_restore", Enum.RenderPriority.Camera.Value - 1, function()
		local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if hrp and _desyncRealCF then
			hrp.CFrame = _desyncRealCF
			hrp.AssemblyLinearVelocity = _desyncRealVel or Vector3.zero
		end
	end)
end

local function startContinuousDesync(targetHead)
	if targetHead then
		_desyncHead = targetHead
	end
	if continuousDesyncConn then return end

	ensureDesyncRestore()

	continuousDesyncConn = RunService.Heartbeat:Connect(function()
		if OrbitEnabled then return end
		if not (RageConfig.on or _rb.active) then return end

		local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if not hrp then return end

		-- 1) 지금 클라 위치 저장 (디싱크 복구용) → 이동 반영됨
		_desyncRealCF = hrp.CFrame
		_desyncRealVel = hrp.AssemblyLinearVelocity

		local head = _desyncHead
		local hasTarget = head and head.Parent

		-- 2) 서버 몸 위치
		if not hasTarget or _rb.isHiding then
			-- 40000 디싱크: 서버만 멀리, 화면은 위 복구로 그대로
			local now = tick()
			if now - _lastHideTP >= 0.1 or not _hideRoamCF then
				_lastHideTP = now
				local ang = math.random() * math.pi * 2
				local dist = 40000
				_hideRoamCF = CFrame.new(math.cos(ang) * dist, math.random(800, 6000), math.sin(ang) * dist)
			end
			hrp.CFrame = _hideRoamCF
			hrp.AssemblyLinearVelocity = Vector3.zero
			return
		end

		_hideRoamCF = nil
		-- ATTACK: 머리 바짝 + 시선 머리 (히트 유도)
		local targetPart = head.Parent:FindFirstChild("HumanoidRootPart")
		local predictedPos = head.Position
		if RageConfig.prediction and targetPart then
			local v = targetPart.AssemblyLinearVelocity
			predictedPos = predictedPos + Vector3.new(v.X, v.Y * 0.1, v.Z) * (RageConfig.predStrength or 0.12)
		end
		local height = RageConfig.height or 0.08
		local dist = Settings.TPDistance or 0.25
		local tpPos = predictedPos + Vector3.new(0, height, 0)
		if Settings.RageTP then
			tpPos = predictedPos + Vector3.new(0, math.max(0.05, dist), 0)
			hrp.CFrame = CFrame.new(tpPos, predictedPos) * CFrame.Angles(math.rad(-80), 0, 0)
		else
			hrp.CFrame = CFrame.lookAt(tpPos, predictedPos)
		end
		hrp.AssemblyLinearVelocity = Vector3.zero
	end)
end

local flyConnection
local function updateFly(dt)
	if not Settings.FlyEnabled then return end
	local char = LocalPlayer.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local hum = char and char:FindFirstChild("Humanoid")
	if not hrp or not hum then return end

	hum.PlatformStand = true
	local camCF = Camera.CFrame
	local moveDir = Vector3.zero

	if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += camCF.LookVector end
	if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= camCF.LookVector end
	if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir -= camCF.RightVector end
	if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir += camCF.RightVector end
	if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0, 1, 0) end
	if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir -= Vector3.new(0, 1, 0) end

	if moveDir.Magnitude > 0 then moveDir = moveDir.Unit * Settings.FlySpeed end

	hrp.CFrame = CFrame.new(hrp.Position + moveDir * dt, hrp.Position + moveDir * dt + camCF.LookVector)
	hrp.AssemblyLinearVelocity = Vector3.zero
end

local function toggleFly(state)
	Settings.FlyEnabled = state
	local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
	if state then
		if flyConnection then flyConnection:Disconnect() end
		flyConnection = RunService.RenderStepped:Connect(updateFly)
	else
		if flyConnection then flyConnection:Disconnect() flyConnection = nil end
		if hum then hum.PlatformStand = false end
	end
end

-- Noclip: 벽 통과 · 카메라/화면은 건드리지 않음
local noclipConnection = nil
local function toggleNoclip(state)
	Settings.NoclipEnabled = state and true or false
	if noclipConnection then
		pcall(function() noclipConnection:Disconnect() end)
		noclipConnection = nil
	end
	if not Settings.NoclipEnabled then
		-- 충돌 복구는 리스폰/기본에 맡김 (화면 흔들림 없음)
		return
	end
	noclipConnection = RunService.Stepped:Connect(function()
		if not Settings.NoclipEnabled then return end
		local char = LocalPlayer.Character
		if not char then return end
		-- 카메라 CFrame 절대 수정 안 함 · 충돌만 해제
		for _, name in ipairs({
			"HumanoidRootPart", "Head", "Torso", "UpperTorso", "LowerTorso",
			"Left Arm", "Right Arm", "Left Leg", "Right Leg",
			"LeftUpperArm", "LeftLowerArm", "LeftHand",
			"RightUpperArm", "RightLowerArm", "RightHand",
			"LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
			"RightUpperLeg", "RightLowerLeg", "RightFoot",
		}) do
			local part = char:FindFirstChild(name)
			if part and part:IsA("BasePart") then
				part.CanCollide = false
			end
		end
	end)
	nnnnTrack(noclipConnection)
end

local function startOrbit()
	if orbitConnection then orbitConnection:Disconnect() end
	local lastOrbitShot = 0
	local lastOrbitTP = 0
	orbitConnection = RunService.Heartbeat:Connect(function()
		if not OrbitEnabled then return end
		local char = LocalPlayer.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChild("Humanoid")
		if not hrp then return end

		local now = tick()
		-- Orbit 전용 이동 (Rage 디싱크와 분리)
		if now - lastOrbitTP > 0.08 then
			lastOrbitTP = now
			local rx = math.random(-40000, 40000)
			local ry = math.random(200, 8000)
			local rz = math.random(-40000, 40000)
			hrp.CFrame = CFrame.new(rx, ry, rz)
			hrp.AssemblyLinearVelocity = Vector3.zero
		end
		if hum then
			hum.PlatformStand = true
			hum.AutoRotate = false
		end

		-- 사격만: Rage가 켜져 있으면 Rage 루프가 쏘므로 여기서 안 쏨 (이중 발사 충돌 방지)
		if RageConfig.on and _rb.active then
			return
		end

		local target = getRageTarget()
		if target then
			local head = getHead(target)
			if head then
				setDesyncHead(head)
				if now - lastOrbitShot >= (RageConfig.fireRate or 0.012) then
					lastOrbitShot = now
					pcall(ShootRemote, target)
					pcall(doMelee)
				end
			end
		end
	end)
end

local function stopOrbit()
	OrbitEnabled = false
	if orbitConnection then orbitConnection:Disconnect() orbitConnection = nil end
	stopContinuousDesync()
	local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
	if hum then
		hum.PlatformStand = false
		hum.AutoRotate = true
	end
end

local function isSpectator(p)
	if not p then return true end
	-- 명시적 관전 플래그
	local ok, v = pcall(function()
		return p:GetAttribute("Spectating")
			or p:GetAttribute("IsSpectator")
			or p:GetAttribute("Spectator")
			or p:GetAttribute("InMenu")
			or p:GetAttribute("IsInMenu")
	end)
	if ok and v then return true end
	-- 팀 이름에 spectator / lobby
	pcall(function()
		local tn = p.Team and tostring(p.Team.Name):lower() or ""
		if tn:find("spect", 1, true) or tn:find("lobby", 1, true) or tn:find("menu", 1, true) then
			v = true
		end
	end)
	if v then return true end
	-- 캐릭터가 월드에 없음 / 고스트
	local char = p.Character
	if not char or not char.Parent then return true end
	if char.Parent ~= Workspace and not char:IsDescendantOf(Workspace) then return true end
	-- 이름/상태
	local nm = string.lower(tostring(p.Name or "") .. " " .. tostring(p.DisplayName or ""))
	if nm:find("spectat", 1, true) then return true end
	return false
end

local function IsEnemy(p)
	if p == LocalPlayer or not p then return false end
	if isSpectator(p) then return false end
	local char = p.Character
	if not char or not char.Parent then return false end
	if not char:IsDescendantOf(Workspace) then return false end
	local h = char:FindFirstChildOfClass("Humanoid")
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not h or not hrp then return false end
	if h.Health <= 0 then return false end
	-- 죽은 상태 / 관전 카메라
	pcall(function()
		if h:GetState() == Enum.HumanoidStateType.Dead then
			h = nil
		end
	end)
	if not h then return false end
	-- TeamCheck
	if Settings.TeamCheck then
		local myTeam = LocalPlayer:GetAttribute("TeamID")
		local theirTeam = p:GetAttribute("TeamID")
		if myTeam ~= nil and theirTeam ~= nil and myTeam == theirTeam then
			return false
		end
		if LocalPlayer.Team and p.Team and LocalPlayer.Team == p.Team then
			return false
		end
	end
	-- Head 없으면 전투 캐릭터 아님
	if not char:FindFirstChild("Head") and not char:FindFirstChild("HitboxHead") then
		return false
	end
	return true
end

local function getHead(p)
	return p.Character and p.Character:FindFirstChild("Head")
end

local function isReloading()
	local r = false
	pcall(function()
		local f = Fighter and Fighter.LocalFighter
		if f and f.EquippedItem then r = f.EquippedItem:Get("IsReloading") or false end
	end)
	return r
end

local function isMelee()
	local n = ""
	pcall(function()
		local f = Fighter and Fighter.LocalFighter
		local item = f and f.EquippedItem
		if item then
			n = tostring(item:Get("Name") or item.Name or ""):lower()
		end
	end)
	if n == "" then
		pcall(function()
			local char = LocalPlayer.Character
			if not char then return end
			for _, tool in ipairs(char:GetChildren()) do
				if tool:IsA("Tool") then
					n = tool.Name:lower()
					break
				end
			end
		end)
	end
	return n:find("knife")
		or n:find("katana")
		or n:find("fist")
		or n:find("bat")
		or n:find("melee")
		or n:find("punch")
		or n:find("sword")
		or n:find("axe")
		or n:find("dagger")
		or n:find("scythe")
		or n:find("claw")
end

local function doMelee()
	local char = LocalPlayer.Character
	if not char then return end
	for _, tool in ipairs(char:GetChildren()) do
		if tool:IsA("Tool") then pcall(function() tool:Activate() end) end
	end
	pcall(function()
		local f = Fighter and Fighter.LocalFighter
		local item = f and f.EquippedItem
		if item and item.Punch then
			item:Punch()
		end
	end)
end

-- 총알 트레이서 (근접 무기/주먹 장착 시 작동 안 함)
local _tracerFolder
local function getTracerFolder()
	if _tracerFolder and _tracerFolder.Parent then return _tracerFolder end
	_tracerFolder = Workspace:FindFirstChild("__NnnnTracers")
	if not _tracerFolder then
		_tracerFolder = Instance.new("Folder")
		_tracerFolder.Name = "__NnnnTracers"
		_tracerFolder.Parent = Workspace
	end
	return _tracerFolder
end

local function rainbowSequence()
	return ColorSequence.new({
		ColorSequenceKeypoint.new(0.00, Color3.fromHSV(0.00, 1, 1)),
		ColorSequenceKeypoint.new(0.16, Color3.fromHSV(0.16, 1, 1)),
		ColorSequenceKeypoint.new(0.33, Color3.fromHSV(0.33, 1, 1)),
		ColorSequenceKeypoint.new(0.50, Color3.fromHSV(0.50, 1, 1)),
		ColorSequenceKeypoint.new(0.66, Color3.fromHSV(0.66, 1, 1)),
		ColorSequenceKeypoint.new(0.83, Color3.fromHSV(0.83, 1, 1)),
		ColorSequenceKeypoint.new(1.00, Color3.fromHSV(1.00, 1, 1)),
	})
end

local function createBulletTracer(fromPos, toPos)
	if not Settings.BulletTracer then return end
	if isMelee() then return end -- 주먹이나 유틸리티/근접무기일 경우 트레이서 적용 안 함
	if not fromPos or not toPos then return end
	local dist = (toPos - fromPos).Magnitude
	if dist < 1 or dist > 2000 then return end

	pcall(function()
		local folder = getTracerFolder()
		local thickness = math.clamp(Settings.TracerThickness or 0.08, 0.04, 0.4)
		local duration = math.clamp(Settings.TracerDuration or 0.28, 0.1, 1.5)
		local mid = (fromPos + toPos) / 2
		local cf = CFrame.lookAt(mid, toPos)

		local bar = Instance.new("Part")
		bar.Name = "TracerBar"
		bar.Anchored = true
		bar.CanCollide = false
		bar.CanQuery = false
		bar.CanTouch = false
		bar.Material = Enum.Material.Neon
		bar.Size = Vector3.new(thickness, thickness, dist)
		bar.CFrame = cf
		bar.CastShadow = false
		if Settings.TracerMode == "Solid" then
			bar.Color = Settings.TracerSolidColor or Color3.fromRGB(100, 255, 220)
		else
			bar.Color = Color3.fromHSV((tick() * 0.4) % 1, 1, 1)
		end
		bar.Parent = folder

		local p0 = Instance.new("Part")
		p0.Anchored = true
		p0.CanCollide = false
		p0.CanQuery = false
		p0.Transparency = 1
		p0.Size = Vector3.new(0.1, 0.1, 0.1)
		p0.Position = fromPos
		p0.Parent = folder
		local p1 = Instance.new("Part")
		p1.Anchored = true
		p1.CanCollide = false
		p1.CanQuery = false
		p1.Transparency = 1
		p1.Size = Vector3.new(0.1, 0.1, 0.1)
		p1.Position = toPos
		p1.Parent = folder
		local a0 = Instance.new("Attachment", p0)
		local a1 = Instance.new("Attachment", p1)
		local beam = Instance.new("Beam")
		beam.Attachment0 = a0
		beam.Attachment1 = a1
		beam.FaceCamera = true
		beam.LightEmission = 1
		beam.LightInfluence = 0
		beam.Width0 = thickness * 1.6
		beam.Width1 = thickness * 0.4
		beam.Segments = 20
		beam.Texture = ""
		if Settings.TracerMode == "Solid" then
			local c = Settings.TracerSolidColor or Color3.fromRGB(100, 255, 220)
			beam.Color = ColorSequence.new(c)
			beam.Transparency = NumberSequence.new(0.15)
		else
			beam.Color = rainbowSequence()
			beam.Transparency = NumberSequence.new(0.05)
		end
		beam.Parent = p0

		task.delay(duration * 0.5, function()
			pcall(function()
				TweenService:Create(bar, TweenInfo.new(duration * 0.5), {Transparency = 1}):Play()
				TweenService:Create(beam, TweenInfo.new(duration * 0.5), {
					Width0 = 0,
					Width1 = 0,
				}):Play()
			end)
		end)
		task.delay(duration + 0.08, function()
			pcall(function() bar:Destroy() end)
			pcall(function() p0:Destroy() end)
			pcall(function() p1:Destroy() end)
		end)
	end)
end

local function tracerFromLocalTo(pos)
	if isMelee() then return end
	local cam = Workspace.CurrentCamera
	local origin = cam and (cam.CFrame.Position + cam.CFrame.LookVector * 2) or nil
	if not origin then
		local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		origin = hrp and (hrp.Position + Vector3.new(0, 1.5, 0))
	end
	if origin and pos then
		createBulletTracer(origin, pos)
	end
end

local _lastAnyTracer = 0
nnnnTrack(UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if not Settings.BulletTracer or isMelee() then return end
	if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
	local now = tick()
	if now - _lastAnyTracer < 0.05 then return end
	_lastAnyTracer = now

	local cam = Workspace.CurrentCamera
	if not cam then return end
	local origin = cam.CFrame.Position + cam.CFrame.LookVector * 1.5
	local dir = cam.CFrame.LookVector * 800
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	local filter = {LocalPlayer.Character}
	pcall(function()
		local f = Workspace:FindFirstChild("__NnnnTracers")
		if f then table.insert(filter, f) end
	end)
	params.FilterDescendantsInstances = filter
	local hit = Workspace:Raycast(origin, dir, params)
	local toPos
	if hit then
		toPos = hit.Position
	else
		toPos = origin + cam.CFrame.LookVector * 250
	end
	createBulletTracer(origin, toPos)
end))

local function ShootRemote(target)
	if not target or not target.Character or isReloading() then return end
	if not Utility or not EnumLib then return end
	local head = getHead(target)
	if not head then return end
	local item
	pcall(function()
		local f = Fighter and (Fighter.LocalFighter or (Fighter.GetFighter and Fighter:GetFighter(LocalPlayer)))
		item = f and f.EquippedItem
	end)
	if not item then return end
	local oid
	pcall(function()
		oid = (item.Get and item:Get("ObjectID")) or item.ObjectID
	end)
	if not oid then return end

	-- 예측 + 머리 중심
	local thrp = target.Character:FindFirstChild("HumanoidRootPart")
	local vel = Vector3.zero
	pcall(function()
		vel = (thrp and thrp.AssemblyLinearVelocity) or head.AssemblyLinearVelocity or Vector3.zero
	end)
	local pred = head.Position + Vector3.new(vel.X, vel.Y * 0.15, vel.Z) * (RageConfig.predStrength or 0.12)
	pred = pred + Vector3.new(0, 0.08, 0)

	-- 조준 원점: 카메라 약간 앞 (총구 느낌)
	local cam = Workspace.CurrentCamera
	local myHrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
	local origin = (cam and cam.CFrame.Position) or (myHrp and myHrp.Position) or pred
	if cam then
		origin = cam.CFrame.Position + cam.CFrame.LookVector * 0.5
	end
	-- 어택 중이면 서버 몸이 적 머리 위 → origin을 머리 근처로
	if _rb.active and not _rb.isHiding then
		origin = pred + Vector3.new(0, 0.2, 0)
	end

	local aimCF = CFrame.lookAt(origin, pred)

	if not isMelee() then
		pcall(tracerFromLocalTo, pred)
	end

	local rem
	pcall(function()
		rem = ReplicatedStorage.Remotes.Replication.Fighter.UseItem
	end)
	if not rem then return end

	local function fireOnce()
		pcall(function()
			local data = {
				[utf8.char(1)] = {
					[utf8.char(0)] = Utility:EncodeCFrame(aimCF),
					[utf8.char(1)] = Utility:EncodeCFrame(aimCF),
					[utf8.char(2)] = head,
					[utf8.char(3)] = Utility:EncodeCFrame(CFrame.new(0.4, 0.2, 0.4)),
				},
			}
			rem:FireServer(oid, EnumLib:ToEnum("StartShooting"), data, nil)
		end)
	end

	-- 버스트: 연타로 히트율 상승
	fireOnce()
	task.defer(fireOnce)
	task.delay(0.02, fireOnce)
end

local function getRageTarget()
	local now = tick()
	-- 타겟 유지 짧게 (0.08s) — 죽은 적 붙잡기 방지
	if now - _rb.lastTargetCheck < 0.08 and _rb.target then
		if IsEnemy(_rb.target) and not (Settings.WaitForShield and hasSpawnShield(_rb.target)) then
			return _rb.target
		end
		_rb.target = nil
	end
	_rb.lastTargetCheck = now
	local best, bestScore = nil, math.huge
	local myPos = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
	myPos = myPos and myPos.Position
	for _, p in ipairs(Players:GetPlayers()) do
		if not IsEnemy(p) then continue end
		if Settings.WaitForShield and hasSpawnShield(p) then continue end
		local head = getHead(p)
		if not head then continue end
		local hum = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then continue end
		local d = myPos and (head.Position - myPos).Magnitude or 999
		-- 거리 + 체력 가중 (약한 적 우선)
		local score = d + (hum.Health / math.max(1, hum.MaxHealth)) * 40
		if score < bestScore then
			bestScore = score
			best = p
		end
	end
	_rb.target = best
	return best
end

local function startRage()
	if not RageConfig.on then return end
	_rb.gen += 1
	local myGen = _rb.gen
	_rb.active = true
	_rb.readyAt = 0 -- 대기 없이 즉시
	_rb.target = nil
	_rb.lastHide = tick()
	_rb.isHiding = false -- 시작하자마자 따라붙기 (40000 히든 먼저 X)
	_rb.lastShot = 0

	RageConfig.fireRate = 0.0001
	RageConfig.desync = true
	applyFastFireAndMelee(true)
	updateRageHUD("")

	-- 켜자마자 타겟 있으면 바로 붙고, 없으면 대기
	pcall(function()
		local t = getRageTarget and getRageTarget()
		local head = t and getHead and getHead(t)
		startContinuousDesync(head)
	end)

	task.spawn(function()
		while _rb.active and myGen == _rb.gen do
			local now = tick()
			local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			if not hrp then task.wait(0.03) continue end

			if isReloading() then task.wait(0.01) continue end

			-- 히든 먼저 → 시간 지나면 어택 → 다시 히든
			if _rb.isHiding then
				if now - _rb.lastHide >= (RageConfig.hide or 0.05) then
					_rb.isHiding = false
					_rb.lastHide = now
				end
			else
				if now - _rb.lastHide >= (RageConfig.attack or 0.88) then
					_rb.isHiding = true
					_rb.lastHide = now
				end
			end

			local target = getRageTarget()
			if target then
				local head = getHead(target)
				if head then
					if Settings.WaitForShield and hasSpawnShield(target) then
						updateRageHUD("")
						task.wait(0.03)
						continue
					end

					updateRageHUD(target.Name or "")

					-- 타겟 있으면 바로 어택 붙기
					if not OrbitEnabled then
						startContinuousDesync(head)
					else
						setDesyncHead(head)
					end

					if not _rb.isHiding then
						-- 어택 중 초고속 사격
						if now - _rb.lastShot >= (RageConfig.fireRate or 0.0001) then
							_rb.lastShot = now
							pcall(ShootRemote, target)
							pcall(doMelee)
							-- 한 틱에 한 번 더
							task.defer(function()
								pcall(ShootRemote, target)
							end)
						end
					end
				end
			else
				-- 죽일 사람 없음 → 40000 배회 유지 (stop 하지 않음)
				updateRageHUD("")
				_rb.isHiding = true
				_desyncHead = nil
				if not OrbitEnabled then
					startContinuousDesync(nil)
				end
			end

			task.wait(0.006)
		end
	end)
end

local function stopRage()
	_rb.gen += 1
	_rb.active = false
	RageConfig.on = false
	_rb.target = nil
	stopContinuousDesync()
	if not Settings.RapidFire then
		applyFastFireAndMelee(false)
	end
	updateRageHUD("")
	task.delay(0.8, function()
		if not RageConfig.on then destroyRageHUD() end
	end)
end

local ESPDrawings = {}
local lastESP = 0

local SKELETON_BONES = {
	{"Head", "UpperTorso"},
	{"Head", "Torso"},
	{"UpperTorso", "LowerTorso"},
	{"UpperTorso", "LeftUpperArm"},
	{"LeftUpperArm", "LeftLowerArm"},
	{"LeftLowerArm", "LeftHand"},
	{"UpperTorso", "RightUpperArm"},
	{"RightUpperArm", "RightLowerArm"},
	{"RightLowerArm", "RightHand"},
	{"LowerTorso", "LeftUpperLeg"},
	{"LeftUpperLeg", "LeftLowerLeg"},
	{"LeftLowerLeg", "LeftFoot"},
	{"LowerTorso", "RightUpperLeg"},
	{"RightUpperLeg", "RightLowerLeg"},
	{"RightLowerLeg", "RightFoot"},
	-- R6
	{"Head", "Torso"},
	{"Torso", "Left Arm"},
	{"Torso", "Right Arm"},
	{"Torso", "Left Leg"},
	{"Torso", "Right Leg"},
}

local function clearESP(p)
	local objs = ESPDrawings[p]
	if not objs then return end
	for k, d in pairs(objs) do
		if k == "Bones" then
			for _, line in pairs(d) do
				pcall(function() line:Remove() end)
			end
		else
			pcall(function() d:Remove() end)
		end
	end
	ESPDrawings[p] = nil
end

local function hideESP(objs)
	if not objs then return end
	objs.Box.Visible = false
	objs.Info.Visible = false
	objs.Dist.Visible = false
	objs.HpBg.Visible = false
	objs.HpFill.Visible = false
	if objs.Bones then
		for _, line in pairs(objs.Bones) do
			line.Visible = false
		end
	end
end

local function ensureESP(p)
	if ESPDrawings[p] then return ESPDrawings[p] end
	if not Drawing then return nil end
	local box = Drawing.new("Square")
	box.Thickness = 1 box.Filled = false box.Color = Color3.fromRGB(255, 255, 255) box.Visible = false
	local infoText = Drawing.new("Text")
	infoText.Size = 13 infoText.Center = true infoText.Outline = true infoText.Color = Color3.fromRGB(255, 255, 255) infoText.Visible = false
	local distText = Drawing.new("Text")
	distText.Size = 13 distText.Center = true distText.Outline = true distText.Color = Color3.fromRGB(255, 255, 255) distText.Visible = false
	-- 체력바 (배경 + 채움)
	local hpBg = Drawing.new("Square")
	hpBg.Filled = true hpBg.Thickness = 0 hpBg.Color = Color3.fromRGB(30, 30, 30) hpBg.Visible = false
	local hpFill = Drawing.new("Square")
	hpFill.Filled = true hpFill.Thickness = 0 hpFill.Color = Color3.fromRGB(80, 255, 80) hpFill.Visible = false
	-- 스켈레톤 라인
	local bones = {}
	for i = 1, 20 do
		local line = Drawing.new("Line")
		line.Thickness = 1.5
		line.Color = Color3.fromRGB(255, 255, 255)
		line.Visible = false
		bones[i] = line
	end
	ESPDrawings[p] = {
		Box = box, Info = infoText, Dist = distText,
		HpBg = hpBg, HpFill = hpFill, Bones = bones,
	}
	return ESPDrawings[p]
end

local function worldToScreen(v3)
	local sp, on = Camera:WorldToViewportPoint(v3)
	return Vector2.new(sp.X, sp.Y), on and sp.Z > 0
end

RunService.Heartbeat:Connect(function()
	if not ESPConfig.enabled then
		for p, objs in pairs(ESPDrawings) do
			hideESP(objs)
		end
		return
	end
	if tick() - lastESP < 0.04 then return end
	lastESP = tick()

	for _, p in ipairs(Players:GetPlayers()) do
		if p == LocalPlayer then continue end
		local objs = ensureESP(p)
		if not objs then continue end

		local char = p.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local head = char and char:FindFirstChild("Head")
		local hum = char and char:FindFirstChildWhichIsA("Humanoid")
		if not hrp or not head or not hum or hum.Health <= 0 then
			hideESP(objs)
			continue
		end

		local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
		if not onScreen or pos.Z < 0 then
			hideESP(objs)
			continue
		end

		local top = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.4, 0))
		local bottom = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 2.5, 0))
		local height = math.max(math.abs(top.Y - bottom.Y), 8)
		local width = height * 0.5
		local x = pos.X - width / 2

		if ESPConfig.box then
			objs.Box.Size = Vector2.new(width, height)
			objs.Box.Position = Vector2.new(x, top.Y)
			objs.Box.Visible = true
		else
			objs.Box.Visible = false
		end

		-- 체력바 (박스 왼쪽)
		if ESPConfig.healthBar then
			local ratio = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
			local barW = 3
			local barH = height
			local barX = x - barW - 3
			local barY = top.Y
			objs.HpBg.Size = Vector2.new(barW, barH)
			objs.HpBg.Position = Vector2.new(barX, barY)
			objs.HpBg.Visible = true
			local fillH = barH * ratio
			objs.HpFill.Size = Vector2.new(barW, fillH)
			objs.HpFill.Position = Vector2.new(barX, barY + (barH - fillH))
			-- 초록→노랑→빨강
			local r = ratio < 0.5 and 1 or (1 - ratio) * 2
			local g = ratio > 0.5 and 1 or ratio * 2
			objs.HpFill.Color = Color3.new(math.clamp(r, 0, 1), math.clamp(g, 0, 1), 0.15)
			objs.HpFill.Visible = true
		else
			objs.HpBg.Visible = false
			objs.HpFill.Visible = false
		end

		local infoParts = {}
		if ESPConfig.name then table.insert(infoParts, p.Name) end
		if ESPConfig.health then table.insert(infoParts, math.floor(hum.Health) .. "HP") end
		if #infoParts > 0 then
			objs.Info.Text = table.concat(infoParts, " | ")
			objs.Info.Position = Vector2.new(pos.X, top.Y - 16)
			objs.Info.Visible = true
		else
			objs.Info.Visible = false
		end

		if ESPConfig.distance then
			local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			local d = myHRP and (hrp.Position - myHRP.Position).Magnitude or 0
			objs.Dist.Text = math.floor(d) .. "m"
			objs.Dist.Position = Vector2.new(pos.X, bottom.Y + 2)
			objs.Dist.Visible = true
		else
			objs.Dist.Visible = false
		end

		-- 스켈레톤
		if ESPConfig.skeleton and objs.Bones then
			local bi = 0
			for _, pair in ipairs(SKELETON_BONES) do
				local a = char:FindFirstChild(pair[1])
				local b = char:FindFirstChild(pair[2])
				if a and b and a:IsA("BasePart") and b:IsA("BasePart") then
					local sa, oa = worldToScreen(a.Position)
					local sb, ob = worldToScreen(b.Position)
					if oa and ob then
						bi += 1
						local line = objs.Bones[bi]
						if line then
							line.From = sa
							line.To = sb
							line.Visible = true
						end
					end
				end
			end
			for i = bi + 1, #objs.Bones do
				objs.Bones[i].Visible = false
			end
		elseif objs.Bones then
			for _, line in pairs(objs.Bones) do
				line.Visible = false
			end
		end
	end
end)
Players.PlayerRemoving:Connect(clearESP)

local FOVCircle
pcall(function()
	if Drawing then
		FOVCircle = Drawing.new("Circle")
		FOVCircle.Thickness = 1.5 FOVCircle.NumSides = 64 FOVCircle.Filled = false
		FOVCircle.Visible = false FOVCircle.Color = Color3.fromRGB(255, 60, 60) FOVCircle.ZIndex = 999
	end
end)

-- FOV 안 · 크로스헤어에 가장 가까운 적 머리
local function getClosest()
	local closest, closestDist = nil, math.huge
	local mousePos = UserInputService:GetMouseLocation()
	local cam = Workspace.CurrentCamera
	if not cam then return nil end
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr == LocalPlayer then continue end
		if Settings.TeamCheck then
			local myTeam = LocalPlayer:GetAttribute("TeamID")
			local theirTeam = plr:GetAttribute("TeamID")
			if myTeam ~= nil and theirTeam ~= nil and myTeam == theirTeam then continue end
			if plr.Team and LocalPlayer.Team and plr.Team == LocalPlayer.Team then continue end
		end
		local char = plr.Character
		if not char then continue end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then continue end
		local head = char:FindFirstChild("Head")
		if not head then continue end
		local screenPoint, onScreen = cam:WorldToViewportPoint(head.Position)
		if screenPoint.Z < 0 then continue end
		-- FOV: 마우스 기준 (화면 밖이어도 Z>0 이면 거리만 체크)
		local distToMouse = (Vector2.new(screenPoint.X, screenPoint.Y) - mousePos).Magnitude
		local fov = Settings.FOVRadius or 220
		if distToMouse <= fov and distToMouse < closestDist then
			closestDist = distToMouse
			closest = head
		end
	end
	return closest
end

RunService:BindToRenderStep("__nnnn_forced_shake", 300, function()
	if Settings.RageEnabled and Settings.IntenseViewShake then
		local rx = math.rad(math.random(-45, 45))
		local ry = math.rad(math.random(-45, 45))
		local rz = math.rad(math.random(-45, 45))
		Camera.CFrame = Camera.CFrame * CFrame.Angles(rx, ry, rz)
	end
end)

-- ============================================================
-- Silent Aim (Rivals) — 카메라 고정, UseItem 조준만 Head로 후킹
-- ============================================================
local _lastTracerShot = 0
local _silentReady = false

local function SA_GetUseItem()
	local ok, rem = pcall(function()
		return ReplicatedStorage.Remotes.Replication.Fighter.UseItem
	end)
	if ok and rem and rem:IsA("RemoteEvent") then return rem end
	return nil
end

local function SA_GetObjectId()
	local oid
	pcall(function()
		if not Fighter then return end
		local lf = Fighter.LocalFighter or (Fighter.GetFighter and Fighter:GetFighter(LocalPlayer))
		if not lf then return end
		local item = lf.EquippedItem
		if item then
			oid = (item.Get and item:Get("ObjectID")) or item.ObjectID
		end
		if not oid and lf.Items then
			for _, it in pairs(lf.Items) do
				local eq = (it.Get and it:Get("IsEquipped")) or it.IsEquipped
				if eq then
					oid = (it.Get and it:Get("ObjectID")) or it.ObjectID
					break
				end
			end
		end
	end)
	return oid
end

local function SA_IsEnemy(plr)
	if not plr or plr == LocalPlayer then return false end
	if isSpectator(plr) then return false end
	local char = plr.Character
	if not char or not char:IsDescendantOf(Workspace) then return false end
	if not Settings.TeamCheck then return true end
	local myT = LocalPlayer:GetAttribute("TeamID")
	local thT = plr:GetAttribute("TeamID")
	if myT ~= nil and thT ~= nil and myT == thT then return false end
	if plr.Team and LocalPlayer.Team and plr.Team == LocalPlayer.Team then return false end
	return true
end

local function SA_GetClosestHead()
	local cam = Workspace.CurrentCamera
	if not cam then return nil end
	local mouse = UserInputService:GetMouseLocation()
	local best, bestD = nil, Settings.FOVRadius or 220
	for _, plr in ipairs(Players:GetPlayers()) do
		if not SA_IsEnemy(plr) then continue end
		local char = plr.Character
		if not char then continue end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then continue end
		local head = char:FindFirstChild("Head")
		if not head then continue end
		local sp, onScreen = cam:WorldToViewportPoint(head.Position)
		if sp.Z < 0 then continue end
		local d = (Vector2.new(sp.X, sp.Y) - mouse).Magnitude
		if d <= bestD then
			bestD = d
			best = head
		end
	end
	return best
end

-- getClosest 호환 (다른 코드에서 사용)
getClosest = SA_GetClosestHead

local function SA_BuildData(head)
	if not head or not head.Parent then return nil end
	if not Utility or not Utility.EncodeCFrame then return nil end

	local thrp = head.Parent:FindFirstChild("HumanoidRootPart")
	local vel = Vector3.zero
	pcall(function()
		vel = (thrp and thrp.AssemblyLinearVelocity) or head.AssemblyLinearVelocity or Vector3.zero
	end)

	-- 예측
	local headPos = head.Position + Vector3.new(vel.X, vel.Y * 0.2, vel.Z) * 0.11 + Vector3.new(0, 0.05, 0)

	local cam = Workspace.CurrentCamera
	local myHrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
	local origin = (cam and cam.CFrame.Position) or (myHrp and myHrp.Position) or headPos
	-- 총구 약간 앞
	if cam then
		origin = cam.CFrame.Position + cam.CFrame.LookVector * 0.8
	end

	local aimCF = CFrame.new(origin, headPos)
	local enc = Utility:EncodeCFrame(aimCF)
	local hitEnc = Utility:EncodeCFrame(CFrame.new(0.43, 0.25, 0.42))
	pcall(function()
		hitEnc = Utility:EncodeCFrame(head.CFrame:ToObjectSpace(CFrame.new(headPos)))
	end)

	return {
		[utf8.char(1)] = {
			[utf8.char(0)] = enc,
			[utf8.char(1)] = enc,
			[utf8.char(2)] = head, -- 타겟 파트 (Head)
			[utf8.char(3)] = hitEnc,
		},
	}, headPos
end

local function SA_FireAt(head)
	if not head then return end
	local rem = SA_GetUseItem()
	if not rem then return end
	if not EnumLib then return end
	local oid = SA_GetObjectId()
	if not oid then return end
	local data, headPos = SA_BuildData(head)
	if not data then return end
	pcall(function()
		rem:FireServer(oid, EnumLib:ToEnum("StartShooting"), data, nil)
	end)
	if Settings.BulletTracer and headPos then
		local now = tick()
		if now - _lastTracerShot >= 0.06 then
			_lastTracerShot = now
			pcall(function()
				if not isMelee or not isMelee() then
					tracerFromLocalTo(headPos)
				end
			end)
		end
	end
end

local function setupSilentAimHook()
	if _silentReady then return end
	if not hookmetamethod or not getnamecallmethod then
		warn("[nnnn] silent: hook 미지원 실행기")
		return
	end
	_silentReady = true

	local wrap = newcclosure or function(f) return f end
	local old
	old = hookmetamethod(game, "__namecall", wrap(function(self, ...)
		local method = getnamecallmethod()
		local args = { ... }

		if method == "FireServer" and Settings.AimbotEnabled then
			local isUse = false
			pcall(function()
				if typeof(self) == "Instance" and self.Name == "UseItem" then
					isUse = true
				end
			end)
			if not isUse then
				local rem = SA_GetUseItem()
				if rem and self == rem then isUse = true end
			end

			if isUse then
				local head = CurrentTarget
				if not (head and head.Parent) then
					head = SA_GetClosestHead()
				end
				if head and head.Parent then
					if head.Name ~= "Head" then
						head = head.Parent:FindFirstChild("Head") or head
					end
					local hc = Settings.HitChance or 100
					if hc >= 100 or math.random(1, 100) <= hc then
						local data = SA_BuildData(head)
						if data then
							-- (ObjectID, Action, shotData, ...)
							if type(args[3]) == "table" or args[3] == nil then
								args[3] = data
							elseif type(args[2]) == "table" then
								args[2] = data
							else
								args[3] = data
							end
						end
					end
				end
				return old(self, table.unpack(args))
			end
		end
		return old(self, ...)
	end))
	print("[nnnn] silent aim hooked")
end

-- LMB: 훅이 실패해도 보조로 Head 사격 데이터 전송
local _saLmb = 0
nnnnTrack(UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if not Settings.AimbotEnabled then return end
	if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
	local now = tick()
	if now - _saLmb < 0.04 then return end
	_saLmb = now
	task.defer(function()
		if not Settings.AimbotEnabled then return end
		local head = CurrentTarget
		if not (head and head.Parent) then head = SA_GetClosestHead() end
		if not head then return end
		if head.Name ~= "Head" then
			head = head.Parent and head.Parent:FindFirstChild("Head") or head
		end
		SA_FireAt(head)
	end)
end))

pcall(setupSilentAimHook)
task.spawn(function()
	for _ = 1, 40 do
		pcall(function()
			if not Utility then Utility = require(ReplicatedStorage.Modules.Utility) end
			if not EnumLib then EnumLib = require(ReplicatedStorage.Modules.EnumLibrary) end
			if not Fighter then
				Fighter = require(LocalPlayer.PlayerScripts.Controllers.FighterController)
			end
		end)
		if Utility and EnumLib then
			pcall(setupSilentAimHook)
			break
		end
		task.wait(0.4)
	end
end)

pcall(function() RunService:UnbindFromRenderStep("__nnnn_silent_lock") end)

-- 몸 돌리기: 애니 덮어쓰기용 Transform 스핀 (C0는 애니에 먹힘)
local _aaMotorList = {}
local _aaBodyT = 0
local _aaSpinConn = nil
local _aaSpinChar = nil

local function resetBodySpinMotors()
	if _aaSpinConn then
		pcall(function() _aaSpinConn:Disconnect() end)
		_aaSpinConn = nil
	end
	for _, motor in ipairs(_aaMotorList) do
		pcall(function()
			if motor and motor.Parent then
				motor.Transform = CFrame.new()
			end
		end)
	end
	_aaMotorList = {}
	_aaSpinChar = nil
end

local function collectMotors(char)
	_aaMotorList = {}
	if not char then return end
	for _, motor in ipairs(char:GetDescendants()) do
		if motor:IsA("Motor6D") and motor.Name ~= "Root" and motor.Name ~= "RootJoint" then
			table.insert(_aaMotorList, motor)
		end
	end
end

local function ensureBodySpinLoop()
	if _aaSpinConn then return end
	-- Stepped = 애니 적용 직후 Transform 덮어쓰기
	_aaSpinConn = RunService.Stepped:Connect(function()
		-- Body Spin 또는 Stretch 모드
		local stretchOn = Settings.AntiAimStretch
		local spinOn = Settings.AntiAimBodySpin
		if not Settings.AntiAimEnabled or (not spinOn and not stretchOn) then
			return
		end
		if OrbitEnabled or Settings.FlyEnabled then return end
		if RageConfig.on or _rb.active then return end

		local char = LocalPlayer.Character
		if not char then return end
		if char ~= _aaSpinChar or #_aaMotorList == 0 then
			_aaSpinChar = char
			collectMotors(char)
		end

		local spd = (Settings.AntiAimSpeed or 180) / 40
		_aaBodyT = _aaBodyT + 0.016 * spd
		local tt = _aaBodyT

		-- 애니 약화해서 Transform이 보이게
		pcall(function()
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hum then
				local tracks = hum:GetPlayingAnimationTracks()
				if type(tracks) == "table" then
					for _, tr in ipairs(tracks) do
						pcall(function()
							tr:AdjustSpeed(0)
							tr:AdjustWeight(0.01)
						end)
					end
				end
			end
		end)

		for i, motor in ipairs(_aaMotorList) do
			if motor and motor.Parent then
				local seed = i * 13.7
				local a = tt * (2.8 + (i % 5) * 0.6) + seed
				local b = tt * (3.4 + (i % 7) * 0.45) + seed * 1.3
				local c = tt * (2.2 + (i % 3) * 0.75) + seed * 0.8
				local rx = math.sin(a) * math.pi
				local ry = math.cos(b) * math.pi
				local rz = math.sin(c) * math.pi

				-- 쭉 늘어남: 관절 오프셋 크게
				local stretch = 0
				if stretchOn then
					stretch = (math.sin(tt * 2.2 + seed) * 0.5 + 0.5) * 2.8 -- 0~2.8 stud
				end
				local ox = math.sin(b) * stretch
				local oy = math.cos(a) * stretch * 1.2
				local oz = math.sin(c) * stretch

				local n = motor.Name or ""
				if n:find("Arm") or n:find("Leg") or n:find("Hand") or n:find("Foot")
					or n:find("Shoulder") or n:find("Hip") then
					rx, ry, rz = rx * 1.6, ry * 1.6, rz * 1.6
					ox, oy, oz = ox * 1.3, oy * 1.4, oz * 1.3
				end

				pcall(function()
					motor.Transform = CFrame.new(ox, oy, oz) * CFrame.Angles(rx, ry, rz)
				end)
			end
		end
	end)
end

local function applyBodySpin(char, dt, spd)
	if not (Settings.AntiAimBodySpin or Settings.AntiAimStretch) then return end
	ensureBodySpinLoop()
	if char and char ~= _aaSpinChar then
		_aaSpinChar = char
		collectMotors(char)
	end
end


local function applyAntiAim(dt)
	-- Lion AA(UpdateCameraRotation) 사용 — 로컬 HRP 스핀은 끔
	if true then return end
	if not Settings.AntiAimEnabled then
		_aaUnderY = nil
		pcall(resetBodySpinMotors)
		return
	end
	-- Rage / Orbit / Fly 중이면 AA가 HRP를 덮지 않음 (레이지 TP 우선)
	if OrbitEnabled or Settings.FlyEnabled then return end
	if RageConfig.on or _rb.active then return end
	if Settings.RageEnabled and Settings.RageTP then return end
	local char = LocalPlayer.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum then return end

	hum.AutoRotate = false
	pcall(function()
		hum.CameraOffset = Vector3.zero
	end)

	dt = dt or 0.016
	local vel = hrp.AssemblyLinearVelocity
	local spd = Settings.AntiAimSpeed or 180
	local pitchMode = Settings.AntiAimPitch or "Disabled"
	local yawMode = Settings.AntiAimYaw or "Disabled"

	local yaw = 0
	if yawMode == "Spin" then
		spinAngle = (spinAngle + spd * dt) % 360
		yaw = math.rad(spinAngle)
	elseif yawMode == "Jitter" then
		spinAngle = (spinAngle + spd * dt * 0.25) % 360
		yaw = math.rad(spinAngle + (math.noise(tick() * 10) * 2 - 1) * 75)
	elseif yawMode == "Random" then
		if tick() - (lastDesyncTick or 0) > 0.07 then
			lastDesyncTick = tick()
			spinAngle = math.random(0, 359)
		end
		yaw = math.rad(spinAngle)
	elseif yawMode == "Backwards" then
		local look = Camera.CFrame.LookVector
		yaw = math.atan2(-look.X, -look.Z) + math.pi
	elseif yawMode == "Left" then
		local look = Camera.CFrame.LookVector
		yaw = math.atan2(-look.X, -look.Z) + math.rad(90)
	elseif yawMode == "Right" then
		local look = Camera.CFrame.LookVector
		yaw = math.atan2(-look.X, -look.Z) - math.rad(90)
	else
		local look = Camera.CFrame.LookVector
		yaw = math.atan2(-look.X, -look.Z)
	end

	local pitch = 0
	if pitchMode == "Up" then
		pitch = math.rad(-80)
	elseif pitchMode == "Down" then
		pitch = math.rad(80)
	elseif pitchMode == "Jitter" then
		pitch = math.rad((math.noise(tick() * 12 + 3) * 2 - 1) * 70)
	elseif pitchMode == "Random" then
		pitch = math.rad((spinAngle % 170) - 85)
	end

	local pos = hrp.Position
	if Settings.AntiAimUnderground then
		if pitchMode == "Disabled" then
			pitch = math.rad(90)
		end
		if not _aaUnderY then
			_aaUnderY = pos.Y - 2.2
		end
		pos = Vector3.new(pos.X, _aaUnderY, pos.Z)
		vel = Vector3.new(vel.X, 0, vel.Z)
	else
		_aaUnderY = nil
	end

	hrp.CFrame = CFrame.new(pos) * CFrame.Angles(pitch, yaw, 0)
	hrp.AssemblyLinearVelocity = vel

	-- 몸 돌리기 (팔·다리·몸통 관절 스핀)
	if Settings.AntiAimBodySpin or Settings.AntiAimStretch ~= false then
		Settings.AntiAimStretch = Settings.AntiAimStretch ~= false
		applyBodySpin(char, dt, spd)
	else
		pcall(resetBodySpinMotors)
	end
end


-- 1인칭: 카메라/캐릭터 컨트롤러보다 늦게 AA 재적용
pcall(function() RunService:UnbindFromRenderStep("__nnnn_aa") end)
RunService:BindToRenderStep("__nnnn_aa", Enum.RenderPriority.Camera.Value + 3, function(dt)
	if Settings.AntiAimEnabled then
		applyAntiAim(dt)
	end
end)


RunService.Heartbeat:Connect(function(dt)
	-- 아무 기능도 안 켜져 있으면 즉시 종료 (렉 방지)
	local any = Settings.AimbotEnabled or Settings.RageEnabled or RageConfig.on
		or Settings.RageTP or Settings.Desync or Settings.Spinbot or Settings.ShowFOV
		or Settings.AntiAimEnabled
	if not any then
		CurrentTarget = nil
		if FOVCircle then FOVCircle.Visible = false end
		return
	end
	if Settings.AimbotEnabled or Settings.RageEnabled or RageConfig.on then
		CurrentTarget = getClosest()
	else
		CurrentTarget = nil
	end
	local char = LocalPlayer.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	-- RageTP: Ultra Rage 어택 중이면 continuousDesync가 담당 (여기선 중복 X)
	if Settings.RageTP and not (RageConfig.on or _rb.active) and Settings.RageEnabled
		and CurrentTarget and CurrentTarget.Parent and hrp then
		local targetPos = CurrentTarget.Position
		local pronePos = targetPos + Vector3.new(0, math.max(0.05, Settings.TPDistance or 0.4), 0)
		hrp.CFrame = CFrame.new(pronePos) * CFrame.Angles(math.rad(-90), 0, 0)
		hrp.AssemblyLinearVelocity = Vector3.zero
		if hum then
			hum.PlatformStand = true
			hum.AutoRotate = false
		end
	end
	
	if Settings.Desync and hrp then
		local now = tick()
		if now - lastDesyncTick > 0.04 then
			lastDesyncTick = now
			local amount = Settings.DesyncAmount
			if Settings.DesyncMode == "AdvancedJitter" then
				desyncOffset = Vector3.new(math.random(-amount, amount), math.sin(tick() * 30) * amount, math.random(-amount, amount))
			elseif Settings.DesyncMode == "Backward" then
				local look = hrp.CFrame.LookVector
				desyncOffset = -look * amount + Vector3.new(0, math.sin(tick() * 20) * 2, 0)
			else
				local right = hrp.CFrame.RightVector
				desyncOffset = right * (math.random() > 0.5 and amount or -amount)
			end
		end
		hrp.CFrame = hrp.CFrame + desyncOffset
		hrp.AssemblyLinearVelocity = Vector3.new(0, math.sin(tick() * 15) * 20, 0)
	end
	
	if FOVCircle then
		FOVCircle.Position = UserInputService:GetMouseLocation()
		FOVCircle.Radius = Settings.FOVRadius
		FOVCircle.Visible = Settings.ShowFOV and (Settings.AimbotEnabled or Settings.RageEnabled)
	end
	
	applyAntiAim(dt)
	if Settings.RageEnabled and Settings.Spinbot and hrp and hum then
		hum.AutoRotate = false
		spinAngle = (spinAngle + Settings.SpinSpeed * dt * 60) % 360
		local vel = hrp.AssemblyLinearVelocity
		hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(spinAngle), 0)
		hrp.AssemblyLinearVelocity = vel
	elseif hum and not RageConfig.on and not Settings.Desync and not Settings.FlyEnabled and not Settings.AntiAimEnabled then
		hum.AutoRotate = true
	end
end)


-- ==================== Weather + 40 Auras (from weather script, no own GUI) ====================
local WeatherAPI = {
	state = {
		rainOn = false, snowOn = false, featherOn = false,
		fogOn = false, mist = false, stars = false,
		rainMult = 1, snowMult = 1, fogDens = 0.5,
	},
	AURAS = {},
	clearAura = function() end,
	buildAura = function() end,
	applyFog = function() end,
	applyRates = function() end,
	applyFeathers = function() end,
	applyGlow = function() end,
	applyRange = function() end,
	auraIndex = function() return 0 end,
	setAuraIndex = function() end,
}
-- 날씨/오라 무거운 초기화 제거 (로드 시 화면 멈춤 방지)
print("[nnnn] weather stub (lightweight)")


-- ==================== Third Person (Lion CameraState POV) ====================
local ThirdPerson = {
	conn = nil,
}

local function getCameraController()
	local ok, cam = pcall(function()
		return require(LocalPlayer.PlayerScripts.Controllers.CameraController)
	end)
	if ok then return cam end
	return nil
end

local function stopThirdPerson()
	if ThirdPerson.conn then
		pcall(task.cancel, ThirdPerson.conn)
		ThirdPerson.conn = nil
	end
	pcall(function()
		local gun = getCameraController()
		if gun and gun.CameraState and gun.CameraState._SetPOVState and gun.CameraState.States then
			local st = gun.CameraState.States.FirstPerson
			if st then
				gun.CameraState:_SetPOVState(st)
			end
		end
	end)
	pcall(function()
		UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
	end)
end

local function startThirdPerson()
	stopThirdPerson()
	ThirdPerson.conn = task.spawn(function()
		while Settings.ThirdPersonEnabled do
			pcall(function()
				local gun = getCameraController()
				if gun and gun.CameraState and gun.CameraState._SetPOVState and gun.CameraState.States then
					local mode = Settings.ThirdPersonMode or "ThirdPerson"
					local states = gun.CameraState.States
					if mode == "ThirdPerson Mirrored" and states.ThirdPersonMirrored then
						gun.CameraState:_SetPOVState(states.ThirdPersonMirrored)
					elseif states.ThirdPerson then
						gun.CameraState:_SetPOVState(states.ThirdPerson)
					end
				end
				if Settings.UnlockMouse then
					UserInputService.MouseBehavior = Enum.MouseBehavior.Default
				end
			end)
			task.wait(0.1)
		end
	end)
	nnnnTrack(ThirdPerson.conn)
end

-- ==================== Hit Sound (전부 개별 재생, 폴백 없음) ====================
local ContentProvider = game:GetService("ContentProvider")

-- 각각 다른 유효 사운드 (이름별 고정 1개, 넘어가기 없음)
local HIT_SOUNDS = {
	{ Name = "Skeet",      Id = "rbxassetid://4817809188" },
	{ Name = "Neverlose",  Id = "rbxassetid://8726881116" },
	{ Name = "Gamesense",  Id = "rbxassetid://6594772085" },
	{ Name = "Onetap",     Id = "rbxassetid://8255306252" },
	{ Name = "Fatality",   Id = "rbxassetid://9103703218" },
	{ Name = "Aimware",    Id = "rbxassetid://6958727243" },
	{ Name = "Rust",       Id = "rbxassetid://12550420634" },
	{ Name = "Bell",       Id = "rbxassetid://6534947240" },
	{ Name = "COD",        Id = "rbxassetid://1622288613" },
	{ Name = "CS2",        Id = "rbxassetid://10722055905" },
	{ Name = "TF2",        Id = "rbxassetid://3455696889" },
	{ Name = "Minecraft",  Id = "rbxassetid://12222216" },
	{ Name = "Quake",      Id = "rbxassetid://1876542910" },
	{ Name = "Beep",       Id = "rbxassetid://131961136" },
	{ Name = "Click",      Id = "rbxassetid://6895079853" },
	{ Name = "Pop",        Id = "rbxassetid://10066951789" },
	{ Name = "Laser",      Id = "rbxassetid://821439273" },
	{ Name = "Headshot",   Id = "rbxassetid://3746776597" },
}

local _hitHpCache = {}
local _hitSoundLast = 0
local _hitSoundFolder = nil
local _hitSoundObjs = {} -- Name -> Sound

local function ensureHitFolder()
	if _hitSoundFolder and _hitSoundFolder.Parent then return _hitSoundFolder end
	local f = Instance.new("Folder")
	f.Name = "__NnnnHitSounds"
	f.Parent = SoundService
	_hitSoundFolder = f
	return f
end

local function preloadAllHitSounds()
	local folder = ensureHitFolder()
	local toPreload = {}
	for _, def in ipairs(HIT_SOUNDS) do
		local existing = _hitSoundObjs[def.Name]
		if existing and existing.Parent then
			table.insert(toPreload, existing)
		else
			local s = Instance.new("Sound")
			s.Name = def.Name
			s.SoundId = def.Id
			s.Volume = Settings.HitSoundVolume or 2
			s.PlaybackSpeed = 1
			s.Parent = folder
			_hitSoundObjs[def.Name] = s
			table.insert(toPreload, s)
		end
	end
	pcall(function()
		ContentProvider:PreloadAsync(toPreload)
	end)
end

-- preload on first hit sound use

local function playHitSound() pcall(showHitMarker)
	if not Settings.HitSoundEnabled then return end
	local now = tick()
	if now - _hitSoundLast < 0.035 then return end
	_hitSoundLast = now

	local name = Settings.HitSoundName or "Skeet"
	local s = _hitSoundObjs[name]
	if not s or not s.Parent then
		preloadAllHitSounds()
		s = _hitSoundObjs[name]
	end
	if not s then return end

	pcall(function()
		s.Volume = Settings.HitSoundVolume or 2
		s.TimePosition = 0
		s:Play()
	end)
end

-- 상대 HP 깎이면 히트
nnnnTrack(RunService.Heartbeat:Connect(function()
	if not Settings.HitSoundEnabled then return end
	local shooting = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
	local combat = shooting or Settings.AimbotEnabled or Settings.RageEnabled or RageConfig.on
	if not combat then
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr == LocalPlayer then continue end
			local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
			if hum then _hitHpCache[plr] = hum.Health end
		end
		return
	end
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr == LocalPlayer then continue end
		if Settings.TeamCheck and plr.Team and LocalPlayer.Team and plr.Team == LocalPlayer.Team then
			continue
		end
		local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
		if not hum then continue end
		local hp = hum.Health
		local prev = _hitHpCache[plr]
		if prev and hp < prev - 0.5 and hp >= 0 then
			playHitSound()
		end
		_hitHpCache[plr] = hp
	end
end))





-- ==================== Name Spoofer (로컬 전용) ====================
local _nameSpoofHooked = false
local _lastNameSpoof = 0

local function getFakeName()
	local n = Settings.SpoofName
	if type(n) == "string" then
		n = n:match("^%s*(.-)%s*$") or n
		if #n > 0 then return n end
	end
	return nil
end

local function applyNameSpoof()
	if not Settings.SpoofNameEnabled then return end
	local fake = getFakeName()
	if not fake then return end

	pcall(function()
		local char = LocalPlayer.Character
		if char then
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hum then
				hum.DisplayName = fake
			end
			for _, d in ipairs(char:GetDescendants()) do
				if d:IsA("TextLabel") or d:IsA("TextButton") then
					local txt = d.Text
					if type(txt) == "string" and #txt > 0 then
						local real = LocalPlayer.Name
						local disp = LocalPlayer.DisplayName
						local neu = txt
						if real and #real > 0 then neu = neu:gsub(real, fake) end
						if disp and #disp > 0 and disp ~= real then neu = neu:gsub(disp, fake) end
						-- 예전에 박힌 nnnn 도 교체
						neu = neu:gsub("nnnn", fake)
						if neu ~= txt then d.Text = neu end
					end
				end
			end
		end
	end)

	if tick() - _lastNameSpoof < 0.3 then return end
	_lastNameSpoof = tick()
	pcall(function()
		local pg = LocalPlayer:FindFirstChild("PlayerGui")
		if not pg then return end
		local real = LocalPlayer.Name
		local disp = LocalPlayer.DisplayName
		for _, obj in ipairs(pg:GetDescendants()) do
			if obj:IsA("TextLabel") or obj:IsA("TextButton") then
				local txt = obj.Text
				if type(txt) ~= "string" or #txt == 0 or #txt > 100 then continue end
				local neu = txt
				if real and #real > 0 then neu = neu:gsub(real, fake) end
				if disp and #disp > 0 and disp ~= real then neu = neu:gsub(disp, fake) end
				neu = neu:gsub("nnnn", fake)
				if neu ~= txt then obj.Text = neu end
			end
		end
	end)
end

local function setupNameSpoof()
	applyNameSpoof()
	if _nameSpoofHooked then return end
	_nameSpoofHooked = true

	nnnnTrack(LocalPlayer.CharacterAdded:Connect(function(char)
		task.wait(0.25)
		applyNameSpoof()
	end))

	-- RenderStepped 스캔 제거 (화면 멈춤 원인)
	nnnnTrack(RunService.Heartbeat:Connect(function()
		if not Settings.SpoofNameEnabled then return end
		if tick() - _lastNameSpoof < 1 then return end
		applyNameSpoof()
	end))
end

-- 하위 호환 (옛 호출명)
local function setupStatSpoof()
	setupNameSpoof()
end
local function applyLeaderstatsSpoof() end
local function applyUiTextSpoof()
	applyNameSpoof()
end
local RANK_NAMES = { "Bronze", "Silver", "Gold", "Platinum", "Diamond", "Master", "Grandmaster", "Champion", "Unreal", "Mythic" }

-- ==================== LinoriaLib UI ====================

-- ===== Animation Player (Character) — Lion style, catalog emote =====
local AnimPlayer = {
	Enabled = false,
	Preset = "Bodybuilder",
	CustomId = "",
	Speed = 1,
	StartPct = 0,
	EndPct = 100,
}

local ANIM_MAP = {
	["Bodybuilder"] = "3994130516",
	["Crawling in a Circle"] = "116935126100338",
	["Dolphin Dance"] = "5938365243",
	["Dance"] = "507771019",
	["Dance Break"] = "94258912028011",
	["French Confidence"] = "116968182519797",
	["Floss"] = "72174079036035",
	["Frosty Flair"] = "10214406616",
	["Full Wiggle"] = "86520127496722",
	["Ghost Floating"] = "75911227509248",
	["Gun"] = "81100102810594",
	["Gangnam Style"] = "78801539668900",
	["Hip Bounce"] = "123602332785269",
	["Hype Dance"] = "93079641847306",
	["Kicking Feet"] = "109814083870185",
	["Line Dance"] = "4049646104",
	["Lay Floating"] = "126579240140537",
	["Let's Drive"] = "17360720445",
	["Long Legs"] = "82416741608012",
	["Rock Out"] = "18225077553",
	["Samba"] = "6869813008",
	["Still Standing"] = "11435177473",
	["Spiral"] = "81926730031709",
	["Solar System"] = "118314972618293",
	["Twirl"] = "3716633898",
	["Take Me Under"] = "6797938823",
	["The Worm"] = "99563207397301",
	["Take the L"] = "110664723286332",
	["Zesty"] = "102901317133934",
}

local ANIM_LIST = {
	"Bodybuilder", "Custom", "Crawling in a Circle", "Dolphin Dance", "Dance",
	"Dance Break", "French Confidence", "Floss", "Frosty Flair", "Full Wiggle",
	"Ghost Floating", "Gun", "Gangnam Style", "Hip Bounce", "Hype Dance",
	"Kicking Feet", "Line Dance", "Lay Floating", "Let's Drive", "Long Legs",
	"Rock Out", "Samba", "Still Standing", "Spiral", "Solar System", "Twirl",
	"Take Me Under", "The Worm", "Take the L", "Zesty",
}

local _apTrack, _apAnim, _apHum, _apAnimator
local _apToken, _apLastReplay = 0, 0

local function animStop()
	_apToken += 1
	if _apTrack then
		pcall(function() _apTrack:Stop(0.1) end)
		pcall(function() _apTrack:Destroy() end)
		_apTrack = nil
	end
	if _apAnim then
		pcall(function() _apAnim:Destroy() end)
		_apAnim = nil
	end
	_apHum, _apAnimator = nil, nil
end

local function animSelectedId()
	if AnimPlayer.Preset == "Custom" then
		local n = tostring(AnimPlayer.CustomId or ""):match("%d+")
		return n
	end
	return ANIM_MAP[AnimPlayer.Preset]
end

local function loadCatalogEmote(humanoid, animator, id)
	local description = humanoid:FindFirstChildOfClass("HumanoidDescription")
	if not description then return nil end
	local emoteName = "NnnnAnimPlayer"
	local captured
	local conn = animator.AnimationPlayed:Connect(function(track)
		captured = track
	end)
	local ok = pcall(function()
		pcall(function() description:RemoveEmote(emoteName) end)
		description:AddEmote(emoteName, tonumber(id))
		humanoid:PlayEmote(emoteName)
	end)
	local deadline = os.clock() + 2
	while ok and not captured and os.clock() < deadline do
		RunService.Heartbeat:Wait()
	end
	pcall(function() conn:Disconnect() end)
	return captured
end

local function animPlay()
	if not AnimPlayer.Enabled then return end
	local id = animSelectedId()
	if not id or id == "" then
		animStop()
		return
	end
	local char = LocalPlayer.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	if not hum then return end
	local animator = hum:FindFirstChildOfClass("Animator")
	if not animator then return end

	animStop()
	_apToken += 1
	local token = _apToken
	local track, animation

	if AnimPlayer.Preset ~= "Custom" then
		track = loadCatalogEmote(hum, animator, id)
	else
		animation = Instance.new("Animation")
		animation.AnimationId = "rbxassetid://" .. id
		local ok
		ok, track = pcall(function() return animator:LoadAnimation(animation) end)
		if not ok then track = nil end
	end

	if not track then
		if animation then pcall(function() animation:Destroy() end) end
		return
	end

	_apAnim = animation
	_apTrack = track
	_apHum = hum
	_apAnimator = animator
	_apLastReplay = os.clock()
	pcall(function()
		track.Priority = Enum.AnimationPriority.Action4
		track.Looped = true
		track:Play(0.1, 1, AnimPlayer.Speed or 1)
	end)

	task.spawn(function()
		while token == _apToken and _apTrack == track and AnimPlayer.Enabled do
			local length = 0
			pcall(function() length = track.Length end)
			local sp = AnimPlayer.StartPct or 0
			local ep = AnimPlayer.EndPct or 100
			if ep <= sp then ep = math.min(100, sp + 1) end
			if length > 0 then
				local st = length * sp / 100
				local et = length * ep / 100
				pcall(function()
					if track.TimePosition < st or track.TimePosition >= et then
						track.TimePosition = st
					end
				end)
			end
			pcall(function()
				if not track.IsPlaying then
					track:Play(0.05, 1, AnimPlayer.Speed or 1)
				end
				track:AdjustSpeed(AnimPlayer.Speed or 1)
				track:AdjustWeight(1, 0)
			end)
			RunService.Heartbeat:Wait()
		end
	end)
end

nnnnTrack(LocalPlayer.CharacterAdded:Connect(function(character)
	task.spawn(function()
		local hum = character:WaitForChild("Humanoid", 10)
		if not hum then return end
		hum:WaitForChild("Animator", 10)
		task.wait(0.5)
		if AnimPlayer.Enabled and LocalPlayer.Character == character then
			animPlay()
		end
	end)
end))

nnnnTrack(RunService.Heartbeat:Connect(function()
	if not AnimPlayer.Enabled or os.clock() - _apLastReplay < 1 then return end
	local char = LocalPlayer.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local animator = hum and hum:FindFirstChildOfClass("Animator")
	local trackValid = false
	if _apTrack then
		pcall(function() trackValid = _apTrack.IsPlaying end)
	end
	if hum and animator and (hum ~= _apHum or animator ~= _apAnimator or not _apTrack or not trackValid) then
		_apLastReplay = os.clock()
		task.spawn(animPlay)
	end
end))


-- ===== Lion Anti Aim (UpdateCameraRotation) =====
local _lionAATask = nil

local function getLionCameraRotation(cameraController)
	local currentRotation = (cameraController and cameraController.Rotation) or Vector2.zero
	local pitch = currentRotation.X or 0
	local yaw = currentRotation.Y or 0
	local cfg = _G.AntiAimPoseConfig or {}
	local pitchMode = string.lower(tostring(cfg.pitch or Settings.AntiAimPitch or "disabled"))
	local yawMode = string.lower(tostring(cfg.yaw or Settings.AntiAimYaw or "disabled"))

	if pitchMode == "up" then
		pitch = math.rad(-89)
	elseif pitchMode == "down" then
		pitch = math.rad(179)
	elseif pitchMode == "zero" then
		pitch = 0
	elseif pitchMode == "random" then
		pitch = math.rad(math.random(-89, 179))
	end

	if yawMode == "backwards" then
		yaw = yaw + math.rad(180)
	elseif yawMode == "spin" then
		yaw = math.rad((tick() * 720) % 360)
	elseif yawMode == "random" then
		yaw = math.rad(math.random(0, 359))
	end

	return Vector2.new(pitch, yaw)
end

local function stopLionAntiAim()
	if _lionAATask then
		pcall(task.cancel, _lionAATask)
		_lionAATask = nil
	end
	_G.AntiAimPoseConfig = nil
end

local function startLionAntiAim()
	stopLionAntiAim()
	_G.AntiAimPoseConfig = {
		enabled = true,
		pitch = string.lower(tostring(Settings.AntiAimPitch or "disabled")),
		yaw = string.lower(tostring(Settings.AntiAimYaw or "disabled")),
		underground = Settings.AntiAimUnderground == true,
	}

	local okUtility, utility = pcall(function()
		return require(ReplicatedStorage.Modules.Utility)
	end)
	if not okUtility or not utility then return end

	local okCamera, cameraController = pcall(function()
		return require(LocalPlayer.PlayerScripts.Controllers.CameraController)
	end)
	if not okCamera or not cameraController then return end

	local rem = ReplicatedStorage:FindFirstChild("Remotes")
	local updateCameraRotation = rem
		and rem:FindFirstChild("Replication")
		and rem.Replication:FindFirstChild("Fighter")
		and rem.Replication.Fighter:FindFirstChild("UpdateCameraRotation")
	if not updateCameraRotation then return end

	_lionAATask = task.spawn(function()
		while _G.AntiAimPoseConfig and _G.AntiAimPoseConfig.enabled and Settings.AntiAimEnabled do
			local cfg = _G.AntiAimPoseConfig or {}
			local randomBurst = (cfg.yaw == "random") and 3 or 1
			for _ = 1, randomBurst do
				local cameraRotation = getLionCameraRotation(cameraController)
				pcall(function()
					updateCameraRotation:FireServer(utility:EncodeCameraRotation(cameraRotation), nil)
				end)
			end
			if cfg.underground then
				pcall(function()
					local char = LocalPlayer.Character
					local hrp = char and char:FindFirstChild("HumanoidRootPart")
					if hrp then
						local pos = hrp.Position
						hrp.CFrame = CFrame.new(pos.X, pos.Y - 2.2, pos.Z) * (hrp.CFrame - hrp.Position)
					end
				end)
			end
			RunService.Heartbeat:Wait()
		end
	end)
end


-- ==================== Lion / Harion World (Skybox / Weather / Ambience) ====================
local LionSkyIDs = {
	["Afternoon"] = {600830446, 600831635, 600832720, 600886090, 600833862, 600835177},
	["Blue Space"] = {149397692, 149397686, 149397697, 149397684, 149397688, 149397702},
	["Classic Roblox"] = {1012890, 1012891, 1012887, 1012889, 1012888, 1014449},
	["Cloudy"] = {591058823, 591059876, 591058104, 591057861, 591057625, 591059642},
	["Dusk"] = {264908339, 264907909, 264909420, 264909758, 264908886, 264907379},
	["Dawn"] = {1417494030, 1417494146, 1417494253, 1417494402, 1417494499, 1417494643},
	["Dark Skies"] = {570557514, 570557775, 570557559, 570557620, 570557672, 570557727},
	["Earth"] = {6444884337, 6444884785, 6444884337, 6444884785, 6444884337, 6444884785},
	["Horizontal Milky Way"] = {159454299, 159454296, 159454293, 159454286, 159454300, 159454288},
	["Heaven"] = {591058823, 591059642, 591059876, 591057625, 591057861, 591058104},
	["Jungle"] = {214253616, 214253616, 214253616, 214253616, 214253616, 214253616},
	["Mountains"] = {452457785, 452457806, 452457839, 452457866, 452457896, 452457928},
	["Nebula"] = {149397697, 149397702, 149397692, 149397688, 149397684, 149397686},
	["Night Light"] = {12064107, 12064152, 12064121, 12063984, 12064115, 12064131},
	["Night"] = {12064121, 12064152, 12064107, 12064115, 12063984, 12064131},
	["Ocean Sky"] = {150335574, 150335585, 150335628, 150335620, 150335610, 150335642},
	["Redshift"] = {401664839, 401664862, 401664960, 401664881, 401664901, 401664936},
	["Space"] = {149397684, 149397686, 149397688, 149397692, 149397697, 149397702},
	["Sunset"] = {264909420, 264907909, 264908339, 264908886, 264909758, 264907379},
	["Storm"] = {570557514, 570557775, 570557559, 570557620, 570557672, 570557727},
	["SFOTH"] = {1012887, 1012891, 1012890, 1012888, 1012889, 1014449},
	["Solid Black"] = {0, 0, 0, 0, 0, 0},
	["Saturn"] = {149397688, 149397686, 149397684, 149397692, 149397702, 149397697},
	["Smoke"] = {570557672, 570557514, 570557727, 570557559, 570557775, 570557620},
	["Vertical Milky Way"] = {159454286, 159454288, 159454299, 159454300, 159454296, 159454293},
	["White"] = {0, 0, 0, 0, 0, 0},
}

local LionSkySettings = {
	["Afternoon"] = {StarCount = 1200, SunAngularSize = 18, MoonAngularSize = 11, ClockTime = 14, Ambient = Color3.fromRGB(150, 150, 150), FogColor = Color3.fromRGB(210, 220, 235)},
	["Blue Space"] = {StarCount = 4500, SunAngularSize = 4, MoonAngularSize = 6, ClockTime = 0, Ambient = Color3.fromRGB(65, 80, 125), FogColor = Color3.fromRGB(25, 30, 60)},
	["Classic Roblox"] = {StarCount = 1200, SunAngularSize = 21, MoonAngularSize = 11, ClockTime = 12, Ambient = Color3.fromRGB(128, 128, 128), FogColor = Color3.fromRGB(192, 192, 192)},
	["Cloudy"] = {StarCount = 0, SunAngularSize = 12, MoonAngularSize = 0, ClockTime = 13, Ambient = Color3.fromRGB(135, 140, 145), FogColor = Color3.fromRGB(180, 185, 190)},
	["Dusk"] = {StarCount = 900, SunAngularSize = 14, MoonAngularSize = 8, ClockTime = 18.4, Ambient = Color3.fromRGB(145, 95, 120), FogColor = Color3.fromRGB(185, 120, 130)},
	["Dawn"] = {StarCount = 600, SunAngularSize = 16, MoonAngularSize = 7, ClockTime = 6.2, Ambient = Color3.fromRGB(170, 135, 115), FogColor = Color3.fromRGB(230, 170, 145)},
	["Dark Skies"] = {StarCount = 2600, SunAngularSize = 0, MoonAngularSize = 12, ClockTime = 0, Ambient = Color3.fromRGB(45, 50, 65), FogColor = Color3.fromRGB(45, 50, 60)},
	["Earth"] = {StarCount = 3000, SunAngularSize = 8, MoonAngularSize = 5, ClockTime = 1, Ambient = Color3.fromRGB(55, 80, 110), FogColor = Color3.fromRGB(35, 50, 80)},
	["Horizontal Milky Way"] = {StarCount = 6000, SunAngularSize = 0, MoonAngularSize = 8, ClockTime = 0, Ambient = Color3.fromRGB(65, 55, 95), FogColor = Color3.fromRGB(35, 25, 60)},
	["Heaven"] = {StarCount = 0, SunAngularSize = 24, MoonAngularSize = 0, ClockTime = 12, Ambient = Color3.fromRGB(205, 205, 230), FogColor = Color3.fromRGB(245, 245, 255)},
	["Jungle"] = {StarCount = 200, SunAngularSize = 18, MoonAngularSize = 0, ClockTime = 15, Ambient = Color3.fromRGB(75, 115, 70), FogColor = Color3.fromRGB(90, 130, 95)},
	["Mountains"] = {StarCount = 600, SunAngularSize = 16, MoonAngularSize = 9, ClockTime = 10, Ambient = Color3.fromRGB(125, 145, 160), FogColor = Color3.fromRGB(180, 200, 215)},
	["Nebula"] = {StarCount = 7000, SunAngularSize = 0, MoonAngularSize = 5, ClockTime = 0, Ambient = Color3.fromRGB(85, 45, 125), FogColor = Color3.fromRGB(40, 20, 70)},
	["Night Light"] = {StarCount = 5000, SunAngularSize = 0, MoonAngularSize = 14, ClockTime = 0, Ambient = Color3.fromRGB(80, 85, 120), FogColor = Color3.fromRGB(35, 40, 70)},
	["Night"] = {StarCount = 3500, SunAngularSize = 0, MoonAngularSize = 11, ClockTime = 0, Ambient = Color3.fromRGB(55, 60, 80), FogColor = Color3.fromRGB(25, 30, 45)},
	["Ocean Sky"] = {StarCount = 400, SunAngularSize = 20, MoonAngularSize = 8, ClockTime = 13, Ambient = Color3.fromRGB(120, 155, 180), FogColor = Color3.fromRGB(135, 180, 210)},
	["Redshift"] = {StarCount = 1800, SunAngularSize = 18, MoonAngularSize = 6, ClockTime = 18, Ambient = Color3.fromRGB(160, 70, 65), FogColor = Color3.fromRGB(150, 55, 50)},
	["Space"] = {StarCount = 6500, SunAngularSize = 0, MoonAngularSize = 6, ClockTime = 0, Ambient = Color3.fromRGB(50, 55, 95), FogColor = Color3.fromRGB(15, 20, 45)},
	["Sunset"] = {StarCount = 700, SunAngularSize = 20, MoonAngularSize = 7, ClockTime = 17.8, Ambient = Color3.fromRGB(170, 105, 95), FogColor = Color3.fromRGB(235, 135, 95)},
	["Storm"] = {StarCount = 0, SunAngularSize = 0, MoonAngularSize = 7, ClockTime = 16, Ambient = Color3.fromRGB(65, 70, 80), FogColor = Color3.fromRGB(70, 75, 85)},
	["SFOTH"] = {StarCount = 1000, SunAngularSize = 21, MoonAngularSize = 11, ClockTime = 14, Ambient = Color3.fromRGB(135, 135, 135), FogColor = Color3.fromRGB(190, 190, 190)},
	["Solid Black"] = {StarCount = 0, SunAngularSize = 0, MoonAngularSize = 0, ClockTime = 0, Ambient = Color3.new(), FogColor = Color3.new()},
	["Saturn"] = {StarCount = 5500, SunAngularSize = 0, MoonAngularSize = 18, ClockTime = 0, Ambient = Color3.fromRGB(105, 85, 65), FogColor = Color3.fromRGB(60, 45, 35)},
	["Smoke"] = {StarCount = 0, SunAngularSize = 0, MoonAngularSize = 5, ClockTime = 3, Ambient = Color3.fromRGB(85, 85, 85), FogColor = Color3.fromRGB(95, 95, 95)},
	["Vertical Milky Way"] = {StarCount = 6200, SunAngularSize = 0, MoonAngularSize = 8, ClockTime = 0, Ambient = Color3.fromRGB(70, 50, 90), FogColor = Color3.fromRGB(30, 20, 55)},
	["White"] = {StarCount = 0, SunAngularSize = 0, MoonAngularSize = 0, ClockTime = 12, Ambient = Color3.new(1, 1, 1), FogColor = Color3.new(1, 1, 1)},
}

local LionWorld = {
	SkyEnabled = false,
	SkyName = "Afternoon",
	WeatherEnabled = false,
	WeatherName = "Rain",
	AmbienceEnabled = false,
	AmbienceName = "Rain",
	AmbienceVol = 0.5,
	ambienceSound = nil,
	weatherPart = nil,
}

local function lionSkyId(id)
	if not id or id == 0 then return "" end
	return "rbxassetid://" .. tostring(id)
end

local function lionApplySkybox(name, enabled)
	name = name or "Afternoon"
	pcall(function()
		for _, child in ipairs(Lighting:GetChildren()) do
			if child:IsA("Sky") then
				if enabled then
					if child.Name ~= "LionSkybox" then child:Destroy() end
				else
					if child.Name == "LionSkybox" or child.Name == "CustomSky" then child:Destroy() end
				end
			end
		end
	end)
	if not enabled then
		currentSkyIndex = 1
		return
	end
	local ids = LionSkyIDs[name] or LionSkyIDs["Afternoon"]
	local settings = LionSkySettings[name] or LionSkySettings["Afternoon"] or {}
	local sky = Lighting:FindFirstChild("LionSkybox")
	if not sky then
		sky = Instance.new("Sky")
		sky.Name = "LionSkybox"
	end
	sky.SkyboxBk = lionSkyId(ids[1])
	sky.SkyboxDn = lionSkyId(ids[2])
	sky.SkyboxFt = lionSkyId(ids[3])
	sky.SkyboxLf = lionSkyId(ids[4])
	sky.SkyboxRt = lionSkyId(ids[5])
	sky.SkyboxUp = lionSkyId(ids[6])
	sky.StarCount = settings.StarCount or 1200
	pcall(function()
		if settings.SunAngularSize ~= nil then sky.SunAngularSize = settings.SunAngularSize end
		if settings.MoonAngularSize ~= nil then sky.MoonAngularSize = settings.MoonAngularSize end
		sky.CelestialBodiesShown = ((settings.SunAngularSize or 0) > 0) or ((settings.MoonAngularSize or 0) > 0)
	end)
	sky.Parent = Lighting
	if settings.ClockTime then pcall(function() Lighting.ClockTime = settings.ClockTime end) end
	if settings.Ambient then
		pcall(function()
			Lighting.Ambient = settings.Ambient
			Lighting.OutdoorAmbient = settings.Ambient
		end)
	end
	if settings.FogColor then pcall(function() Lighting.FogColor = settings.FogColor end) end
	currentSkyIndex = 2
end

local function lionClearWeather()
	if LionWorld.weatherPart then
		pcall(function() LionWorld.weatherPart:Destroy() end)
		LionWorld.weatherPart = nil
	end
end

local function lionApplyWeather(name)
	lionClearWeather()
	name = name or "Rain"
	local part = Instance.new("Part")
	part.Name = "NnnnWeather"
	part.Anchored = true
	part.CanCollide = false
	part.Transparency = 1
	part.Size = Vector3.new(1, 1, 1)
	part.Parent = Workspace
	LionWorld.weatherPart = part

	local tex = "rbxassetid://241876422"
	local rate, speed, life = 420, 105, 0.85
	if name == "Snow" then
		tex = "rbxassetid://92367298778210"
		rate, speed, life = 180, 16, 5
	elseif name == "Heavy Rain" or name == "Heavy Rain Night" then
		rate, speed, life = 850, 125, 0.75
	elseif name == "Forest Rain Night" or name == "Light Rain And Thunder" then
		rate, speed, life = 500, 110, 0.8
	elseif name == "City" or name == "Windy Day" then
		tex = "rbxassetid://130207970"
		rate, speed, life = 40, 8, 3
	end

	local att = Instance.new("Attachment")
	att.Parent = part
	local pe = Instance.new("ParticleEmitter")
	pe.Texture = tex
	pe.Rate = rate
	pe.Speed = NumberRange.new(speed * 0.8, speed)
	pe.Lifetime = NumberRange.new(life * 0.7, life)
	pe.SpreadAngle = Vector2.new(8, 8)
	pe.Size = NumberSequence.new(0.08)
	pe.Parent = att

	nnnnTrack(RunService.Heartbeat:Connect(function()
		if not LionWorld.WeatherEnabled then return end
		local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if root and LionWorld.weatherPart then
			LionWorld.weatherPart.CFrame = root.CFrame * CFrame.new(0, 25, 0)
		end
	end))
end

local AmbiencePresets = {
	["Rain"] = "rbxassetid://1516791621",
	["Heavy Rain"] = "rbxassetid://1516791621",
	["City"] = "rbxassetid://9112823651",
	["Windy Day"] = "rbxassetid://9112823651",
}

local function lionStopAmbience()
	if LionWorld.ambienceSound then
		pcall(function()
			LionWorld.ambienceSound:Stop()
			LionWorld.ambienceSound:Destroy()
		end)
		LionWorld.ambienceSound = nil
	end
end

local function lionPlayAmbience(name)
	lionStopAmbience()
	local id = AmbiencePresets[name] or AmbiencePresets["Rain"]
	local s = Instance.new("Sound")
	s.Name = "NnnnAmbience"
	s.SoundId = id
	s.Volume = LionWorld.AmbienceVol or 0.5
	s.Looped = true
	s.Parent = SoundService
	s:Play()
	LionWorld.ambienceSound = s
end


-- ==================== Extra Features pack ====================
-- Triggerbot
local _tbLast = 0
nnnnTrack(RunService.Heartbeat:Connect(function()
	if not Settings.Triggerbot then return end
	local now = tick()
	if now - _tbLast < (Settings.TriggerDelay or 0.05) then return end
	local head = nil
	pcall(function()
		if SA_GetClosestHead then head = SA_GetClosestHead() end
	end)
	if not head then return end
	local cam = Workspace.CurrentCamera
	if not cam then return end
	local sp, on = cam:WorldToViewportPoint(head.Position)
	if not on or sp.Z < 0 then return end
	local center = cam.ViewportSize / 2
	local d = (Vector2.new(sp.X, sp.Y) - Vector2.new(center.X, center.Y)).Magnitude
	if d > (Settings.TriggerFOV or 40) then return end
	_tbLast = now
	pcall(function()
		if mouse1press then
			mouse1press()
			task.delay(0.02, function() pcall(mouse1release) end)
		elseif VirtualInputManager then
			-- fallback none
		end
	end)
end))

-- Resolver (simple yaw flip guess for anti-aim targets)
local _resolverYaw = 0
nnnnTrack(RunService.Heartbeat:Connect(function()
	if not Settings.Resolver then return end
	_resolverYaw = (_resolverYaw + 15) % 360
end))

-- Target HUD
local TargetHUDGui, TargetHUDLabel
local function ensureTargetHUD()
	if TargetHUDGui and TargetHUDGui.Parent then return TargetHUDLabel end
	pcall(function()
		local g = Instance.new("ScreenGui")
		g.Name = "NnnnTargetHUD"
		g.ResetOnSpawn = false
		g.Parent = CoreGui
		local f = Instance.new("Frame")
		f.Size = UDim2.new(0, 180, 0, 48)
		f.Position = UDim2.new(0.5, 20, 0.5, 40)
		f.BackgroundColor3 = Color3.fromRGB(14, 14, 16)
		f.BorderSizePixel = 0
		f.Parent = g
		Instance.new("UICorner", f).CornerRadius = UDim.new(0, 4)
		local s = Instance.new("UIStroke")
		s.Color = Color3.fromRGB(70, 140, 220)
		s.Thickness = 1
		s.Parent = f
		local l = Instance.new("TextLabel")
		l.Size = UDim2.new(1, -10, 1, 0)
		l.Position = UDim2.new(0, 8, 0, 0)
		l.BackgroundTransparency = 1
		l.Font = Enum.Font.Code
		l.TextSize = 13
		l.TextColor3 = Color3.fromRGB(230, 230, 235)
		l.TextXAlignment = Enum.TextXAlignment.Left
		l.Text = "Target: -"
		l.Parent = f
		TargetHUDGui = g
		TargetHUDLabel = l
	end)
	return TargetHUDLabel
end

nnnnTrack(RunService.Heartbeat:Connect(function()
	if not Settings.TargetHUD then
		if TargetHUDGui then pcall(function() TargetHUDGui:Destroy() end) TargetHUDGui = nil TargetHUDLabel = nil end
		return
	end
	local lab = ensureTargetHUD()
	if not lab then return end
	local t = CurrentTarget or (_rb and _rb.target)
	local name, hp = "-", "-"
	pcall(function()
		if typeof(t) == "Instance" and t:IsA("BasePart") then
			local m = t.Parent
			local plr = m and Players:GetPlayerFromCharacter(m)
			if plr then
				name = plr.Name
				local h = m:FindFirstChildOfClass("Humanoid")
				if h then hp = math.floor(h.Health) end
			end
		elseif typeof(t) == "Instance" and t:IsA("Player") then
			name = t.Name
			local h = t.Character and t.Character:FindFirstChildOfClass("Humanoid")
			if h then hp = math.floor(h.Health) end
		end
	end)
	lab.Text = "Target: " .. tostring(name) .. "\nHP: " .. tostring(hp)
end))

-- Hit marker (simple screen flash on damage number if possible)
local function showHitMarker()
	if not Settings.HitMarker then return end
	pcall(function()
		local g = Instance.new("ScreenGui")
		g.Name = "NnnnHitMarker"
		g.ResetOnSpawn = false
		g.Parent = CoreGui
		local l = Instance.new("TextLabel")
		l.Size = UDim2.new(0, 40, 0, 40)
		l.Position = UDim2.new(0.5, -20, 0.5, -20)
		l.BackgroundTransparency = 1
		l.Text = "x"
		l.Font = Enum.Font.Code
		l.TextSize = 28
		l.TextColor3 = Color3.fromRGB(255, 255, 255)
		l.Parent = g
		task.delay(0.12, function() pcall(function() g:Destroy() end) end)
	end)
end

-- Inf jump
nnnnTrack(UserInputService.JumpRequest:Connect(function()
	if not Settings.InfJump then return end
	local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
	if hum then
		pcall(function()
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end)
	end
end))

-- No fall / jump power apply (CustomJump ON 일 때만)
nnnnTrack(RunService.Heartbeat:Connect(function()
	local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
	if not hum then return end
	if Settings.CustomJump then
		pcall(function()
			hum.UseJumpPower = true
			hum.JumpPower = Settings.JumpPower or 50
		end)
	end
	if Settings.NoFall then
		pcall(function()
			hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
			hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
		end)
	end
end))

-- Viewmodel offset (best-effort on camera arms)
nnnnTrack(RunService.RenderStepped:Connect(function()
	if not Settings.ViewmodelOffset then return end
	pcall(function()
		local cam = Workspace.CurrentCamera
		if not cam then return end
		for _, obj in ipairs(cam:GetChildren()) do
			if obj:IsA("Model") or obj:IsA("Folder") then
				local off = Vector3.new(Settings.VM_X or 0, Settings.VM_Y or 0, Settings.VM_Z or 0)
				for _, p in ipairs(obj:GetDescendants()) do
					if p:IsA("BasePart") then
						-- light touch only — skip heavy rewrite
					end
				end
			end
		end
	end)
end))


-- ==================== Cfg: Themes + Config Manager ====================
local CfgSys = {
	folder = "nnnnLinoria",
	cfgFolder = "nnnnLinoria/configs",
	themeFolder = "nnnnLinoria/themes",
	autoloadFile = "nnnnLinoria/autoload.txt",
	currentName = "default",
	autoload = "none",
	theme = {
		Background = Color3.fromRGB(10, 10, 12),
		Main = Color3.fromRGB(14, 14, 16),
		Accent = Color3.fromRGB(70, 140, 220),
		Outline = Color3.fromRGB(40, 40, 45),
		Font = Color3.fromRGB(230, 230, 235),
	},
}

local function cfgEnsureFolders()
	if not makefolder then return end
	pcall(makefolder, CfgSys.folder)
	pcall(makefolder, CfgSys.cfgFolder)
	pcall(makefolder, CfgSys.themeFolder)
end

local function colorToTable(c)
	if typeof(c) ~= "Color3" then return {R=1,G=1,B=1} end
	return {R = c.R, G = c.G, B = c.B}
end

local function tableToColor(t)
	if type(t) ~= "table" then return Color3.new(1,1,1) end
	return Color3.new(t.R or 1, t.G or 1, t.B or 1)
end

local function serializeSettings()
	local out = {}
	for k, v in pairs(Settings) do
		local ty = typeof(v)
		if ty == "number" or ty == "string" or ty == "boolean" then
			out[k] = v
		elseif ty == "Color3" then
			out[k] = {__color = true, R = v.R, G = v.G, B = v.B}
		elseif ty == "EnumItem" then
			out[k] = {__enum = true, EnumType = tostring(v.EnumType), Name = v.Name}
		end
	end
	out.__theme = {
		Background = colorToTable(CfgSys.theme.Background),
		Main = colorToTable(CfgSys.theme.Main),
		Accent = colorToTable(CfgSys.theme.Accent),
		Outline = colorToTable(CfgSys.theme.Outline),
		Font = colorToTable(CfgSys.theme.Font),
	}
	return out
end

local function applySerialized(data)
	if type(data) ~= "table" then return end
	for k, v in pairs(data) do
		if k == "__theme" and type(v) == "table" then
			if v.Background then CfgSys.theme.Background = tableToColor(v.Background) end
			if v.Main then CfgSys.theme.Main = tableToColor(v.Main) end
			if v.Accent then CfgSys.theme.Accent = tableToColor(v.Accent) Settings.ThemeColor = CfgSys.theme.Accent end
			if v.Outline then CfgSys.theme.Outline = tableToColor(v.Outline) end
			if v.Font then CfgSys.theme.Font = tableToColor(v.Font) end
		elseif type(v) == "table" and v.__color then
			Settings[k] = Color3.new(v.R, v.G, v.B)
		elseif type(v) == "table" and v.__enum then
			pcall(function()
				local et = v.EnumType
				if et == "Enum.KeyCode" or tostring(et):find("KeyCode") then
					Settings[k] = Enum.KeyCode[v.Name]
				end
			end)
		elseif Settings[k] ~= nil and type(Settings[k]) == type(v) then
			Settings[k] = v
		elseif type(v) == "boolean" or type(v) == "number" or type(v) == "string" then
			Settings[k] = v
		end
	end
end

local function listJsonIn(folder)
	local list = {}
	if not listfiles then return list end
	pcall(function()
		for _, f in ipairs(listfiles(folder)) do
			local name = tostring(f):match("([^/\\]+)%.json$")
			if name then table.insert(list, name) end
		end
	end)
	table.sort(list)
	return list
end

local function saveConfig(name)
	if not name or name == "" or not writefile then
		showNotification("config name empty")
		return false
	end
	cfgEnsureFolders()
	local path = CfgSys.cfgFolder .. "/" .. name .. ".json"
	local ok = pcall(function()
		writefile(path, HttpService:JSONEncode(serializeSettings()))
	end)
	if ok then
		CfgSys.currentName = name
		showNotification("config saved: " .. name)
	else
		showNotification("save failed")
	end
	return ok
end

local function loadConfig(name)
	if not name or name == "" or not readfile then return false end
	local path = CfgSys.cfgFolder .. "/" .. name .. ".json"
	local ok, data = pcall(function()
		if isfile and not isfile(path) then error("missing") end
		return HttpService:JSONDecode(readfile(path))
	end)
	if ok and type(data) == "table" then
		applySerialized(data)
		CfgSys.currentName = name
		showNotification("config loaded: " .. name)
		return true
	end
	showNotification("load failed")
	return false
end

local function saveTheme(name)
	if not name or name == "" or not writefile then return end
	cfgEnsureFolders()
	pcall(function()
		writefile(CfgSys.themeFolder .. "/" .. name .. ".json", HttpService:JSONEncode({
			Background = colorToTable(CfgSys.theme.Background),
			Main = colorToTable(CfgSys.theme.Main),
			Accent = colorToTable(CfgSys.theme.Accent),
			Outline = colorToTable(CfgSys.theme.Outline),
			Font = colorToTable(CfgSys.theme.Font),
		}))
	end)
	showNotification("theme saved: " .. name)
end

local function loadTheme(name)
	if not name or name == "" or not readfile then return end
	local path = CfgSys.themeFolder .. "/" .. name .. ".json"
	pcall(function()
		if isfile and not isfile(path) then return end
		local data = HttpService:JSONDecode(readfile(path))
		if type(data) == "table" then
			CfgSys.theme.Background = tableToColor(data.Background)
			CfgSys.theme.Main = tableToColor(data.Main)
			CfgSys.theme.Accent = tableToColor(data.Accent)
			CfgSys.theme.Outline = tableToColor(data.Outline)
			CfgSys.theme.Font = tableToColor(data.Font)
			Settings.ThemeColor = CfgSys.theme.Accent
			showNotification("theme loaded: " .. name)
		end
	end)
end

local function setAutoload(name)
	cfgEnsureFolders()
	if writefile then
		pcall(writefile, CfgSys.autoloadFile, tostring(name or ""))
		CfgSys.autoload = name or "none"
		showNotification("autoload: " .. tostring(CfgSys.autoload))
	end
end

local function removeAutoload()
	setAutoload("")
	CfgSys.autoload = "none"
end

task.defer(function()
	pcall(function()
		if readfile and isfile and isfile(CfgSys.autoloadFile) then
			local n = readfile(CfgSys.autoloadFile)
			n = tostring(n or ""):gsub("%s+", "")
			if n ~= "" then
				CfgSys.autoload = n
				loadConfig(n)
			end
		end
	end)
end)


-- ==================== Name Spoofer + Device Spoof ====================
local NameSpoof = {
	enabled = false,
	name = "nnnn",
	origUser = LocalPlayer.Name,
	origDisplay = LocalPlayer.DisplayName,
	textCache = setmetatable({}, {__mode = "k"}),
	humCache = setmetatable({}, {__mode = "k"}),
}

local function nsReplace(text, search, replacement)
	if search == "" or text == "" then return text end
	local result, cursor = {}, 1
	while true do
		local first, last = string.find(text, search, cursor, true)
		if not first then
			table.insert(result, string.sub(text, cursor))
			break
		end
		table.insert(result, string.sub(text, cursor, first - 1))
		table.insert(result, replacement)
		cursor = last + 1
	end
	return table.concat(result)
end

local function nsSpoofText(text)
	if NameSpoof.name == "" then return tostring(text) end
	local result = tostring(text)
	if NameSpoof.origDisplay ~= NameSpoof.origUser then
		result = nsReplace(result, NameSpoof.origDisplay, NameSpoof.name)
	end
	result = nsReplace(result, "@" .. NameSpoof.origUser, "@" .. NameSpoof.name)
	result = nsReplace(result, NameSpoof.origUser, NameSpoof.name)
	return result
end

local function nsApplyText(object)
	if not (object:IsA("TextLabel") or object:IsA("TextButton")) then return end
	pcall(function()
		local current = object.Text
		local cached = NameSpoof.textCache[object]
		if cached == nil then
			cached = {Original = current, Spoofed = nil}
			NameSpoof.textCache[object] = cached
		elseif current ~= cached.Spoofed then
			cached.Original = current
		end
		local spoofed = nsSpoofText(cached.Original)
		cached.Spoofed = spoofed
		if current ~= spoofed then object.Text = spoofed end
	end)
end

local function applyNameSpoof()
	if not NameSpoof.enabled or NameSpoof.name == "" then return end
	pcall(function()
		local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
		if hum then
			if NameSpoof.humCache[hum] == nil then
				NameSpoof.humCache[hum] = hum.DisplayName
			end
			hum.DisplayName = NameSpoof.name
		end
	end)
	for _, root in ipairs({LocalPlayer:FindFirstChildOfClass("PlayerGui"), CoreGui}) do
		if root then
			pcall(function()
				for _, object in ipairs(root:GetDescendants()) do
					nsApplyText(object)
				end
			end)
		end
	end
end

local function restoreNameSpoof()
	for object, cached in pairs(NameSpoof.textCache) do
		if object and object.Parent then
			pcall(function() object.Text = cached.Original end)
		end
	end
	table.clear(NameSpoof.textCache)
	for hum, original in pairs(NameSpoof.humCache) do
		if hum and hum.Parent then
			pcall(function() hum.DisplayName = original end)
		end
	end
	table.clear(NameSpoof.humCache)
end

-- metamethod hook once
do
	local st = env.__NnnnNameSpoofState
	if not st then
		st = {Enabled = false, Name = "", Player = LocalPlayer}
		env.__NnnnNameSpoofState = st
		if hookmetamethod and newcclosure then
			pcall(function()
				local oldIndex
				oldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
					if st.Enabled and st.Name ~= "" and self == st.Player
						and (not checkcaller or not checkcaller())
						and (key == "Name" or key == "DisplayName") then
						return st.Name
					end
					return oldIndex(self, key)
				end))
			end)
		end
	end
	NameSpoof.state = st
end

nnnnTrack(LocalPlayer.CharacterAdded:Connect(function()
	if NameSpoof.enabled then
		task.defer(applyNameSpoof)
	end
end))

-- Device spoof
local function setDeviceMode(mode)
	pcall(function()
		local rem = ReplicatedStorage:FindFirstChild("Remotes")
		local rep = rem and rem:FindFirstChild("Replication")
		local fighter = rep and rep:FindFirstChild("Fighter")
		local setc = fighter and fighter:FindFirstChild("SetControls")
		if setc then
			setc:FireServer(mode)
		end
	end)
end

local function getRealDevice()
	if game:GetService("VRService").VREnabled then return "VR" end
	if UserInputService.TouchEnabled then return "Touch" end
	if UserInputService.GamepadEnabled and not UserInputService.KeyboardEnabled then return "Gamepad" end
	return "MouseKeyboard"
end


-- ==================== Emote Hop (ONYX exact: anim + speed, no CFrame break) ====================
local emoteHopTrack = nil
local emoteHopAnim = nil

local function resolve_anim(asset)
	local ok, objs = pcall(function()
		return game:GetObjects(asset)
	end)
	if ok and type(objs) == "table" then
		for i = 1, #objs do
			if objs[i] and objs[i]:IsA("Animation") then
				return objs[i].AnimationId
			end
		end
	end
	return asset
end

task.spawn(function()
	local id = "rbxassetid://92281817840531"
	pcall(function() id = resolve_anim(id) end)
	local a = Instance.new("Animation")
	a.AnimationId = id
	emoteHopAnim = a
end)

local function stop_emote()
	if emoteHopTrack then
		pcall(function() emoteHopTrack:Stop() end)
		emoteHopTrack = nil
	end
end

local function play_emote(char)
	if not Settings.EmoteHop then return end
	if not char then return end
	if not emoteHopAnim then return end
	local hum = char:FindFirstChildWhichIsA("Humanoid")
	if not hum then return end
	stop_emote()
	local animator = hum:FindFirstChildOfClass("Animator") or hum
	local ok, track = pcall(function()
		return animator:LoadAnimation(emoteHopAnim)
	end)
	if not ok or not track then return end
	emoteHopTrack = track
	pcall(function()
		track.Priority = Enum.AnimationPriority.Action4
		track:Play()
		track:AdjustSpeed(Settings.EmoteSpeed or 40)
	end)
	track.Stopped:Connect(function()
		if Settings.EmoteHop and LocalPlayer.Character == char then
			play_emote(char)
		end
	end)
end

nnnnTrack(LocalPlayer.CharacterAdded:Connect(function(char)
	if Settings.EmoteHop then
		task.defer(function()
			char:WaitForChild("Humanoid", 5)
			task.wait(0.1)
			play_emote(char)
		end)
	end
end))


local function applyUIAccent(col)
	if typeof(col) ~= "Color3" then return end
	Settings.ThemeColor = col
	if CfgSys and CfgSys.theme then CfgSys.theme.Accent = col end
	pcall(function()
		local gui = CoreGui:FindFirstChild("NnnnHubUI") or (LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("NnnnHubUI"))
		if not gui then return end
		for _, d in ipairs(gui:GetDescendants()) do
			if d.Name == "AccentBar" and d:IsA("Frame") then
				d.BackgroundColor3 = col
			end
			if d:IsA("ScrollingFrame") then
				d.ScrollBarImageColor3 = col
			end
		end
		-- selected tab button (bright)
		local main = gui:FindFirstChild("Main")
		if main then
			for _, b in ipairs(main:GetDescendants()) do
				if b:IsA("TextButton") and b.Parent and b.Parent.Name ~= "Main" then
					-- skip
				end
			end
		end
	end)
	if env.__NnnnSetAccent then
		pcall(env.__NnnnSetAccent, col)
	end
end

local function buildCustomUI()
	pcall(function()
		for _, n in ipairs({"NnnnHubUI", "NnnnToggleGui"}) do
			local o = CoreGui:FindFirstChild(n)
			if o then o:Destroy() end
			local pg = LocalPlayer:FindFirstChild("PlayerGui")
			if pg then
				local o2 = pg:FindFirstChild(n)
				if o2 then o2:Destroy() end
			end
		end
	end)

	-- Ronaldo Hook palette (black + gold)
	local ACCENT = Color3.fromRGB(212, 175, 55)
	Settings.ThemeColor = Settings.ThemeColor or ACCENT
	local BG = Color3.fromRGB(12, 12, 14)
	local PANEL = Color3.fromRGB(20, 20, 24)
	local CARD = Color3.fromRGB(26, 26, 32)
	local STROKE = Color3.fromRGB(55, 48, 30)
	local TEXT = Color3.fromRGB(245, 240, 230)
	local MUTED = Color3.fromRGB(150, 140, 120)

	local gui = Instance.new("ScreenGui")
	gui.Name = "NnnnHubUI"
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	gui.DisplayOrder = 999
	gui.IgnoreGuiInset = true
	gui.Enabled = true
	pcall(function()
		if gethui then gui.Parent = gethui() end
	end)
	if not gui.Parent then pcall(function() gui.Parent = CoreGui end) end
	if not gui.Parent then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end
	print("[Ronaldo Hook] UI parent =", gui.Parent and gui.Parent:GetFullName())

	local main = Instance.new("Frame")
	main.Name = "Main"
	main.Size = UDim2.new(0, 760, 0, 540)
	main.Position = UDim2.new(0.5, -380, 0.5, -270)
	main.BackgroundColor3 = BG
	main.BackgroundTransparency = 0
	main.BorderSizePixel = 0
	main.ClipsDescendants = true
	main.Visible = true
	main.Parent = gui
	Instance.new("UICorner", main).CornerRadius = UDim.new(0, 14)
	local mainStroke = Instance.new("UIStroke")
	mainStroke.Color = ACCENT
	mainStroke.Thickness = 1.5
	mainStroke.Transparency = 0.25
	mainStroke.Parent = main

	-- gold accent edge
	local leftGlow = Instance.new("Frame")
	leftGlow.Size = UDim2.new(0, 4, 1, 0)
	leftGlow.BackgroundColor3 = ACCENT
	leftGlow.BorderSizePixel = 0
	leftGlow.ZIndex = 3
	leftGlow.Parent = main

	local topLine = Instance.new("Frame")
	topLine.Size = UDim2.new(1, 0, 0, 2)
	topLine.BackgroundColor3 = ACCENT
	topLine.BorderSizePixel = 0
	topLine.ZIndex = 12
	topLine.Parent = main

	-- title bar
	local titleBar = Instance.new("Frame")
	titleBar.Size = UDim2.new(1, 0, 0, 42)
	titleBar.BackgroundColor3 = PANEL
	titleBar.BackgroundTransparency = 0
	titleBar.BorderSizePixel = 0
	titleBar.ZIndex = 10
	titleBar.Parent = main

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -80, 1, 0)
	title.Position = UDim2.new(0, 18, 0, 0)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.GothamBold
	title.TextSize = 16
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.TextColor3 = ACCENT
	title.Text = "RONALDO HOOK"
	title.ZIndex = 11
	title.Parent = titleBar

	local closeBtn = Instance.new("TextButton")
	closeBtn.Size = UDim2.new(0, 28, 0, 28)
	closeBtn.Position = UDim2.new(1, -34, 0.5, -14)
	closeBtn.BackgroundColor3 = Color3.fromRGB(35, 28, 20)
	closeBtn.Text = "×"
	closeBtn.TextColor3 = ACCENT
	closeBtn.Font = Enum.Font.GothamBold
	closeBtn.TextSize = 16
	closeBtn.ZIndex = 11
	closeBtn.Parent = titleBar
	Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 4)

	-- TOP horizontal tabs (ONYX style)
	local tabBar = Instance.new("Frame")
	tabBar.Size = UDim2.new(1, -20, 0, 32)
	tabBar.Position = UDim2.new(0, 12, 0, 48)
	tabBar.BackgroundTransparency = 1
	tabBar.ZIndex = 10
	tabBar.Parent = main

	local tabLay = Instance.new("UIListLayout")
	tabLay.FillDirection = Enum.FillDirection.Horizontal
	tabLay.Padding = UDim.new(0, 4)
	tabLay.SortOrder = Enum.SortOrder.LayoutOrder
	tabLay.Parent = tabBar

	-- content under tabs
	local content = Instance.new("Frame")
	content.Size = UDim2.new(1, -20, 1, -92)
	content.Position = UDim2.new(0, 12, 0, 86)
	content.BackgroundTransparency = 1
	content.ZIndex = 10
	content.Parent = main

	local pages = {}
	local tabBtns = {}
	local tabNames = {"Aim", "Rage", "Character", "ESP", "Visuals", "World", "Misc", "Cfg"}
	local tabLabels = {
		Aim = "main",
		Rage = "rage",
		Character = "character",
		ESP = "esp",
		Visuals = "visuals",
		World = "world",
		Misc = "misc",
		Cfg = "settings",
	}

	local function makePage(name)
		-- two-column page container
		local holder = Instance.new("Frame")
		holder.Name = name
		holder.Size = UDim2.new(1, 0, 1, 0)
		holder.BackgroundTransparency = 1
		holder.Visible = false
		holder.Parent = content

		local left = Instance.new("ScrollingFrame")
		left.Name = "Left"
		left.Size = UDim2.new(0.5, -4, 1, 0)
		left.Position = UDim2.new(0, 0, 0, 0)
		left.BackgroundTransparency = 1
		left.BorderSizePixel = 0
		left.ScrollBarThickness = 3
		left.ScrollBarImageColor3 = ACCENT
		left.CanvasSize = UDim2.new(0, 0, 0, 0)
		left.Parent = holder
		local ll = Instance.new("UIListLayout")
		ll.Padding = UDim.new(0, 8)
		ll.SortOrder = Enum.SortOrder.LayoutOrder
		ll.Parent = left
		ll:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
			left.CanvasSize = UDim2.new(0, 0, 0, ll.AbsoluteContentSize.Y + 12)
		end)
		local lp = Instance.new("UIPadding")
		lp.PaddingTop = UDim.new(0, 2)
		lp.PaddingBottom = UDim.new(0, 8)
		lp.PaddingRight = UDim.new(0, 4)
		lp.Parent = left

		local right = Instance.new("ScrollingFrame")
		right.Name = "Right"
		right.Size = UDim2.new(0.5, -4, 1, 0)
		right.Position = UDim2.new(0.5, 4, 0, 0)
		right.BackgroundTransparency = 1
		right.BorderSizePixel = 0
		right.ScrollBarThickness = 3
		right.ScrollBarImageColor3 = ACCENT
		right.CanvasSize = UDim2.new(0, 0, 0, 0)
		right.Parent = holder
		local rl = Instance.new("UIListLayout")
		rl.Padding = UDim.new(0, 8)
		rl.SortOrder = Enum.SortOrder.LayoutOrder
		rl.Parent = right
		rl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
			right.CanvasSize = UDim2.new(0, 0, 0, rl.AbsoluteContentSize.Y + 12)
		end)
		local rp = Instance.new("UIPadding")
		rp.PaddingTop = UDim.new(0, 2)
		rp.PaddingBottom = UDim.new(0, 8)
		rp.PaddingLeft = UDim.new(0, 4)
		rp.Parent = right

		-- pages[name] = left by default for backward compat (addToggle parents to pages.X)
		-- Also store left/right
		pages[name] = left
		pages[name .. "_L"] = left
		pages[name .. "_R"] = right
		pages[name .. "_Holder"] = holder
		return left, right
	end

	local function makeRow(page)
		local row = Instance.new("Frame")
		row.BackgroundTransparency = 1
		row.Size = UDim2.new(1, 0, 0, 40)
		row.AutomaticSize = Enum.AutomaticSize.Y
		row.Parent = page
		local lay = Instance.new("UIListLayout")
		lay.FillDirection = Enum.FillDirection.Vertical
		lay.Padding = UDim.new(0, 8)
		lay.SortOrder = Enum.SortOrder.LayoutOrder
		lay.Parent = row
		return row
	end

	for _, n in ipairs(tabNames) do makePage(n) end

	-- put some sections on right column for balance
	local function pageSide(name, side)
		if side == "R" then return pages[name .. "_R"] end
		return pages[name .. "_L"]
	end

	local function switchTab(name)
		for _, n in ipairs(tabNames) do
			local h = pages[n .. "_Holder"]
			if h then h.Visible = (n == name) end
		end
		for k, b in pairs(tabBtns) do
			if k == name then
				b.BackgroundColor3 = ACCENT
				b.TextColor3 = Color3.fromRGB(255, 255, 255)
			else
				b.BackgroundColor3 = CARD
				b.TextColor3 = MUTED
			end
		end
	end

	for i, n in ipairs(tabNames) do
		local b = Instance.new("TextButton")
		b.Size = UDim2.new(0, 86, 1, 0)
		b.BackgroundColor3 = CARD
		b.Text = tabLabels[n] or string.lower(n)
		b.Font = Enum.Font.GothamBold
		b.TextSize = 11
		b.TextColor3 = MUTED
		b.LayoutOrder = i
		b.AutoButtonColor = false
		b.Parent = tabBar
		Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
		local st = Instance.new("UIStroke")
		st.Color = STROKE
		st.Thickness = 1
		st.Parent = b
		b.MouseButton1Click:Connect(function() switchTab(n) end)
		tabBtns[n] = b
	end
	switchTab("Aim")
	env.__NnnnSetAccent = function(col)
		if typeof(col) ~= "Color3" then return end
		ACCENT = col
		Settings.ThemeColor = col
		if CfgSys and CfgSys.theme then CfgSys.theme.Accent = col end
		for _, d in ipairs(gui:GetDescendants()) do
			if d.Name == "AccentBar" and d:IsA("Frame") then
				d.BackgroundColor3 = col
			end
			if d:IsA("ScrollingFrame") then
				d.ScrollBarImageColor3 = col
			end
		end
		for k, b in pairs(tabBtns) do
			local h = pages[k .. "_Holder"]
			if h and h.Visible then
				b.BackgroundColor3 = col
			end
		end
	end

	-- helpers
	-- Linoria 스타일 이름 있는 그룹 박스 (가로 절반 가능)
	local function section(page, text, half)
		local box = Instance.new("Frame")
		box.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
		box.BorderSizePixel = 0
		box.Size = UDim2.new(1, 0, 0, 40)
		box.AutomaticSize = Enum.AutomaticSize.Y
		box.Parent = page
		Instance.new("UICorner", box).CornerRadius = UDim.new(0, 10)
		local st = Instance.new("UIStroke")
		st.Color = STROKE
		st.Thickness = 1
		st.Parent = box

		local head = Instance.new("Frame")
		head.Size = UDim2.new(1, 0, 0, 28)
		head.BackgroundColor3 = Color3.fromRGB(26, 26, 32)
		head.BorderSizePixel = 0
		head.Parent = box
		Instance.new("UICorner", head).CornerRadius = UDim.new(0, 6)
		local accentBar = Instance.new("Frame")
		accentBar.Name = "AccentBar"
		accentBar.Size = UDim2.new(0, 3, 1, -8)
		accentBar.Position = UDim2.new(0, 6, 0, 4)
		accentBar.BackgroundColor3 = ACCENT
		accentBar.BorderSizePixel = 0
		accentBar.Parent = head
		Instance.new("UICorner", accentBar).CornerRadius = UDim.new(1, 0)
		-- 하단만 각지게 덮개
		local headFix = Instance.new("Frame")
		headFix.Size = UDim2.new(1, 0, 0, 10)
		headFix.Position = UDim2.new(0, 0, 1, -10)
		headFix.BackgroundColor3 = Color3.fromRGB(22, 25, 32)
		headFix.BorderSizePixel = 0
		headFix.Parent = head

		local accentBar = Instance.new("Frame")
		accentBar.Size = UDim2.new(0, 3, 1, -8)
		accentBar.Position = UDim2.new(0, 6, 0, 4)
		accentBar.BackgroundColor3 = ACCENT
		accentBar.BorderSizePixel = 0
		accentBar.Parent = head
		Instance.new("UICorner", accentBar).CornerRadius = UDim.new(1, 0)

		local title = Instance.new("TextLabel")
		title.Size = UDim2.new(1, -24, 1, 0)
		title.Position = UDim2.new(0, 16, 0, 0)
		title.BackgroundTransparency = 1
		title.Font = Enum.Font.GothamBold
		title.TextSize = 12
		title.TextColor3 = TEXT
		title.TextXAlignment = Enum.TextXAlignment.Left
		title.Text = text
		title.Parent = head

		local body = Instance.new("Frame")
		body.Name = "Body"
		body.BackgroundTransparency = 1
		body.Size = UDim2.new(1, -12, 0, 0)
		body.Position = UDim2.new(0, 6, 0, 32)
		body.AutomaticSize = Enum.AutomaticSize.Y
		body.Parent = box
		local lay = Instance.new("UIListLayout")
		lay.Parent = body
		lay.Padding = UDim.new(0, 6)
		lay.SortOrder = Enum.SortOrder.LayoutOrder
		local pad = Instance.new("UIPadding")
		pad.PaddingBottom = UDim.new(0, 8)
		pad.Parent = body

		-- 하위 컨트롤이 body에 붙도록 page 대용 반환
		return body
	end

	local _uiParent = nil
	local function setParent(p) _uiParent = p end
	local function parentOf(page) return _uiParent or page end

	local function addToggle(page, text, default, cb)
		page = parentOf(page)

		local row = Instance.new("Frame")
		row.Size = UDim2.new(1, 0, 0, 38)
		row.BackgroundColor3 = CARD
		row.BorderSizePixel = 0
		row.Parent = page
		Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)

		local lab = Instance.new("TextLabel")
		lab.Size = UDim2.new(1, -56, 1, 0)
		lab.Position = UDim2.new(0, 12, 0, 0)
		lab.BackgroundTransparency = 1
		lab.Font = Enum.Font.Gotham
		lab.TextSize = 14
		lab.TextColor3 = TEXT
		lab.TextXAlignment = Enum.TextXAlignment.Left
		lab.Text = text
		lab.Parent = row

		local track = Instance.new("Frame")
		track.Size = UDim2.new(0, 40, 0, 20)
		track.Position = UDim2.new(1, -48, 0.5, -10)
		track.BackgroundColor3 = default and ACCENT or Color3.fromRGB(45, 48, 58)
		track.BorderSizePixel = 0
		track.Parent = row
		Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

		local knob = Instance.new("Frame")
		knob.Size = UDim2.new(0, 16, 0, 16)
		knob.Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
		knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		knob.BorderSizePixel = 0
		knob.Parent = track
		Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

		local state = default and true or false
		local hit = Instance.new("TextButton")
		hit.Size = UDim2.new(1, 0, 1, 0)
		hit.BackgroundTransparency = 1
		hit.Text = ""
		hit.Parent = row
		hit.MouseButton1Click:Connect(function()
			state = not state
			TweenService:Create(track, TweenInfo.new(0.15), {
				BackgroundColor3 = state and ACCENT or Color3.fromRGB(45, 48, 58)
			}):Play()
			TweenService:Create(knob, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
				Position = state and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
			}):Play()
			pcall(cb, state)
		end)
	end

	local function addSlider(page, text, min, max, cur, cb)
		page = parentOf(page)
		local row = Instance.new("Frame")
		row.Size = UDim2.new(1, 0, 0, 48)
		row.BackgroundColor3 = CARD
		row.BorderSizePixel = 0
		row.Parent = page
		Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)

		local lab = Instance.new("TextLabel")
		lab.Size = UDim2.new(1, -16, 0, 20)
		lab.Position = UDim2.new(0, 12, 0, 4)
		lab.BackgroundTransparency = 1
		lab.Font = Enum.Font.Gotham
		lab.TextSize = 12
		lab.TextColor3 = TEXT
		lab.TextXAlignment = Enum.TextXAlignment.Left
		lab.Text = text .. ": " .. tostring(cur)
		lab.Parent = row

		local bar = Instance.new("TextButton")
		bar.Size = UDim2.new(1, -24, 0, 8)
		bar.Position = UDim2.new(0, 12, 0, 30)
		bar.BackgroundColor3 = Color3.fromRGB(40, 44, 54)
		bar.Text = ""
		bar.BorderSizePixel = 0
		bar.Parent = row
		Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

		local fill = Instance.new("Frame")
		fill.BackgroundColor3 = ACCENT
		fill.BorderSizePixel = 0
		fill.Size = UDim2.new(math.clamp((cur - min) / math.max(1e-6, max - min), 0, 1), 0, 1, 0)
		fill.Parent = bar
		Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

		bar.MouseButton1Down:Connect(function()
			local conn
			conn = RunService.RenderStepped:Connect(function()
				if not UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
					if conn then conn:Disconnect() end
					return
				end
				local rel = math.clamp((UserInputService:GetMouseLocation().X - bar.AbsolutePosition.X) / math.max(1, bar.AbsoluteSize.X), 0, 1)
				local val = min + (max - min) * rel
				if max - min >= 5 then val = math.floor(val + 0.5)
				else val = math.floor(val * 10 + 0.5) / 10 end
				fill.Size = UDim2.new(rel, 0, 1, 0)
				lab.Text = text .. ": " .. tostring(val)
				pcall(cb, val)
			end)
		end)
	end

	local function addButton(page, text, cb)
		page = parentOf(page)
		local b = Instance.new("TextButton")
		b.Size = UDim2.new(1, 0, 0, 34)
		b.BackgroundColor3 = Color3.fromRGB(32, 36, 46)
		b.Text = text
		b.Font = Enum.Font.GothamMedium
		b.TextSize = 13
		b.TextColor3 = TEXT
		b.Parent = page
		Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
		local st = Instance.new("UIStroke")
		st.Color = STROKE
		st.Thickness = 1
		st.Parent = b
		b.MouseButton1Click:Connect(function() pcall(cb) end)
	end

	local function addDropdown(page, text, options, defaultIdx, cb)
		page = parentOf(page)
		local wrap = Instance.new("Frame")
		wrap.BackgroundTransparency = 1
		wrap.Size = UDim2.new(1, 0, 0, 34)
		wrap.ClipsDescendants = true
		wrap.Parent = page

		local header = Instance.new("TextButton")
		header.Size = UDim2.new(1, 0, 0, 34)
		header.BackgroundColor3 = CARD
		header.BorderSizePixel = 0
		header.AutoButtonColor = true
		header.Text = ""
		header.Parent = wrap
		Instance.new("UICorner", header).CornerRadius = UDim.new(0, 8)

		local lab = Instance.new("TextLabel")
		lab.Size = UDim2.new(0.4, 0, 1, 0)
		lab.Position = UDim2.new(0, 12, 0, 0)
		lab.BackgroundTransparency = 1
		lab.Font = Enum.Font.Gotham
		lab.TextSize = 12
		lab.TextColor3 = MUTED
		lab.TextXAlignment = Enum.TextXAlignment.Left
		lab.Text = text
		lab.Parent = header

		local idx = defaultIdx or 1
		local val = options[idx] or options[1]
		local valL = Instance.new("TextLabel")
		valL.Size = UDim2.new(0.45, 0, 1, 0)
		valL.Position = UDim2.new(0.4, 0, 0, 0)
		valL.BackgroundTransparency = 1
		valL.Font = Enum.Font.GothamMedium
		valL.TextSize = 12
		valL.TextColor3 = TEXT
		valL.TextXAlignment = Enum.TextXAlignment.Right
		valL.Text = tostring(val)
		valL.Parent = header

		local arrow = Instance.new("TextLabel")
		arrow.Size = UDim2.new(0, 20, 1, 0)
		arrow.Position = UDim2.new(1, -24, 0, 0)
		arrow.BackgroundTransparency = 1
		arrow.Font = Enum.Font.GothamBold
		arrow.TextSize = 12
		arrow.TextColor3 = ACCENT
		arrow.Text = "▾"
		arrow.Parent = header

		local list = Instance.new("Frame")
		list.BackgroundColor3 = Color3.fromRGB(20, 22, 28)
		list.BorderSizePixel = 0
		list.Position = UDim2.new(0, 0, 0, 36)
		list.Size = UDim2.new(1, 0, 0, 0)
		list.ClipsDescendants = true
		list.Parent = wrap
		Instance.new("UICorner", list).CornerRadius = UDim.new(0, 8)
		local listLay = Instance.new("UIListLayout")
		listLay.Parent = list
		listLay.SortOrder = Enum.SortOrder.LayoutOrder
		listLay.Padding = UDim.new(0, 2)
		local listPad = Instance.new("UIPadding")
		listPad.PaddingTop = UDim.new(0, 4)
		listPad.PaddingBottom = UDim.new(0, 4)
		listPad.PaddingLeft = UDim.new(0, 4)
		listPad.PaddingRight = UDim.new(0, 4)
		listPad.Parent = list

		local open = false
		local itemH = 28
		local maxShow = math.min(#options, 8)
		local fullH = 4 + maxShow * (itemH + 2) + 4

		for i, opt in ipairs(options) do
			local ob = Instance.new("TextButton")
			ob.Size = UDim2.new(1, 0, 0, itemH)
			ob.BackgroundColor3 = Color3.fromRGB(30, 33, 42)
			ob.Text = "  " .. tostring(opt)
			ob.Font = Enum.Font.Gotham
			ob.TextSize = 12
			ob.TextColor3 = TEXT
			ob.TextXAlignment = Enum.TextXAlignment.Left
			ob.LayoutOrder = i
			ob.Parent = list
			Instance.new("UICorner", ob).CornerRadius = UDim.new(0, 5)
			ob.MouseButton1Click:Connect(function()
				idx = i
				val = opt
				valL.Text = tostring(val)
				pcall(cb, val)
				open = false
				arrow.Text = "▾"
				TweenService:Create(wrap, TweenInfo.new(0.22, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, 34)
				}):Play()
				TweenService:Create(list, TweenInfo.new(0.22, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, 0)
				}):Play()
			end)
		end

		header.MouseButton1Click:Connect(function()
			open = not open
			if open then
				arrow.Text = "▴"
				TweenService:Create(wrap, TweenInfo.new(0.28, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, 34 + fullH + 4)
				}):Play()
				TweenService:Create(list, TweenInfo.new(0.28, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, fullH)
				}):Play()
			else
				arrow.Text = "▾"
				TweenService:Create(wrap, TweenInfo.new(0.22, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, 34)
				}):Play()
				TweenService:Create(list, TweenInfo.new(0.22, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
					Size = UDim2.new(1, 0, 0, 0)
				}):Play()
			end
		end)
	end

	-- ===== AIM =====
	setParent(section(pages.Aim, "silent aim"))
	addToggle(pages.Aim, "enabled", Settings.AimbotEnabled, function(v)
		Settings.AimbotEnabled = v
		if v then pcall(setupSilentAimHook) end
		showNotification("Silent Aim: " .. tostring(v))
	end)
	addToggle(pages.Aim, "show fov", Settings.ShowFOV, function(v) Settings.ShowFOV = v end)
	addToggle(pages.Aim, "team check", Settings.TeamCheck, function(v) Settings.TeamCheck = v end)
	addSlider(pages.Aim, "fov radius", 30, 500, Settings.FOVRadius, function(v) Settings.FOVRadius = v end)
	addSlider(pages.Aim, "hit chance", 1, 100, Settings.HitChance or 100, function(v) Settings.HitChance = v end)

	setParent(section(pages.Aim_R, "triggerbot"))
	addToggle(pages.Aim_R, "enabled", Settings.Triggerbot, function(v)
		Settings.Triggerbot = v
		showNotification("Triggerbot: " .. tostring(v))
	end)
	addSlider(pages.Aim_R, "reaction time", 1, 30, math.floor((Settings.TriggerDelay or 0.05) * 100), function(v)
		Settings.TriggerDelay = v / 100
	end)
	addSlider(pages.Aim_R, "trigger fov", 10, 120, Settings.TriggerFOV or 40, function(v)
		Settings.TriggerFOV = v
	end)
	addToggle(pages.Aim_R, "resolver", Settings.Resolver, function(v)
		Settings.Resolver = v
	end)

	setParent(section(pages.Aim_R, "weapons"))
	addToggle(pages.Aim_R, "no spread", Settings.NoSpread, function(v)
		Settings.NoSpread = v
		if applyNoSpread then pcall(applyNoSpread, v) end
		showNotification("No Spread: " .. tostring(v))
	end)
	addToggle(pages.Aim_R, "rapid fire", Settings.RapidFire, function(v)
		Settings.RapidFire = v
		if applyFastFireAndMelee then pcall(applyFastFireAndMelee, v) end
	end)

	-- ===== RAGE =====
	do
		local row = makeRow(pages.Rage)
		setParent(section(row, "Ultra Rage", true))
		addToggle(pages.Rage, "ULTRA RAGE", RageConfig.on, function(v)
			RageConfig.on = v
			if v then startRage() else stopRage() end
			showNotification("Ultra Rage: " .. tostring(v))
		end)
		addToggle(pages.Rage, "Continuous Desync", RageConfig.desync, function(v)
			RageConfig.desync = v
			if not v then stopContinuousDesync() end
		end)
		addToggle(pages.Rage, "Wait For Shield", Settings.WaitForShield, function(v) Settings.WaitForShield = v end)
		addToggle(pages.Rage, "View Shake", Settings.IntenseViewShake, function(v) Settings.IntenseViewShake = v end)
		setParent(section(row, "Timing", true))
		addSlider(pages.Rage, "Hidden x0.01", 1, 100, math.floor((RageConfig.hide or 0.05) * 100), function(v) RageConfig.hide = v / 100 end)
		addSlider(pages.Rage, "Attack x0.01", 1, 100, math.floor((RageConfig.attack or 0.88) * 100), function(v) RageConfig.attack = v / 100 end)
		setParent(nil)
	end

	-- ===== CHARACTER (Lion Anti Aim + Animation Player) =====
	setParent(section(pages.Character, "Anti Aim"))
	addToggle(pages.Character, "Enabled", Settings.AntiAimEnabled, function(v)
		Settings.AntiAimEnabled = v
		if v then
			pcall(startLionAntiAim)
		else
			pcall(stopLionAntiAim)
		end
	end)
	addDropdown(pages.Character, "Pitch", {"disabled", "up", "down", "zero", "random"}, 1, function(v)
		Settings.AntiAimPitch = v
		if _G.AntiAimPoseConfig then _G.AntiAimPoseConfig.pitch = v end
	end)
	addDropdown(pages.Character, "Yaw", {"disabled", "backwards", "spin", "random"}, 1, function(v)
		Settings.AntiAimYaw = v
		if _G.AntiAimPoseConfig then _G.AntiAimPoseConfig.yaw = v end
	end)
	addToggle(pages.Character, "Underground", Settings.AntiAimUnderground, function(v)
		Settings.AntiAimUnderground = v
		if _G.AntiAimPoseConfig then _G.AntiAimPoseConfig.underground = v end
	end)

	-- Animation Player
	setParent(section(pages.Character_R, "Animation Player"))
	addToggle(pages.Character_R, "Anim Enabled", AnimPlayer.Enabled, function(v)
		AnimPlayer.Enabled = v
		if v then
			pcall(animPlay)
		else
			pcall(animStop)
		end
	end)
	addDropdown(pages.Character_R, "Animation", ANIM_LIST, 1, function(v)
		AnimPlayer.Preset = v
		if AnimPlayer.Enabled then pcall(animPlay) end
	end)
	do
		local page = pages.Character
		local row = Instance.new("Frame")
		row.Size = UDim2.new(1, 0, 0, 36)
		row.BackgroundColor3 = Color3.fromRGB(24, 27, 34)
		row.BorderSizePixel = 0
		row.Parent = page
		Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)
		local lab = Instance.new("TextLabel")
		lab.Size = UDim2.new(0.4, 0, 1, 0)
		lab.Position = UDim2.new(0, 10, 0, 0)
		lab.BackgroundTransparency = 1
		lab.Font = Enum.Font.Gotham
		lab.TextSize = 12
		lab.TextColor3 = Color3.fromRGB(140, 148, 165)
		lab.TextXAlignment = Enum.TextXAlignment.Left
		lab.Text = "Custom ID"
		lab.Parent = row
		local box = Instance.new("TextBox")
		box.Size = UDim2.new(0.52, -8, 0, 26)
		box.Position = UDim2.new(0.42, 0, 0.5, -13)
		box.BackgroundColor3 = Color3.fromRGB(32, 36, 46)
		box.Text = tostring(AnimPlayer.CustomId or "")
		box.PlaceholderText = "id... (ex: 4049646104)"
		box.Font = Enum.Font.Gotham
		box.TextSize = 12
		box.TextColor3 = Color3.fromRGB(235, 238, 245)
		box.ClearTextOnFocus = false
		box.Parent = row
		Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
		box.FocusLost:Connect(function()
			AnimPlayer.CustomId = box.Text
			if AnimPlayer.Enabled and AnimPlayer.Preset == "Custom" then
				pcall(animPlay)
			end
		end)
	end
	addSlider(pages.Character_R, "Speed x0.1", 1, 50, math.floor((AnimPlayer.Speed or 1) * 10), function(v)
		AnimPlayer.Speed = v / 10
		if _apTrack then pcall(function() _apTrack:AdjustSpeed(AnimPlayer.Speed) end) end
	end)
	addSlider(pages.Character_R, "Start %", 0, 99, AnimPlayer.StartPct or 0, function(v)
		AnimPlayer.StartPct = v
	end)
	addSlider(pages.Character_R, "End %", 1, 100, AnimPlayer.EndPct or 100, function(v)
		AnimPlayer.EndPct = v
	end)

	-- ===== ESP =====
	setParent(section(pages.ESP, "ESP"))
	addToggle(pages.ESP, "ESP Enabled", ESPConfig.enabled, function(v) ESPConfig.enabled = v end)
	addToggle(pages.ESP, "Box", ESPConfig.box, function(v) ESPConfig.box = v end)
	addToggle(pages.ESP, "Distance", ESPConfig.distance, function(v) ESPConfig.distance = v end)
	addToggle(pages.ESP, "Name", ESPConfig.name, function(v) ESPConfig.name = v end)
	addToggle(pages.ESP, "Health Text", ESPConfig.health, function(v) ESPConfig.health = v end)
	addToggle(pages.ESP, "Health Bar", ESPConfig.healthBar, function(v) ESPConfig.healthBar = v end)
	addToggle(pages.ESP, "Skeleton", ESPConfig.skeleton, function(v) ESPConfig.skeleton = v end)

	-- ===== VISUALS (Lion: show cosmetics changer) =====
	setParent(section(pages.Visuals, "Viewmodel"))
	addToggle(pages.Visuals, "show cosmetics changer", false, function(v)
		if v then
			Settings.UnlockAllSkins = true
			pcall(enableAllSkins)
			pcall(openSkinPicker)
			showNotification("Cosmetics changer: ON")
		else
			pcall(function()
				local g = CoreGui:FindFirstChild("NnnnSkinPicker")
					or (LocalPlayer.PlayerGui and LocalPlayer.PlayerGui:FindFirstChild("NnnnSkinPicker"))
				if g then g:Destroy() end
			end)
			showNotification("Cosmetics changer: OFF")
		end
	end)
	addToggle(pages.Visuals, "Rapid Fire", Settings.RapidFire, function(v)
		Settings.RapidFire = v
		if v then applyFastFireAndMelee(true)
		elseif not RageConfig.on then applyFastFireAndMelee(false) end
	end)
	addToggle(pages.Visuals, "Target HUD", Settings.TargetHUD, function(v)
		Settings.TargetHUD = v
	end)
	addToggle(pages.Visuals, "Hit Marker", Settings.HitMarker, function(v)
		Settings.HitMarker = v
	end)
	addToggle(pages.Visuals, "Viewmodel Offset", Settings.ViewmodelOffset, function(v)
		Settings.ViewmodelOffset = v
	end)
	addSlider(pages.Visuals, "VM X", -10, 10, Settings.VM_X or 0, function(v) Settings.VM_X = v end)
	addSlider(pages.Visuals, "VM Y", -10, 10, Settings.VM_Y or 0, function(v) Settings.VM_Y = v end)
	addSlider(pages.Visuals, "VM Z", -10, 10, Settings.VM_Z or 0, function(v) Settings.VM_Z = v end)
	addButton(pages.Visuals, "Save Cosmetics Config", function()
		Settings.UnlockAllSkins = true
		pcall(enableAllSkins)
		local api = env.__NnnnSkinAPI
		if api and api.savecfg then api.savecfg() showNotification("Cosmetics saved")
		else showNotification("API 없음 — changer 먼저 켜기") end
	end)
	addButton(pages.Visuals, "Load Cosmetics Config", function()
		Settings.UnlockAllSkins = true
		pcall(enableAllSkins)
		local api = env.__NnnnSkinAPI
		if api and api.loadcfg then
			api.loadcfg()
			pcall(api.rebuildinv)
			pcall(api.saferep, "WeaponInventory")
			pcall(api.saferep, "CosmeticInventory")
			showNotification("Cosmetics loaded")
		else showNotification("API 없음") end
	end)


	-- ===== WORLD (Harion / Lion style) =====
	setParent(section(pages.World, "Camera"))
	addSlider(pages.World, "Field of View", 50, 120, Camera.FieldOfView, function(v)
		pcall(function() Camera.FieldOfView = v end)
	end)

	-- skybox (Harion style)
	setParent(section(pages.World, "skybox"))
	addToggle(pages.World, "enabled", false, function(v)
		LionWorld.SkyEnabled = v
		pcall(lionApplySkybox, LionWorld.SkyName, v)
		showNotification("skybox: " .. tostring(v))
	end)
	addDropdown(pages.World, "Selected", {
		"Afternoon", "Blue Space", "Classic Roblox", "Cloudy", "Dusk", "Dawn",
		"Dark Skies", "Earth", "Horizontal Milky Way", "Heaven", "Jungle", "Mountains",
		"Nebula", "Night Light", "Night", "Ocean Sky", "Redshift", "Space", "Sunset",
		"Storm", "SFOTH", "Solid Black", "Saturn", "Smoke", "Vertical Milky Way", "White",
	}, 1, function(v)
		LionWorld.SkyName = v
		if LionWorld.SkyEnabled then
			pcall(lionApplySkybox, v, true)
		end
	end)

	-- weather
	setParent(section(pages.World_R, "weather"))
	addToggle(pages.World, "enabled", false, function(v)
		LionWorld.WeatherEnabled = v
		if v then
			pcall(lionApplyWeather, LionWorld.WeatherName)
		else
			pcall(lionClearWeather)
		end
		showNotification("weather: " .. tostring(v))
	end)
	addDropdown(pages.World, "Selected", {
		"Heavy Rain", "Rain", "Snow", "City", "Windy Day",
		"Heavy Rain Night", "Forest Rain Night", "Light Rain And Thunder",
	}, 1, function(v)
		LionWorld.WeatherName = v
		if LionWorld.WeatherEnabled then
			pcall(lionApplyWeather, v)
		end
	end)

	-- ambience
	setParent(section(pages.World_R, "ambience"))
	addToggle(pages.World, "enabled", false, function(v)
		LionWorld.AmbienceEnabled = v
		if v then
			pcall(lionPlayAmbience, LionWorld.AmbienceName)
		else
			pcall(lionStopAmbience)
		end
	end)
	addDropdown(pages.World, "Selected", {
		"Heavy Rain", "Rain", "City", "Windy Day",
		"Heavy Rain Night", "Forest Rain Night", "Light Rain And Thunder",
	}, 1, function(v)
		LionWorld.AmbienceName = v
		if LionWorld.AmbienceEnabled then
			pcall(lionPlayAmbience, v)
		end
	end)
	addSlider(pages.World, "Volume x0.1", 0, 20, 5, function(v)
		LionWorld.AmbienceVol = v / 10
		if LionWorld.ambienceSound then
			pcall(function() LionWorld.ambienceSound.Volume = LionWorld.AmbienceVol end)
		end
	end)


	-- ===== MISC =====
	do
		local row = makeRow(pages.Misc)
		setParent(section(row, "Player", true))
		addToggle(pages.Misc, "Fly", Settings.FlyEnabled, function(v) toggleFly(v) end)
		addToggle(pages.Misc, "Noclip", Settings.NoclipEnabled, function(v)
			pcall(function() toggleNoclip(v) end)
		end)
		addSlider(pages.Misc, "Fly Speed", 1, 100, Settings.FlySpeed, function(v) Settings.FlySpeed = v end)
		addSlider(pages.Misc, "WalkSpeed", 16, 150, Settings.WalkSpeed, function(v)
			Settings.WalkSpeed = v
			local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
			if h then h.WalkSpeed = v end
		end)
		addToggle(pages.Misc, "Custom Jump", Settings.CustomJump, function(v)
			Settings.CustomJump = v
			if not v then
				pcall(function()
					local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
					if hum then
						hum.UseJumpPower = false
						hum.JumpHeight = 7.2
					end
				end)
			end
		end)
		addSlider(pages.Misc, "JumpPower", 30, 120, Settings.JumpPower or 50, function(v)
			Settings.JumpPower = v
		end)
		addToggle(pages.Misc, "Infinite Jump", Settings.InfJump, function(v)
			Settings.InfJump = v
		end)
		addToggle(pages.Misc, "No Fall", Settings.NoFall, function(v)
			Settings.NoFall = v
		end)
		setParent(section(row, "Tracer", true))
		addToggle(pages.Misc, "Bullet Tracer", Settings.BulletTracer, function(v) Settings.BulletTracer = v end)
		addToggle(pages.Misc, "Rainbow Tracer", Settings.TracerMode == "Rainbow", function(v)
			Settings.TracerMode = v and "Rainbow" or "Solid"
		end)
		setParent(nil)
	end
	setParent(section(pages.Misc, "Third Person"))
	addToggle(pages.Misc, "Enabled", Settings.ThirdPersonEnabled, function(v)
		Settings.ThirdPersonEnabled = v
		if v then
			pcall(startThirdPerson)
		else
			pcall(stopThirdPerson)
		end
		showNotification("Third Person: " .. tostring(v))
	end)
	addDropdown(pages.Misc, "Mode", {"ThirdPerson", "ThirdPerson Mirrored"}, 1, function(v)
		Settings.ThirdPersonMode = v
	end)
	addToggle(pages.Misc, "Unlock Mouse", Settings.UnlockMouse, function(v)
		Settings.UnlockMouse = v
	end)

	setParent(section(pages.Misc, "name spoofer"))
	addToggle(pages.Misc, "enabled", Settings.NameSpoofEnabled, function(v)
		Settings.NameSpoofEnabled = v
		NameSpoof.enabled = v
		if NameSpoof.state then
			NameSpoof.state.Enabled = v
			NameSpoof.state.Name = Settings.NameSpoofText or NameSpoof.name
		end
		NameSpoof.name = Settings.NameSpoofText or NameSpoof.name
		if v then
			applyNameSpoof()
		else
			restoreNameSpoof()
		end
		showNotification("name spoofer: " .. tostring(v))
	end)
	do
		local page = pages.Misc
		local row = Instance.new("Frame")
		row.Size = UDim2.new(1, 0, 0, 36)
		row.BackgroundColor3 = Color3.fromRGB(24, 27, 34)
		row.BorderSizePixel = 0
		row.Parent = page
		Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)
		local lab = Instance.new("TextLabel")
		lab.Size = UDim2.new(0.35, 0, 1, 0)
		lab.Position = UDim2.new(0, 10, 0, 0)
		lab.BackgroundTransparency = 1
		lab.Font = Enum.Font.Gotham
		lab.TextSize = 12
		lab.TextColor3 = Color3.fromRGB(140, 148, 165)
		lab.TextXAlignment = Enum.TextXAlignment.Left
		lab.Text = "name"
		lab.Parent = row
		local box = Instance.new("TextBox")
		box.Size = UDim2.new(0.55, -8, 0, 26)
		box.Position = UDim2.new(0.38, 0, 0.5, -13)
		box.BackgroundColor3 = Color3.fromRGB(32, 36, 46)
		box.Text = Settings.NameSpoofText or "nnnn"
		box.PlaceholderText = "spoof name"
		box.Font = Enum.Font.Gotham
		box.TextSize = 12
		box.TextColor3 = Color3.fromRGB(235, 238, 245)
		box.ClearTextOnFocus = false
		box.Parent = row
		Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
		box.FocusLost:Connect(function()
			Settings.NameSpoofText = box.Text
			NameSpoof.name = box.Text
			if NameSpoof.state then NameSpoof.state.Name = box.Text end
			if NameSpoof.enabled then applyNameSpoof() end
		end)
	end

	setParent(section(pages.Misc, "spoof device"))
	addToggle(pages.Misc, "enabled", Settings.DeviceSpoofEnabled, function(v)
		Settings.DeviceSpoofEnabled = v
		if v then
			setDeviceMode(Settings.DeviceSpoofMode or "MouseKeyboard")
		else
			setDeviceMode(getRealDevice())
		end
		showNotification("spoof device: " .. tostring(v))
	end)
	addDropdown(pages.Misc, "Mode", {"Touch", "Gamepad", "MouseKeyboard", "VR"}, 3, function(v)
		Settings.DeviceSpoofMode = v
		if Settings.DeviceSpoofEnabled then
			setDeviceMode(v)
		end
	end)

	setParent(section(pages.Misc_R, "EMOTE HOP"))
	addToggle(pages.Misc_R, "Emote Hop", Settings.EmoteHop, function(v)
		Settings.EmoteHop = v
		if v then
			play_emote(LocalPlayer.Character)
		else
			stop_emote()
		end
		showNotification("Emote Hop: " .. tostring(v))
	end)
	addSlider(pages.Misc_R, "Emote Speed", 1, 40, Settings.EmoteSpeed or 40, function(v)
		Settings.EmoteSpeed = v
		if emoteHopTrack then
			pcall(function() emoteHopTrack:AdjustSpeed(v) end)
		end
	end)

	setParent(section(pages.Misc_R, "Hit Sound"))
	addToggle(pages.Misc_R, "Hit Sound", Settings.HitSoundEnabled, function(v) Settings.HitSoundEnabled = v end)
	local hitNames = {}
	pcall(function() for _, s in ipairs(HIT_SOUNDS) do table.insert(hitNames, s.Name) end end)
	if #hitNames == 0 then hitNames = {"Skeet"} end
	addDropdown(pages.Misc_R, "Sound", hitNames, 1, function(v) Settings.HitSoundName = v end)
	addSlider(pages.Misc_R, "Volume", 0.5, 5, Settings.HitSoundVolume or 2, function(v) Settings.HitSoundVolume = v end)
	addButton(pages.Misc_R, "Preview Hit Sound", function()
		local was = Settings.HitSoundEnabled
		Settings.HitSoundEnabled = true
		pcall(playHitSound)
		Settings.HitSoundEnabled = was
	end)

	-- ===== CFG / Settings (clean: color + config) =====
	do
		local pageL = pages.Cfg
		local pageR = pages.Cfg_R or pages.Cfg
		local ACC = ACCENT

		local function mkCard(parent, titleText)
			local card = Instance.new("Frame")
			card.Size = UDim2.new(1, 0, 0, 40)
			card.AutomaticSize = Enum.AutomaticSize.Y
			card.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
			card.BorderSizePixel = 0
			card.Parent = parent
			Instance.new("UICorner", card).CornerRadius = UDim.new(0, 6)
			local st = Instance.new("UIStroke")
			st.Color = Color3.fromRGB(40, 44, 56)
			st.Thickness = 1
			st.Parent = card
			local head = Instance.new("Frame")
			head.Size = UDim2.new(1, 0, 0, 28)
			head.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
			head.BorderSizePixel = 0
			head.Parent = card
			local bar = Instance.new("Frame")
			bar.Size = UDim2.new(0, 3, 1, -8)
			bar.Position = UDim2.new(0, 6, 0, 4)
			bar.BackgroundColor3 = ACC
			bar.BorderSizePixel = 0
			bar.Parent = head
			Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
			local tl = Instance.new("TextLabel")
			tl.Size = UDim2.new(1, -20, 1, 0)
			tl.Position = UDim2.new(0, 16, 0, 0)
			tl.BackgroundTransparency = 1
			tl.Font = Enum.Font.GothamBold
			tl.TextSize = 12
			tl.TextColor3 = Color3.fromRGB(235, 238, 245)
			tl.TextXAlignment = Enum.TextXAlignment.Left
			tl.Text = titleText
			tl.Parent = head
			local body = Instance.new("Frame")
			body.BackgroundTransparency = 1
			body.Size = UDim2.new(1, -16, 0, 0)
			body.Position = UDim2.new(0, 8, 0, 32)
			body.AutomaticSize = Enum.AutomaticSize.Y
			body.Parent = card
			local lay = Instance.new("UIListLayout")
			lay.Padding = UDim.new(0, 6)
			lay.SortOrder = Enum.SortOrder.LayoutOrder
			lay.Parent = body
			local pad = Instance.new("UIPadding")
			pad.PaddingBottom = UDim.new(0, 10)
			pad.Parent = body
			return body
		end

		local function mkBtn(parent, text, primary, cb)
			local b = Instance.new("TextButton")
			b.Size = UDim2.new(1, 0, 0, 32)
			b.BackgroundColor3 = primary and ACC or Color3.fromRGB(32, 34, 42)
			b.BorderSizePixel = 0
			b.Text = text
			b.Font = Enum.Font.GothamMedium
			b.TextSize = 12
			b.TextColor3 = primary and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(220, 224, 235)
			b.Parent = parent
			Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
			b.MouseButton1Click:Connect(function() pcall(cb) end)
			return b
		end

		local function mkInput(parent, placeholder, default)
			local box = Instance.new("TextBox")
			box.Size = UDim2.new(1, 0, 0, 32)
			box.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
			box.BorderSizePixel = 0
			box.Text = default or ""
			box.PlaceholderText = placeholder or ""
			box.PlaceholderColor3 = Color3.fromRGB(100, 108, 125)
			box.Font = Enum.Font.Gotham
			box.TextSize = 12
			box.TextColor3 = Color3.fromRGB(235, 238, 245)
			box.ClearTextOnFocus = false
			box.TextXAlignment = Enum.TextXAlignment.Left
			box.Parent = parent
			Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
			local pad = Instance.new("UIPadding")
			pad.PaddingLeft = UDim.new(0, 10)
			pad.Parent = box
			return box
		end

		local function mkLabel(parent, text)
			local l = Instance.new("TextLabel")
			l.Size = UDim2.new(1, 0, 0, 16)
			l.BackgroundTransparency = 1
			l.Font = Enum.Font.Gotham
			l.TextSize = 11
			l.TextColor3 = Color3.fromRGB(130, 138, 155)
			l.TextXAlignment = Enum.TextXAlignment.Left
			l.Text = text
			l.Parent = parent
			return l
		end

		-- LEFT: UI Color
		local colorBody = mkCard(pageL, "UI Color")
		mkLabel(colorBody, "Accent color")
		local swatchRow = Instance.new("Frame")
		swatchRow.Size = UDim2.new(1, 0, 0, 30)
		swatchRow.BackgroundTransparency = 1
		swatchRow.Parent = colorBody
		local swLay = Instance.new("UIListLayout")
		swLay.FillDirection = Enum.FillDirection.Horizontal
		swLay.Padding = UDim.new(0, 6)
		swLay.Parent = swatchRow
		local accents = {
			{Color3.fromRGB(70, 140, 220), "Blue"},
			{Color3.fromRGB(220, 40, 40), "Red"},
			{Color3.fromRGB(160, 80, 220), "Purple"},
			{Color3.fromRGB(100, 255, 220), "Mint"},
			{Color3.fromRGB(255, 180, 60), "Gold"},
			{Color3.fromRGB(255, 255, 255), "White"},
			{Color3.fromRGB(255, 100, 180), "Pink"},
			{Color3.fromRGB(80, 220, 120), "Green"},
		}
		local preview = Instance.new("Frame")
		for _, info in ipairs(accents) do
			local col, name = info[1], info[2]
			local sw = Instance.new("TextButton")
			sw.Size = UDim2.new(0, 28, 0, 28)
			sw.BackgroundColor3 = col
			sw.Text = ""
			sw.BorderSizePixel = 0
			sw.Parent = swatchRow
			Instance.new("UICorner", sw).CornerRadius = UDim.new(0, 6)
			sw.MouseButton1Click:Connect(function()
				preview.BackgroundColor3 = col
				applyUIAccent(col)
				showNotification("color: " .. name)
			end)
		end
		mkLabel(colorBody, "Current")
		preview.Size = UDim2.new(1, 0, 0, 8)
		preview.BackgroundColor3 = Settings.ThemeColor or ACC
		preview.BorderSizePixel = 0
		preview.Parent = colorBody
		Instance.new("UICorner", preview).CornerRadius = UDim.new(1, 0)

		-- RGB inputs
		mkLabel(colorBody, "Custom RGB (0-255)")
		local rgbRow = Instance.new("Frame")
		rgbRow.Size = UDim2.new(1, 0, 0, 32)
		rgbRow.BackgroundTransparency = 1
		rgbRow.Parent = colorBody
		local rgbLay = Instance.new("UIListLayout")
		rgbLay.FillDirection = Enum.FillDirection.Horizontal
		rgbLay.Padding = UDim.new(0, 6)
		rgbLay.Parent = rgbRow
		local function mkRgbBox(def)
			local box = Instance.new("TextBox")
			box.Size = UDim2.new(0.33, -4, 1, 0)
			box.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
			box.BorderSizePixel = 0
			box.Text = tostring(def)
			box.Font = Enum.Font.Gotham
			box.TextSize = 12
			box.TextColor3 = Color3.fromRGB(235, 238, 245)
			box.ClearTextOnFocus = false
			box.Parent = rgbRow
			Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
			return box
		end
		local cur = Settings.ThemeColor or ACC
		local rBox = mkRgbBox(math.floor(cur.R * 255 + 0.5))
		local gBox = mkRgbBox(math.floor(cur.G * 255 + 0.5))
		local bBox = mkRgbBox(math.floor(cur.B * 255 + 0.5))
		mkBtn(colorBody, "Apply RGB", true, function()
			local r = math.clamp(tonumber(rBox.Text) or 70, 0, 255)
			local g = math.clamp(tonumber(gBox.Text) or 140, 0, 255)
			local b = math.clamp(tonumber(bBox.Text) or 220, 0, 255)
			local col = Color3.fromRGB(r, g, b)
			preview.BackgroundColor3 = col
			applyUIAccent(col)
			showNotification(string.format("RGB %d,%d,%d", r, g, b))
		end)

		-- RIGHT: Config
		local cfgBody = mkCard(pageR, "Config")
		mkLabel(cfgBody, "Config name")
		local cfgBox = mkInput(cfgBody, "e.g. default", (CfgSys and CfgSys.currentName) or "default")
		env.__NnnnCfgNameBox = cfgBox

		mkLabel(cfgBody, "Saved configs (click to select)")
		local listFrame = Instance.new("ScrollingFrame")
		listFrame.Size = UDim2.new(1, 0, 0, 110)
		listFrame.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
		listFrame.BorderSizePixel = 0
		listFrame.ScrollBarThickness = 4
		listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
		listFrame.Parent = cfgBody
		Instance.new("UICorner", listFrame).CornerRadius = UDim.new(0, 6)
		local listLay = Instance.new("UIListLayout")
		listLay.Padding = UDim.new(0, 3)
		listLay.SortOrder = Enum.SortOrder.LayoutOrder
		listLay.Parent = listFrame
		listLay:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
			listFrame.CanvasSize = UDim2.new(0, 0, 0, listLay.AbsoluteContentSize.Y + 6)
		end)
		local listPad = Instance.new("UIPadding")
		listPad.PaddingTop = UDim.new(0, 4)
		listPad.PaddingBottom = UDim.new(0, 4)
		listPad.PaddingLeft = UDim.new(0, 4)
		listPad.PaddingRight = UDim.new(0, 4)
		listPad.Parent = listFrame

		local function refreshConfigList()
			for _, ch in ipairs(listFrame:GetChildren()) do
				if ch:IsA("TextButton") then ch:Destroy() end
			end
			local list = {}
			pcall(function()
				list = listJsonIn((CfgSys and CfgSys.cfgFolder) or "nnnnLinoria/configs")
			end)
			if #list == 0 then
				local empty = Instance.new("TextLabel")
				empty.Size = UDim2.new(1, 0, 0, 22)
				empty.BackgroundTransparency = 1
				empty.Text = "(no configs)"
				empty.Font = Enum.Font.Gotham
				empty.TextSize = 11
				empty.TextColor3 = Color3.fromRGB(120, 128, 140)
				empty.TextXAlignment = Enum.TextXAlignment.Left
				empty.Parent = listFrame
				return
			end
			for i, name in ipairs(list) do
				local btn = Instance.new("TextButton")
				btn.Size = UDim2.new(1, 0, 0, 24)
				btn.BackgroundColor3 = Color3.fromRGB(34, 36, 44)
				btn.BorderSizePixel = 0
				btn.Text = "  " .. name
				btn.Font = Enum.Font.Gotham
				btn.TextSize = 12
				btn.TextColor3 = Color3.fromRGB(230, 234, 245)
				btn.TextXAlignment = Enum.TextXAlignment.Left
				btn.LayoutOrder = i
				btn.Parent = listFrame
				Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
				btn.MouseButton1Click:Connect(function()
					cfgBox.Text = name
					showNotification("selected: " .. name)
				end)
			end
		end
		refreshConfigList()

		local status = Instance.new("TextLabel")
		status.Size = UDim2.new(1, 0, 0, 18)
		status.BackgroundTransparency = 1
		status.Font = Enum.Font.Gotham
		status.TextSize = 11
		status.TextColor3 = Color3.fromRGB(120, 180, 255)
		status.TextXAlignment = Enum.TextXAlignment.Left
		status.Text = "autoload: " .. tostring((CfgSys and CfgSys.autoload) or "none")
		status.Parent = cfgBody

		mkBtn(cfgBody, "Save Config", true, function()
			local n = cfgBox.Text ~= "" and cfgBox.Text or "default"
			if saveConfig then pcall(saveConfig, n) end
			refreshConfigList()
			showNotification("saved: " .. n)
		end)
		mkBtn(cfgBody, "Create Config", false, function()
			local n = cfgBox.Text ~= "" and cfgBox.Text or ("cfg_" .. tostring(os.time() % 100000))
			cfgBox.Text = n
			if saveConfig then pcall(saveConfig, n) end
			refreshConfigList()
			showNotification("created: " .. n)
		end)
		mkBtn(cfgBody, "Load Config", false, function()
			local n = cfgBox.Text ~= "" and cfgBox.Text or "default"
			if loadConfig then pcall(loadConfig, n) end
			showNotification("loaded: " .. n)
		end)
		mkBtn(cfgBody, "Refresh List", false, function()
			refreshConfigList()
			showNotification("list refreshed")
		end)
		mkBtn(cfgBody, "Delete Config", false, function()
			local n = cfgBox.Text ~= "" and cfgBox.Text or "default"
			pcall(function()
				local folder = (CfgSys and CfgSys.cfgFolder) or "nnnnLinoria/configs"
				local path = folder .. "/" .. n .. ".json"
				if isfile and isfile(path) and delfile then
					delfile(path)
					showNotification("deleted: " .. n)
				else
					showNotification("delete failed / no file")
				end
			end)
			refreshConfigList()
		end)
		mkBtn(cfgBody, "Set Autoload", false, function()
			local n = cfgBox.Text ~= "" and cfgBox.Text or "default"
			if setAutoload then pcall(setAutoload, n) end
			status.Text = "autoload: " .. n
			showNotification("autoload: " .. n)
		end)
		mkBtn(cfgBody, "Unload Script", false, function()
			if env.__NnnnUnload then pcall(env.__NnnnUnload) end
		end)
	end

	-- drag
	local dragging, dragStart, startPos
	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = main.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local d = input.Position - dragStart
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end)

	local menuOpen = true
	closeBtn.MouseButton1Click:Connect(function()
		menuOpen = false
		main.Visible = false
	end)
	UserInputService.InputBegan:Connect(function(input, gp)
		if gp then return end
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == (Settings.MenuKey or Enum.KeyCode.RightShift) then
			menuOpen = not menuOpen
			main.Visible = menuOpen
		end
	end)

	local tgui = Instance.new("ScreenGui")
	tgui.Name = "NnnnToggleGui"
	tgui.ResetOnSpawn = false
	tgui.DisplayOrder = 1000
	tgui.IgnoreGuiInset = true
	pcall(function() if gethui then tgui.Parent = gethui() end end)
	if not tgui.Parent then pcall(function() tgui.Parent = CoreGui end) end
	if not tgui.Parent then tgui.Parent = LocalPlayer:WaitForChild("PlayerGui") end
	local tb = Instance.new("TextButton")
	tb.Size = UDim2.new(0, 100, 0, 32)
	tb.Position = UDim2.new(0, 12, 0, 12)
	tb.BackgroundColor3 = PANEL
	tb.Text = "RH"
	tb.Font = Enum.Font.GothamBold
	tb.TextSize = 14
	tb.TextColor3 = TEXT
	tb.Parent = tgui
	Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 6)
	local ts = Instance.new("UIStroke")
	ts.Color = ACCENT
	ts.Thickness = 1
	ts.Parent = tb
	tb.MouseButton1Click:Connect(function()
		menuOpen = not menuOpen
		main.Visible = menuOpen
	end)

	env.__NnnnLibrary = { Toggle = function() menuOpen = not menuOpen main.Visible = menuOpen end }
	menuOpen = true
	main.Visible = true
	gui.Enabled = true
	-- re-parent in case executor blocked earlier
	pcall(function()
		if not gui.Parent and gethui then gui.Parent = gethui() end
	end)
	if not gui.Parent then pcall(function() gui.Parent = CoreGui end) end
	if not gui.Parent then
		pcall(function() gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
	end
	showNotification("Ronaldo Hook ready")
	print("[Ronaldo Hook] UI ready visible=", main.Visible, "parent=", gui.Parent and gui.Parent.Name)
	warn("[Ronaldo Hook] MENU OPEN - RightShift to toggle")
end

print("[Ronaldo Hook] script loaded, starting UI...")

local function forceShowUI()
	-- 1) try full UI
	local ok, err = pcall(buildCustomUI)
	if not ok then
		warn("[Ronaldo Hook] buildCustomUI error:", err)
	else
		print("[Ronaldo Hook] UI build ok")
	end

	-- 2) force any existing hub visible
	pcall(function()
		local parents = {}
		pcall(function() if gethui then table.insert(parents, gethui()) end end)
		table.insert(parents, CoreGui)
		pcall(function()
			local pg = LocalPlayer:FindFirstChild("PlayerGui")
			if pg then table.insert(parents, pg) end
		end)
		for _, parent in ipairs(parents) do
			local g = parent:FindFirstChild("NnnnHubUI")
			if g then
				g.Enabled = true
				local main = g:FindFirstChild("Main")
				if main then main.Visible = true end
			end
			local tgui = parent:FindFirstChild("NnnnToggleGui")
			if tgui then tgui.Enabled = true end
		end
	end)

	-- 3) if still no UI, emergency window
	local has = false
	pcall(function()
		if gethui and gethui():FindFirstChild("NnnnHubUI") then has = true end
	end)
	pcall(function()
		if CoreGui:FindFirstChild("NnnnHubUI") then has = true end
	end)
	pcall(function()
		local pg = LocalPlayer:FindFirstChild("PlayerGui")
		if pg and pg:FindFirstChild("NnnnHubUI") then has = true end
	end)

	if not has then
		warn("[Ronaldo Hook] emergency UI")
		local g = Instance.new("ScreenGui")
		g.Name = "NnnnHubUI"
		g.ResetOnSpawn = false
		g.DisplayOrder = 9999
		g.IgnoreGuiInset = true
		g.Enabled = true
		pcall(function() if gethui then g.Parent = gethui() end end)
		if not g.Parent then pcall(function() g.Parent = CoreGui end) end
		if not g.Parent then g.Parent = LocalPlayer:WaitForChild("PlayerGui") end

		local main = Instance.new("Frame")
		main.Name = "Main"
		main.Size = UDim2.new(0, 420, 0, 280)
		main.Position = UDim2.new(0.5, -210, 0.5, -140)
		main.BackgroundColor3 = Color3.fromRGB(12, 12, 14)
		main.BorderSizePixel = 0
		main.Visible = true
		main.Parent = g
		Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)
		local st = Instance.new("UIStroke")
		st.Color = Color3.fromRGB(212, 175, 55)
		st.Thickness = 2
		st.Parent = main

		local title = Instance.new("TextLabel")
		title.Size = UDim2.new(1, 0, 0, 40)
		title.BackgroundColor3 = Color3.fromRGB(20, 20, 24)
		title.BorderSizePixel = 0
		title.Font = Enum.Font.GothamBold
		title.TextSize = 18
		title.TextColor3 = Color3.fromRGB(212, 175, 55)
		title.Text = "RONALDO HOOK"
		title.Parent = main

		local msg = Instance.new("TextLabel")
		msg.Size = UDim2.new(1, -24, 1, -60)
		msg.Position = UDim2.new(0, 12, 0, 48)
		msg.BackgroundTransparency = 1
		msg.Font = Enum.Font.Gotham
		msg.TextSize = 14
		msg.TextColor3 = Color3.fromRGB(230, 230, 230)
		msg.TextWrapped = true
		msg.TextYAlignment = Enum.TextYAlignment.Top
		msg.Text = "UI ON\n\nFull menu failed to build.\nError: " .. tostring(err or "unknown") .. "\n\nRightShift / Insert still bound if full UI loads later."
		msg.Parent = main
	end

	-- 4) keep forcing visible for a bit
	task.spawn(function()
		for _ = 1, 20 do
			pcall(function()
				local g = (gethui and gethui():FindFirstChild("NnnnHubUI"))
					or CoreGui:FindFirstChild("NnnnHubUI")
					or (LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("NnnnHubUI"))
				if g then
					g.Enabled = true
					local m = g:FindFirstChild("Main")
					if m then m.Visible = true end
				end
			end)
			task.wait(0.25)
		end
	end)

	warn("[Ronaldo Hook] UI FORCE ON")
end

-- 바로 켜기
task.spawn(forceShowUI)

-- 다른 스크립트와 공존: nnnn만 정리
env.__NnnnUnload = function()
	pcall(function()
		for _, c in ipairs(nnnnConns) do
			pcall(function()
				if typeof(c) == "RBXScriptConnection" then c:Disconnect() end
			end)
		end
		table.clear(nnnnConns)
	end)
	pcall(function()
		local only = {
			"NnnnHubUI", "NnnnToggleGui", "CerberusNotifyGui",
			"RageHUD", "ActiveFeaturesHUD", "NnnnSkinPicker",
			"NnnnTargetHUD", "NnnnHitMarker", "RonaldoHookBoot",
		}
		local parents = { CoreGui }
		local pg = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
		if pg then table.insert(parents, pg) end
		pcall(function() if gethui then table.insert(parents, gethui()) end end)
		for _, parent in ipairs(parents) do
			for _, name in ipairs(only) do
				local o = parent:FindFirstChild(name)
				if o then o:Destroy() end
			end
		end
	end)
	print("[nnnn] unloaded (other scripts untouched)")
end

print("[Ronaldo Hook] ready - UI turning ON")
print("[Ronaldo Hook] menu: RightShift / Insert / left RH button")
