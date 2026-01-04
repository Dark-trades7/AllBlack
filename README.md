-- 🌑 NightShadow Hub - Teste de execução mínima
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criando GUI
local gui = Instance.new("ScreenGui", playerGui)
gui.Name = "NightShadowHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 200)
frame.Position = UDim2.new(0.5, -150, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(50,50,50)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame)

-- Função para criar botões
local function createButton(text, y)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(0.9,0,0,40)
    b.Position = UDim2.new(0.05,0,0,y)
    b.Text = text
    b.TextScaled = true
    b.BackgroundColor3 = Color3.fromRGB(100,100,100)
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b)
    return b
end

-- Criando botões de teste
local autoBtn = createButton("AUTO FARM: OFF", 20)
autoBtn.MouseButton1Click:Connect(function()
    autoBtn.Text = autoBtn.Text == "AUTO FARM: OFF" and "AUTO FARM: ON" or "AUTO FARM: OFF"
end)

local killBtn = createButton("KILL AURA: OFF", 80)
killBtn.MouseButton1Click:Connect(function()
    killBtn.Text = killBtn.Text == "KILL AURA: OFF" and "KILL AURA: ON" or "KILL AURA: OFF"
end)

local flyBtn = createButton("FLY: OFF", 140)
flyBtn.MouseButton1Click:Connect(function()
    flyBtn.Text = flyBtn.Text == "FLY: OFF" and "FLY: ON" or "FLY: OFF"
end)
