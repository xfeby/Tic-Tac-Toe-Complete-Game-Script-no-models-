local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")

local setModel = script.Parent
local boardModel = setModel:WaitForChild("TicTacToeGameBoard")
local gamePieces = boardModel:WaitForChild("GamePieces")
local cellsFolder = boardModel:WaitForChild("Cells")

local xTemplate = gamePieces:WaitForChild("X")
local oTemplate = gamePieces:WaitForChild("O")
local tieTemplate = gamePieces:WaitForChild("Tie")

local ALLOW_SOLO_TEST = false
local SHOW_DEBUG_CELLS = false
local RESET_DELAY = 3

local PIECE_HEIGHT_OFFSET = 0.004

local TIE_MODEL_HEIGHT_OFFSET = -0.05
local TIE_MODEL_ROTATION_DEGREES = 45
local X_ROTATION_DEGREES = 0

local WIN_LINE_HEIGHT_OFFSET = 0.055
local WIN_LINE_THICKNESS = 0.10
local WIN_LINE_EXTRA_LENGTH = 0.28
local WIN_LINE_TRANSPARENCY = 0.28
local WIN_LINE_MATERIAL = Enum.Material.SmoothPlastic

local SEAT_INDICATOR_HEIGHT = 1.9
local SEAT_INDICATOR_MAX_DISTANCE = 18

local TURN_INDICATOR_HEIGHT = 0.9
local TURN_INDICATOR_MAX_DISTANCE = 28

local RESULT_TEXT_MAX_DISTANCE = 35

local X_COLOR = Color3.fromRGB(255, 60, 60)
local O_COLOR = Color3.fromRGB(110, 210, 255)
local TIE_COLOR = Color3.fromRGB(220, 240, 255)

local WIN_SOUND_ID = "rbxassetid://73743814975286"
local TIE_SOUND_ID = "rbxassetid://139856019851463"
local X_PLACE_SOUND_ID = "rbxassetid://138316900117394"
local O_PLACE_SOUND_ID = "rbxassetid://136839407049381"

local WIN_SOUND_VOLUME = 3
local TIE_SOUND_VOLUME = 2
local PLACE_SOUND_VOLUME = 4

local CONFETTI_BURST_TIME = 0.4
local DEFAULT_CONFETTI_EMIT_COUNT = 45

local function setHiddenRecursive(root)
	if not root then
		return
	end

	if root:IsA("BasePart") then
		root.Transparency = 1
		root.CanCollide = false
		root.CanTouch = false
		root.CanQuery = false
	end

	for _, d in ipairs(root:GetDescendants()) do
		if d:IsA("BasePart") then
			d.Transparency = 1
			d.CanCollide = false
			d.CanTouch = false
			d.CanQuery = false
		end
	end
end

local function setVisibleRecursive(root)
	if not root then
		return
	end

	if root:IsA("BasePart") then
		root.Transparency = 0
		root.CanCollide = false
		root.CanTouch = false
		root.CanQuery = false
	end

	for _, d in ipairs(root:GetDescendants()) do
		if d:IsA("BasePart") then
			d.Transparency = 0
			d.CanCollide = false
			d.CanTouch = false
			d.CanQuery = false
		end
	end
end

setHiddenRecursive(xTemplate)
setHiddenRecursive(oTemplate)
setHiddenRecursive(tieTemplate)

local activePiecesFolder = boardModel:FindFirstChild("ActivePieces")
if not activePiecesFolder then
	activePiecesFolder = Instance.new("Folder")
	activePiecesFolder.Name = "ActivePieces"
	activePiecesFolder.Parent = boardModel
end

local effectsFolder = boardModel:FindFirstChild("Effects")
if not effectsFolder then
	effectsFolder = Instance.new("Folder")
	effectsFolder.Name = "Effects"
	effectsFolder.Parent = boardModel
end

local uiFolder = boardModel:FindFirstChild("SeatIndicators")
if not uiFolder then
	uiFolder = Instance.new("Folder")
	uiFolder.Name = "SeatIndicators"
	uiFolder.Parent = boardModel
