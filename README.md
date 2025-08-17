--[[
    Script educativo para testar seu anti-cheat em jogo próprio no Roblox.
    Interface: bolinha preta para abrir menu
    Funções: Speed Boost, Super Pulo, NoClip, "Anti Hit" que só bloqueia dano
    NÃO USE em jogos de terceiros!
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BrainrotTestGui"
if syn and syn.protect_gui then
    syn.protect_gui(screenGui)
    screenGui.Parent = game.CoreGui
else
    screenGui.Parent = game:GetService("CoreGui")
end

local openButton = Instance.new("TextButton")
openButton.Name = "OpenMenu"
openButton.Size = UDim2.new(0, 50, 0, 50)
openButton.Position = UDim2.new(0, 20, 0, 150)
openButton.BackgroundColor3 = Color3.new(0,0,0)
openButton.Text = ""
openButton.BorderSizePixel = 0
openButton.Parent = screenGui
openButton.AutoButtonColor = false
openButton.ZIndex = 2

local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 240, 0, 260)
menuFrame.Position = UDim2.new(0, 80, 0, 120)
menuFrame.BackgroundColor3 = Color3.fromRGB(40,40,40)
menuFrame.Visible = false
menuFrame.Parent = screenGui
menuFrame.Active = true
menuFrame.Draggable = true

local function createButton(text, y)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 45)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 20
    btn.Parent = menuFrame
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = true
    return btn
end

local speed = false
local jump = false
local noclip = false
local antihit = false

-- Buttons
local speedBtn = createButton("Speed Boost [OFF]", 10)
local jumpBtn = createButton("Super Pulo [OFF]", 60)
local noclipBtn = createButton("Atravessar Paredes [OFF]", 110)
local antihitBtn = createButton("Anti Hit [OFF]", 160)

-- Toggle functions
speedBtn.MouseButton1Click:Connect(function()
    speed = not speed
    speedBtn.Text = "Speed Boost [" .. (speed and "ON" or "OFF") .. "]"
end)

jumpBtn.MouseButton1Click:Connect(function()
    jump = not jump
    jumpBtn.Text = "Super Pulo [" .. (jump and "ON" or "OFF") .. "]"
end)

noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipBtn.Text = "Atravessar Paredes [" .. (noclip and "ON" or "OFF") .. "]"
end)

antihitBtn.MouseButton1Click:Connect(function()
    antihit = not antihit
    antihitBtn.Text = "Anti Hit [" .. (antihit and "ON" or "OFF") .. "]"
    if antihit then
        startAntiHit()
    end
end)

-- Funções principais (loop)
local function applyEffects()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    -- Speed Boost
    if speed then
        hum.WalkSpeed = 40
    else
        hum.WalkSpeed = 16
    end
    -- Super Pulo
    if jump then
        hum.JumpPower = 120
    else
        hum.JumpPower = 50
    end
end

-- Noclip
RunService.RenderStepped:Connect(function()
    if noclip and LocalPlayer.Character then
        for _,v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") and v.CanCollide == true then
                v.CanCollide = false
            end
        end
    elseif LocalPlayer.Character then
        for _,v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") and v.CanCollide == false then
                v.CanCollide = true
            end
        end
    end
end)

-- Speed/Jump
RunService.RenderStepped:Connect(function()
    pcall(applyEffects)
end)

-- Anti Hit (novo) - só impede dano, não cura
local humConn
function startAntiHit()
    -- Desconecta conexões anteriores
    if humConn then
        humConn:Disconnect()
        humConn = nil
    end
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end

    local lastHealth = hum.Health

    humConn = hum.HealthChanged:Connect(function(newHealth)
        if antihit and newHealth < lastHealth then
            hum.Health = lastHealth -- Impede que a vida diminua ao tomar hit
        else
            lastHealth = newHealth -- Atualiza o último valor válido
        end
    end)
end

-- Garantir anti hit ao respawn
LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    applyEffects()
    if antihit then
        startAntiHit()
    end
end)

-- Abrir/fechar menu
openButton.MouseButton1Click:Connect(function()
    menuFrame.Visible = not menuFrame.Visible
end)

-- Fechar menu ao apertar ESC
UIS.InputBegan:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.Escape then
        menuFrame.Visible = false
    end
end)

-- Garantir que o menu não suma se mudar de personagem
if LocalPlayer.Character then
    applyEffects()
    if antihit then
        startAntiHit()
    end
end
