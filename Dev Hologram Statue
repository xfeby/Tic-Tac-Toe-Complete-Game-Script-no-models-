local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local hologram = script.Parent
local base = hologram:WaitForChild("Base")
local spawnPoint = hologram:WaitForChild("SpawnPoint")

local transparencies = {
	body = 0.4,
	images = 0.35,
	textStroke = 0.3,
	highlight = 0.25,
	highlightFill = 0.5,	
}

local devs = {
	{userId = 47535287, name = "Feby // @xf_by", role = "Owner"},
	{userId = 72364014, name = "GuyAwesome // @Guy_Awe", role = "Builder"},
	{userId = 43194841, name = "Relic // @RelicCadcaa", role = "Scripter"},
	{userId = 55393996, name = "Drago // @yoyodrago", role = "Scripter"},
	{userId = 284597834, name = "Hitman // @MFHitman", role = "contributor"},
	
}

local SWITCH_TIME = 20
local DESINTEGRATE_DELAY = 1
local FLOAT_SPEED = 1.5
local FLOAT_HEIGHT = 0.4
local ROTATION_SPEED = math.rad(20)

local TweenService = game:GetService("TweenService")
local TWEEN_INFO = TweenInfo.new(DESINTEGRATE_DELAY, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
local TWEEN_INFO2 = TweenInfo.new(DESINTEGRATE_DELAY/2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)

local function TweenHolo(Item, FillColor : Color3, OutlineColor : Color3, FillTransparency : number, OutlineTransparency : number)
	local Goal = {}
	Goal.FillTransparency = FillTransparency
	Goal.OutlineTransparency = OutlineTransparency
	Goal.FillColor = FillColor
	Goal.OutlineColor = OutlineColor

	local Tween = TweenService:Create(Item, TWEEN_INFO, Goal)
	Tween:Play()
end

local function TweenPendingScreen(Transparency : number)
	task.spawn(function()
		local Goal = {}
		Goal.ImageTransparency = Transparency
		local Tween = TweenService:Create(script.Parent.SpawnPoint.PendScreen.PlayerImage, TWEEN_INFO2, Goal)
		Tween:Play()
	end)
end

local R6_Y_OFFSET = 1.2
local R15_Y_OFFSET = 0

local currentRig
local currentYOffset = 0
local currentIndex = 1
local spawnTimestamp = tick()
local hologramHighlight

local beamTargetPart
local mainBeam
local outerBeam
local basePointLight
local baseSpotLight

local function getOrCreateAttachment(parent, name, position)
	local attachment = parent:FindFirstChild(name)
	if not attachment then
		attachment = Instance.new("Attachment")
		attachment.Name = name
		attachment.Parent = parent
	end

	if position then
		attachment.Position = position
	end

	return attachment
end

local function getOrCreatePart(parent, name)
	local part = parent:FindFirstChild(name)
	if not part then
		part = Instance.new("Part")
		part.Name = name
		part.Parent = parent
	end
	return part
end

-- no need for these, I have my own effects

--local function setupProjectorEffects()
--	local baseAttach = getOrCreateAttachment(base, "BeamStart", Vector3.new(0, base.Size.Y * 0.5, 0))

--	beamTargetPart = getOrCreatePart(hologram, "BeamTarget")
--	beamTargetPart.Anchored = true
--	beamTargetPart.CanCollide = false
--	beamTargetPart.CanTouch = false
--	beamTargetPart.CanQuery = false
--	beamTargetPart.Transparency = 1
--	beamTargetPart.Size = Vector3.new(1, 1, 1)
--	beamTargetPart.CFrame = spawnPoint.CFrame * CFrame.new(0, 1.4, 0)

--	local targetAttach = getOrCreateAttachment(beamTargetPart, "BeamEnd")

--	mainBeam = base:FindFirstChild("MainBeam")
--	if not mainBeam then
--		mainBeam = Instance.new("Beam")
--		mainBeam.Name = "MainBeam"
--		mainBeam.Parent = base
--	end
--	mainBeam.Attachment0 = baseAttach
--	mainBeam.Attachment1 = targetAttach
--	mainBeam.FaceCamera = true
--	mainBeam.LightEmission = 0.65
--	mainBeam.LightInfluence = 0
--	mainBeam.Brightness = 0.65
--	mainBeam.Width0 = 0.28
--	mainBeam.Width1 = 1.15
--	mainBeam.Segments = 8
--	mainBeam.Color = ColorSequence.new(Color3.fromRGB(120, 225, 255))
--	mainBeam.Transparency = NumberSequence.new{
--		NumberSequenceKeypoint.new(0, 0.8),
--		NumberSequenceKeypoint.new(0.25, 0.72),
--		NumberSequenceKeypoint.new(1, 0.93),
--	}

--	outerBeam = base:FindFirstChild("OuterBeam")
--	if not outerBeam then
--		outerBeam = Instance.new("Beam")
--		outerBeam.Name = "OuterBeam"
--		outerBeam.Parent = base
--	end
--	outerBeam.Attachment0 = baseAttach
--	outerBeam.Attachment1 = targetAttach
--	outerBeam.FaceCamera = true
--	outerBeam.LightEmission = 0.45
--	outerBeam.LightInfluence = 0
--	outerBeam.Brightness = 0.22
--	outerBeam.Width0 = 0.65
--	outerBeam.Width1 = 1.8
--	outerBeam.Segments = 8
--	outerBeam.Color = ColorSequence.new(Color3.fromRGB(100, 205, 245))
--	outerBeam.Transparency = NumberSequence.new{
--		NumberSequenceKeypoint.new(0, 0.92),
--		NumberSequenceKeypoint.new(0.3, 0.88),
--		NumberSequenceKeypoint.new(1, 1),
--	}

--	basePointLight = base:FindFirstChild("HologramPointLight")
--	if not basePointLight then
--		basePointLight = Instance.new("PointLight")
--		basePointLight.Name = "HologramPointLight"
--		basePointLight.Parent = base
--	end
--	basePointLight.Color = Color3.fromRGB(110, 220, 255)
--	basePointLight.Brightness = 0.2
--	basePointLight.Range = 8
--	basePointLight.Shadows = false

--	baseSpotLight = base:FindFirstChild("HologramSpotLight")
--	if not baseSpotLight then
--		baseSpotLight = Instance.new("SpotLight")
--		baseSpotLight.Name = "HologramSpotLight"
--		baseSpotLight.Parent = base
--	end
--	baseSpotLight.Color = Color3.fromRGB(110, 220, 255)
--	baseSpotLight.Brightness = 0.35
--	baseSpotLight.Range = 8
--	baseSpotLight.Angle = 45
--	baseSpotLight.Face = Enum.NormalId.Top
--	baseSpotLight.Shadows = false

--	base.Material = Enum.Material.Metal
--end

local function getRigYOffset(rig)
	local humanoid = rig:FindFirstChildOfClass("Humanoid")
	if not humanoid then
		return 0
	end

	if humanoid.RigType == Enum.HumanoidRigType.R6 then
		return R6_Y_OFFSET
	end

	return R15_Y_OFFSET
end

local function removeOldBillboard(rig)
	local old = rig:FindFirstChild("DevNameBillboard", true)
	if old then
		old:Destroy()
	end
end

local function attachNameBillboard(rig, dev)
	removeOldBillboard(rig)

	local hrp = rig:FindFirstChild("HumanoidRootPart")
	if not hrp then
		return
	end

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "DevNameBillboard"
	billboard.Adornee = hrp
	billboard.Parent = rig
	billboard.AlwaysOnTop = true
	billboard.LightInfluence = 0
	billboard.MaxDistance = 35
	billboard.Size = UDim2.new(8, 0, 3, 0)
	billboard.StudsOffset = Vector3.new(0, -.5, 0)

	local holder = Instance.new("Frame")
	holder.BackgroundTransparency = 1
	holder.Size = UDim2.fromScale(1, 1)
	holder.Parent = billboard

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Name = "NameLabel"
	nameLabel.Parent = holder
	nameLabel.BackgroundTransparency = 1
	nameLabel.Size = UDim2.new(1, 0, .2, 0)
	nameLabel.Position = UDim2.new(0, 0, 0, 0)
	nameLabel.Font = Enum.Font.GothamBold
	nameLabel.Text = dev.name
	nameLabel.TextWrapped = false
	nameLabel.TextScaled = true
	nameLabel.TextColor3 = Color3.fromRGB(185, 245, 255)
	nameLabel.TextStrokeTransparency = transparencies.textStroke
	nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 80, 120)

	local roleLabel = Instance.new("TextLabel")
	roleLabel.Name = "RoleLabel"
	roleLabel.Parent = holder
	roleLabel.BackgroundTransparency = 1
	roleLabel.Size = UDim2.new(1, 0, 0.15, 0)
	roleLabel.Position = UDim2.new(0, 0, 0.2, 0)
	roleLabel.Font = Enum.Font.GothamMedium
	roleLabel.Text = dev.role
	roleLabel.TextWrapped = false
	roleLabel.TextScaled = true
	roleLabel.TextColor3 = Color3.fromRGB(120, 220, 255)
	roleLabel.TextStrokeTransparency = transparencies.textStroke
	roleLabel.TextStrokeColor3 = Color3.fromRGB(0, 70, 110)
