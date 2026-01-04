-- =========================
-- 🌑 NightShadow Hub (TEST GAME)
-- Auto Farm + Kill Aura + Fly + Infinite Jump + Speed + ESP + Missões
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- =========================
-- PERSONAGEM
local function getChar()
    local c = player.Character or player.CharacterAdded:Wait()
    return c,
        c:WaitForChild("Humanoid"),
        c:WaitForChild("HumanoidRootPart")
end

local char, humanoid, hrp = getChar()
player.CharacterAdded:Connect(function()
    char, humanoid, hrp = getChar()
end)

-- =========================
-- CONFIGURAÇÃO
local autoFarm = false
local killAura = false
local auraRadius = 12
local auraDamage = 15
local auraDelay = 0.2
local farmHeight = 6

local flying = false
local flySpeed = 80

local infJump = false
local speeds = {16,50,100,200}
local speedIndex = 1

local npcESPEnabled = false
local missionTarget = nil

-- =========================
-- GUI BASE
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,320,0,400)
frame.Position = UDim2.new(0,20,0.5,-200)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame)

local function createButton(text, y)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(0.9,0,0,40)
    b.Position = UDim2.new(0.05,0,0,y)
    b.Text = text
    b.TextScaled = true
    b.BackgroundColor3 = Color3.fromRGB(40,40,40)
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b)
    return b
end

local function createLabel(text, y)
    local l = Instance.new("TextLabel", frame)
    l.Size = UDim2.new(0.9,0,0,30)
    l.Position = UDim2.new(0.05,0,0,y)
    l.Text = text
    l.TextScaled = true
    l.BackgroundTransparency = 1
    l.TextColor3 = Color3.new(1,1,1)
    return l
end

-- =========================
-- FUNÇÕES DE NPC
local function getClosestNPC()
    local closest, dist = nil, math.huge
    for _, npc in pairs(workspace.NPCs:GetChildren()) do
        local hum = npc:FindFirstChild("Humanoid")
        local root = npc:FindFirstChild("HumanoidRootPart")
        if hum and root and hum.Health > 0 then
            local d = (root.Position - hrp.Position).Magnitude
            if d < dist then
                dist = d
                closest = npc
            end
        end
    end
    return closest
end

-- =========================
-- AUTO FARM
RunService.Heartbeat:Connect(function()
    if not autoFarm or not hrp then return end
    local npc = getClosestNPC()
    if npc then
        hrp.CFrame = CFrame.new(npc.HumanoidRootPart.Position + Vector3.new(0,farmHeight,0), npc.HumanoidRootPart.Position)
        hrp.Velocity = Vector3.zero
    end
end)

-- =========================
-- KILL AURA
task.spawn(function()
    while task.wait(auraDelay) do
        if not killAura then continue end
        for _, npc in pairs(workspace.NPCs:GetChildren()) do
            local hum = npc:FindFirstChild("Humanoid")
            local root = npc:FindFirstChild("HumanoidRootPart")
            if hum and root and hum.Health > 0 then
                if (root.Position - hrp.Position).Magnitude <= auraRadius then
                    hum:TakeDamage(auraDamage)
                end
            end
        end
    end
end)

-- =========================
-- FLY
RunService.RenderStepped:Connect(function()
    if not flying then return end
    local cam = workspace.CurrentCamera
    local move = Vector3.zero
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then move += cam.CFrame.LookVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then move -= cam.CFrame.LookVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then move -= cam.CFrame.RightVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then move += cam.CFrame.RightVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move += Vector3.new(0,1,0) end
    hrp.Velocity = move.Magnitude > 0 and move.Unit * flySpeed or Vector3.zero
end)

-- =========================
-- INFINITE JUMP
UserInputService.JumpRequest:Connect(function()
    if infJump then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- =========================
-- NPC ESP
local espBoxes = {}
RunService.RenderStepped:Connect(function()
    if not npcESPEnabled then
        for _, box in pairs(espBoxes) do box:Destroy() end
        espBoxes = {}
        return
    end
    for _, npc in pairs(workspace.NPCs:GetChildren()) do
        if not espBoxes[npc] and npc:FindFirstChild("HumanoidRootPart") then
            local box = Instance.new("BillboardGui", npc)
            box.Name = "ESPBox"
            box.Size = UDim2.new(0,50,0,50)
            box.Adornee = npc.HumanoidRootPart
            local txt = Instance.new("TextLabel", box)
            txt.Text = npc.Name
            txt.TextScaled = true
            txt.BackgroundTransparency = 1
            txt.Size = UDim2.new(1,0,1,0)
            txt.TextColor3 = Color3.new(1,0,0)
            espBoxes[npc] = box
        end
    end
end)

-- =========================
-- BOTÕES
createButton("AUTO FARM: OFF", 20).MouseButton1Click:Connect(function(b)
    autoFarm = not autoFarm
    b.Text = autoFarm and "AUTO FARM: ON" or "AUTO FARM: OFF"
end)

createButton("KILL AURA: OFF", 70).MouseButton1Click(function(b)
    killAura = not killAura
    b.Text = killAura and "KILL AURA: ON" or "KILL AURA: OFF"
end)

createButton("FLY: OFF", 120).MouseButton1Click(function(b)
    flying = not flying
    b.Text = flying and "FLY: ON" or "FLY: OFF"
end)

createButton("INFINITE JUMP: OFF", 170).MouseButton1Click(function(b)
    infJump = not infJump
    b.Text = infJump and "INFINITE JUMP: ON" or "INFINITE JUMP: OFF"
end)

createButton("SPEED", 220).MouseButton1Click(function()
    speedIndex += 1
    if speedIndex > #speeds then speedIndex = 1 end
    humanoid.WalkSpeed = speeds[speedIndex]
end)

createButton("NPC ESP: OFF", 270).MouseButton1Click(function(b)
    npcESPEnabled = not npcESPEnabled
    b.Text = npcESPEnabled and "NPC ESP: ON" or "NPC ESP: OFF"
end)
