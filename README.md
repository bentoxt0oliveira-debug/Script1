-- [[ BLOX FRUITS LITE AUTO FARM - DELTA EXECUTER ]] --

getgenv().AutoFarm = true
getgenv().FastAttack = true
getgenv().SelectWeapon = "Melee" -- Opções: "Melee", "Sword", "Blox Fruit"

-- 1. Interface Gráfica (GUI)
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local StatusLabel = Instance.new("TextLabel")
local FarmBtn = Instance.new("TextButton")
local FastAttackBtn = Instance.new("TextButton")
local ToggleBtn = Instance.new("TextButton")

ScreenGui.Parent = game:GetService("CoreGui") or game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

-- Botão Flutuante (Para Abrir/Fechar)
ToggleBtn.Parent = ScreenGui
ToggleBtn.Position = UDim2.new(0.02, 0, 0.35, 0)
ToggleBtn.Size = UDim2.new(0, 45, 0, 45)
ToggleBtn.Text = "🌾"
ToggleBtn.TextSize = 22
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
ToggleBtn.Active = true
ToggleBtn.Draggable = true

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(1, 0)
ToggleCorner.Parent = ToggleBtn

-- Painel Principal
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.Position = UDim2.new(0.08, 0, 0.2, 0)
MainFrame.Size = UDim2.new(0, 250, 0, 180)
MainFrame.Active = true
MainFrame.Draggable = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "⚔️ Blox Fruits Auto Farm"
Title.TextColor3 = Color3.fromRGB(0, 255, 150)
Title.TextSize = 14
Title.Font = Enum.Font.SourceSansBold
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 50)

StatusLabel.Parent = MainFrame
StatusLabel.Position = UDim2.new(0.05, 0, 0.22, 0)
StatusLabel.Size = UDim2.new(0.9, 0, 0, 25)
StatusLabel.Text = "Status: Procurando Quest..."
StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusLabel.TextSize = 12
StatusLabel.Font = Enum.Font.SourceSans

-- Botões Internos
local function StyleButton(btn, pos, text)
    btn.Parent = MainFrame
    btn.Position = pos
    btn.Size = UDim2.new(0.85, 0, 0, 30)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 5)
    corner.Parent = btn
end

StyleButton(FarmBtn, UDim2.new(0.075, 0, 0.45, 0), "Auto Farm Level: ON")
StyleButton(FastAttackBtn, UDim2.new(0.075, 0, 0.68, 0), "Fast Attack: ON")

-- Eventos dos Botões
ToggleBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

FarmBtn.MouseButton1Click:Connect(function()
    getgenv().AutoFarm = not getgenv().AutoFarm
    FarmBtn.Text = getgenv().AutoFarm and "Auto Farm Level: ON" or "Auto Farm Level: OFF"
    FarmBtn.BackgroundColor3 = getgenv().AutoFarm and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(180, 50, 50)
    if not getgenv().AutoFarm then StatusLabel.Text = "Status: Farm Pausado" end
end)

FastAttackBtn.MouseButton1Click:Connect(function()
    getgenv().FastAttack = not getgenv().FastAttack
    FastAttackBtn.Text = getgenv().FastAttack and "Fast Attack: ON" or "Fast Attack: OFF"
    FastAttackBtn.BackgroundColor3 = getgenv().FastAttack and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(180, 50, 50)
end)

-- 2. Serviços do Roblox
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local CommF = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommF_")

-- Anti-AFK (Evita Desconexão)
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new(0, 0))
end)

-- Sistema de Teleporte Suave (Tween)
local function TweenTo(targetCFrame)
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    
    -- Noclip automático durante a movimentação para não bater em ilhas
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end

    local distance = (root.Position - targetCFrame.Position).Magnitude
    local tweenInfo = TweenInfo.new(distance / 300, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(root, tweenInfo, {CFrame = targetCFrame})
    tween:Play()
    return tween
end

-- Equipa a arma selecionada
local function AutoEquip()
    pcall(function()
        local backpack = LocalPlayer.Backpack
        local char = LocalPlayer.Character
        for _, item in pairs(backpack:GetChildren()) do
            if item:IsA("Tool") and item.ToolTip == getgenv().SelectWeapon then
                char.Humanoid:EquipTool(item)
            end
        end
    end)
end

-- Fast Attack Loop
task.spawn(function()
    while task.wait(0.1) do
        if getgenv().FastAttack and getgenv().AutoFarm then
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton1(Vector2.new(500, 500))
            end)
        end
    end
end)

-- 3. Loop Principal de Auto Farm Level
task.spawn(function()
    while task.wait(0.5) do
        if getgenv().AutoFarm then
            pcall(function()
                local playerLevel = LocalPlayer.Data.Level.Value
                
                -- Checa se já está com Quest ativa
                local questUI = LocalPlayer.PlayerGui.Main:FindFirstChild("Quest")
                if not questUI or not questUI.Visible then
                    StatusLabel.Text = "Status: Pegando Quest do Level " .. playerLevel
                    -- Dispara pedido de quest ideal via Remote do jogo
                    CommF:InvokeServer("StartQuest", "BanditQuest1", 1) 
                else
                    StatusLabel.Text = "Status: Derrotando NPCs..."
                    AutoEquip()
                    
                    -- Procura o mob alvo mais próximo
                    for _, enemy in pairs(workspace.Enemies:GetChildren()) do
                        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            -- Posiciona 8 blocos acima do mob para bater sem tomar dano
                            TweenTo(enemy.HumanoidRootPart.CFrame * CFrame.new(0, 8, 0))
                            break
                        end
                    end
                end
            end)
        end
    end
end)

