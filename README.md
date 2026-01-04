-- =========================
-- 🌑 NightShadow Hub - Executor Delta Confiável
-- Infinite Jump + Speed + NPC ESP
-- =========================

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

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
local infJump = false
local speeds = {16,50,100,200}
local speedIndex = 1

local npcESPEnabled = false
local espBoxes = {}
local npcFolderName = "NPCs" -- Mude para o nome correto da pasta de inimigos do jogo

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
-- INFINITE JUMP
UserInputService.JumpRequest:Connect(function()
    if infJump and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- =========================
-- NPC ESP
local function getNPCs()
    if workspace:FindFirstChild(npcFolderName) then
        return workspace[npcFolderName]:GetChildren()
    end
    return {}
end

RunService.RenderStepped:Connect(function()
    if npcESPEnabled then
        for _, npc in pairs(getNPCs()) do
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
    else
        for _, box in pairs(espBoxes) do box:Destroy() end
        espBoxes = {}
    end
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
    humanoid.WalkSpeed = speeds[speedIndex]
    speedBtn.Text = "SPEED: "..speeds[speedIndex]
end)

local espBtn = createButton("NPC ESP: OFF", 120)
espBtn.MouseButton1Click:Connect(function()
    npcESPEnabled = not npcESPEnabled
    espBtn.Text = npcESPEnabled and "NPC ESP: ON" or "NPC ESP: OFF"
end)