end

local function getSeats()
	local seats = {}
	for _, obj in ipairs(setModel:GetDescendants()) do
		if obj:IsA("Seat") then
			table.insert(seats, obj)
		end
	end

	table.sort(seats, function(a, b)
		return a:GetFullName() < b:GetFullName()
	end)

	print("[TicTacToe] Seats found:", #seats)
	return seats[1], seats[2]
end

local seatA, seatB = getSeats()

if not seatA or not seatB then
	warn("[TicTacToe] Missing 2 real Seat objects inside the stools")
end

local function getPlayerFromSeat(seat)
	if not seat then
		return nil
	end

	local hum = seat.Occupant
	if not hum then
		return nil
	end

	local character = hum.Parent
	if not character then
		return nil
	end

	return Players:GetPlayerFromCharacter(character)
end

local cells = {}
for i = 1, 9 do
	local cell = cellsFolder:WaitForChild(tostring(i))
	cells[i] = cell

	cell.Anchored = true
	cell.CanCollide = false
	cell.CanTouch = false
	cell.CanQuery = true
	cell.Transparency = SHOW_DEBUG_CELLS and 0.5 or 1

	if SHOW_DEBUG_CELLS then
		cell.Color = Color3.fromRGB(255, 0, 0)
	end
end

local board = {"","","","","","","","",""}
local playerX = nil
local playerO = nil
local currentMark = "X"
local gameActive = false
local resetId = 0
local swapSeatRolesNextRound = false

local winLines = {
	{1,2,3},
	{4,5,6},
	{7,8,9},
	{1,4,7},
	{2,5,8},
	{3,6,9},
	{1,5,9},
	{3,5,7},
}

local seatIndicatorA = nil
local seatIndicatorB = nil
local turnIndicatorAnchor = nil
local turnIndicatorGui = nil
local confettiContainer = nil

local function clearEffects()
	for _, obj in ipairs(effectsFolder:GetChildren()) do
		obj:Destroy()
	end
end

local function clearBoard()
	for _, obj in ipairs(activePiecesFolder:GetChildren()) do
		obj:Destroy()
	end

	clearEffects()

	for i = 1, 9 do
		board[i] = ""
	end
end

local function getWinner()
	for _, line in ipairs(winLines) do
		local a, b, c = line[1], line[2], line[3]
		if board[a] ~= "" and board[a] == board[b] and board[b] == board[c] then
			return board[a], line
		end
	end
	return nil, nil
end

local function boardFull()
	for i = 1, 9 do
		if board[i] == "" then
			return false
		end
	end
	return true
end

local function getBoardTopPosition()
	local minX, maxX = math.huge, -math.huge
	local minY, maxY = math.huge, -math.huge
	local minZ, maxZ = math.huge, -math.huge

	for i = 1, 9 do
		local cell = cells[i]
		if cell then
			local pos = cell.Position
			local half = cell.Size * 0.5

			minX = math.min(minX, pos.X - half.X)
			maxX = math.max(maxX, pos.X + half.X)

			minY = math.min(minY, pos.Y - half.Y)
			maxY = math.max(maxY, pos.Y + half.Y)

			minZ = math.min(minZ, pos.Z - half.Z)
			maxZ = math.max(maxZ, pos.Z + half.Z)
		end
	end

	return Vector3.new(
		(minX + maxX) * 0.5,
		maxY,
		(minZ + maxZ) * 0.5
	)
end

local function placeObjectAtCell(obj, cell, extraRotation)
	local pos = cell.Position + Vector3.new(0, cell.Size.Y * 0.5 + PIECE_HEIGHT_OFFSET, 0)
	local cf = CFrame.new(pos) * (extraRotation or CFrame.new())

	if obj:IsA("Model") then
		obj:PivotTo(cf)
	elseif obj:IsA("BasePart") then
		obj.CFrame = cf
	end
end

local function placeObjectAtPivot(obj, cf)
	if obj:IsA("Model") then
		obj:PivotTo(cf)
	elseif obj:IsA("BasePart") then
		obj.CFrame = cf
	end
end

local function playEffectSound(soundId, parent, volume)
	if not soundId or soundId == "" or not parent then
		return
	end

	local sound = Instance.new("Sound")
	sound.SoundId = soundId
	sound.Volume = volume or 2.8
	sound.RollOffMaxDistance = 20
	sound.RollOffMinDistance = 4
	sound.RollOffMode = Enum.RollOffMode.InverseTapered
	sound.EmitterSize = 5
	sound.Parent = parent
	sound:Play()
	Debris:AddItem(sound, 5)
end

local function spawnMark(mark, index)
	local template = (mark == "X") and xTemplate or oTemplate
	local clone = template:Clone()
	clone.Name = mark .. "_" .. tostring(index)
	clone.Parent = activePiecesFolder

	setVisibleRecursive(clone)

	local extraRotation = nil
	if mark == "X" then
		extraRotation = CFrame.Angles(0, math.rad(X_ROTATION_DEGREES), 0)
	end

	placeObjectAtCell(clone, cells[index], extraRotation)

	local soundAnchor = Instance.new("Part")
	soundAnchor.Name = "PlaceSoundAnchor"
	soundAnchor.Anchored = true
	soundAnchor.CanCollide = false
	soundAnchor.CanTouch = false
	soundAnchor.CanQuery = false
	soundAnchor.Transparency = 1
	soundAnchor.Size = Vector3.new(1, 1, 1)
	soundAnchor.CFrame = CFrame.new(cells[index].Position)
	soundAnchor.Parent = effectsFolder

	if mark == "X" then
		playEffectSound(X_PLACE_SOUND_ID, soundAnchor, PLACE_SOUND_VOLUME)
	else
		playEffectSound(O_PLACE_SOUND_ID, soundAnchor, PLACE_SOUND_VOLUME)
	end

	Debris:AddItem(soundAnchor, 3)

	print("[TicTacToe] Spawned", mark, "on", index)
end

local function showFloatingText(text, color, sizeScale, lifetime)
	local life = lifetime or 1.5

	local anchor = Instance.new("Part")
	anchor.Name = "FloatingTextAnchor"
	anchor.Anchored = true
	anchor.CanCollide = false
	anchor.CanTouch = false
	anchor.CanQuery = false
	anchor.Transparency = 1
	anchor.Size = Vector3.new(1, 1, 1)
	anchor.CFrame = CFrame.new(getBoardTopPosition() + Vector3.new(0, 1.2, 0))
	anchor.Parent = effectsFolder

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ResultBillboard"
	billboard.AlwaysOnTop = false
	billboard.MaxDistance = RESULT_TEXT_MAX_DISTANCE
	billboard.Size = UDim2.fromScale(3.5, 1)
	billboard.LightInfluence = 0
	billboard.Parent = anchor

	local label = Instance.new("TextLabel")
	label.BackgroundTransparency = 1
	label.Size = UDim2.fromScale(1, 1)
	label.Text = text
	label.TextColor3 = color
	label.TextStrokeTransparency = 0.2
	label.TextStrokeColor3 = Color3.new(0, 0, 0)
	label.Font = Enum.Font.GothamBlack
	label.TextScaled = true
	label.Parent = billboard

	local scale = Instance.new("UIScale")
	scale.Scale = 0.7
	scale.Parent = label

	TweenService:Create(
		scale,
		TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
		{ Scale = sizeScale or 1 }
	):Play()

	task.delay(life * 0.6, function()
		if label and label.Parent then
			TweenService:Create(
				label,
				TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
				{
					TextTransparency = 1,
					TextStrokeTransparency = 1,
				}
			):Play()
		end
	end)

	Debris:AddItem(anchor, life)
end

local function resolveConfettiContainer()
	if confettiContainer and confettiContainer.Parent then
		return confettiContainer
	end

	local gameTable = setModel:FindFirstChild("GameTable")
	if gameTable then
		local base = gameTable:FindFirstChild("Base")
		if base then
			local basePart = base:FindFirstChild("BasePart")
			if basePart then
				local exact = basePart:FindFirstChild("ConfettiVFX")
				if exact then
					confettiContainer = exact
					print("[TicTacToe] ConfettiVFX found at exact path:", confettiContainer:GetFullName())
					return confettiContainer
				end
			end
		end
	end

	for _, obj in ipairs(setModel:GetDescendants()) do
		if obj.Name == "ConfettiVFX" then
			confettiContainer = obj
			print("[TicTacToe] ConfettiVFX found by descendant search:", confettiContainer:GetFullName())
			return confettiContainer
		end
	end

	warn("[TicTacToe] ConfettiVFX not found anywhere under " .. setModel:GetFullName())
	return nil
end

local function stopAllConfettiEmitters()
	local container = resolveConfettiContainer()
	if not container then
		return
	end

	for _, obj in ipairs(container:GetDescendants()) do
		if obj:IsA("ParticleEmitter") then
			obj.Enabled = false
		end
	end
end

local function burstConfettiEmitters()
	local container = resolveConfettiContainer()
	if not container then
		return
	end

	local emitters = {}

	for _, obj in ipairs(container:GetDescendants()) do
		if obj:IsA("ParticleEmitter") then
			table.insert(emitters, obj)
		end
	end

	if #emitters == 0 then
		warn("[TicTacToe] ConfettiVFX found, but no ParticleEmitters are inside it")
		return
	end

	print("[TicTacToe] Firing confetti emitters:", #emitters)

	for _, emitter in ipairs(emitters) do
		local emitCount = emitter:GetAttribute("EmitCount")
		if typeof(emitCount) ~= "number" then
			emitCount = DEFAULT_CONFETTI_EMIT_COUNT
		end

		emitter.Enabled = true
		emitter:Emit(emitCount)
	end

	task.delay(CONFETTI_BURST_TIME, function()
		for _, emitter in ipairs(emitters) do
			if emitter and emitter.Parent then
				emitter.Enabled = false
			end
		end
	end)
end

local function spawnTieModelEffect()
	local clone = tieTemplate:Clone()
	clone.Name = "TieModelEffect"
	clone.Parent = effectsFolder

	setVisibleRecursive(clone)

	local boardCenter = getBoardTopPosition()
	local cf =
		CFrame.new(boardCenter + Vector3.new(0, TIE_MODEL_HEIGHT_OFFSET, 0))
		* CFrame.Angles(0, math.rad(TIE_MODEL_ROTATION_DEGREES), 0)

	placeObjectAtPivot(clone, cf)
	return clone
end

local function spawnDynamicWinLine(winningLine, winnerMark)
	if not winningLine then
		return nil
	end

	local firstCell = cells[winningLine[1]]
	local lastCell = cells[winningLine[3]]
	if not firstCell or not lastCell then
		return nil
	end

	local startPos = firstCell.Position + Vector3.new(0, firstCell.Size.Y * 0.5 + WIN_LINE_HEIGHT_OFFSET, 0)
	local endPos = lastCell.Position + Vector3.new(0, lastCell.Size.Y * 0.5 + WIN_LINE_HEIGHT_OFFSET, 0)
	local midPos = (startPos + endPos) * 0.5
	local dist = (endPos - startPos).Magnitude
	local winnerColor = (winnerMark == "O") and O_COLOR or X_COLOR

	local linePart = Instance.new("Part")
	linePart.Name = "WinLineEffect"
	linePart.Anchored = true
	linePart.CanCollide = false
	linePart.CanTouch = false
	linePart.CanQuery = false
	linePart.Material = WIN_LINE_MATERIAL
	linePart.Color = winnerColor
	linePart.Transparency = WIN_LINE_TRANSPARENCY
	linePart.Size = Vector3.new(WIN_LINE_THICKNESS, WIN_LINE_THICKNESS, 0.05)
	linePart.CFrame = CFrame.lookAt(midPos, endPos)
	linePart.Parent = effectsFolder

	TweenService:Create(
		linePart,
		TweenInfo.new(0.18, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
		{
			Size = Vector3.new(WIN_LINE_THICKNESS, WIN_LINE_THICKNESS, dist + WIN_LINE_EXTRA_LENGTH)
		}
	):Play()

	return linePart
end

local function createSeatIndicator(name)
	local anchor = Instance.new("Part")
	anchor.Name = name
	anchor.Anchored = true
	anchor.CanCollide = false
	anchor.CanTouch = false
	anchor.CanQuery = false
	anchor.Transparency = 1
	anchor.Size = Vector3.new(1, 1, 1)
	anchor.Parent = uiFolder

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "Billboard"
	billboard.AlwaysOnTop = false
	billboard.MaxDistance = SEAT_INDICATOR_MAX_DISTANCE
	billboard.Size = UDim2.fromScale(1, 1)
	billboard.LightInfluence = 0
	billboard.Parent = anchor

	local label = Instance.new("TextLabel")
	label.Name = "Role"
	label.BackgroundTransparency = 1
	label.Size = UDim2.fromScale(1, 1)
	label.Font = Enum.Font.GothamBold
	label.TextScaled = true
	label.TextStrokeTransparency = 0.7
	label.TextStrokeColor3 = Color3.new(0, 0, 0)
	label.TextColor3 = Color3.new(1, 1, 1)
	label.Parent = billboard

	return anchor
end

local function createTurnIndicator()
	local anchor = Instance.new("Part")
	anchor.Name = "TurnIndicatorAnchor"
	anchor.Anchored = true
	anchor.CanCollide = false
	anchor.CanTouch = false
	anchor.CanQuery = false
	anchor.Transparency = 1
	anchor.Size = Vector3.new(1, 1, 1)
	anchor.Parent = uiFolder

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "TurnBillboard"
	billboard.AlwaysOnTop = false
	billboard.MaxDistance = TURN_INDICATOR_MAX_DISTANCE
	billboard.Size = UDim2.fromScale(3.5, 1)
	billboard.LightInfluence = 0
	billboard.Enabled = false
	billboard.Parent = anchor

	local label = Instance.new("TextLabel")
	label.Name = "Label"
	label.BackgroundTransparency = 1
	label.Size = UDim2.fromScale(1, 1)
	label.Font = Enum.Font.GothamBold
	label.TextScaled = true
	label.TextStrokeTransparency = 0.7
	label.TextStrokeColor3 = Color3.new(0, 0, 0)
	label.Parent = billboard

	return anchor, billboard
end

local function ensureIndicators()
	if not seatIndicatorA then
		seatIndicatorA = createSeatIndicator("SeatAIndicator")
	end

	if not seatIndicatorB then
		seatIndicatorB = createSeatIndicator("SeatBIndicator")
	end

	if not turnIndicatorAnchor or not turnIndicatorGui then
		turnIndicatorAnchor, turnIndicatorGui = createTurnIndicator()
	end
end

local function getSeatRoleText(seat)
	if seat == seatA then
		if swapSeatRolesNextRound then
			return "O", O_COLOR
		else
			return "X", X_COLOR
		end
	end

	if seat == seatB then
		if swapSeatRolesNextRound then
			return "X", X_COLOR
		else
			return "O", O_COLOR
		end
	end

	return "", Color3.fromRGB(255, 255, 255)
end

local function updateSeatIndicator(indicatorAnchor, seat)
	if not indicatorAnchor or not seat then
		return
	end

	local seatedPlayer = getPlayerFromSeat(seat)
	indicatorAnchor.CFrame = seat.CFrame * CFrame.new(0, SEAT_INDICATOR_HEIGHT, 0)

	local billboard = indicatorAnchor:FindFirstChild("Billboard")
	if not billboard then
		return
	end

	local roleLabel = billboard:FindFirstChild("Role")
	if not roleLabel then
		return
	end

	if seatedPlayer then
		billboard.Enabled = false
		return
	end

	local roleText, roleColor = getSeatRoleText(seat)
	roleLabel.Text = roleText
	roleLabel.TextColor3 = roleColor
	billboard.Enabled = true
end

local function getCurrentPlayer()
	return currentMark == "X" and playerX or playerO
end

local function updateTurnIndicator()
	ensureIndicators()

	if not turnIndicatorAnchor or not turnIndicatorGui then
		return
	end

	local label = turnIndicatorGui:FindFirstChild("Label")
	if not label then
		return
	end

	turnIndicatorAnchor.CFrame = CFrame.new(getBoardTopPosition() + Vector3.new(0, TURN_INDICATOR_HEIGHT, 0))

	if not gameActive then
		turnIndicatorGui.Enabled = false
		return
	end

	local currentPlayer = getCurrentPlayer()

	if currentPlayer then
		label.Text = currentPlayer.Name .. "'s turn"
	else
		label.Text = currentMark .. " TURN"
	end

	label.TextColor3 = (currentMark == "X") and X_COLOR or O_COLOR
	turnIndicatorGui.Enabled = true
end

local function updateIndicators()
	ensureIndicators()

	if seatA then
		updateSeatIndicator(seatIndicatorA, seatA)
	end

	if seatB then
		updateSeatIndicator(seatIndicatorB, seatB)
	end

	updateTurnIndicator()
end

local function showWinEffect(winnerMark, winningLine)
	local winnerColor = (winnerMark == "O") and O_COLOR or X_COLOR
	local winnerText = winnerMark .. " WINS!"

	showFloatingText(winnerText, winnerColor, 1.08, 1.7)
	spawnDynamicWinLine(winningLine, winnerMark)
	burstConfettiEmitters()

	local soundAnchor = Instance.new("Part")
	soundAnchor.Name = "WinSoundAnchor"
	soundAnchor.Anchored = true
	soundAnchor.CanCollide = false
	soundAnchor.CanTouch = false
	soundAnchor.CanQuery = false
	soundAnchor.Transparency = 1
	soundAnchor.Size = Vector3.new(1, 1, 1)
	soundAnchor.CFrame = CFrame.new(getBoardTopPosition() + Vector3.new(0, 1, 0))
	soundAnchor.Parent = effectsFolder

	playEffectSound(WIN_SOUND_ID, soundAnchor, WIN_SOUND_VOLUME)
	Debris:AddItem(soundAnchor, 4)
end

local function showTieEffect()
	showFloatingText("TIE", TIE_COLOR, 1.0, 1.35)
	spawnTieModelEffect()

	local soundAnchor = Instance.new("Part")
	soundAnchor.Name = "TieSoundAnchor"
	soundAnchor.Anchored = true
	soundAnchor.CanCollide = false
	soundAnchor.CanTouch = false
	soundAnchor.CanQuery = false
	soundAnchor.Transparency = 1
	soundAnchor.Size = Vector3.new(1, 1, 1)
	soundAnchor.CFrame = CFrame.new(getBoardTopPosition() + Vector3.new(0, 1, 0))
	soundAnchor.Parent = effectsFolder

	playEffectSound(TIE_SOUND_ID, soundAnchor, TIE_SOUND_VOLUME)
	Debris:AddItem(soundAnchor, 4)
end

local function assignPlayersForRound(seatedA, seatedB)
	if ALLOW_SOLO_TEST then
		local player = seatedA or seatedB

		if player then
			playerX = player
			playerO = player
			currentMark = "X"
			gameActive = true
			return
		end

		playerX = nil
		playerO = nil
		gameActive = false
		return
	end

	if seatedA and seatedB and seatedA ~= seatedB then
		if swapSeatRolesNextRound then
			playerX = seatedB
			playerO = seatedA
		else
			playerX = seatedA
			playerO = seatedB
		end

		currentMark = "X"
		gameActive = true
	else
		playerX = nil
		playerO = nil
		gameActive = false
	end
end

local function resetRound()
	resetId += 1
	clearBoard()
	stopAllConfettiEmitters()

	local seatedA = getPlayerFromSeat(seatA)
	local seatedB = getPlayerFromSeat(seatB)

	print(
		"[TicTacToe] resetRound seatedA =",
		seatedA and seatedA.Name,
		"seatedB =",
		seatedB and seatedB.Name,
		"swap =",
		swapSeatRolesNextRound
	)

	assignPlayersForRound(seatedA, seatedB)
	updateIndicators()
end

local function scheduleReset(seconds, swapNextRound)
	local myId = resetId + 1
	resetId = myId

	task.delay(seconds, function()
		if resetId ~= myId then
			return
		end

		if swapNextRound and not ALLOW_SOLO_TEST then
			local seatedA = getPlayerFromSeat(seatA)
			local seatedB = getPlayerFromSeat(seatB)

			if seatedA and seatedB and seatedA ~= seatedB then
				swapSeatRolesNextRound = not swapSeatRolesNextRound
			end
		end

		resetRound()
	end)
end

local function canMove(player, index)
	if not gameActive then
		print("[TicTacToe] blocked: game not active")
		return false
	end

	if board[index] ~= "" then
		print("[TicTacToe] blocked: cell occupied", index)
		return false
	end

	local seatedA = getPlayerFromSeat(seatA)
	local seatedB = getPlayerFromSeat(seatB)

	if ALLOW_SOLO_TEST then
		local seatedSoloPlayer = seatedA or seatedB
		if not seatedSoloPlayer or player ~= seatedSoloPlayer then
			print("[TicTacToe] blocked: not the solo seated player")
			task.defer(resetRound)
			return false
		end

		return true
	else
		local duoNormal = seatedA and seatedB and seatedA ~= seatedB and playerX == seatedA and playerO == seatedB
		local duoSwapped = seatedA and seatedB and seatedA ~= seatedB and playerX == seatedB and playerO == seatedA

		if not (duoNormal or duoSwapped) then
			print("[TicTacToe] blocked: seat players changed")
			task.defer(function()
				swapSeatRolesNextRound = false
				resetRound()
			end)
			return false
		end

		if player ~= getCurrentPlayer() then
			print("[TicTacToe] blocked: not your turn / not a seated player")
			return false
		end
	end

	return true
end

local function doMove(player, index)
	print("[TicTacToe] click", index, "from", player and player.Name)

	if not canMove(player, index) then
		return
	end

	board[index] = currentMark
	spawnMark(currentMark, index)

	local winnerMark, winningLine = getWinner()
	if winnerMark then
		gameActive = false
		updateIndicators()
		showWinEffect(winnerMark, winningLine)
		scheduleReset(RESET_DELAY, true)
		return
	end

	if boardFull() then
		gameActive = false
		updateIndicators()
		showTieEffect()
		scheduleReset(RESET_DELAY, true)
		return
	end

	currentMark = (currentMark == "X") and "O" or "X"
	updateIndicators()
end

for i = 1, 9 do
	local cell = cells[i]

	local clickDetector = cell:FindFirstChildOfClass("ClickDetector")
	if not clickDetector then
		clickDetector = Instance.new("ClickDetector")
		clickDetector.Parent = cell
	end

	clickDetector.MaxActivationDistance = 32
	clickDetector.MouseClick:Connect(function(player)
		doMove(player, i)
	end)
end

local function onSeatOccupantChanged()
	task.defer(resetRound)
end

if seatA then
	seatA:GetPropertyChangedSignal("Occupant"):Connect(onSeatOccupantChanged)
end

if seatB then
	seatB:GetPropertyChangedSignal("Occupant"):Connect(onSeatOccupantChanged)
end

resetRound()
updateIndicators()
print("[TicTacToe] Server script started")
