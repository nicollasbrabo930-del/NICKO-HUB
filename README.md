-- NICKO HUB v1.0 - Copy of Nullfire Style for Steal a Brainrot
-- Keyless | Mobile OK | Obsidian UI | Use Delta/KRNL
-- AVISO: Use alt account! Pode ban.

local repo = "https://raw.githubusercontent.com/deividcomsono/Obsidian/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

Library:CreateWindow({
    Title = "NICKO HUB | Steal a Brainrot 🔥",
    ConfigFolder = "NickoHub",
    Footer = "v1.0 | Keyless | Mobile Support",
})

local Tabs = {
    Main = Library:AddTab("Main", "rbxassetid://7733715400"),
    Player = Library:AddTab("Player", "rbxassetid://6034833295"),
    Teleport = Library:AddTab("Teleport", "rbxassetid://13416847540"),
    Visuals = Library:AddTab("Visuals", "rbxassetid://4483345998"),
    Misc = Library:AddTab("Misc", "rbxassetid://10734927860"),
}

-- Variables
getgenv().AutoSteal = false
getgenv().AutoCollect = false
getgenv().FlyEnabled = false
getgenv().NoclipEnabled = false
getgenv().InfJump = false
local Player = game.Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Functions
local function GetBrainrots()
    local brainrots = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj.Name:find("Brainrot") and obj.Parent and obj.Parent.Parent == workspace.Bases then -- Assume structure
            table.insert(brainrots, obj)
        end
    end
    return brainrots
end

local function StealBrainrot(target)
    if target and target.Parent then
        -- Assume remote or proximity
        local remote = game.ReplicatedStorage.Remotes:FindFirstChild("Steal") -- Common
        if remote then
            remote:FireServer(target)
        end
        -- Or tp and interact
        RootPart.CFrame = target.CFrame * CFrame.new(0,0,-5)
        wait(0.1)
        fireclickdetector(target:FindFirstChild("ClickDetector"))
    end
end

local function AutoStealLoop()
    spawn(function()
        while getgenv().AutoSteal do
            local brainrots = GetBrainrots()
            for _, br in pairs(brainrots) do
                if br.Owner.Value ~= Player.Name then -- Assume Owner
                    StealBrainrot(br)
                    break
                end
            end
            wait(Options.StealDelay.Value or 1)
        end
    end)
end

-- Fly Function
local BodyVelocity, BodyAngularVelocity
local function ToggleFly()
    if getgenv().FlyEnabled then
        local BV = Instance.new("BodyVelocity")
        BV.MaxForce = Vector3.new(4000, 4000, 4000)
        BV.Velocity = Vector3.new(0, 0.1, 0)
        BV.Parent = RootPart
        BodyVelocity = BV
        
        local BA = Instance.new("BodyAngularVelocity")
        BA.MaxTorque = Vector3.new(4000, 4000, 4000)
        BA.AngularVelocity = Vector3.new(0, 0, 0)
        BA.Parent = RootPart
        BodyAngularVelocity = BA
        
        spawn(function()
            while getgenv().FlyEnabled and Character.Parent do
                local cam = workspace.CurrentCamera
                local vel = Vector3.new(0,0,0)
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.W) then vel = vel + cam.CFrame.LookVector end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.S) then vel = vel - cam.CFrame.LookVector end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.A) then vel = vel - cam.CFrame.RightVector end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.D) then vel = vel + cam.CFrame.RightVector end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then vel = vel + Vector3.new(0,1,0) end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.LeftShift) then vel = vel - Vector3.new(0,1,0) end
                BodyVelocity.Velocity = vel * (Options.FlySpeed.Value or 50)
                wait()
            end
            if BodyVelocity then BodyVelocity:Destroy() end
            if BodyAngularVelocity then BodyAngularVelocity:Destroy() end
        end)
    else
        if BodyVelocity then BodyVelocity:Destroy() end
        if BodyAngularVelocity then BodyAngularVelocity:Destroy() end
    end
end

