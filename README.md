-- =========================
-- 🌑 NightShadow Hub (TEST GAME)
-- Auto Farm + Kill Aura REAL + Fly + Jump + Speed
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

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
-- CONFIG
local autoFarm = false
local killAura = false
local farmHeight = 6
local auraRadius = 12
local auraDamage = 15
local auraDelay = 0.2

-- =========================
-- GUI
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,260,0,300)
frame.Position = UDim2.new(0,20,0.5,-150)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
Instance.new("UICorner", frame)

local function button(text, y)
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

-- =========================
-- AUTO FARM (NPCs)
local function getClosestNPC()
    local closest, dist = nil, math.huge
    for _, npc in pairs(workspace.NPCs:GetChildren()) do
        if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
            local root = npc:FindFirstChild("HumanoidRootPart")
            if root then
                local d = (root.Position - hrp.Position).Magnitude
                if d < dist then
                    dist = d
                    closest = npc
                end
            end
        end
    end
    return closest
end

RunService.Heartbeat:Connect(function()
    if not autoFarm or not hrp then return end
    local npc = getClosestNPC()
    if npc and npc:FindFirstChild("HumanoidRootPart") then
        hrp.CFrame = CFrame.new(
            npc.HumanoidRootPart.Position + Vector3.new(0,farmHeight,0),
            npc.HumanoidRootPart.Position
        )
        hrp.Velocity = Vector3.zero
    end
end)

-- =========================
-- KILL AURA REAL
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
local flying = false
local flySpeed = 80

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
local infJump = false
UserInputService.JumpRequest:Connect(function()
    if infJump then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- =========================
-- SPEED
local speeds = {16,50,100,200}
local speedIndex = 1

-- =========================
-- BOTÕES
button("AUTO FARM: OFF",20).MouseButton1Click:Connect(function(b)
    autoFarm = not autoFarm
    b.Text = autoFarm and "AUTO FARM: ON" or "AUTO FARM: OFF"
end)

button("KILL AURA: OFF",70).MouseButton1Click:Connect(function(b)
    killAura = not killAura
    b.Text = killAura and "KILL AURA: ON" or "KILL AURA: OFF"
end)

button("FLY: OFF",120).MouseButton1Click:Connect(function(b)
    flying = not flying
    b.Text = flying and "FLY: ON" or "FLY: OFF"
end)

button("INFINITE JUMP: OFF",170).MouseButton1Click:Connect(function(b)
    infJump = not infJump
    b.Text = infJump and "INFINITE JUMP: ON" or "INFINITE JUMP: OFF"
end)

button("SPEED",220).MouseButton1Click:Connect(function()
    speedIndex += 1
    if speedIndex > #speeds then speedIndex = 1 end
    humanoid.WalkSpeed = speeds[speedIndex]
end)
