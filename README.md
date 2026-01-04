-- =========================
-- 🌑 NightShadow Hub - Executor Delta
-- Auto Farm + Kill Aura + Fly + Infinite Jump + Speed + NPC ESP
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- PLAYER E CHARACTER
local player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(c)
    char = c
    humanoid = char:WaitForChild("Humanoid")
    hrp = char:WaitForChild("HumanoidRootPart")
end)

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
local espBoxes = {}

-- =========================
-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
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

-- =========================
-- FUNÇÕES DE NPC
local function getClosestNPC()
    if not workspace:FindFirstChild("NPCs") then return nil end
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
    if autoFarm and hrp then
        local npc = getClosestNPC()
        if npc and npc:FindFirstChild("HumanoidRootPart") then
            local targetPos = npc.HumanoidRootPart.Position + Vector3.new(0,farmHeight,0)
            local tween = TweenService:Create(hrp, TweenInfo.new(0.1), {CFrame = CFrame.new(targetPos, npc.HumanoidRootPart.Position)})
            tween:Play()
            hrp.Velocity = Vector3.zero
        end
    end
end)

-- =========================
-- KILL AURA
task.spawn(function()
    while task.wait(auraDelay) do
        if killAura and workspace:FindFirstChild("NPCs") then
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
    end
end)

-- =========================
-- FLY
RunService.RenderStepped:Connect(function()
    if flying and hrp then
        local cam = workspace.CurrentCamera
        local move = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then move = move + cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then move = move - cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then move = move - cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then move = move + cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move = move + Vector3.new(0,1,0) end
        if move.Magnitude > 0 then
            hrp.Velocity = move.Unit * flySpeed
        else
            hrp.Velocity = Vector3.zero
        end
    end
end)

-- =========================
-- INFINITE JUMP
UserInputService.JumpRequest:Connect(function()
    if infJump and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- =========================
-- NPC ESP
RunService.RenderStepped:Connect(function()
    if npcESPEnabled then
        if workspace:FindFirstChild("NPCs") then
            for _, npc in pairs(workspace.NPCs:GetChildren()) do
                if not espBoxes[npc] and npc:FindFirstChild("HumanoidRootPart") then
                    local box = Instance.new("BillboardGui", npc)
                    box.Name = "ESPBox"
                    box.Size = UDim2.new(0,50,0,50)
                    box.Adornee = npc.HumanoidRootPart
                    box.AlwaysOnTop = true
                    local txt = Instance.new("TextLabel", box)
                    txt.Text = npc.Name
                    txt.TextScaled = true
                    txt.BackgroundTransparency = 1
                    txt.Size = UDim2.new(1,0,1,0)
                    txt.TextColor3 = Color3.new(1,0,0)
                    espBoxes[npc] = box
                end
            end
        end
    else
        for _, box in pairs(espBoxes) do box:Destroy() end
        espBoxes = {}
    end
end)

-- =========================
-- BOTÕES
local autoBtn = createButton("AUTO FARM: OFF", 20)
autoBtn.MouseButton1Click:Connect(function()
    autoFarm = not autoFarm
    autoBtn.Text = autoFarm and "AUTO FARM: ON" or "AUTO FARM: OFF"
end)

local auraBtn = createButton("KILL AURA: OFF", 70)
auraBtn.MouseButton1Click:Connect(function()
    killAura = not killAura
    auraBtn.Text = killAura and "KILL AURA: ON" or "KILL AURA: OFF"
end)

local flyBtn = createButton("FLY: OFF", 120)
flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    flyBtn.Text = flying and "FLY: ON" or "FLY: OFF"
end)

local jumpBtn = createButton("INFINITE JUMP: OFF", 170)
jumpBtn.MouseButton1Click:Connect(function()
    infJump = not infJump
    jumpBtn.Text = infJump and "INFINITE JUMP: ON" or "INFINITE JUMP: OFF"
end)

local speedBtn = createButton("SPEED: 16", 220)
speedBtn.MouseButton1Click:Connect(function()
    speedIndex = speedIndex + 1
    if speedIndex > #speeds then speedIndex = 1 end
    humanoid.WalkSpeed = speeds[speedIndex]
    speedBtn.Text = "SPEED: "..speeds[speedIndex]
end)

local espBtn = createButton("NPC ESP: OFF", 270)
espBtn.MouseButton1Click:Connect(function()
    npcESPEnabled = not npcESPEnabled
    espBtn.Text = npcESPEnabled and "NPC ESP: ON" or "NPC ESP: OFF"
end)