-- Noclip
local function ToggleNoclip()
    spawn(function()
        while getgenv().NoclipEnabled do
            for _, part in pairs(Character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
            wait()
        end
    end)
end

-- Infinite Jump
game:GetService("UserInputService").JumpRequest:Connect(function()
    if getgenv().InfJump then
        Humanoid:ChangeState("Jumping")
    end
end)

-- Main Tab
local MainLeft = Tabs.Main:AddLeftGroupbox("Auto Farm")

MainLeft:AddToggle("AutoStealT", {
    Text = "Auto Steal",
    Default = false,
    Callback = function(v)
        getgenv().AutoSteal = v
        if v then AutoStealLoop() end
    end,
})

MainLeft:AddSlider("StealDelay", {
    Text = "Steal Delay",
    Default = 1,
    Min = 0.1,
    Max = 5,
    Rounding = 1,
})

MainLeft:AddToggle("AutoCollectT", {
    Text = "Auto Collect",
    Default = false,
    Callback = function(v)
        getgenv().AutoCollect = v
        -- Implement collect loop
    end,
})

local MainRight = Tabs.Main:AddRightGroupbox("Steal Tools")

MainRight:AddToggle("InstantSteal", {
    Text = "Instant Steal (Aimbot)",
    Default = false,
    Callback = function(v)
        -- Loop closest
    end,
})

MainRight:AddButton({
    Text = "Steal All Visible",
    Func = function()
        local brs = GetBrainrots()
        for _, br in pairs(brs) do
            StealBrainrot(br)
        end
    end,
})

MainRight:AddDropdown("TargetPlayer", {
    Values = game.Players:GetPlayers(),
    Text = "TP & Steal from Player",
    Multi = false,
    Callback = function(p)
        if p then
            -- TP to player base
            local target = game.Players[p]
            RootPart.CFrame = target.Character.HumanoidRootPart.CFrame
        end
    end,
})

-- Player Tab
local PlayerGB = Tabs.Player:AddLeftGroupbox("Movement")

PlayerGB:AddSlider("WalkSpeed", {
    Text = "Walk Speed",
    Default = 16,
    Min = 16,
    Max = 200,
    Rounding = 0,
    Callback = function(v)
        Humanoid.WalkSpeed = v
    end,
})

PlayerGB:AddSlider("JumpPower", {
    Text = "Jump Power",
    Default = 50,
    Min = 50,
    Max = 200,
    Rounding = 0,
    Callback = function(v)
        Humanoid.JumpPower = v
    end,
})

PlayerGB:AddToggle("FlyT", {
    Text = "Fly",
    Default = false,
    Callback = function(v)
        getgenv().FlyEnabled = v
        ToggleFly()
    end,
})

PlayerGB:AddSlider("FlySpeed", {
    Text = "Fly Speed",
    Default = 50,
    Min = 10,
    Max = 200,
})

PlayerGB:AddToggle("NoclipT", {
    Text = "Noclip",
    Default = false,
    Callback = function(v)
        getgenv().NoclipEnabled = v
        ToggleNoclip()
    end,
})

PlayerGB:AddToggle("InfJumpT", {
    Text = "Infinite Jump",
    Default = false,
    Callback = function(v)
        getgenv().InfJump = v
    end,
})

-- Teleport Tab
local TPGB = Tabs.Teleport:AddLeftGroupbox("Teleports")

local tpLocations = {"Shop", "Spawn", "Leaderboard"} -- Add more CFrames

TPGB:AddDropdown("TPLoc", {
    Values = tpLocations,
    Text = "Select Location",
})

TPGB:AddButton({
    Text = "Teleport",
    Func = function()
        local loc = Options.TPLoc.Value
        -- TP to loc CFrame
        if loc == "Shop" then
            RootPart.CFrame = workspace.Shop.CFrame -- Assume
        end
    end,
})

-- Visuals
local VisualGB = Tabs.Visuals:AddLeftGroupbox("ESP")

VisualGB:AddToggle("BrainrotESP", {
    Text = "Brainrot ESP",
    Default = false,
    Callback = function(v)
        -- ESP script
    end,
})

-- Misc
local MiscGB = Tabs.Misc:AddLeftGroupbox("Misc")

MiscGB:AddButton({
    Text = "Rejoin Server",
    Func = function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
    end,
})

MiscGB:AddToggle("AntiAFK", {
    Text = "Anti AFK",
    Default = false,
    Callback = function(v)
        -- Anti afk loop
    end,
})

-- Notify on load
Library:Notify("NICKO HUB Loaded! 🔥 OP for Steal a Brainrot", 5)

SaveManager:SetLibrary(Library)
ThemeManager:SetLibrary(Library)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ "StealDelay", "FlySpeed" })
ThemeManager:SetFolder("NickoHub")
SaveManager:SetFolder("NickoHub")
Library:LoadConfig("default")

-- Character respawn
Player.CharacterAdded:Connect(function(char)
    Character = char
    Humanoid = char:WaitForChild("Humanoid")
    RootPart = char:WaitForChild("HumanoidRootPart")
end)
