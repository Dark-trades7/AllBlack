-- =========================
-- 🌑 NightShadow Hub - Executor Delta (Final Visual)
-- Infinite Jump + Speed + NPC ESP + Auto Farm & Kill Aura Visual
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

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
local autoFarmVisual = false
local killAuraVisual = false

local espBoxes = {}
local combatNPCs = {"Bandit","Pirate","Mink"} -- inimigos de combate
local missionNPCs = {"QuestNPC1","QuestNPC2"} -- NPCs de missão
local targetNPC = nil
local auraRadius = 15 -- raio do Kill Aura visual

-- =========================
-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,320,0,500)
frame.Position = UDim2.new(0,20,0.5,-250)
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
-- FUNÇÕES PARA NPCS
local function getAllNPCs()
    local npcs = {}
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and (table.find(combatNPCs,npc.Name) or table.find(missionNPCs,npc.Name)) then
            table.insert(npcs, npc)
        end
    end
    return npcs
end

local function getCombatNPCs()
    local npcs = {}
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and table.find(combatNPCs,npc.Name) then
            table.insert(npcs, npc)
        end
    end
    return npcs
end

local function getMissionNPCs()
    local npcs = {}
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and table.find(missionNPCs,npc.Name) then
            table.insert(npcs, npc)
        end
    end
    return npcs
end

-- =========================
-- ESP + Auto Farm & Kill Aura VISUAL
RunService.RenderStepped:Connect(function()
    if not npcESPEnabled and not autoFarmVisual and not killAuraVisual then
        for _, box in pairs(espBoxes) do box:Destroy() end
        espBoxes = {}
        targetNPC = nil
        return
    end

    for _, npc in pairs(getAllNPCs()) do
        if npc:FindFirstChild("HumanoidRootPart") and not espBoxes[npc] then
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
            if table.find(combatNPCs,npc.Name) then
                txt.TextColor3 = Color3.fromRGB(255,0,0) -- vermelho para inimigos
            else
                txt.TextColor3 = Color3.fromRGB(0,255,0) -- verde para missão
            end
            espBoxes[npc] = box
        end
    end

    -- Target NPC mais próximo
    local closest, dist = nil, math.huge
    for _, npc in pairs(getCombatNPCs()) do
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

local farmBtn = createButton("AUTO FARM VISUAL: OFF", 170)
farmBtn.MouseButton1Click:Connect(function()
    autoFarmVisual = not autoFarmVisual
    farmBtn.Text = autoFarmVisual and "AUTO FARM VISUAL: ON" or "AUTO FARM VISUAL: OFF"
end)

local auraBtn = createButton("KILL AURA VISUAL: OFF", 220)
auraBtn.MouseButton1Click:Connect(function()
    killAuraVisual = not killAuraVisual
    auraBtn.Text = killAuraVisual and "KILL AURA VISUAL: ON" or "KILL AURA VISUAL: OFF"
end)

-- =========================
-- TECLA E PARA “ATACAR” NPC MAIS PRÓXIMO (VISUAL)
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.E and targetNPC then
        print("NPC alvo selecionado:", targetNPC.Name)
        -- Aqui você poderia disparar RemoteEvent do jogo, se existir
        -- Exemplo:
        -- game:GetService("ReplicatedStorage").Remotes.CommF_:FireServer("Damage", targetNPC, 10)
    end
end)

-- =========================
-- FUNÇÃO PARA DESTACAR NPCS NO RAIO DO KILL AURA
RunService.RenderStepped:Connect(function()
    if killAuraVisual then
        for _, npc in pairs(getCombatNPCs()) do
            local root = npc:FindFirstChild("HumanoidRootPart")
            if root then
                local dist = (hrp.Position - root.Position).Magnitude
                if dist <= auraRadius then
                    if espBoxes[npc] then
                        espBoxes[npc].TextLabel.TextColor3 = Color3.fromRGB(0,0,255) -- azul para dentro do Kill Aura
                    end
                end
            end
        end
    end
end)