end

local function styleRig(rig)
	for _, obj in ipairs(rig:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.CanCollide = false
			obj.CanTouch = false
			obj.CanQuery = false
			obj.CastShadow = false
			obj.Massless = true

			if obj.Name == "HumanoidRootPart" then
				obj.Anchored = true
				obj.Transparency = 1
			else
				obj.Anchored = false
				if obj.Transparency ~= 1 then
					obj.Transparency = transparencies.body
				end
				--obj.Color = Color3.fromRGB(120, 220, 255)
				--obj.Material = Enum.Material.ForceField
			end
		elseif obj:IsA("Decal") then
			obj.Transparency = 1
		elseif obj:IsA("Accessory") then
			local handle = obj:FindFirstChild("Handle")
			if handle and handle:IsA("BasePart") then
				handle.CanCollide = false
				handle.CanTouch = false
				handle.CanQuery = false
				handle.CastShadow = false
				handle.Massless = true
				if handle.Transparency ~= 1 then
					handle.Transparency = transparencies.body
				end
				--handle.Color = Color3.fromRGB(120, 220, 255)
				--handle.Material = Enum.Material.ForceField
			end
		end
	end
	
	local humanoid = rig:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
		humanoid.AutoRotate = false
		humanoid.PlatformStand = true
	end

	local animate = rig:FindFirstChild("Animate")
	if animate then
		animate:Destroy()
	end

	hologramHighlight = rig:FindFirstChild("HologramHighlight")
	if hologramHighlight then
		hologramHighlight:Destroy()
	end
	
	-- get rid of particles, noises, & scripts
	for _, obj : Instance in ipairs(rig:GetDescendants()) do
		if obj:IsA("ParticleEmitter") or obj:IsA("Sound") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("BaseScript") then
			if obj.Parent.Parent:IsA("Accessory") or obj.Parent:IsA("Accessory") then
				obj:Destroy()
			end
		end
	end

	hologramHighlight = Instance.new("Highlight")
	hologramHighlight.Name = "HologramHighlight"
	hologramHighlight.FillColor = Color3.fromRGB(255, 255, 255)
	hologramHighlight.OutlineColor = Color3.fromRGB(255, 255, 255)
	hologramHighlight.FillTransparency = 0
	hologramHighlight.OutlineTransparency = 0
	hologramHighlight.DepthMode = Enum.HighlightDepthMode.Occluded
	hologramHighlight.Parent = rig
	
	task.wait(DESINTEGRATE_DELAY)
	
	TweenPendingScreen(1)
	TweenHolo(hologramHighlight, Color3.fromRGB(85, 224, 255), Color3.fromRGB(24, 205, 255), transparencies.highlightFill, transparencies.highlight)
	
end

local function clearCurrentRig()
	if currentRig then
		local Holo = currentRig:FindFirstChild("HologramHighlight")
		if Holo then
			--task.spawn(function()
			TweenHolo(Holo, Color3.fromRGB(255, 255, 255), Color3.fromRGB(255, 255, 255), 0, 0)
			--end)
			task.wait(DESINTEGRATE_DELAY)
			TweenPendingScreen(0.1)
			task.wait(DESINTEGRATE_DELAY/4)
		end
		--for _, obj in ipairs(currentRig:GetDescendants()) do
		--	if obj:IsA("BasePart") then
		--		obj.Anchored = true
		--	end
		--end
		--for _, obj in ipairs(currentRig:GetDescendants()) do
		--	if obj:IsA("BasePart") then
		--		task.wait(DESINTEGRATE_DELAY)
		--		obj:Destroy()
		--	end
		--end
		currentRig:Destroy()
		currentRig = nil
		hologramHighlight = nil
	end
end

local function spawnAvatar(dev)
	clearCurrentRig()

	local ok, rig = pcall(function()
		return Players:CreateHumanoidModelFromUserId(dev.userId)
	end)

	if not ok or not rig then
		warn("Failed to create hologram rig for userId:", dev.userId)
		return
	end

	rig.Name = "CurrentRig"
	rig.Parent = hologram

	local hrp = rig:FindFirstChild("HumanoidRootPart")
	if not hrp then
		hrp = rig:WaitForChild("HumanoidRootPart", 5)
	end

	if not hrp then
		rig:Destroy()
		warn("No HumanoidRootPart for userId:", dev.userId)
		return
	end

	styleRig(rig)
	attachNameBillboard(rig, dev)

	currentYOffset = getRigYOffset(rig)
	rig:PivotTo(spawnPoint.CFrame * CFrame.new(0, currentYOffset, 0))
	rig:ScaleTo(2.5)

	currentRig = rig
	spawnTimestamp = tick()
end

local function cycleToNextDev()
	local dev = devs[currentIndex]
	spawnAvatar(dev)

	currentIndex += 1
	if currentIndex > #devs then
		currentIndex = 1
	end
end

--local function updateBeamTarget(baseY)
--	if beamTargetPart then
--		beamTargetPart.CFrame = CFrame.new(
--			spawnPoint.Position + Vector3.new(0, currentYOffset + 0.95 + baseY * 0.35, 0)
--		)
--	end
--end

--local function applyHologramFlicker(timeNow)
--	local flicker = 0.97 + math.noise(timeNow * 7, 0, 0) * 0.03
--	local shimmer = (math.sin(timeNow * 10) + 1) * 0.5

--	if basePointLight then
--		basePointLight.Brightness = 0.16 * flicker + shimmer * 0.02
--	end

--	if baseSpotLight then
--		baseSpotLight.Brightness = 0.28 * flicker + shimmer * 0.03
--	end

--	if mainBeam then
--		mainBeam.Brightness = 0.58 * flicker + shimmer * 0.03
--		mainBeam.Width0 = 0.27 + shimmer * 0.015
--		mainBeam.Width1 = 1.1 + shimmer * 0.03
--	end

--	if outerBeam then
--		outerBeam.Brightness = 0.18 * flicker
--		outerBeam.Width0 = 0.64 + shimmer * 0.02
--		outerBeam.Width1 = 1.75 + shimmer * 0.03
--	end

--	if hologramHighlight then
--		hologramHighlight.FillTransparency = 0.83 + (1 - flicker) * 0.04
--		hologramHighlight.OutlineTransparency = 0.17 + (1 - flicker) * 0.06
--	end

--	if currentRig then
--		local scanPulse = 0.42 + math.sin(timeNow * 8) * 0.02
--		local glitch = math.noise(timeNow * 15, 5, 0) * 0.012

--		for _, obj in ipairs(currentRig:GetDescendants()) do
--			if obj:IsA("BasePart") and obj.Name ~= "HumanoidRootPart" then
--				obj.Transparency = math.clamp(scanPulse + glitch, 0.36, 0.5)
--			elseif obj:IsA("Decal") then
--				obj.Transparency = math.clamp(0.42 + glitch, 0.34, 0.55)
--			end
--		end
--	end
--end

--setupProjectorEffects()
cycleToNextDev()

task.spawn(function()
	while true do
		task.wait(SWITCH_TIME)
		cycleToNextDev()
	end
end)

RunService.Heartbeat:Connect(function()
	local timeNow = tick()
	local elapsed = timeNow - spawnTimestamp
	local floatY = math.sin(elapsed * FLOAT_SPEED) * FLOAT_HEIGHT
	local rotationY = elapsed * ROTATION_SPEED

	--updateBeamTarget(floatY)
	--applyHologramFlicker(timeNow)

	if not currentRig then
		return
	end

	local basePosition = spawnPoint.Position + Vector3.new(0, currentYOffset + floatY, 0)
	local targetCF = CFrame.new(basePosition) * CFrame.Angles(0, rotationY, 0)

	currentRig:PivotTo(targetCF)
end)
