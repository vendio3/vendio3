local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/jensonhirst/Orion/main/source')))()

local Window = OrionLib:MakeWindow({
    Name = "MH",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "OrionConfig"
})

local MainTab = Window:MakeTab({
    Name = "Main",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

MainTab:AddLabel("Main")

local SettingsTab = Window:MakeTab({
    Name = "Settings",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

SettingsTab:AddLabel("Auto Layout Switch Settings")

-- Control Variables
local runningLayout1, runningLayout2, runningLayout3 = false, false, false
local autoRebirth, autoDrop = false, false
local autoTeleport = false -- Master toggle control
local autoLayoutSwitch12, autoLayoutSwitch13 = false, false
local autoRegularBox, autoUnrealBox, autoInfernoBox = false, false, false
local autoPulse = false
local lastLoadedLayout = nil -- Keeps track of the last loaded layout
local layoutSwitchThreshold = 1e3 -- Default threshold value

-- Function to load a layout (prevents unnecessary reloads)
local function loadLayout(layoutName)
    if lastLoadedLayout ~= layoutName then
        game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", layoutName)
        lastLoadedLayout = layoutName
    end
end

-- Auto Layout Switching Loop (Loops Layout 1, Loads Layout 2 Once)
local function layoutSwitchLoop12()
    spawn(function()
        local loadedLayout2 = false
        while autoLayoutSwitch12 do
            local moneyValue = game:GetService("Players").LocalPlayer.PlayerGui.GUI.Money.Value
            if moneyValue >= layoutSwitchThreshold then
                if not loadedLayout2 then
                    game:GetService("ReplicatedStorage").DestroyAll:InvokeServer()
                    game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", "Layout2")
                    loadedLayout2 = true
                end
            else
                game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", "Layout1")
                loadedLayout2 = false
            end
            wait(0.001)
        end
    end)
end

-- Auto Layout Switching Loop (Loops Layout 1, Loads Layout 3 Once)
local function layoutSwitchLoop13()
    spawn(function()
        local loadedLayout3 = false
        while autoLayoutSwitch13 do
            local moneyValue = game:GetService("Players").LocalPlayer.PlayerGui.GUI.Money.Value
            if moneyValue >= layoutSwitchThreshold then
                if not loadedLayout3 then
                    game:GetService("ReplicatedStorage").DestroyAll:InvokeServer()
                    game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", "Layout3")
                    loadedLayout3 = true
                end
            else
                game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", "Layout1")
                loadedLayout3 = false
            end
            wait(0.001)
        end
    end)
end

-- Dropdown for Auto Layout Switch Threshold
local thresholdOptions = {
    "1K (Thousand)", "1M (Million)", "1B (Billion)", "1T (Trillion)", "1qd (Quadrillion)", "1Qn (Quintillion)",
    "1sx (Sextillion)", "1Sp (Septillion)", "1O (Octillion)", "1N (Nonillion)", "1de (Decillion)", "1Ud (Undecillion)",
    "1DD (Duodecillion)", "1tdD (Tredecillion)", "1qdD (Quattuordecillion)", "1QnD (Quindecillion)", "1sxD (Sedecillion)", "1SpD (Septendecillion)"
}

local thresholdValues = {
    ["1K (Thousand)"] = 1e3,
    ["1M (Million)"] = 1e6,
    ["1B (Billion)"] = 1e9,
    ["1T (Trillion)"] = 1e12,
    ["1qd (Quadrillion)"] = 1e15,
    ["1Qn (Quintillion)"] = 1e18,
    ["1sx (Sextillion)"] = 1e21,
    ["1Sp (Septillion)"] = 1e24,
    ["1O (Octillion)"] = 1e27,
    ["1N (Nonillion)"] = 1e30,
    ["1de (Decillion)"] = 1e33,
    ["1Ud (Undecillion)"] = 1e36,
    ["1DD (Duodecillion)"] = 1e39,
    ["1tdD (Tredecillion)"] = 1e42,
    ["1qdD (Quattuordecillion)"] = 1e45,
    ["1QnD (Quindecillion)"] = 1e48,
    ["1sxD (Sedecillion)"] = 1e51,
    ["1SpD (Septendecillion)"] = 1e54,
}

OrionLib:Init()


-- Dropdown for Layout Switch Threshold
SettingsTab:AddDropdown({
    Name = "Select Layout Switch (What cash value to switch)",
    Options = thresholdOptions,
    Default = "1K (Thousand)",
    Callback = function(selected)
        layoutSwitchThreshold = thresholdValues[selected] or 1e3
    end
})

-- Toggle for Auto Layout Switching (1 and 2)
SettingsTab:AddToggle({
    Name = "Auto Layout Switch (1 to 2)",
    Default = false,
    Callback = function(state)
        autoLayoutSwitch12 = state
        if state then
            layoutSwitchLoop12()
        end
    end
})

-- Toggle for Auto Layout Switching (1 and 3)
SettingsTab:AddToggle({
    Name = "Auto Layout Switch (1 to 3)",
    Default = false,
    Callback = function(state)
        autoLayoutSwitch13 = state
        if state then
            layoutSwitchLoop13()
        end
    end
})

-- Layout Toggles
local function startLayoutLoop(layoutName, runningVariable)
    spawn(function()
        while _G[runningVariable] do
            game:GetService("ReplicatedStorage").Layouts:InvokeServer("Load", layoutName)
            wait(0.001)
        end
    end)
end

MainTab:AddToggle({
    Name = "Layout 1",
    Default = false,
    Callback = function(state)
        _G.runningLayout1 = state
        if state then
            startLayoutLoop("Layout1", "runningLayout1")
        end
    end
})

MainTab:AddToggle({
    Name = "Layout 2",
    Default = false,
    Callback = function(state)
        _G.runningLayout2 = state
        if state then
            startLayoutLoop("Layout2", "runningLayout2")
        end
    end
})

MainTab:AddToggle({
    Name = "Layout 3",
    Default = false,
    Callback = function(state)
        _G.runningLayout3 = state
        if state then
            startLayoutLoop("Layout3", "runningLayout3")
        end
    end
})

-- Auto Rebirth Toggle
MainTab:AddToggle({
    Name = "Auto Rebirth",
    Default = false,
    Callback = function(state)
        autoRebirth = state
        spawn(function()
            while autoRebirth do
                game:GetService("ReplicatedStorage").Rebirth:InvokeServer(26)
                wait(0.001)
            end
        end)
    end
})

-- Auto Drop Toggle
MainTab:AddToggle({
    Name = "Auto Drop",
    Default = false,
    Callback = function(state)
        autoDrop = state
        spawn(function()
            while autoDrop do
                game:GetService("ReplicatedStorage").RemoteDrop:FireServer()
                wait(0.02)
            end
        end)
    end
})

-- Auto Regular Mystery Box Toggle
MainTab:AddToggle({
    Name = "Auto Regular Box",
    Default = false,
    Callback = function(state)
        autoRegularBox = state
        spawn(function()
            while autoRegularBox do
                game:GetService("ReplicatedStorage").MysteryBox:InvokeServer("Regular")
                wait(0.001)
            end
        end)
    end
})

-- Auto Unreal Mystery Box Toggle
MainTab:AddToggle({
    Name = "Auto Unreal Box",
    Default = false,
    Callback = function(state)
        autoUnrealBox = state
        spawn(function()
            while autoUnrealBox do
                game:GetService("ReplicatedStorage").MysteryBox:InvokeServer("Unreal")
                wait(0.001)
            end
        end)
    end
})

-- Auto Inferno Mystery Box Toggle
MainTab:AddToggle({
    Name = "Auto Inferno Box",
    Default = false,
    Callback = function(state)
        autoInfernoBox = state
        spawn(function()
            while autoInfernoBox do
                game:GetService("ReplicatedStorage").MysteryBox:InvokeServer("Inferno")
                wait(0.001)
            end
        end)
    end
})

-- Auto Pulse Toggle
MainTab:AddToggle({
    Name = "Auto Pulse",
    Default = false,
    Callback = function(state)
        autoPulse = state
        spawn(function()
            while autoPulse do
                game:GetService("ReplicatedStorage").Pulse:FireServer()
                wait(0.001)
            end
        end)
    end
})

-- Control Variables
local autoRebirth = false
local rebirthThreshold = 1e3 -- Default rebirth threshold value

-- Auto Rebirth Threshold Options
local rebirthOptions = {
    "1K (Thousand)", "1M (Million)", "1B (Billion)", "1T (Trillion)", "1qd (Quadrillion)", "1Qn (Quintillion)",
    "1sx (Sextillion)", "1Sp (Septillion)", "1O (Octillion)", "1N (Nonillion)", "1de (Decillion)", "1Ud (Undecillion)",
    "1DD (Duodecillion)", "1tdD (Tredecillion)", "1qdD (Quattuordecillion)", "1QnD (Quindecillion)", "1sxD (Sedecillion)", "1SpD (Septendecillion)",
    "1OcD (Octodecillion)", "1NvD (Novemdecillion)", "1Vgn (Vigintillion)", "1UVg (Unvigintillion)", "1DVg (Duovigintillion)", "1TVg (Tresvigintillion)",
    "1qtV (Quattuorvigintillion)", "1QnV (Quinvigintillion)", "1SeV (Sesvigintillion)", "1SPG (Septemvigintillion)", "1OVG (Octovigintillion)", "1NVG (Novemvigintillion)",
    "1TGN (Trigintillion)", "1UTG (Untrigintillion)", "1DTG (Duotrigintillion)", "1tsTG (Trestrigintillion)", "1qtTG (Quattuortrigintillion)", "1QnTG (Quintrigintillion)",
    "1ssTG (Sestrigintillion)", "1SpTG (Septentrigintillion)", "1OcTG (Octotrigintillion)", "1NoTG (Novemtrigintillion)", "1QdDR (Quadragintillion)", "1uQDR (Unquadragintillion)",
    "1dQDR (Duoquadragintillion)", "1tQDR (Tresquadragintillion)", "1qdQDR (Quattuorquadragintillion)", "1QnQDR (Quinquadragintillion)", "1sxQDR (Sesquadragintillion)", "1SpQDR (Septenquadragintillion)",
    "1OQDDr (Octoquadragintillion)", "1NQDDr (Novemquadragintillion)", "1qQGNT (Quinquagintillion)", "1uQGNT (Unquinquagintillion)", "1dQGNT (Duoquinquagintillion)", "1tQGNT (Tresquinquagintillion)",
    "1qdQGNT (Quattuorquinquagintillion)", "1QnQGNT (Quinquinquagintillion)", "1sxQGNT (Sesquinquagintillion)", "1SpQGNT (Septenquinquagintillion)", "1OQQGNT (Octoquinquagintillion)", "1NQQGNT (Novemquinquagintillion)",
    "1SXGNTL (Sexagintillion)", "1USXGNTL (Unsexagintillion)", "1DSXGNTL (Duosexagintillion)", "1TSXGNTL (Tresexagintillion)", "1QTSXGNTL (Quattuorsexagintillion)", "1QNSXGNTL (Quinsexagintillion)",
    "1SXSXGNTL (Sesexagintillion)", "1SPSXGNTL (Septensexagintillion)", "1OSXGNTL (Octosexagintillion)", "1NVSXGNTL (Novemsexagintillion)", "1SPTGNTL (Septuagintillion)", "1USPTGNTL (Unseptuagintillion)",
    "1DSPTGNTL (Duoseptuagintillion)", "1TSPTGNTL (Treseptuagintillion)", "1QTSPTGNTL (Quattuorseptuagintillion)", "1QNSPTGNTL (Quinseptuagintillion)", "1SXSPTGNTL (Seseptuagintillion)", "1SPSPTGNTL (Septenseptuagintillion)",
    "1OSPTGNTL (Octoseptuagintillion)", "1NVSPTGNTL (Novemseptuagintillion)", "1OTGNTL (Octogintillion)", "1UOTGNTL (Unoctogintillion)", "1DOTGNTL (Duooctogintillion)", "1TOTGNTL (Treoctogintillion)",
    "1QTOTGNTL (Quattuoroctogintillion)", "1QNOTGNTL (Quinoctogintillion)", "1SXOTGNTL (Sexoctogintillion)", "1SPOTGNTL (Septemoctogintillion)", "1OTOTGNTL (Octooctogintillion)", "1NVOTGNTL (Novemoctogintillion)",
    "1NONGNTL (Nonagintillion)", "1UNONGNTL (Unnonagintillion)", "1DNONGNTL (Duononagintillion)", "1TNONGNTL (Trenonagintillion)", "1QTNONGNTL (Quattuornonagintillion)", "1QNNONGNTL (Quinnonagintillion)",
    "1SXNONGNTL (Senonagintillion)", "1SPNONGNTL (Septenonagintillion)", "1OTNONGNTL (Octononagintillion)", "1NONONGNTL (Novemnonagintillion)", "1CENT (Centillion)", "1UNCENT (Uncentillion)", 
    "inf (Infinity)"
}

local rebirthValues = {}
for i, v in pairs(rebirthOptions) do
    rebirthValues[v] = 10^(3 * i)
end

SettingsTab:AddDropdown({
    Name = "Select Auto Rebirth (Only rebirth when the cash is meet",
    Options = rebirthOptions,
    Default = "1K (Thousand)",
    Callback = function(selected)
        rebirthThreshold = rebirthValues[selected] or 1e3
    end
})

-- Auto Rebirth Toggle
SettingsTab:AddToggle({
    Name = "Auto Rebirth",
    Default = false,
    Description = "Toggle Auto Rebirth based on selected threshold.",
    Callback = function(state)
        autoRebirth = state
        spawn(function()
            while autoRebirth do
                local moneyValue = game:GetService("Players").LocalPlayer.PlayerGui.GUI.Money.Value
                if moneyValue >= rebirthThreshold then
                    game:GetService("ReplicatedStorage").Rebirth:InvokeServer(1)
                end
                task.wait(0.001)
            end
        end)
    end
})

-- Auto Teleport & Stay Toggle
MainTab:AddToggle({
    Name = "Auto Teleport to Boxes & Stay",
    Default = false,
    Description = "Teleports to each box and stays at the last detected box, only looping back to the original position when no boxes are detected.",
    Callback = function(state)
        autoTeleport = state -- Update toggle state

        local player = game.Players.LocalPlayer
        local character = player.Character

        if character and character:FindFirstChild("HumanoidRootPart") then
            if state then
                teleportLoop()
            end
        end
    end
})

-- Teleport Loop Function
function teleportLoop()
    local player = game.Players.LocalPlayer
    local character = player.Character

    if character and character:FindFirstChild("HumanoidRootPart") then
        local humanoidRootPart = character.HumanoidRootPart
        local originalPosition = humanoidRootPart.CFrame -- Store original position

        while autoTeleport do -- Only run if the toggle is ON
            task.wait(0.001) -- Fast teleporting

            local boxesFolder = game.Workspace:FindFirstChild("Boxes") -- Locate the boxes
            local hasBoxes = false -- Flag to check if any boxes exist

            if boxesFolder and #boxesFolder:GetChildren() > 0 then
                for _, box in pairs(boxesFolder:GetChildren()) do
                    if not autoTeleport then return end -- Stop if toggle is turned off
                    if box:IsA("Part") or box:IsA("MeshPart") then
                        hasBoxes = true -- At least one box exists
                        -- Move the player directly onto the box (touching it)
                        humanoidRootPart.CFrame = box.CFrame + Vector3.new(0, humanoidRootPart.Size.Y / 2, 0)
                        task.wait(0.05) -- Small delay to ensure touch
                    end
                end
            end

            -- If no boxes are found, loop teleport to the original position
            if not hasBoxes then
                while autoTeleport and (not boxesFolder or #boxesFolder:GetChildren() == 0) do
                    humanoidRootPart.CFrame = originalPosition -- Continuously teleport to the original position
                    task.wait(0.1) -- Prevents excessive looping
                end
            end
        end
    end
end

-- Load OrionLib
-- egg
-- Variables
local eggFolders = {
    workspace.Easter:FindFirstChild("EASTER ISLAND EGG SPAWNS"),
    workspace.Map:FindFirstChild("EGG_SPAWNS")
}

local ESPEnabled = false
local allESPs = {}
local connections = {}

-- Function to create ESP for an egg
local function createESP(egg)
    for _, child in ipairs(egg:GetChildren()) do
        if child:IsA("BasePart") or child:IsA("Model") then
            local esp = Instance.new("BillboardGui")
            esp.Name = "EggESP"
            esp.Size = UDim2.new(0, 100, 0, 40)
            esp.AlwaysOnTop = true
            esp.StudsOffset = Vector3.new(0, 3, 0)
            esp.Parent = egg

            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, 0, 1, 0)
            label.BackgroundTransparency = 1
            label.Text = child.Name
            label.TextColor3 = Color3.fromRGB(255, 255, 0)
            label.TextStrokeTransparency = 0
            label.TextScaled = true
            label.Font = Enum.Font.SourceSansBold
            label.Parent = esp

            -- Attach to part
            if egg:IsA("Model") and egg:FindFirstChild("PrimaryPart") then
                esp.Adornee = egg.PrimaryPart
            elseif egg:IsA("BasePart") then
                esp.Adornee = egg
            elseif child:IsA("BasePart") then
                esp.Adornee = child
            end

            -- Cleanup if egg is deleted
            local conn = egg.AncestryChanged:Connect(function(_, parent)
                if not parent then
                    if esp and esp.Parent then
                        esp:Destroy()
                        table.remove(allESPs, table.find(allESPs, esp))
                    end
                end
            end)

            table.insert(allESPs, esp)
            table.insert(connections, conn)
            break
        end
    end
end

-- Enable ESP
local function enableESP()
    for _, folder in ipairs(eggFolders) do
        if folder then
            for _, egg in ipairs(folder:GetChildren()) do
                createESP(egg)
            end
            -- Listen for new eggs
            local conn = folder.ChildAdded:Connect(function(newEgg)
                task.wait(0.1)
                createESP(newEgg)
            end)
            table.insert(connections, conn)
        end
    end
end

-- Disable ESP
local function disableESP()
    for _, esp in ipairs(allESPs) do
        if esp and esp.Parent then
            esp:Destroy()
        end
    end
    allESPs = {}

    -- Disconnect all events
    for _, conn in ipairs(connections) do
        if conn then
            conn:Disconnect()
        end
    end
    connections = {}
end

-- Create a tab
local EspTab = Window:MakeTab({
    Name = "Esp Egg",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Create a Toggle
EspTab:AddToggle({
    Name = "Egg ESP",
    Default = false,
    Callback = function(Value)
        ESPEnabled = Value
        if ESPEnabled then
            enableESP()
        else
            disableESP()
        end
    end
})

OrionLib:Init()
