--[[
    Script educativo para testar anti-cheat em seu próprio jogo Roblox.
    Interface simples: bolinha preta para abrir menu.
    Funções: Speed Boost, Super Pulo, NoClip, Anti RIT.
    NÃO USE em jogos de terceiros!
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- GUI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
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
menuFrame.Size = UDim2.new(0, 200, 0, 220)
menuFrame.Position = UDim2.new(0, 80, 0, 120)
menuFrame.BackgroundColor3 = Color3.fromRGB(40,40,40)
menuFrame.Visible = false
menuFrame.Parent = screenGui

local function createButton(text, y)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.Parent = menuFrame
    return btn
end

local speed = false
local jump = false
local noclip = false
local antiRIT = false

-- Buttons
local speedBtn = createButton("Speed Boost [OFF]", 10)
local jumpBtn = createButton("Super Pulo [OFF]", 60)
local noclipBtn = createButton("Atravessar Paredes [OFF]", 110)
local antiRITBtn = createButton("Anti RIT [OFF]", 160)

-- Toggle functions
speedBtn.MouseButton1Click:Connect(function()
    speed = not speed
    speedBtn.Text = "Speed Boost [" .. (speed and "ON" or "OFF") .. "]"
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speed and 40 or 16
    end
end)

jumpBtn.MouseButton1Click:Connect(function()
    jump = not jump
    jumpBtn.Text = "Super Pulo [" .. (jump and "ON" or "OFF") .. "]"
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = jump and 120 or 50
    end
end)

noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipBtn.Text = "Atravessar Paredes [" .. (noclip and "ON" or "OFF") .. "]"
end)

antiRITBtn.MouseButton1Click:Connect(function()
    antiRIT = not antiRIT
    antiRITBtn.Text = "Anti RIT [" .. (antiRIT and "ON" or "OFF") .. "]"
end)

-- NoClip logic
RunService.Stepped:Connect(function()
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

-- Anti RIT dummy logic (impede kick simples)
if antiRIT then
    LocalPlayer.OnKick = function() return true end
end

-- Abrir/fechar menu
openButton.MouseButton1Click:Connect(function()
    menuFrame.Visible = not menuFrame.Visible
end)

-- Fechar menu ao apertar ESC
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Escape then
        menuFrame.Visible = false
    end
end)

-- Reset ao morrer (mantém valores)
LocalPlayer.CharacterAdded:Connect(function(char)
    if speed then char:WaitForChild("Humanoid").WalkSpeed = 40 end
    if jump then char:WaitForChild("Humanoid").JumpPower = 120 end
end)
