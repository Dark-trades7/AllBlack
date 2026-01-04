-- =========================
-- 🌑 NightShadow Hub (TEST GAME) - VERSÃO REVISADA
-- Auto Farm + Kill Aura + Fly + Infinite Jump + Speed + NPC ESP
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
local espBoxes = {}

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
        local targetPos
