-- BeanzZZ Anti-Detection AutoRob for Emergency Hamburg (2026 Update)
-- Tween slower + WalkSpeed boost instead of instant TP

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid = Character:WaitForChild("Humanoid")

-- Normal WalkSpeed backup
local normalSpeed = Humanoid.WalkSpeed

LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
    Humanoid = newChar:WaitForChild("Humanoid")
    normalSpeed = Humanoid.WalkSpeed
end)

-- Safer teleport: slower tween + speed boost
local function safeTeleport(cframe)
    -- Geçici speed boost (anticheat bypass)
    Humanoid.WalkSpeed = 40  -- 50+ yapma, detection yer
    wait(0.5)
    
    local distance = (HumanoidRootPart.Position - cframe.Position).Magnitude
    local tweenTime = math.clamp(distance / 30, 4, 8)  -- Mesafeye göre 4-8 sn
    
    local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(tweenTime, Enum.EasingStyle.Linear), {CFrame = cframe})
    tween:Play()
    tween.Completed:Wait()
    
    wait(math.random(5,10)/10)  -- Random delay detection kaçsın
    Humanoid.WalkSpeed = normalSpeed
end

-- Locations (oyunda F9 ile güncelle!)
local locations = {
    dealer = CFrame.new(-100, 5, 200),
    clubDoorBehind = CFrame.new(148, 5, 102),
    clubSafe = CFrame.new(152, 5, 98),
    bankVaultFront = CFrame.new(0, 5, 0),
    bankEntrance = CFrame.new(10, 5, 20),
    bankLootPositions = {CFrame.new(5,5,5), CFrame.new(-5,5,5), CFrame.new(0,5,10)},
    jewelerSafeFront = CFrame.new(200, 5, -100),
    jewelerEntranceFar = CFrame.new(205, 5, -120),
    jewelerSafe = CFrame.new(198, 5, -98),
    parkingGarage = CFrame.new(50, -20, 10)
}

local function buyGrenade()
    safeTeleport(locations.dealer)
    wait(1.5)
    -- Remote isimlerini inspect et (genelde BuyTool veya events.Grenade)
    fireclickdetector(game:GetService("Workspace").Dealer.ClickDetector)  -- Veya FireServer
    wait(2)
end

local function throwGrenade()
    local tool = LocalPlayer.Backpack:FindFirstChild("Grenade") or Character:FindFirstChild("Grenade")
    if tool then
        Humanoid:EquipTool(tool)
        wait(1)
        mouse1press()
        wait(0.2)
        mouse1release()
        wait(4)  -- Patlama bekle
    end
end

local function collectLoot(positions)
    for _, pos in pairs(positions) do
        safeTeleport(pos)
        wait(0.8)
        mouse1click()  -- Loot al
        wait(math.random(3,8)/10)
    end
end

local function sellLoot()
    safeTeleport(locations.dealer)
    wait(2)
    -- Sell remote
    fireclickdetector(game:GetService("Workspace").Dealer.SellDetector)
    wait(1.5)
end

-- Vault open check (basit proximity)
local function isVaultOpen(pos)
    safeTeleport(pos * CFrame.new(0,0,-5))  -- Yakına git
    wait(1)
    return true  -- Gerçekte part check ekle
end

-- GUI aynı minimal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(1, -320, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Text = "BeanzZZ Anti-Detect | discord.gg/beanzhub"
Title.Parent = MainFrame
-- ... (aynı butonlar)

local autoRobEnabled = false
local AutoRobBtn = -- aynı

AutoRobBtn.MouseButton1Click:Connect(function()
    autoRobEnabled = not autoRobEnabled
    if autoRobEnabled then
        spawn(function()
            while autoRobEnabled do
                -- Club
                buyGrenade()
                safeTeleport(locations.clubDoorBehind)
                throwGrenade()
                if isVaultOpen(locations.clubSafe) then
                    collectLoot({locations.clubSafe})
                end
                sellLoot()

                -- Bank
                if isVaultOpen(locations.bankVaultFront) then
                    buyGrenade()
                    safeTeleport(locations.bankVaultFront)
                    throwGrenade()
                    safeTeleport(locations.bankEntrance)
                    collectLoot(locations.bankLootPositions)
                    sellLoot()
                end

                -- Jeweler
                if isVaultOpen(locations.jewelerSafeFront) then
                    buyGrenade()
                    safeTeleport(locations.jewelerSafeFront)
                    throwGrenade()
                    safeTeleport(locations.jewelerEntranceFar)
                    collectLoot({locations.jewelerSafe})
                    sellLoot()
                end

                safeTeleport(locations.parkingGarage)
                wait(5)

                -- Server hop low pop (max 5)
                -- Ekstra scriptle yap, TeleportService bazen çalışmaz
            end
        end)
    end
end)

-- Drag aynı
