-- =========================
-- 🌑 NightShadow Hub v1.8
-- Auto Win (NPC) + Kill Aura + Auto Rebirth + Fly + Jump + Speed
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- =========================
-- PERSONAGEM
-- =========================
local function getChar()
    local c = player.Character or player.CharacterAdded:Wait()
    return c,
        c:WaitForChild("Humanoid"),
        c:WaitForChild("HumanoidRootPart")
end

local char, humanoid, hrp = getChar()

player.CharacterAdded:Connect(function()
    char, humanoid, hrp = getChar()
    humanoid.WalkSpeed = 16
end)

-- =========================
-- AUTO WIN (NPCs PRÓXIMOS)
-- =========================
local tpEnabled = false
local tpConnection
local npcList = {}
local npcIndex = 1
local searchRadius = 500
local npcHeightOffset = 6 -- altura acima do NPC (proteção)

local function getNearbyNPCs()
    local list = {}
    if not hrp then return list end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model")
            and obj ~= char
            and obj:FindFirstChild("Humanoid")
            and obj.Humanoid.Health > 0
        then
            local root = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
            if root then
                local dist = (root.Position - hrp.Position).Magnitude
                if dist <= searchRadius then
                    table.insert(list, root)
                end
            end
        end
    end

    table.sort(list, function(a, b)
        return (a.Position - hrp.Position).Magnitude <
               (b.Position - hrp.Position).Magnitude
    end)

    return list
end

-- =========================
-- KILL AURA (HITBOX)
-- =========================
local killAura = false
local auraRadius = 10
local auraDamageDelay = 0.25
local lastAuraHit = {}
local auraPart
local auraConnection

local function createAura()
    if auraPart or not hrp then return end

    auraPart = Instance.new("Part")
    auraPart.Name = "KillAura"
    auraPart.Size = Vector3.new(auraRadius, auraRadius * 1.5, auraRadius)
    auraPart.Transparency = 1
    auraPart.CanCollide = false
    auraPart.Massless = true
    auraPart.Parent = workspace

    local weld = Instance.new("WeldConstraint")
    weld.Part0 = auraPart
    weld.Part1 = hrp
    weld.Parent = auraPart
end

local function removeAura()
    if auraConnection then
        auraConnection:Disconnect()
        auraConnection = nil
    end
    if auraPart then
        auraPart:Destroy()
        auraPart = nil
    end
    lastAuraHit = {}
end

local function onAuraTouch(hit)
    if not killAura then return end

    local model = hit:FindFirstAncestorOfClass("Model")
    if not model or model == char then return end

    local hum = model:FindFirstChild("Humanoid")
    if not hum or hum.Health <= 0 then return end

    local now = tick()
    if lastAuraHit[hum] and now - lastAuraHit[hum] < auraDamageDelay then
        return
    end
    lastAuraHit[hum] = now

    pcall(function()
        hum:TakeDamage(10)
    end)
end

local function startKillAura()
    createAura()
    auraConnection = auraPart.Touched:Connect(onAuraTouch)
end

local function stopKillAura()
    removeAura()
end

-- =========================
-- AUTO WIN LOOP
-- =========================
local function startTP()
    if tpConnection then return end

    tpConnection = RunService.Heartbeat:Connect(function()
        if not tpEnabled or not hrp then return end

        if #npcList == 0 or npcIndex > #npcList then
            npcList = getNearbyNPCs()
            npcIndex = 1
            if #npcList == 0 then return end
        end

        local npcRoot = npcList[npcIndex]
        if npcRoot and npcRoot.Parent then
            -- TP acima do NPC olhando para ele
            hrp.CFrame = CFrame.new(
                npcRoot.Position + Vector3.new(0, npcHeightOffset, 0),
                npcRoot.Position
            )

            -- trava movimento para não cair / knockback
            hrp.AssemblyLinearVelocity = Vector3.zero

            npcIndex += 1
        else
            npcIndex += 1
        end
    end)
end

local function stopTP()
    if tpConnection then
        tpConnection:Disconnect()
        tpConnection = nil
    end
end

-- =========================
-- AUTO REBIRTH
-- =========================
local autoRebirth = false
local lastWins = player:GetAttribute("Wins") or 0
local rebirthRemote

for _, v in pairs(ReplicatedStorage:GetDescendants()) do
    if v:IsA("RemoteEvent") and v.Name:lower():find("rebirth") then
        rebirthRemote = v
        break
    end
end

task.spawn(function()
    while task.wait(2) do
        if not autoRebirth or not rebirthRemote then continue end
        local wins = player:GetAttribute("Wins")
        if wins and wins > lastWins then
            lastWins = wins
            pcall(function()
                rebirthRemote:FireServer()
            end)
        end
    end
end)

-- =========================
-- GUI
-- =========================
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 420)
frame.Position = UDim2.new(0, 20, 0.5, -210)
frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(170,0,0)
stroke.Thickness = 3

-- DRAG
do
    local dragging, dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- TÍTULO
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,40)
title.Text = "🩸 NightShadow Hub v1.8"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextScaled = true

-- FUNÇÃO BOTÃO
local function createButton(text, y)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(0.85,0,0,40)
    b.Position = UDim2.new(0.075,0,0,y)
    b.Text = text
    b.TextScaled = true
    b.BackgroundColor3 = Color3.fromRGB(35,35,35)
    b.TextColor3 = Color3.fromRGB(255,255,255)
    Instance.new("UICorner", b)
    return b
end

-- BOTÕES
local tpButton = createButton("AUTO WIN: OFF", 50)
tpButton.MouseButton1Click:Connect(function()
    tpEnabled = not tpEnabled
    tpButton.Text = tpEnabled and "AUTO WIN: ON" or "AUTO WIN: OFF"
    tpButton.BackgroundColor3 = tpEnabled and Color3.fromRGB(0,170,0) or Color3.fromRGB(170,0,0)
    if tpEnabled then startTP() else stopTP() end
end)

local auraBtn = createButton("KILL AURA: OFF", 100)
auraBtn.MouseButton1Click:Connect(function()
    killAura = not killAura
    auraBtn.Text = killAura and "KILL AURA: ON" or "KILL AURA: OFF"
    auraBtn.BackgroundColor3 = killAura and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
    if killAura then startKillAura() else stopKillAura() end
end)

local rebBtn = createButton("AUTO REBIRTH: OFF", 150)
rebBtn.MouseButton1Click:Connect(function()
    autoRebirth = not autoRebirth
    rebBtn.Text = autoRebirth and "AUTO REBIRTH: ON" or "AUTO REBIRTH: OFF"
    rebBtn.BackgroundColor3 = autoRebirth and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
end)

-- FLY
local flyButton = createButton("FLY: OFF", 200)
local flying = false
local flyConn
local flySpeed = 100

local function startFly()
    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    flyConn = RunService.RenderStepped:Connect(function()
        if not flying or not hrp then return end
        local cam = workspace.CurrentCamera
        local move = Vector3.zero

        if UserInputService:IsKeyDown(Enum.KeyCode.W) then move += cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then move -= cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then move -= cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then move += cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move += Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then move -= Vector3.new(0,1,0) end

        hrp.Velocity = move.Magnitude > 0 and move.Unit * flySpeed or Vector3.zero
    end)
end

local function stopF
