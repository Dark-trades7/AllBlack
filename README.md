-- =========================
-- 🌑 NightShadow Hub v1.8
-- Auto Win (NPC) + Auto Kill + Auto Rebirth + Fly + Jump + Speed
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
-- AUTO KILL
-- =========================
local autoKill = false
local killDistance = 8
local attackDelay = 0.15
local lastAttack = 0

local function getTool()
    if char then
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("Tool") then
                return v
            end
        end
    end

    for _, v in pairs(player.Backpack:GetChildren()) do
        if v:IsA("Tool") then
            humanoid:EquipTool(v)
            return v
        end
    end
end

local function tryKillNPC(npcRoot)
    if not autoKill or not npcRoot or not npcRoot.Parent then return end

    local npcHumanoid = npcRoot.Parent:FindFirstChild("Humanoid")
    if not npcHumanoid or npcHumanoid.Health <= 0 then return end

    if (npcRoot.Position - hrp.Position).Magnitude > killDistance then return end

    local tool = getTool()
    if not tool then return end

    if tick() - lastAttack >= attackDelay then
        lastAttack = tick()
        pcall(function()
            tool:Activate()
        end)
    end
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
            hrp.CFrame = npcRoot.CFrame * CFrame.new(0, 0, -2)
            tryKillNPC(npcRoot)
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

local killBtn = createButton("AUTO KILL: OFF", 100)
killBtn.MouseButton1Click:Connect(function()
    autoKill = not autoKill
    killBtn.Text = autoKill and "AUTO KILL: ON" or "AUTO KILL: OFF"
    killBtn.BackgroundColor3 = autoKill and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
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

local function stopFly()
    if flyConn then flyConn:Disconnect() flyConn = nil end
    humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
end

flyButton.MouseButton1Click:Connect(function()
    flying = not flying
    flyButton.Text = flying and "FLY: ON (100)" or "FLY: OFF"
    flyButton.BackgroundColor3 = flying and Color3.fromRGB(0,170,0) or Color3.fromRGB(35,35,35)
    if flying then startFly() else stopFly() end
end)

-- INFINITE JUMP
local infJumpButton = createButton("INFINITE JUMP: OFF", 250)
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

-- SPEED
local speedButton = createButton("SPEED: 100", 300)
local speeds = {50,100,150,200,250,300}
local index = 2

speedButton.MouseButton1Click:Connect(function()
    index += 1
    if index > #speeds then index = 1 end
    humanoid.WalkSpeed = speeds[index]
    speedButton.Text = "SPEED: "..speeds[index]
end)

-- RESET AO MORRER
humanoid.Died:Connect(function()
    humanoid.WalkSpeed = 16
    flying = false
    infJump = false
    autoRebirth = false
    autoKill = false
    stopFly()
    stopTP()
end)
