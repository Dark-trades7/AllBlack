-- =========================
-- 🌑 NightShadow Hub - Executor Delta
-- Infinite Jump + Speed + NPC ESP + Target Attack (visual)
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- PLAYER
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(c)
    char = c
    humanoid = char:WaitForChild("Humanoid")
    hrp = char:WaitForChild("HumanoidRootPart")
end)

-- CONFIGURAÇÃO
local infJump = false
local speeds = {16,50,100,200}
local speedIndex = 1

local npcESPEnabled = false
local espBoxes = {}
local combatNPCs = {"Bandit","Pirate","Mink"} -- nomes dos NPCs de combate
local targetNPC = nil

-- =========================
-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,320,0,400)
frame.Position = UDim2.new(0,20,0.5,-200)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame)

local function createButton(text, y)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(0.9,0,0,40)
    b.Position = UDim2.new(0.05,0,0,y)
    b.Text = text
    b.TextScaled = true
    b.BackgroundColor3 = Color3.fromRGB(40,40,40)
    b.TextColor3 = Color3.fromRGB(255,255,255)
    Instance.new("UICorner", b)
    return b
end

-- =========================
-- INFINITE JUMP
UserInputService.JumpRequest:Connect(function()
    if infJump and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- =========================
-- SPEED
local function setSpeed()
    humanoid.WalkSpeed = speeds[speedIndex]
end
setSpeed()

-- =========================
-- NPC ESP + TARGET
local function getAllNPCs()
    local npcs = {}
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and table.find(combatNPCs, npc.Name) then
            table.insert(npcs, npc)
        end
    end
    return npcs
end

RunService.RenderStepped:Connect(function()
    -- Limpar ESP se desativado
    if not npcESPEnabled then
        for _, box in pairs(espBoxes) do box:Destroy() end
        espBoxes = {}
        targetNPC = nil
        return
    end

    -- Criar ESP nos NPCs
    for _, npc in pairs(getAllNPCs()) do
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

    -- Escolher NPC mais próximo como alvo
    local closest, dist = nil, math.huge
    for _, npc in pairs(getAllNPCs()) do
        local root = npc:FindFirstChild("HumanoidRootPart")
        if root then
            local d = (hrp.Position - root.Position).Magnitude
            if d < dist then
                dist = d
                closest = npc
            end
        end
    end
    targetNPC = closest
end)

-- =========================
-- BOTÕES
local jumpBtn = createButton("INFINITE JUMP: OFF", 20)
jumpBtn.MouseButton1Click:Connect(function()
    infJump = not infJump
    jumpBtn.Text = infJump and "INFINITE JUMP: ON" or "INFINITE JUMP: OFF"
end)

local speedBtn = createButton("SPEED: 16", 70)
speedBtn.MouseButton1Click:Connect(function()
    speedIndex = speedIndex + 1
    if speedIndex > #speeds then speedIndex = 1 end
    setSpeed()
    speedBtn.Text = "SPEED: "..speeds[speedIndex]
end)

local espBtn = createButton("NPC ESP: OFF", 120)
espBtn.MouseButton1Click:Connect(function()
    npcESPEnabled = not npcESPEnabled
    espBtn.Text = npcESPEnabled and "NPC ESP: ON" or "NPC ESP: OFF"
end)

-- =========================
-- “TARGET ATTACK” SIMULADO (Auto Kill/Auto Farm visual)
-- Você pode clicar para atacar o NPC mais próximo
local function attackTarget()
    if targetNPC then
        print("Ataque simulado no NPC:", targetNPC.Name)
        -- Aqui você pode disparar o RemoteEvent do seu jogo, se existir
        -- Exemplo:
        -- ReplicatedStorage.Remotes.CommF_:FireServer("Damage", targetNPC, 10)
    end
end

UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.E then
        attackTarget()
    end
end)
