-- =========================
-- 🌑 NightShadow Hub v1.8
-- Auto Win (NPC) + Kill Aura + Auto Rebirth + Fly + Jump + Speed
-- Delta Executor Friendly
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
-- CONFIGURAÇÕES AUTO WIN / KILL AURA
local tpEnabled = false
local tpConnection
local npcList = {}
local npcIndex = 1
local searchRadius = 500
local npcHeightOffset = 6 -- altura acima do NPC

local killAura = false
local auraRadius = 10
local auraDamageDelay = 0.25
local lastAuraHit = {}
local auraPart
local auraConnection

-- =========================
-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "NightShadowHub"
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 380)
frame.Position = UDim2.new(0, 20, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

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

-- =========================
-- AUTO WIN
local function getNearbyNPCs()
    local list = {}
    if not hrp then return list end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= char and obj:FindFirstChild("Humanoid") and obj.Humanoid.Health > 0 then
            local root = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
            if root then
                local dist = (root.Position - hrp.Position).Magnitude
                if dist <= searchRadius then
                    table.insert(list, root)
                end
            end
        end
    end
    table.sort(list, function(a,b)
        return (a.Position-hrp.Position).Magnitude < (b.Position-hrp.Position).Magnitude
    end)
    return list
end

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
            hrp.CFrame = CFrame.new(npcRoot.Position + Vector3.new(0,npcHeightOffset,0), npcRoot.Position)
            hrp.Velocity = Vector3.zero
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
-- KILL AURA
local function createAura()
    if auraPart or not hrp then return end
    auraPart = Instance.new("Part")
    auraPart.Size = Vector3.new(auraRadius, auraRadius*1.5, auraRadius)
    auraPart.Transparency = 1
    auraPart.CanCollide = false
    auraPart.Massless = true
    auraPart.Anchored = false
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
    if lastAuraHit[hum] and now - lastAuraHit[hum] < auraDamageDelay then return end
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
-- AUTO REBIRTH
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
-- FLY
local flyButton = createButton("FLY: OFF", 170)
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

local function stopFly()
    if flyConn then flyConn:Disconnect() flyConn=nil end
    humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
end

flyButton.MouseButton1Click:Connect(function()
    flying = not flying
    flyButton.Text = flying and "FLY: ON" or "FLY: OFF"
    flyButton.BackgroundColor3 = flying and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
    if flying then startFly() else stopFly() end
end)

-- =========================
-- INFINITE JUMP
local infJumpButton = createButton("INFINITE JUMP: OFF", 220)
local infJump = false
UserInputService.JumpRequest:Connect(function()
    if infJump and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

infJumpButton.MouseButton1Click:Connect(function()
    infJump = not infJump
    infJumpButton.Text = infJump and "INFINITE JUMP: ON" or "INFINITE JUMP: OFF"
    infJumpButton.BackgroundColor3 = infJump and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
end)

-- =========================
-- SPEED
local speedButton = createButton("SPEED: 100", 270)
local speeds = {50,100,150,200,250,300}
local index = 2
speedButton.MouseButton1Click:Connect(function()
    index += 1
    if index > #speeds then index = 1 end
    humanoid.WalkSpeed = speeds[index]
    speedButton.Text = "SPEED: "..speeds[index]
end)

-- =========================
-- AUTO WIN / KILL AURA / REBIRTH BOTÕES
local tpBtn = createButton("AUTO WIN: OFF", 20)
tpBtn.MouseButton1Click:Connect(function()
    tpEnabled = not tpEnabled
    tpBtn.Text = tpEnabled and "AUTO WIN: ON" or "AUTO WIN: OFF"
    tpBtn.BackgroundColor3 = tpEnabled and Color3.fromRGB(0,170,0) or Color3.fromRGB(170,0,0)
    if tpEnabled then startTP() else stopTP() end
end)

local auraBtn = createButton("KILL AURA: OFF", 70)
auraBtn.MouseButton1Click:Connect(function()
    killAura = not killAura
    auraBtn.Text = killAura and "KILL AURA: ON" or "KILL AURA: OFF"
    auraBtn.BackgroundColor3 = killAura and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
    if killAura then startKillAura() else stopKillAura() end
end)

local rebBtn = createButton("AUTO REBIRTH: OFF", 120)
rebBtn.MouseButton1Click:Connect(function()
    autoRebirth = not autoRebirth
    rebBtn.Text = autoRebirth and "AUTO REBIRTH: ON" or "AUTO REBIRTH: OFF"
    rebBtn.BackgroundColor3 = autoRebirth and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
end)

-- =========================
-- RESET AO MORRER
humanoid.Died:Connect(function()
    humanoid.WalkSpeed = 16
    flying = false
    infJump = false
    autoRebirth = false
    killAura = false
    tpEnabled = false
    stopFly()
    stopTP()
    stopKillAura()
end)
