--==================================================
-- Zaystrap Remake V4 (Enhanced Desktop & Mobile)
-- Aesthetic Green Glass Theme & Interactive Widgets
-- Made by Zay
-- v4.3 – Register/Login Auth, Profile & Settings Tabs,
--         Draggable TopBar + Z Overlay, Password Masking,
--         Custom Theme Colors, Scrollable Sidebar
--==================================================

local Players         = game:GetService("Players")
local Lighting        = game:GetService("Lighting")
local RunService      = game:GetService("RunService")
local TweenService    = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local HttpService      = game:GetService("HttpService")

local player = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--==================================================
-- CONFIG SYSTEM  (writefile / readfile safe)
--==================================================
local CONFIG_FILE = "zaystrap_v4_config.json"

local defaultConfig = {
    keyVerified  = false,
    gravStrength = 0.4,
    skyR = 135, skyG = 206, skyB = 235,
    ambR = 127, ambG = 127, ambB = 127,
    brightness = 1,
    fogEnd = 100000,
    fogR = 192, fogG = 192, fogB = 192,
    timeOfDay = "14:00:00",
    windowW = 540, windowH = 480,
    username = "Zay",
    password = "revampv4",
    key = "Zaystrap2026",
    profilePic = "rbxassetid://6033788226",
    accentR = 46, accentG = 213, accentB = 115,
    zButtonVisible = false,
    showTopBarControls = true,
    isRegistered = false,
    minimizedType = "bar",
    rememberedUsername = "",
    rememberedPassword = "",
}

local cfg = {}

local function loadConfig()
    local ok, raw = pcall(function() return readfile(CONFIG_FILE) end)
    if ok and raw and raw ~= "" then
        local ok2, data = pcall(function() return HttpService:JSONDecode(raw) end)
        if ok2 and type(data) == "table" then
            for k, v in pairs(defaultConfig) do
                cfg[k] = (data[k] ~= nil) and data[k] or v
            end
            return
        end
    end
    for k, v in pairs(defaultConfig) do cfg[k] = v end
end

local function saveConfig()
    pcall(function()
        writefile(CONFIG_FILE, HttpService:JSONEncode(cfg))
    end)
end

loadConfig()

--==================================================
-- INPUT LOGIC (Freeze & Triangle Reset)
--==================================================
local isLagging = false
local LagDuration = 0.4
local freezeMacroEnabled = false

local function causeLag(duration)
    local startTime = os.clock()
    while os.clock() - startTime < duration do
        for i = 1, 1000 do local x = i * i end
    end
end

local function triggerLag()
    if isLagging then return end
    isLagging = true
    causeLag(LagDuration)
    isLagging = false
end

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.ButtonL1 then
        if freezeMacroEnabled then
            triggerLag()
        end
    end
    if _G.ResetMacroEnabled and input.KeyCode == Enum.KeyCode.ButtonY then
        local character = player.Character
        if character then
            local hum = character:FindFirstChildOfClass("Humanoid")
            if hum then hum.Health = 0 end
        end
    end
end)

--==================================================
-- DEPENDENCIES
--==================================================
-- Replaced broken/truncated ToggleFFlag remote import with a local dummy function to prevent crashes.
local ToggleFFlag = function(...) end

--==================================================
-- FF3 ULTIMATE LOGIC (STABILIZED)
--==================================================
local ff3Enabled = false
local TIME_QUOTA = 0.0015
local ff3Physics = PhysicalProperties.new(0.7, 0.3, 0.3, 100, 100)

local function shouldSkipSimplify(obj)
    -- Skip player characters (names, numbers, outfits, accessories)
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character and obj:IsDescendantOf(p.Character) then
            return true
        end
    end
    -- Skip local camera objects (viewmodels, arms)
    if obj:IsDescendantOf(Camera) then
        return true
    end
    -- Skip game balls
    local name = obj.Name:lower()
    if name == "ball" or name == "football" or name == "tpsball" or obj:FindFirstAncestor("Ball") or obj:FindFirstAncestor("Football") or obj:FindFirstAncestor("TPSBall") then
        return true
    end
    -- Skip field, lines, goals, nets, turf, and posts
    if name:find("field") or name:find("grass") or name:find("turf") or name:find("goal") or name:find("net") or name:find("line") or name:find("post") or name:find("court") or name:find("pitch") then
        return true
    end
    -- Skip uniform models
    if obj:FindFirstAncestor("Uniform") or name == "uniform" then
        return true
    end
    return false
end

local function simplify(obj)
    if not ff3Enabled then return end
    pcall(function()
        if shouldSkipSimplify(obj) then return end
        
        if obj:IsA("BasePart") then
            -- Set to SmoothPlastic to strip native textures and disable shadow casting for performance.
            obj.Material = Enum.Material.SmoothPlastic
            obj.CastShadow = false
            if obj.CanCollide then obj.CustomPhysicalProperties = ff3Physics end
        elseif obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("SurfaceAppearance") then
            -- Destroy custom decals/textures on stadium/background parts
            obj:Destroy()
        elseif obj:IsA("MeshPart") then
            -- Clear custom mesh textures on stadium/background parts to free VRAM
            obj.TextureID = ""
            obj.CastShadow = false
            if obj.CanCollide then obj.CustomPhysicalProperties = ff3Physics end
        elseif obj:IsA("SpecialMesh") then
            obj.TextureId = ""
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then
            -- Disable particle effects for performance
            obj.Enabled = false
        elseif obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
            obj:Destroy()
        end
    end)
end

Workspace.DescendantAdded:Connect(function(newObj)
    if ff3Enabled then simplify(newObj) end
end)

--==================================================
-- THEME & UTILS
--==================================================
local theme = {
    accent       = Color3.fromRGB(cfg.accentR or 46, cfg.accentG or 213, cfg.accentB or 115),
    accent_light = Color3.fromRGB(math.min((cfg.accentR or 46) + 40, 255), math.min((cfg.accentG or 213) + 40, 255), math.min((cfg.accentB or 115) + 40, 255)),
    bg_main      = Color3.fromRGB(12, 26, 17),
    bg_sidebar   = Color3.fromRGB(8, 18, 12),
    bg_card      = Color3.fromRGB(18, 38, 25),
    text         = Color3.fromRGB(255, 255, 255),
    text_dim     = Color3.fromRGB(160, 195, 175),
    outline      = Color3.fromRGB(cfg.accentR or 46, cfg.accentG or 213, cfg.accentB or 115),
    danger       = Color3.fromRGB(255, 76, 76),
    discord      = Color3.fromRGB(88, 101, 242),
    terminal     = Color3.fromRGB(8, 16, 12),
    success      = Color3.fromRGB(cfg.accentR or 46, cfg.accentG or 213, cfg.accentB or 115)
}

local function updateThemeColors(newColor)
    local oldAccent = theme.accent
    theme.accent = newColor
    local h, s, v = newColor:ToHSV()
    theme.accent_light = Color3.fromHSV(h, s * 0.7, math.min(v * 1.3, 1))
    theme.outline = newColor
    theme.success = newColor

    cfg.accentR = math.round(newColor.R * 255)
    cfg.accentG = math.round(newColor.G * 255)
    cfg.accentB = math.round(newColor.B * 255)
    saveConfig()

    for _, v in ipairs(gui:GetDescendants()) do
        if v:IsA("TextButton") then
            if v.BackgroundColor3 == oldAccent then
                v.BackgroundColor3 = theme.accent
            end
            if v.TextColor3 == oldAccent then
                v.TextColor3 = theme.accent
            end
        elseif v:IsA("TextLabel") or v:IsA("TextBox") then
            if v.TextColor3 == oldAccent then
                v.TextColor3 = theme.accent
            end
        elseif v:IsA("UIStroke") then
            if v.Color == oldAccent then
                v.Color = theme.accent
            end
        elseif v:IsA("Frame") then
            if v.BackgroundColor3 == oldAccent then
                v.BackgroundColor3 = theme.accent
            end
        elseif v:IsA("ImageLabel") then
            if v.ImageColor3 == oldAccent then
                v.ImageColor3 = theme.accent
            end
        end
    end
    if navIndicator then navIndicator.BackgroundColor3 = theme.accent end
    if screenFps then screenFps.TextColor3 = theme.accent end
    if zButton then zButton.TextColor3 = theme.accent end
end

local function applyStyle(obj, radius, stroke, strokeColor)
    local corner = Instance.new("UICorner", obj)
    corner.CornerRadius = UDim.new(0, radius)
    if stroke then
        local s = Instance.new("UIStroke", obj)
        s.Color = strokeColor or theme.outline
        s.Thickness = 1.2
        s.Transparency = 0.45
        s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        return s
    end
end

local function applyGradient(obj, startColor, endColor)
    local g = Instance.new("UIGradient", obj)
    g.Color = ColorSequence.new(startColor, endColor)
    g.Rotation = 45
    return g
end

local function makeDraggable(frame, handle)
    handle = handle or frame
    local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
    local function update(input)
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging  = true
            dragStart = input.Position
            startPos  = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then update(input) end
    end)
end

local function makeScrollable(scrollFrame)
    scrollFrame.ScrollBarThickness = 3
    scrollFrame.ScrollBarImageColor3 = theme.accent
    scrollFrame.ScrollBarImageTransparency = 0.3
    scrollFrame.ScrollingDirection = Enum.ScrollingDirection.Y
    scrollFrame.ElasticBehavior = Enum.ElasticBehavior.Always
    local layout = scrollFrame:FindFirstChildOfClass("UIListLayout") or scrollFrame:FindFirstChildOfClass("UIGridLayout")
    if layout then
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scrollFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 15)
        end)
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 15)
    end
end

--==================================================
-- COMPONENT CREATORS
--==================================================
local function MakeInput(parent, placeholder, default, callback)
    local input = Instance.new("TextBox", parent)
    input.Size = UDim2.new(1, 0, 0, 35)
    input.BackgroundColor3 = theme.bg_sidebar
    input.BackgroundTransparency = 0.2
    input.Text = default or ""
    input.PlaceholderText = placeholder
    input.TextColor3 = theme.text
    input.PlaceholderColor3 = theme.text_dim
    input.Font = Enum.Font.GothamMedium
    input.TextSize = 13
    applyStyle(input, 6, true)
    input:GetPropertyChangedSignal("Text"):Connect(function() callback(input.Text) end)
    return input
end

local function MakeButton(parent, text, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.BackgroundColor3 = theme.accent
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 13
    btn.AutoButtonColor = false
    local stroke = applyStyle(btn, 6, true, theme.accent_light)
    stroke.Transparency = 0.5
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {BackgroundColor3 = theme.accent_light}):Play()
        TweenService:Create(stroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {BackgroundColor3 = theme.accent}):Play()
        TweenService:Create(stroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
    end)
    btn.MouseButton1Click:Connect(function() callback(btn) end)
    return btn
end

local function createToggle(parent, text, callback, startState)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(1, 0, 0, 42)
    frame.BackgroundColor3 = theme.bg_card
    frame.BackgroundTransparency = 0.25
    applyStyle(frame, 8, true)

    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -60, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = theme.text
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextWrapped = true

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 38, 0, 18)
    btn.Position = UDim2.new(1, -48, 0.5, -9)
    btn.BackgroundColor3 = theme.bg_sidebar
    btn.Text = ""
    applyStyle(btn, 9, true)

    local dot = Instance.new("Frame", btn)
    dot.Size = UDim2.new(0, 12, 0, 12)
    dot.Position = UDim2.new(0, 3, 0.5, -6)
    dot.BackgroundColor3 = theme.text_dim
    applyStyle(dot, 9, false)

    local active = startState == true
    local function refresh()
        dot.Position = active and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6)
        dot.BackgroundColor3 = active and Color3.new(1,1,1) or theme.text_dim
        btn.BackgroundColor3 = active and theme.accent or theme.bg_sidebar
    end
    refresh()

    local function setState(state)
        active = state
        TweenService:Create(dot, TweenInfo.new(0.2, Enum.EasingStyle.Quart), {
            Position = active and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6),
            BackgroundColor3 = active and Color3.new(1,1,1) or theme.text_dim
        }):Play()
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = active and theme.accent or theme.bg_sidebar
        }):Play()
    end

    btn.MouseButton1Click:Connect(function()
        active = not active
        setState(active)
        callback(active)
    end)
    return { Button = btn, SetState = setState }
end

-- Labeled section header
local function MakeSectionLabel(parent, text)
    local lbl = Instance.new("TextLabel", parent)
    lbl.Size = UDim2.new(1, 0, 0, 22)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = theme.accent
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    return lbl
end

-- Small inline number input (returns textbox)
local function MakeNumberInput(parent, label, defaultVal, minVal, maxVal, onChanged)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 36)
    row.BackgroundColor3 = theme.bg_card
    row.BackgroundTransparency = 0.3
    applyStyle(row, 6, true)

    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(0.6, -4, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = theme.text
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextWrapped = true

    local box = Instance.new("TextBox", row)
    box.Size = UDim2.new(0.35, 0, 0.72, 0)
    box.Position = UDim2.new(0.63, 0, 0.14, 0)
    box.BackgroundColor3 = theme.bg_sidebar
    box.Text = tostring(defaultVal)
    box.TextColor3 = theme.accent
    box.Font = Enum.Font.GothamMedium
    box.TextSize = 12
    box.ClearTextOnFocus = false
    applyStyle(box, 4, true)

    box.FocusLost:Connect(function()
        local v = tonumber(box.Text)
        if v then
            if minVal then v = math.max(minVal, v) end
            if maxVal then v = math.min(maxVal, v) end
            box.Text = tostring(v)
            onChanged(v)
        else
            box.Text = tostring(defaultVal)
        end
    end)
    return box
end

-- Color row (R, G, B sliders)
local function MakeColorRow(parent, label, r, g, b, onChange)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 95)
    row.BackgroundColor3 = theme.bg_card
    row.BackgroundTransparency = 0.3
    applyStyle(row, 6, true)

    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -10, 0, 18)
    lbl.Position = UDim2.new(0, 10, 0, 4)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = theme.text_dim
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local preview = Instance.new("Frame", row)
    preview.Size = UDim2.new(0, 14, 0, 14)
    preview.Position = UDim2.new(1, -22, 0, 5)
    preview.BackgroundColor3 = Color3.fromRGB(r, g, b)
    preview.BorderSizePixel = 0
    applyStyle(preview, 3, false)

    local vals = {r, g, b}
    local sliderColors = {
        Color3.fromRGB(255, 76, 76), -- Red
        Color3.fromRGB(46, 213, 115), -- Green
        Color3.fromRGB(88, 101, 242) -- Blue
    }
    local labels = {"R", "G", "B"}
    
    for i = 1, 3 do
        local yPos = 22 + (i - 1) * 23
        
        local sLabel = Instance.new("TextLabel", row)
        sLabel.Size = UDim2.new(0, 12, 0, 16)
        sLabel.Position = UDim2.new(0, 10, 0, yPos)
        sLabel.BackgroundTransparency = 1
        sLabel.Text = labels[i]
        sLabel.TextColor3 = theme.text_dim
        sLabel.Font = Enum.Font.GothamBold
        sLabel.TextSize = 10
        sLabel.TextXAlignment = Enum.TextXAlignment.Left
        
        local track = Instance.new("Frame", row)
        track.Size = UDim2.new(1, -70, 0, 6)
        track.Position = UDim2.new(0, 28, 0, yPos + 5)
        track.BackgroundColor3 = theme.bg_sidebar
        track.BorderSizePixel = 0
        applyStyle(track, 3, false)
        
        local fill = Instance.new("Frame", track)
        fill.Size = UDim2.new(vals[i] / 255, 0, 1, 0)
        fill.BackgroundColor3 = sliderColors[i]
        fill.BorderSizePixel = 0
        applyStyle(fill, 3, false)
        
        local knob = Instance.new("TextButton", track)
        knob.Size = UDim2.new(0, 12, 0, 12)
        knob.Position = UDim2.new(vals[i] / 255, -6, 0.5, -6)
        knob.BackgroundColor3 = Color3.new(1, 1, 1)
        knob.Text = ""
        knob.AutoButtonColor = false
        applyStyle(knob, 6, true, theme.accent)
        
        local valLabel = Instance.new("TextLabel", row)
        valLabel.Size = UDim2.new(0, 28, 0, 16)
        valLabel.Position = UDim2.new(1, -34, 0, yPos)
        valLabel.BackgroundTransparency = 1
        valLabel.Text = tostring(vals[i])
        valLabel.TextColor3 = theme.text_dim
        valLabel.Font = Enum.Font.Code
        valLabel.TextSize = 10
        valLabel.TextXAlignment = Enum.TextXAlignment.Right
        
        local function updateVal(inputX)
            local trackAbsSize = track.AbsoluteSize.X
            local trackAbsPos = track.AbsolutePosition.X
            if trackAbsSize == 0 then return end
            local relX = math.clamp(inputX - trackAbsPos, 0, trackAbsSize)
            local pct = relX / trackAbsSize
            local val = math.round(pct * 255)
            
            vals[i] = val
            valLabel.Text = tostring(val)
            fill.Size = UDim2.new(pct, 0, 1, 0)
            knob.Position = UDim2.new(pct, -6, 0.5, -6)
            
            local newColor = Color3.fromRGB(vals[1], vals[2], vals[3])
            preview.BackgroundColor3 = newColor
            onChange(vals[1], vals[2], vals[3])
        end
        
        local activeDrag = false
        knob.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                activeDrag = true
                updateVal(input.Position.X)
            end
        end)
        
        UserInputService.InputChanged:Connect(function(input)
            if activeDrag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                updateVal(input.Position.X)
            end
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                activeDrag = false
            end
        end)
        
        track.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                activeDrag = true
                updateVal(input.Position.X)
            end
        end)
    end
    return row, {}
end

local FlagDocs = {
    ["TextureQualityOverride"] = "Forces textures to a specific resolution (0 is highest).",
    ["TaskSchedulerTargetFps"] = "Sets the internal engine thread frame rate limit.",
    ["DebugDisplayFPS"]        = "Toggles the built-in FPS counter overlay.",
    ["S2PhysicsSenderRate"]    = "Controls how often physics data is sent to the server.",
    ["RakNetResendRttMultiple"]= "Adjusts network packet resend timing to reduce lag.",
    ["MaxProcessPacketsSteps"] = "Optimizes engine packet processing speed to prevent desync.",
    ["TerrainArraySliceSize"]  = "Changes how terrain is loaded; 0 often disables terrain for FPS.",
    ["DebugSkyGray"]           = "Removes sky textures to reduce GPU load.",
    ["DisablePostFx"]          = "Disables Bloom, Blur, and Sunrays for performance.",
    ["ConnectionMTUSize"]      = "Adjusts maximum packet size for network stability."
}

-- ANTI-CHEAT SAFE PHYSICS ENGINE
local stealthPhysics = { jumpVelocity = 0, active = false }

UserInputService.JumpRequest:Connect(function()
    if stealthPhysics.active and stealthPhysics.jumpVelocity > 0 then
        local ch = player.Character
        if ch then
            local hrp2 = ch:FindFirstChild("HumanoidRootPart")
            local hum2 = ch:FindFirstChildOfClass("Humanoid")
            local state = hum2 and hum2:GetState()
            if hrp2 and hum2 and (state == Enum.HumanoidStateType.Running or state == Enum.HumanoidStateType.RunningNoPhysics) then
                hum2:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
                hrp2.CFrame = hrp2.CFrame + Vector3.new(0, 0.5, 0)
                hrp2.AssemblyLinearVelocity = Vector3.new(hrp2.AssemblyLinearVelocity.X, stealthPhysics.jumpVelocity, hrp2.AssemblyLinearVelocity.Z)
                task.delay(0.2, function()
                    if hum2 then hum2:SetStateEnabled(Enum.HumanoidStateType.Jumping, true) end
                end)
            end
        end
    end
end)

local function coerceValue(flagName, value)
    local lowerName = flagName:lower()
    local valStr = tostring(value):lower()
    
    if lowerName:find("flag") or lowerName:find("bool") then
        if valStr == "true" or valStr == "1" or value == true or value == 1 then
            return true
        elseif valStr == "false" or valStr == "0" or value == false or value == 0 then
            return false
        end
    elseif lowerName:find("int") or lowerName:find("double") or lowerName:find("float") or lowerName:find("num") then
        local num = tonumber(value)
        if num then
            if lowerName:find("int") then
                return math.round(num)
            end
            return num
        end
    end
    return value
end

local function trySetFlag(func, name, value)
    local coerced = coerceValue(name, value)
    local success = pcall(function() func(name, coerced) end)
    if success then return true end
    
    if coerced ~= value then
        success = pcall(function() func(name, value) end)
        if success then return true end
    end
    
    local strVal = tostring(value)
    if strVal ~= tostring(coerced) then
        success = pcall(function() func(name, strVal) end)
        if success then return true end
    end
    
    return false
end

local function trySetFlagAll(func, f, stripped, v)
    if trySetFlag(func, f, v) then return true end
    if trySetFlag(func, stripped, v) then return true end
    return false
end

local function TFF(f, v)
    local stripped = f:gsub("^D?F[IS][nt][tr][in]*g?", "")
    pcall(function()
        if setthreadidentity then setthreadidentity(8)
        elseif setidentity then setidentity(8) end
    end)
    local lowerFlag = stripped:lower()
    
    -- Only intercept actual jump customized flags (e.g. FFlagUserJumpPower, FFlagJumpVelocity)
    -- This prevents standard flags like FFlagJumpDelay from disabling normal jumps.
    if lowerFlag:find("jumppower") or lowerFlag:find("jumpvelocity") or lowerFlag:find("jumpheight") or lowerFlag:find("jumpboost") or lowerFlag == "stealthjump" then
        local val = tonumber(v)
        if val then
            if val <= 0 then
                stealthPhysics.active = false
                stealthPhysics.jumpVelocity = 0
            else
                stealthPhysics.active = true
                stealthPhysics.jumpVelocity = lowerFlag:find("multiplier") and 50 * val or val
            end
        else
            -- If value is boolean false or string "false"/"nil"/"none", disable stealth jump
            if v == false or tostring(v):lower() == "false" or tostring(v):lower() == "nil" or tostring(v):lower() == "none" then
                stealthPhysics.active = false
                stealthPhysics.jumpVelocity = 0
            end
        end
    end
    
    local successState = 0
    for _, n in ipairs({setpatchedfflag, setpfflag}) do
        if n and trySetFlagAll(n, f, stripped, v) then
            successState = 2; break
        end
    end
    if successState == 0 then
        for _, n in ipairs({setfflag, setfastflag}) do
            if n and trySetFlagAll(n, f, stripped, v) then
                successState = 1; break
            end
        end
    end
    return successState == 0 and 1 or successState
end

--==================================================
-- MOBILE DETECTION HELPER
--==================================================
local function isMobileDevice()
    local screenSize = Camera.ViewportSize
    if screenSize.X == 0 or screenSize.Y == 0 then screenSize = Vector2.new(1280, 720) end
    return UserInputService.TouchEnabled or screenSize.X < 700
end

--==================================================
-- ROOT UI
--==================================================
local gui = Instance.new("ScreenGui")
gui.Name = "Zaystrap Remake V4"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = player:WaitForChild("PlayerGui")

--==================================================
-- MAIN FRAME + LAYOUT
--==================================================
local main = Instance.new("Frame", gui)
main.Position    = UDim2.new(0.5, 0, 0.5, 0)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = Color3.new(1, 1, 1)
main.BackgroundTransparency = 0.15
main.ClipsDescendants = true
main.Visible = false
main.Active  = true
applyStyle(main, 12, true)
applyGradient(main, theme.bg_main, Color3.fromRGB(8, 18, 12))
makeDraggable(main)

local sidebar   = Instance.new("Frame", main)
sidebar.BackgroundColor3 = Color3.new(1, 1, 1)
sidebar.BackgroundTransparency = 0.1
sidebar.ClipsDescendants = true
applyStyle(sidebar, 12, false)
applyGradient(sidebar, theme.bg_sidebar, Color3.fromRGB(5, 12, 8))

local container = Instance.new("Frame", main)
container.BackgroundTransparency = 1

local fGl  -- font grid layout ref

local MIN_W, MIN_H = 400, 340
local curW = math.max(cfg.windowW or 540, MIN_W)
local curH = math.max(cfg.windowH or 480, MIN_H)

local function getTargetSize()
    local screenSize = Camera.ViewportSize
    if screenSize.X == 0 or screenSize.Y == 0 then screenSize = Vector2.new(1280, 720) end
    local mobile = isMobileDevice()
    if mobile then
        local w = math.clamp(screenSize.X * 0.95, 290, 480)
        local h = math.clamp(screenSize.Y * 0.88, 260, 420)
        return UDim2.new(0, w, 0, h)
    else
        return UDim2.new(0, curW, 0, curH)
    end
end

local function layoutUI()
    local screenSize = Camera.ViewportSize
    if screenSize.X == 0 or screenSize.Y == 0 then screenSize = Vector2.new(1280, 720) end
    local mobile = isMobileDevice()
    local targetSize = getTargetSize()
    main.Size = targetSize
    local sidebarW = mobile and 52 or 58
    local padding  = mobile and 6 or 12
    sidebar.Size    = UDim2.new(0, sidebarW, 1, 0)
    container.Size  = UDim2.new(1, -sidebarW - (padding * 2), 1, -36)
    container.Position = UDim2.new(0, sidebarW + padding, 0, 28)
    if fGl then
        if mobile then
            fGl.CellSize = UDim2.new(0.95, 0, 0, 28)
            fGl.CellPadding = UDim2.new(0, 0, 0, 5)
        else
            fGl.CellSize = UDim2.new(0.48, 0, 0, 32)
            fGl.CellPadding = UDim2.new(0, 8, 0, 6)
        end
    end
end

Camera:GetPropertyChangedSignal("ViewportSize"):Connect(layoutUI)

--==================================================
-- RESIZE HANDLE (bottom-right corner drag)
--==================================================
local resizeHandle = Instance.new("TextButton", main)
resizeHandle.Size = UDim2.new(0, 18, 0, 18)
resizeHandle.Position = UDim2.new(1, -18, 1, -18)
resizeHandle.BackgroundColor3 = theme.accent
resizeHandle.BackgroundTransparency = 0.4
resizeHandle.Text = ""
resizeHandle.ZIndex = 20
resizeHandle.AutoButtonColor = false
applyStyle(resizeHandle, 4, false)

-- Small grip icon
local gripLabel = Instance.new("TextLabel", resizeHandle)
gripLabel.Size = UDim2.new(1, 0, 1, 0)
gripLabel.BackgroundTransparency = 1
gripLabel.Text = "⌟"
gripLabel.TextColor3 = theme.accent_light
gripLabel.Font = Enum.Font.GothamBold
gripLabel.TextSize = 14
gripLabel.ZIndex = 21

resizeHandle.MouseEnter:Connect(function()
    TweenService:Create(resizeHandle, TweenInfo.new(0.15), {BackgroundTransparency = 0.1}):Play()
end)
resizeHandle.MouseLeave:Connect(function()
    TweenService:Create(resizeHandle, TweenInfo.new(0.15), {BackgroundTransparency = 0.4}):Play()
end)

do
    local resizing = false
    local resizeStart, startSize, startPos2

    resizeHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            resizing    = true
            resizeStart = input.Position
            startSize   = Vector2.new(main.AbsoluteSize.X, main.AbsoluteSize.Y)
            startPos2   = main.AbsolutePosition
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then resizing = false end
            end)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not resizing then return end
        if input.UserInputType ~= Enum.UserInputType.MouseMovement and
           input.UserInputType ~= Enum.UserInputType.Touch then return end

        local delta = input.Position - resizeStart
        local newW = math.max(MIN_W, startSize.X + delta.X)
        local newH = math.max(MIN_H, startSize.Y + delta.Y)

        curW = newW
        curH = newH
        cfg.windowW = newW
        cfg.windowH = newH

        main.Size = UDim2.new(0, newW, 0, newH)
        layoutUI()
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            if resizing then
                resizing = false
                saveConfig()
            end
        end
    end)
end

--==================================================
-- FLOATING WIDGETS (No Z button — only topBar)
--==================================================
-- The top bar with − and × is now DRAGGABLE and the only
-- minimized widget (the big Z button has been removed).
local topBar = Instance.new("Frame", gui)
topBar.Size  = UDim2.new(0, 150, 0, 36)
topBar.Position = UDim2.new(0.5, -75, 0, -50)
topBar.BackgroundColor3 = Color3.new(1, 1, 1)
topBar.BackgroundTransparency = 0.25
topBar.Visible = false
topBar.Active  = true
applyStyle(topBar, 8, true)
applyGradient(topBar, theme.bg_main, Color3.fromRGB(8, 18, 12))

local topDragHandle = Instance.new("TextLabel", topBar)
topDragHandle.Size = UDim2.new(0, 70, 1, 0)
topDragHandle.Position = UDim2.new(0, 40, 0, 0)
topDragHandle.BackgroundTransparency = 1
topDragHandle.Text = "⠿⠿"
topDragHandle.TextColor3 = theme.text_dim
topDragHandle.TextTransparency = 0.3
topDragHandle.Font = Enum.Font.GothamBold
topDragHandle.TextSize = 14
topDragHandle.Active = true

makeDraggable(topBar, topDragHandle) -- DRAGGABLE via the center handle!

local topMini = Instance.new("TextButton", topBar)
topMini.Size  = UDim2.new(0, 35, 1, 0)
topMini.Position = UDim2.new(0, 5, 0, 0)
topMini.BackgroundTransparency = 1
topMini.Text  = "−"
topMini.TextColor3 = theme.accent
topMini.TextSize = 22
topMini.Font  = Enum.Font.GothamBold

local topClose = Instance.new("TextButton", topBar)
topClose.Size  = UDim2.new(0, 35, 1, 0)
topClose.Position = UDim2.new(1, -40, 0, 0)
topClose.BackgroundTransparency = 1
topClose.Text  = "×"
topClose.TextColor3 = theme.danger
topClose.TextSize = 22
topClose.Font  = Enum.Font.GothamBold

local function setupWidgetHover(btn, normalColor, hoverColor)
    btn.MouseEnter:Connect(function() TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = hoverColor}):Play() end)
    btn.MouseLeave:Connect(function() TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = normalColor}):Play() end)
end
setupWidgetHover(topMini, theme.accent, theme.accent_light)
setupWidgetHover(topClose, theme.danger, Color3.fromRGB(255, 120, 120))

--==================================================
-- KEY SYSTEM  (persistent via config)
--==================================================
local masterKey = "Zaystrap2026"
local keyVerified = cfg.keyVerified == true
local isRegistered = cfg.isRegistered == true

local keyFrame = Instance.new("Frame", gui)
keyFrame.Size  = UDim2.new(0, 320, 0, 225) -- default size for Login Mode, will resize dynamically
keyFrame.Position  = UDim2.new(0.5, 0, 0.5, 0)
keyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
keyFrame.BackgroundColor3 = Color3.new(1, 1, 1)
keyFrame.BackgroundTransparency = 0.15
keyFrame.BorderSizePixel = 0
keyFrame.Active  = true
applyStyle(keyFrame, 10, true)
applyGradient(keyFrame, theme.bg_main, Color3.fromRGB(8, 18, 12))
makeDraggable(keyFrame)

local keyTitle = Instance.new("TextLabel", keyFrame)
keyTitle.Size  = UDim2.new(1, -10, 0, 45)
keyTitle.Position = UDim2.new(0, 5, 0, 4)
keyTitle.BackgroundTransparency = 1
keyTitle.RichText = true
keyTitle.TextWrapped = true
keyTitle.TextColor3 = theme.text
keyTitle.Font  = Enum.Font.GothamBold
keyTitle.TextSize = 14
keyTitle.TextXAlignment = Enum.TextXAlignment.Center
keyTitle.TextYAlignment = Enum.TextYAlignment.Center

local userInput = Instance.new("TextBox", keyFrame)
userInput.Size  = UDim2.new(0.85, 0, 0, 30)
userInput.AnchorPoint = Vector2.new(0.5, 0.5)
userInput.BackgroundColor3 = theme.bg_sidebar
userInput.BackgroundTransparency = 0.2
userInput.Text  = cfg.rememberedUsername or ""
userInput.PlaceholderText = "Username..."
userInput.TextColor3 = theme.text
userInput.PlaceholderColor3 = theme.text_dim
userInput.Font  = Enum.Font.GothamMedium
userInput.TextSize = 12
applyStyle(userInput, 6, true)

local passInput = Instance.new("TextBox", keyFrame)
passInput.Size  = UDim2.new(0.72, 0, 0, 30)
passInput.AnchorPoint = Vector2.new(0, 0.5)
passInput.BackgroundColor3 = theme.bg_sidebar
passInput.BackgroundTransparency = 0.2
passInput.Text  = cfg.rememberedPassword or ""
passInput.PlaceholderText = "Password..."
passInput.TextColor3 = Color3.new(0,0,0,0) -- start invisible (mask overlay covers it)
passInput.PlaceholderColor3 = theme.text_dim
passInput.Font  = Enum.Font.GothamMedium
passInput.TextSize = 12
passInput.ClearTextOnFocus = false
applyStyle(passInput, 6, true)

-- Bullet overlay: sits on top of passInput, shows '•' chars, click-through to input
local passMaskLabel = Instance.new("TextLabel", keyFrame)
passMaskLabel.Size  = UDim2.new(0.72, -12, 0, 30) -- slightly inset so it doesn't block edges
passMaskLabel.AnchorPoint = Vector2.new(0, 0.5)
passMaskLabel.BackgroundTransparency = 1
passMaskLabel.TextColor3 = theme.text
passMaskLabel.Font  = Enum.Font.GothamMedium
passMaskLabel.TextSize = 12
passMaskLabel.TextXAlignment = Enum.TextXAlignment.Left
passMaskLabel.ZIndex = 5
passMaskLabel.Text = string.rep("•", #passInput.Text)

-- Keep bullet count in sync with real text length (no logic, just count)
passInput:GetPropertyChangedSignal("Text"):Connect(function()
    passMaskLabel.Text = string.rep("•", #passInput.Text)
end)

-- Password reveal toggle button
local passRevealBtn = Instance.new("TextButton", keyFrame)
passRevealBtn.Size = UDim2.new(0, 28, 0, 28)
passRevealBtn.AnchorPoint = Vector2.new(1, 0.5)
passRevealBtn.BackgroundColor3 = theme.bg_sidebar
passRevealBtn.BackgroundTransparency = 0.2
passRevealBtn.Text = "👁"
passRevealBtn.TextSize = 13
passRevealBtn.Font = Enum.Font.GothamBold
passRevealBtn.TextColor3 = theme.text_dim
passRevealBtn.AutoButtonColor = false
applyStyle(passRevealBtn, 6, true)

local passRevealed = false
passRevealBtn.MouseButton1Click:Connect(function()
    passRevealed = not passRevealed
    if passRevealed then
        passMaskLabel.Visible = false
        passInput.TextColor3 = theme.text
        passRevealBtn.TextColor3 = theme.accent
        passRevealBtn.Text = "🙈"
    else
        passMaskLabel.Visible = true
        passInput.TextColor3 = Color3.new(0,0,0,0)
        passRevealBtn.TextColor3 = theme.text_dim
        passRevealBtn.Text = "👁"
    end
end)


local keyInput = Instance.new("TextBox", keyFrame)
keyInput.Size  = UDim2.new(0.85, 0, 0, 30)
keyInput.AnchorPoint = Vector2.new(0.5, 0.5)
keyInput.BackgroundColor3 = theme.bg_sidebar
keyInput.BackgroundTransparency = 0.2
keyInput.Text  = ""
keyInput.PlaceholderText = "Master Key..."
keyInput.TextColor3 = theme.text
keyInput.PlaceholderColor3 = theme.text_dim
keyInput.Font  = Enum.Font.GothamMedium
keyInput.TextSize = 12
applyStyle(keyInput, 6, true)

-- Checkbox: remember key
local rememberRow = Instance.new("Frame", keyFrame)
rememberRow.Size  = UDim2.new(0.85, 0, 0, 24)
rememberRow.AnchorPoint = Vector2.new(0.5, 0.5)
rememberRow.BackgroundTransparency = 1

local rememberBox = Instance.new("TextButton", rememberRow)
rememberBox.Size  = UDim2.new(0, 20, 0, 20)
rememberBox.BackgroundColor3 = theme.bg_sidebar
rememberBox.Text  = ""
rememberBox.AutoButtonColor = false
applyStyle(rememberBox, 4, true)

local rememberCheck = Instance.new("TextLabel", rememberBox)
rememberCheck.Size  = UDim2.new(1, 0, 1, 0)
rememberCheck.BackgroundTransparency = 1
rememberCheck.Text  = "✓"
rememberCheck.TextColor3 = theme.accent
rememberCheck.Font  = Enum.Font.GothamBold
rememberCheck.TextSize = 13
rememberCheck.Visible = true -- default: remember ON

local rememberLbl = Instance.new("TextLabel", rememberRow)
rememberLbl.Size  = UDim2.new(1, -28, 1, 0)
rememberLbl.Position = UDim2.new(0, 26, 0, 0)
rememberLbl.BackgroundTransparency = 1
rememberLbl.Text  = "Remember credentials"
rememberLbl.TextColor3 = theme.text_dim
rememberLbl.Font  = Enum.Font.GothamMedium
rememberLbl.TextSize = 12
rememberLbl.TextXAlignment = Enum.TextXAlignment.Left

local rememberEnabled = true
rememberBox.MouseButton1Click:Connect(function()
    rememberEnabled = not rememberEnabled
    rememberCheck.Visible = rememberEnabled
    TweenService:Create(rememberBox, TweenInfo.new(0.15), {
        BackgroundColor3 = rememberEnabled and theme.bg_card or theme.bg_sidebar
    }):Play()
end)

local keySubmit = Instance.new("TextButton", keyFrame)
keySubmit.AnchorPoint = Vector2.new(0.5, 0.5)
keySubmit.BackgroundColor3 = theme.accent
keySubmit.TextColor3 = Color3.new(1, 1, 1)
keySubmit.Font  = Enum.Font.GothamBold
keySubmit.TextSize = 13
keySubmit.AutoButtonColor = false
local keySubmitStroke = applyStyle(keySubmit, 6, true, theme.accent_light)
keySubmitStroke.Transparency = 0.5

keySubmit.MouseEnter:Connect(function()
    TweenService:Create(keySubmit, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent_light}):Play()
    TweenService:Create(keySubmitStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
end)
keySubmit.MouseLeave:Connect(function()
    TweenService:Create(keySubmit, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent}):Play()
    TweenService:Create(keySubmitStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
end)

local credsTooltip = Instance.new("TextButton", keyFrame)
credsTooltip.AnchorPoint = Vector2.new(0.5, 0.5)
credsTooltip.BackgroundTransparency = 1
credsTooltip.Font  = Enum.Font.GothamMedium
credsTooltip.TextSize = 9

-- Layout now also positions passRevealBtn alongside passInput
local function drawAuthScreen()
    local screenSize = Camera.ViewportSize
    if screenSize.X == 0 or screenSize.Y == 0 then screenSize = Vector2.new(1280, 720) end
    local mobile = isMobileDevice()

    if not isRegistered then
        -- Registration Mode
        keyFrame.Size = mobile and UDim2.new(0, math.min(290, screenSize.X * 0.95), 0, 275) or UDim2.new(0, 320, 0, 280)
        keyTitle.Text = "REGISTER NEW CLIENT\n<font size='10' color='#A0C3AF'>made by zay</font>"

        userInput.Position = UDim2.new(0.5, 0, 0.28, 0)
        passInput.Position = UDim2.new(0.08, 0, 0.42, 0)
        passRevealBtn.Position = UDim2.new(0.85, 0, 0.42, 0)
        passMaskLabel.Position = UDim2.new(0.09, 0, 0.42, 0)
        keyInput.Position  = UDim2.new(0.5, 0, 0.56, 0)
        keyInput.Visible   = true

        rememberRow.Position = UDim2.new(0.5, 0, 0.70, 0)
        keySubmit.Size       = UDim2.new(0.85, 0, 0, 30)
        keySubmit.Position   = UDim2.new(0.5, 0, 0.83, 0)
        keySubmit.Text       = "REGISTER ACCOUNT"

        credsTooltip.Size    = UDim2.new(0.85, 0, 0, 15)
        credsTooltip.Position = UDim2.new(0.5, 0, 0.94, 0)
        credsTooltip.Text    = "Enter Master Key to register: Zaystrap2026"
        credsTooltip.TextColor3 = theme.text_dim
    else
        -- Login Mode
        keyFrame.Size = mobile and UDim2.new(0, math.min(290, screenSize.X * 0.95), 0, 180) or UDim2.new(0, 320, 0, 185)
        keyTitle.Text = "CLIENT LOG IN\n<font size='10' color='#A0C3AF'>made by zay</font>"

        userInput.Position = UDim2.new(0.5, 0, 0.32, 0)
        passInput.Position = UDim2.new(0.08, 0, 0.50, 0)
        passRevealBtn.Position = UDim2.new(0.85, 0, 0.50, 0)
        passMaskLabel.Position = UDim2.new(0.09, 0, 0.50, 0)
        keyInput.Visible   = false

        rememberRow.Position = UDim2.new(0.5, 0, 0.67, 0)
        keySubmit.Size       = UDim2.new(0.85, 0, 0, 30)
        keySubmit.Position   = UDim2.new(0.5, 0, 0.82, 0)
        keySubmit.Text       = "LOG IN"

        credsTooltip.Size    = UDim2.new(0.85, 0, 0, 15)
        credsTooltip.Position = UDim2.new(0.5, 0, 0.93, 0)
        credsTooltip.Text    = "Reset Account / Re-register"
        credsTooltip.TextColor3 = theme.accent
    end
end


local function layoutKeyFrame()
    drawAuthScreen()
end
Camera:GetPropertyChangedSignal("ViewportSize"):Connect(layoutKeyFrame)
drawAuthScreen()

credsTooltip.MouseButton1Click:Connect(function()
    if isRegistered then
        isRegistered = false
        cfg.isRegistered = false
        saveConfig()
        drawAuthScreen()
    end
end)

--==================================================
-- UI TRANSITIONS (With customizable Z button & controls)
--==================================================
local activePage = nil

local zButton = Instance.new("TextButton", gui)
zButton.Size = UDim2.new(0, 50, 0, 50)
zButton.Position = UDim2.new(0.05, 0, 0.8, 0)
zButton.BackgroundColor3 = Color3.fromRGB(8, 18, 12)
zButton.BackgroundTransparency = 0.2
zButton.Text = "Z"
zButton.TextColor3 = theme.accent
zButton.Font = Enum.Font.GothamBold
zButton.TextSize = 22
zButton.Visible = false
zButton.Active = true
zButton.ZIndex = 100
applyStyle(zButton, 25, true)
applyGradient(zButton, theme.bg_sidebar, Color3.fromRGB(5, 12, 8))
makeDraggable(zButton)

local function toggleUI(state)
    if state == "close_to_widgets" or state == "hide" then
        local shrinkTween = TweenService:Create(main, TweenInfo.new(0.35, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
            Size = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 1
        })
        shrinkTween:Play()
        shrinkTween.Completed:Connect(function()
            main.Visible = false
            if cfg.minimizedType == "z" then
                zButton.Visible = true
                topBar.Visible = false
            else
                -- In bar mode, hide Z overlay (the bar takes over)
                zButton.Visible = false
                topBar.Visible = true
                topBar.Position = UDim2.new(0.5, -75, 0, -50)
                TweenService:Create(topBar, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0.5, -75, 0, 85)}):Play()
            end
        end)
    elseif state == "restore" or state == "show" then
        -- Hide minimized widgets first
        zButton.Visible = false
        TweenService:Create(topBar, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(0.5, -75, 0, -50)}):Play()
        task.delay(0.25, function()
            topBar.Visible = false
        end)
        main.Visible = true
        main.Size    = UDim2.new(0, 0, 0, 0)
        main.BackgroundTransparency = 0.15
        layoutUI()
        local restoredSize = main.Size
        main.Size = UDim2.new(0, 0, 0, 0)
        local restoreTween = TweenService:Create(main, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = restoredSize})
        restoreTween:Play()
        -- After restore animation, re-show persistent Z overlay if enabled
        restoreTween.Completed:Connect(function()
            if cfg.zButtonVisible and main.Visible then
                zButton.Visible = true
            end
        end)
    elseif state == "destroy" or state == "close" then
        zButton.Visible = false
        TweenService:Create(main, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
        TweenService:Create(topBar, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(0.5,-75,0,-50)}):Play()
        task.wait(0.25)
        gui:Destroy()
    end
end


zButton.MouseButton1Click:Connect(function()
    if main.Visible then
        -- If main window is open, Z button acts as a quick-minimize
        toggleUI("close_to_widgets")
    else
        zButton.Visible = false
        toggleUI("restore")
    end
end)


-- topMini (−) restores the main window, topClose (×) destroys everything
topMini.MouseButton1Click:Connect(function() toggleUI("restore") end)
topClose.MouseButton1Click:Connect(function() toggleUI("destroy") end)

local topBarBodyBtn = Instance.new("TextButton", topBar)
topBarBodyBtn.Size = UDim2.new(1, 0, 1, 0)
topBarBodyBtn.BackgroundTransparency = 1
topBarBodyBtn.Text = ""
topBarBodyBtn.ZIndex = 0
topBarBodyBtn.MouseButton1Click:Connect(function()
    if not cfg.showTopBarControls then
        toggleUI("restore")
    end
end)

-- FPS Display Label
local screenFps = Instance.new("TextLabel", gui)
screenFps.Size  = UDim2.new(0, 100, 0, 30)
screenFps.Position = UDim2.new(1, -110, 1, -40)
screenFps.BackgroundTransparency = 1
screenFps.TextColor3 = theme.accent
screenFps.Font  = Enum.Font.Code
screenFps.TextSize = 15
screenFps.TextXAlignment = Enum.TextXAlignment.Right
screenFps.Visible = false
screenFps.ZIndex = 10

-- Window controls (inside main)
local controls = Instance.new("Frame", main)
controls.Size  = UDim2.new(0, 80, 0, 30)
controls.Position = UDim2.new(1, -90, 0, 10)
controls.BackgroundTransparency = 1

local closeBtn = Instance.new("TextButton", controls)
closeBtn.Size  = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -30, 0, 0)
closeBtn.BackgroundTransparency = 1
closeBtn.Text  = "×"
closeBtn.TextColor3 = theme.text
closeBtn.TextSize = 24
closeBtn.Font  = Enum.Font.Gotham
closeBtn.MouseButton1Click:Connect(function() toggleUI("close_to_widgets") end)

local miniBtn = Instance.new("TextButton", controls)
miniBtn.Size  = UDim2.new(0, 30, 0, 30)
miniBtn.Position = UDim2.new(0, 10, 0, 0)
miniBtn.BackgroundTransparency = 1
miniBtn.Text  = "−"
miniBtn.TextColor3 = theme.text
miniBtn.TextSize = 24
miniBtn.Font  = Enum.Font.Gotham
miniBtn.MouseButton1Click:Connect(function() toggleUI("close_to_widgets") end)

closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {TextColor3 = theme.danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {TextColor3 = theme.text}):Play() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn, TweenInfo.new(0.15), {TextColor3 = theme.accent}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn, TweenInfo.new(0.15), {TextColor3 = theme.text}):Play() end)

local function updateControlsVisibility()
    local visible = cfg.showTopBarControls == true
    closeBtn.Visible = visible
    miniBtn.Visible = visible
    topMini.Visible = visible
    topClose.Visible = visible
end

-- Key submit logic
local function openMainUI()
    keyFrame:Destroy()
    layoutUI()
    main.Visible = true
    toggleUI("restore")
end

local function doSubmit()
    if not isRegistered then
        -- Registration Flow
        if keyInput.Text == masterKey then
            if userInput.Text ~= "" and passInput.Text ~= "" then
                cfg.username = userInput.Text
                cfg.password = passInput.Text
                cfg.isRegistered = true
                cfg.keyVerified = true
                saveConfig()

                keySubmit.BackgroundColor3 = theme.success
                keySubmit.Text = "ACCOUNT CREATED"
                task.wait(1)

                isRegistered = true
                drawAuthScreen()

                keySubmit.BackgroundColor3 = theme.accent
                keySubmit.Text = "LOG IN"
            else
                keySubmit.BackgroundColor3 = theme.danger
                keySubmit.Text = "FILL ALL FIELDS"
                task.wait(1.2)
                keySubmit.BackgroundColor3 = theme.accent
                keySubmit.Text = "REGISTER ACCOUNT"
            end
        else
            keySubmit.BackgroundColor3 = theme.danger
            keySubmit.Text = "INVALID KEY"
            task.wait(1.2)
            keySubmit.BackgroundColor3 = theme.accent
            keySubmit.Text = "REGISTER ACCOUNT"
        end
    else
        -- Login Flow
        local reqUser = cfg.username or "Zay"
        local reqPass = cfg.password or "revampv4"

        if userInput.Text == reqUser and passInput.Text == reqPass then
            keySubmit.BackgroundColor3 = theme.success
            keySubmit.Text = "ACCESS GRANTED"
            if rememberEnabled then
                cfg.keyVerified = true
                cfg.rememberedUsername = userInput.Text
                cfg.rememberedPassword = passInput.Text
                saveConfig()
            else
                cfg.keyVerified = false
                cfg.rememberedUsername = ""
                cfg.rememberedPassword = ""
                saveConfig()
            end
            task.wait(0.4)
            openMainUI()
        else
            keySubmit.BackgroundColor3 = theme.danger
            keySubmit.Text = "INVALID CREDENTIALS"
            task.wait(1.2)
            keySubmit.BackgroundColor3 = theme.accent
            keySubmit.Text = "LOG IN"
        end
    end
end

keySubmit.MouseButton1Click:Connect(doSubmit)

userInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then task.spawn(doSubmit) end
end)
passInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then task.spawn(doSubmit) end
end)
keyInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then task.spawn(doSubmit) end
end)


--==================================================
-- SIDEBAR LOGO (compact icon-only bar)
--==================================================
local logoLabel = Instance.new("TextLabel", sidebar)
logoLabel.Size     = UDim2.new(1, 0, 0, 52)
logoLabel.Position = UDim2.new(0, 0, 0, 0)
logoLabel.BackgroundTransparency = 1
logoLabel.Text     = "Z"
logoLabel.TextColor3 = theme.accent
logoLabel.Font     = Enum.Font.GothamBlack
logoLabel.TextSize = 26

local logoDivider = Instance.new("Frame", sidebar)
logoDivider.Size     = UDim2.new(0.55, 0, 0, 1)
logoDivider.Position = UDim2.new(0.225, 0, 0, 52)
logoDivider.BackgroundColor3 = theme.accent
logoDivider.BackgroundTransparency = 0.6
logoDivider.BorderSizePixel = 0

--==================================================
-- SHARED HOVER TOOLTIP
--==================================================
local tooltip = Instance.new("Frame", main)
tooltip.Size        = UDim2.new(0, 175, 0, 56)
tooltip.AnchorPoint = Vector2.new(0, 0.5)
tooltip.BackgroundColor3 = Color3.fromRGB(5, 20, 14)
tooltip.BorderSizePixel = 0
tooltip.Visible     = false
tooltip.ZIndex      = 50
applyStyle(tooltip, 10, true)

-- Left accent notch
local ttAccentBar = Instance.new("Frame", tooltip)
ttAccentBar.Size     = UDim2.new(0, 3, 0.6, 0)
ttAccentBar.Position = UDim2.new(0, 0, 0.2, 0)
ttAccentBar.BackgroundColor3 = theme.accent
ttAccentBar.BorderSizePixel = 0
Instance.new("UICorner", ttAccentBar).CornerRadius = UDim.new(0, 2)

local ttName = Instance.new("TextLabel", tooltip)
ttName.Size     = UDim2.new(1, -16, 0, 22)
ttName.Position = UDim2.new(0, 12, 0, 7)
ttName.BackgroundTransparency = 1
ttName.TextColor3 = Color3.new(1, 1, 1)
ttName.Font     = Enum.Font.GothamBold
ttName.TextSize = 13
ttName.TextXAlignment = Enum.TextXAlignment.Left
ttName.ZIndex   = 51

local ttDesc = Instance.new("TextLabel", tooltip)
ttDesc.Size     = UDim2.new(1, -16, 0, 18)
ttDesc.Position = UDim2.new(0, 12, 0, 27)
ttDesc.BackgroundTransparency = 1
ttDesc.TextColor3 = theme.text_dim
ttDesc.Font     = Enum.Font.Gotham
ttDesc.TextSize = 11
ttDesc.TextXAlignment = Enum.TextXAlignment.Left
ttDesc.ZIndex   = 51
ttDesc.TextWrapped = true

-- Sliding nav indicator on right edge of sidebar
local navIndicator = Instance.new("Frame", sidebar)
navIndicator.Size     = UDim2.new(0, 3, 0, 28)
navIndicator.Position = UDim2.new(1, -3, 0, 70)
navIndicator.BackgroundColor3 = theme.accent
navIndicator.BorderSizePixel = 0
navIndicator.ZIndex  = 5
Instance.new("UICorner", navIndicator).CornerRadius = UDim.new(0, 2)

--==================================================
-- PAGES & TAB NAVIGATION (icon sidebar)
--==================================================
local pages = {}
local tabDefs = {
    {name="Profile",    icon="profile.png",   fallback="👤",  desc="Your profile & stats"},
    {name="Presets",    icon="presets.png",   fallback="⚡",  desc="World & visual presets"},
    {name="Mods",       icon="mods.png",      fallback="🎮",  desc="Game modifications"},
    {name="World",      icon="world.png",     fallback="🌐",  desc="Lighting & world"},
    {name="Fonts",      icon="fonts.png",     fallback="Aa",  desc="Font selector"},
    {name="Uni mod",    icon="uni.png",       fallback="👕",  desc="Jersey customization"},
    {name="Sleeves",    icon="sleeves.png",   fallback="💪",  desc="Sleeve customization"},
    {name="Ball mod",   icon="ball.png",      fallback="⚽",  desc="Ball customization"},
    {name="FPS",        icon="fps.png",       fallback="🖥️",  desc="FPS limiter & display"},
    {name="Fast Flags", icon="flag.png",      fallback="🚩",  desc="FastFlag injector"},
    {name="Settings",   icon="settings.png",  fallback="⚙️",  desc="App settings"},
    {name="Discord",    icon="discord.png",   fallback="💬",  desc="Community discord"},
}

local function getIconAsset(fileName)
    local path = "zaystrap_icons/" .. fileName
    local getAsset = getcustomasset or getsynasset
    if getAsset and isfile then
        local exists = false
        pcall(function() exists = isfile(path) end)
        if exists then
            local ok, res = pcall(function() return getAsset(path) end)
            if ok and res then return res end
        end
    elseif getAsset then
        local ok, res = pcall(function() return getAsset(path) end)
        if ok and res then return res end
    end
    return nil
end

local tabScrollFrame = Instance.new("ScrollingFrame", sidebar)
tabScrollFrame.Size     = UDim2.new(1, 0, 1, -56)
tabScrollFrame.Position = UDim2.new(0, 0, 0, 56)
tabScrollFrame.BackgroundTransparency = 1
tabScrollFrame.ScrollBarThickness = 0
tabScrollFrame.ScrollingDirection = Enum.ScrollingDirection.Y
tabScrollFrame.ElasticBehavior = Enum.ElasticBehavior.Always
tabScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
tabScrollFrame.ClipsDescendants = true
tabScrollFrame.BorderSizePixel = 0

local tabLayout = Instance.new("UIListLayout", tabScrollFrame)
tabLayout.Padding = UDim.new(0, 5)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

local tabPad = Instance.new("UIPadding", tabScrollFrame)
tabPad.PaddingTop = UDim.new(0, 6)

tabLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    tabScrollFrame.CanvasSize = UDim2.new(0, 0, 0, tabLayout.AbsoluteContentSize.Y + 12)
end)

for i, def in ipairs(tabDefs) do
    -- Content page
    local p = Instance.new("ScrollingFrame", container)
    p.Size     = UDim2.new(1, 0, 1, 0)
    p.BackgroundTransparency = 1
    p.Visible  = (i == 1)
    p.ClipsDescendants = true
    local layout = Instance.new("UIListLayout", p)
    layout.Padding = UDim.new(0, 8)
    pages[def.name] = p
    if i == 1 then activePage = p end
    makeScrollable(p)
    local pad = Instance.new("UIPadding", p)
    pad.PaddingLeft   = UDim.new(0, 6)
    pad.PaddingRight  = UDim.new(0, 14)
    pad.PaddingTop    = UDim.new(0, 4)
    pad.PaddingBottom = UDim.new(0, 12)

    -- Icon button
    local btn = Instance.new("TextButton", tabScrollFrame)
    btn.Size     = UDim2.new(0, 44, 0, 44)
    btn.BackgroundColor3 = (i == 1) and theme.accent or theme.bg_sidebar
    btn.BackgroundTransparency = (i == 1) and 0.15 or 0.55
    btn.Font     = Enum.Font.GothamBold
    btn.TextSize = 17
    btn.AutoButtonColor = false
    btn.LayoutOrder = i
    btn.ZIndex   = 3
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)

    local btnStroke = Instance.new("UIStroke", btn)
    btnStroke.Thickness    = 1.5
    btnStroke.Color        = theme.accent
    btnStroke.Transparency = (i == 1) and 0.3 or 1

    local imgLabel
    local assetUrl = getIconAsset(def.icon)
    if assetUrl then
        btn.Text = ""
        imgLabel = Instance.new("ImageLabel", btn)
        imgLabel.Size = UDim2.new(0, 24, 0, 24)
        imgLabel.Position = UDim2.new(0.5, -12, 0.5, -12)
        imgLabel.BackgroundTransparency = 1
        imgLabel.Image = assetUrl
        imgLabel.ImageColor3 = (i == 1) and theme.accent or theme.text_dim
        imgLabel.ImageTransparency = (i == 1) and 0 or 0.35
        imgLabel.ZIndex = 4
    else
        btn.Text = def.fallback
        btn.TextColor3 = (i == 1) and theme.accent or theme.text_dim
        btn.TextTransparency = (i == 1) and 0 or 0.35
    end

    -- Hover: expand tooltip
    btn.MouseEnter:Connect(function()
        if activePage == p then return end
        local targetColor = Color3.new(1, 1, 1)
        local btnProps = { BackgroundTransparency = 0.35 }
        if not imgLabel then
            btnProps.TextColor3 = targetColor
            btnProps.TextTransparency = 0
        end
        TweenService:Create(btn, TweenInfo.new(0.15), btnProps):Play()
        if imgLabel then
            TweenService:Create(imgLabel, TweenInfo.new(0.15), { 
                ImageColor3 = targetColor,
                ImageTransparency = 0
            }):Play()
        end
        local sw = sidebar.AbsoluteSize.X
        local btnMidY = btn.AbsolutePosition.Y - main.AbsolutePosition.Y + btn.AbsoluteSize.Y / 2
        tooltip.Position = UDim2.new(0, sw + 10, 0, btnMidY)
        ttName.Text = def.name
        ttDesc.Text = def.desc
        tooltip.Size = UDim2.new(0, 0, 0, 56)
        tooltip.Visible = true
        TweenService:Create(tooltip, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 175, 0, 56)
        }):Play()
    end)

    btn.MouseLeave:Connect(function()
        if activePage ~= p then
            local targetColor = theme.text_dim
            local btnProps = { BackgroundTransparency = 0.55 }
            if not imgLabel then
                btnProps.TextColor3 = targetColor
                btnProps.TextTransparency = 0.35
            end
            TweenService:Create(btn, TweenInfo.new(0.15), btnProps):Play()
            if imgLabel then
                TweenService:Create(imgLabel, TweenInfo.new(0.15), { 
                    ImageColor3 = targetColor,
                    ImageTransparency = 0.35
                }):Play()
            end
        end
        TweenService:Create(tooltip, TweenInfo.new(0.14), {
            Size = UDim2.new(0, 0, 0, 56)
        }):Play()
        task.delay(0.15, function()
            if not activePage or activePage ~= p then
                tooltip.Visible = false
            end
        end)
    end)

    btn.MouseButton1Click:Connect(function()
        if p == activePage then return end
        if activePage then activePage.Visible = false end
        tooltip.Visible = false

        -- Deactivate all buttons
        for _, child in ipairs(tabScrollFrame:GetChildren()) do
            if child:IsA("TextButton") then
                local childImg = child:FindFirstChildOfClass("ImageLabel")
                local btnProps = {
                    BackgroundTransparency = 0.55
                }
                if not childImg then
                    btnProps.TextColor3 = theme.text_dim
                    btnProps.TextTransparency = 0.35
                end
                TweenService:Create(child, TweenInfo.new(0.2), btnProps):Play()
                if childImg then
                    TweenService:Create(childImg, TweenInfo.new(0.2), { 
                        ImageColor3 = theme.text_dim,
                        ImageTransparency = 0.35
                    }):Play()
                end
                local s = child:FindFirstChildOfClass("UIStroke")
                if s then TweenService:Create(s, TweenInfo.new(0.2), {Transparency = 1}):Play() end
            end
        end

        -- Activate this button
        local actBtnProps = {
            BackgroundTransparency = 0.15
        }
        if not imgLabel then
            actBtnProps.TextColor3 = theme.accent
            actBtnProps.TextTransparency = 0
        end
        TweenService:Create(btn, TweenInfo.new(0.2), actBtnProps):Play()
        if imgLabel then
            TweenService:Create(imgLabel, TweenInfo.new(0.2), { 
                ImageColor3 = theme.accent,
                ImageTransparency = 0
            }):Play()
        end
        TweenService:Create(btnStroke, TweenInfo.new(0.2), {Transparency = 0.3}):Play()

        -- Slide nav indicator to this button
        local btnPos = btn.AbsolutePosition.Y - tabScrollFrame.AbsolutePosition.Y + tabScrollFrame.CanvasPosition.Y
        TweenService:Create(navIndicator, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            Position = UDim2.new(1, -3, 0, 56 + btnPos + 8)
        }):Play()

        -- Slide in content page
        p.Visible  = true
        p.Position = UDim2.new(0.05, 0, 0, 0)
        p.Size     = UDim2.new(0.95, 0, 1, 0)
        TweenService:Create(p, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Position = UDim2.new(0, 0, 0, 0),
            Size     = UDim2.new(1, 0, 1, 0)
        }):Play()
        activePage = p
    end)
end
--==================================================
-- PRESETS TAB
--==================================================
MakeSectionLabel(pages.Presets, "World presets")

createToggle(pages.Presets, "Grey Fog", function(s)
    if s then
        Lighting.Ambient = Color3.fromRGB(100, 100, 110)
        Lighting.OutdoorAmbient = Color3.fromRGB(70, 70, 80)
        Lighting.Brightness = 0.5
        Lighting.FogColor   = Color3.fromRGB(150, 150, 160)
        Lighting.FogEnd     = 500
        Lighting.FogStart   = 50
        local sr = Lighting:FindFirstChildOfClass("SunRaysEffect")
        if sr then sr.Intensity = 0 end
    else
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Brightness = 1
        Lighting.FogColor   = Color3.fromRGB(192,192,192)
        Lighting.FogEnd     = 100000
        Lighting.FogStart   = 0
        local sr = Lighting:FindFirstChildOfClass("SunRaysEffect")
        if sr then sr.Intensity = 0.25 end
    end
end)

createToggle(pages.Presets, "FF3 No-Textures / FPS+", function(s)
    ff3Enabled = s
    if s then
        pcall(function()
            if setfflag then
                setfflag("FIntDebugTextureManagerSkipMips", "8")
            elseif setpatchedfflag then
                setpatchedfflag("FIntDebugTextureManagerSkipMips", "8")
            end
        end)
        task.spawn(function()
            local descendants = Workspace:GetDescendants()
            for i, v in ipairs(descendants) do
                simplify(v)
                if i % 1000 == 0 then task.wait() end
            end
        end)
    else
        pcall(function()
            if setfflag then
                setfflag("FIntDebugTextureManagerSkipMips", "0")
            elseif setpatchedfflag then
                setpatchedfflag("FIntDebugTextureManagerSkipMips", "0")
            end
        end)
    end
end)

MakeSectionLabel(pages.Presets, "Advanced rendering")

createToggle(pages.Presets, "Fullbright / No Fog", function(s)
    if s then
        Lighting.Ambient    = Color3.new(1,1,1)
        Lighting.Brightness = 2
        Lighting.FogEnd     = 100000
    else
        Lighting.Ambient    = Color3.new(0.5,0.5,0.5)
        Lighting.Brightness = 1
        Lighting.FogEnd     = 1000
    end
end)

createToggle(pages.Presets, "Clear Atmosphere", function(s)
    for _, v in pairs(Lighting:GetChildren()) do
        if v:IsA("Atmosphere") or v:IsA("Clouds") or v:IsA("Sky") then
            v.Parent = s and ReplicatedStorage or Lighting
        end
    end
end)

createToggle(pages.Presets, "Pink Sky", function(on)
    if on then
        local s = Instance.new("Sky", Lighting)
        s.Name = "S2_PinkSky"
        s.SkyboxBk = "rbxassetid://271042516"
        s.SkyboxDn = "rbxassetid://271077243"
        s.SkyboxFt = "rbxassetid://271042556"
        s.SkyboxLf = "rbxassetid://271042310"
        s.SkyboxRt = "rbxassetid://271042467"
        s.SkyboxUp = "rbxassetid://271077958"
    else
        local existing = Lighting:FindFirstChild("S2_PinkSky")
        if existing then existing:Destroy() end
    end
end)

createToggle(pages.Presets, "No Particles", function(s)
    task.spawn(function()
        local count = 0
        for _, v in pairs(game.Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
                v.Enabled = not s
            end
            count = count + 1
            if count % 500 == 0 then task.wait() end
        end
    end)
end)

-- Additional preset toggles
createToggle(pages.Presets, "Bright Day", function(s)
    if s then
        Lighting.Ambient = Color3.fromRGB(255,255,255)
        Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
        Lighting.Brightness = 3
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
    else
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Brightness = 1
        Lighting.FogEnd = 500
        Lighting.FogStart = 50
    end
end)

createToggle(pages.Presets, "Night Mode", function(s)
    if s then
        Lighting.Ambient = Color3.fromRGB(10,10,30)
        Lighting.OutdoorAmbient = Color3.fromRGB(5,5,20)
        Lighting.Brightness = 0.2
        Lighting.FogEnd = 200
        Lighting.FogStart = 0
    else
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Brightness = 1
        Lighting.FogEnd = 500
        Lighting.FogStart = 50
    end
end)

createToggle(pages.Presets, "Storm", function(s)
    if s then
        Lighting.Ambient = Color3.fromRGB(60,70,90)
        Lighting.OutdoorAmbient = Color3.fromRGB(50,60,80)
        Lighting.Brightness = 0.3
        Lighting.FogColor = Color3.fromRGB(100,100,120)
        Lighting.FogEnd = 200
        Lighting.FogStart = 0
    else
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Brightness = 1
        Lighting.FogEnd = 500
        Lighting.FogStart = 50
    end
end)

--==================================================
-- NIKE TECH & SPASM
--==================================================
--==================================================
-- NIKE TECH & SPASM
--==================================================
local spasmConnection = nil
local function setupSpasm(char)
    if spasmConnection then
        pcall(function() spasmConnection:Disconnect() end)
        spasmConnection = nil
    end
    local humanoid = char:WaitForChild("Humanoid", 5)
    local rootPart = char:WaitForChild("HumanoidRootPart", 5)
    if not humanoid or not rootPart then return end
    spasmConnection = humanoid.Jumping:Connect(function()
        task.wait(0.05)
        if rootPart and rootPart.Parent then
            local v = rootPart.AssemblyLinearVelocity
            rootPart.AssemblyLinearVelocity = Vector3.new(v.X, 0, v.Z)
            rootPart.AssemblyLinearVelocity = Vector3.new(v.X, 68, v.Z)
        end
    end)
end

local nikeTechConnection = nil
local function setupNikeTech(char)
    if nikeTechConnection then
        pcall(function() nikeTechConnection:Disconnect() end)
        nikeTechConnection = nil
    end
    local humanoid = char:WaitForChild("Humanoid", 5)
    local rootPart = char:WaitForChild("HumanoidRootPart", 5)
    if not humanoid or not rootPart then return end
    nikeTechConnection = humanoid.Jumping:Connect(function()
        pcall(function()
            if rootPart and rootPart.Parent then
                rootPart.Anchored = true
                task.wait(0.0035)
                if rootPart and rootPart.Parent then
                    rootPart.Anchored = false
                    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 80, rootPart.Velocity.Z)
                end
            end
        end)
    end)
end

--==================================================
-- JP MACRO LOGIC
--==================================================
local jpEnabled, angleEnabled, antiAceEnabled = false, false, false
local hasBoosted = false
local localPlayer = Players.LocalPlayer
local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart", 5)
local hum = char:WaitForChild("Humanoid", 5)

localPlayer.CharacterAdded:Connect(function(newChar)
    char = newChar
    hrp = char:WaitForChild("HumanoidRootPart", 5)
    hum = char:WaitForChild("Humanoid", 5)
end)

--==================================================
-- MODS TAB (Scrollable)
--==================================================
MakeSectionLabel(pages.Mods, "Macros & automation")

createToggle(pages.Mods, "Triangle Reset Macro", function(state) _G.ResetMacroEnabled = state end)
createToggle(pages.Mods, "Freeze Macro (L1)",    function(s) freezeMacroEnabled = s end)

createToggle(pages.Mods, "Nike Tech", function(on)
    if on then
        if player.Character then setupNikeTech(player.Character) end
        _G.NikeTechConn = player.CharacterAdded:Connect(setupNikeTech)
    else
        if nikeTechConnection then nikeTechConnection:Disconnect(); nikeTechConnection = nil end
        if _G.NikeTechConn then _G.NikeTechConn:Disconnect() end
    end
end)

createToggle(pages.Mods, "Spasm Macro", function(on)
    if on then
        if player.Character then setupSpasm(player.Character) end
        _G.SpasmConn = player.CharacterAdded:Connect(setupSpasm)
    else
        if spasmConnection then spasmConnection:Disconnect(); spasmConnection = nil end
        if _G.SpasmConn then _G.SpasmConn:Disconnect() end
    end
end)

createToggle(pages.Mods, "JP Macro",           function(state) jpEnabled = state end)
createToggle(pages.Mods, "Angle Macro",         function(s)    angleEnabled = s end)
createToggle(pages.Mods, "Anti Ace (No Uni)",   function(s)    antiAceEnabled = s end)
createToggle(pages.Mods, "FPS Display",          function(s)    screenFps.Visible = s end)

--==================================================
-- GRAVITY CONTROLS
--==================================================
MakeSectionLabel(pages.Mods, "GRAVITY CONTROLS")

local gravActive   = false
local gravStrength = cfg.gravStrength or 0.4

local function applySmartPhysics()
    local ch = player.Character
    if not ch then return end
    local hrp2 = ch:FindFirstChild("HumanoidRootPart")
    local hum2 = ch:FindFirstChild("Humanoid")
    if not hrp2 or not hum2 then return end

    for _, v in pairs(hrp2:GetChildren()) do
        if v.Name == "S2_SmartGrav" or v.Name == "S2_GravAttach" then v:Destroy() end
    end

    if gravActive then
        local attach = Instance.new("Attachment", hrp2)
        attach.Name  = "S2_GravAttach"
        local vf = Instance.new("VectorForce", hrp2)
        vf.Name  = "S2_SmartGrav"
        vf.Attachment0 = attach
        vf.RelativeTo  = Enum.ActuatorRelativeTo.World

        task.spawn(function()
            while gravActive and vf and vf.Parent do
                local state = hum2:GetState()
                local mass  = hrp2.AssemblyMass
                if state == Enum.HumanoidStateType.Freefall or state == Enum.HumanoidStateType.Jumping then
                    vf.Force = Vector3.new(0, mass * 196.2 * gravStrength, 0)
                elseif state == Enum.HumanoidStateType.Physics then
                    vf.Force = Vector3.new(0, mass * 196.2 * (gravStrength * 0.7), 0)
                else
                    vf.Force = Vector3.new(0, 0, 0)
                end
                task.wait(0.03)
            end
            pcall(function() vf:Destroy() end)
            pcall(function() attach:Destroy() end)
        end)
    end
end

-- Preset gravity toggles
createToggle(pages.Mods, "Legit Gravity (Safe)", function(state)
    gravActive   = state
    gravStrength = 0.3
    cfg.gravStrength = gravStrength
    saveConfig()
    applySmartPhysics()
end)

createToggle(pages.Mods, "Gravity Macro (Noticeable)", function(state)
    gravActive   = state
    gravStrength = 0.6
    cfg.gravStrength = gravStrength
    saveConfig()
    applySmartPhysics()
end)

-- Custom gravity input
local gravToggleActive = false
local gravCustomBox = MakeNumberInput(pages.Mods, "Custom Gravity (0.1 – 2.0)", gravStrength, 0.05, 2.0, function(v)
    gravStrength = v
    cfg.gravStrength = v
    saveConfig()
    if gravActive then applySmartPhysics() end
end)

-- Enable/disable custom gravity
createToggle(pages.Mods, "Enable Custom Gravity", function(state)
    gravActive       = state
    gravToggleActive = state
    applySmartPhysics()
end)

player.CharacterAdded:Connect(function()
    task.wait(1.5)
    if gravActive then applySmartPhysics() end
end)

--==================================================
-- WORLD TAB  (Scrollable)
--==================================================
MakeSectionLabel(pages.World, "SKY & ATMOSPHERE")

-- Sky color (ambient)
MakeColorRow(pages.World, "Sky Ambient Color (RGB)", cfg.skyR, cfg.skyG, cfg.skyB, function(r, g, b)
    cfg.skyR, cfg.skyG, cfg.skyB = r, g, b
    Lighting.Ambient = Color3.fromRGB(r, g, b)
    saveConfig()
end)

MakeColorRow(pages.World, "Outdoor Ambient (RGB)", cfg.ambR, cfg.ambG, cfg.ambB, function(r, g, b)
    cfg.ambR, cfg.ambG, cfg.ambB = r, g, b
    Lighting.OutdoorAmbient = Color3.fromRGB(r, g, b)
    saveConfig()
end)

MakeNumberInput(pages.World, "Brightness (0.1 – 5)", cfg.brightness, 0.1, 5, function(v)
    cfg.brightness = v
    Lighting.Brightness = v
    saveConfig()
end)

MakeSectionLabel(pages.World, "FOG")

MakeColorRow(pages.World, "Fog Color (RGB)", cfg.fogR, cfg.fogG, cfg.fogB, function(r, g, b)
    cfg.fogR, cfg.fogG, cfg.fogB = r, g, b
    Lighting.FogColor = Color3.fromRGB(r, g, b)
    saveConfig()
end)

MakeNumberInput(pages.World, "Fog Start (0 – 10000)", Lighting.FogStart, 0, 10000, function(v)
    Lighting.FogStart = v
    saveConfig()
end)

MakeNumberInput(pages.World, "Fog End (100 – 200000)", cfg.fogEnd, 100, 200000, function(v)
    cfg.fogEnd      = v
    Lighting.FogEnd = v
    saveConfig()
end)

MakeSectionLabel(pages.World, "TIME OF DAY & SUN")

-- Time of day input
do
    local tod = Instance.new("Frame", pages.World)
    tod.Size  = UDim2.new(1, 0, 0, 36)
    tod.BackgroundColor3 = theme.bg_card
    tod.BackgroundTransparency = 0.3
    applyStyle(tod, 6, true)

    local todLbl = Instance.new("TextLabel", tod)
    todLbl.Size  = UDim2.new(0.55, 0, 1, 0)
    todLbl.Position = UDim2.new(0, 10, 0, 0)
    todLbl.BackgroundTransparency = 1
    todLbl.Text  = "Time of Day (HH:MM:SS)"
    todLbl.TextColor3 = theme.text
    todLbl.Font  = Enum.Font.GothamMedium
    todLbl.TextSize = 11
    todLbl.TextXAlignment = Enum.TextXAlignment.Left
    todLbl.TextWrapped = true

    local todBox = Instance.new("TextBox", tod)
    todBox.Size  = UDim2.new(0.38, 0, 0.72, 0)
    todBox.Position = UDim2.new(0.60, 0, 0.14, 0)
    todBox.BackgroundColor3 = theme.bg_sidebar
    todBox.Text  = cfg.timeOfDay or "14:00:00"
    todBox.TextColor3 = theme.accent
    todBox.Font  = Enum.Font.GothamMedium
    todBox.TextSize = 11
    todBox.ClearTextOnFocus = false
    applyStyle(todBox, 4, true)

    todBox.FocusLost:Connect(function()
        local t = todBox.Text
        if t:match("^%d%d:%d%d:%d%d$") then
            pcall(function() Lighting.TimeOfDay = t end)
            cfg.timeOfDay = t
            saveConfig()
        end
    end)
end

MakeNumberInput(pages.World, "Clock Time (0 – 24)", Lighting.ClockTime, 0, 24, function(v)
    Lighting.ClockTime = v
end)

-- Shadows & effects toggles
MakeSectionLabel(pages.World, "EFFECTS & SHADOWS")

createToggle(pages.World, "Global Shadows", function(s)
    Lighting.GlobalShadows = s
end)

createToggle(pages.World, "Bloom Effect", function(s)
    local bloom = Lighting:FindFirstChildOfClass("BloomEffect")
    if bloom then bloom.Enabled = s
    elseif s then Instance.new("BloomEffect", Lighting) end
end)

createToggle(pages.World, "Sun Rays", function(s)
    local sr = Lighting:FindFirstChildOfClass("SunRaysEffect")
    if sr then sr.Enabled = s
    elseif s then Instance.new("SunRaysEffect", Lighting) end
end)

createToggle(pages.World, "Depth of Field Blur", function(s)
    local dof = Lighting:FindFirstChildOfClass("DepthOfFieldEffect")
    if dof then dof.Enabled = s
    elseif s then Instance.new("DepthOfFieldEffect", Lighting) end
end)

createToggle(pages.World, "Color Correction", function(s)
    local cc = Lighting:FindFirstChildOfClass("ColorCorrectionEffect")
    if cc then cc.Enabled = s
    elseif s then Instance.new("ColorCorrectionEffect", Lighting) end
end)

-- Quick presets inside World tab
MakeSectionLabel(pages.World, "QUICK SKY PRESETS")

local skyPresets = {
    {"Dawn 🌅",   {ambR=255,ambG=160,ambB=100, outR=200,outG=140,outB=80,  bright=0.8, time="6:00:00"}},
    {"Noon ☀️",   {ambR=180,ambG=200,ambB=220, outR=160,outG=190,outB=210, bright=2,   time="14:00:00"}},
    {"Dusk 🌆",   {ambR=255,ambG=100,ambB=60,  outR=200,outG=80, ambB=50,  bright=0.6, time="18:30:00"}},
    {"Night 🌙",  {ambR=10, ambG=20, ambB=60,  outR=5,  outG=10, outB=40,  bright=0.1, time="00:00:00"}},
    {"Storm ⛈️", {ambR=60, ambG=70, ambB=90,  outR=50, outG=60, outB=80,  bright=0.3, time="12:00:00"}},
    {"Pink Sky 🌸",{ambR=255,ambG=180,ambB=220,outR=220,outG=150,outB=200, bright=1.5, time="07:30:00"}},
}

for _, preset in ipairs(skyPresets) do
    local name, data = preset[1], preset[2]
    MakeButton(pages.World, name, function()
        pcall(function()
            Lighting.Ambient        = Color3.fromRGB(data.ambR or 127, data.ambG or 127, data.ambB or 127)
            Lighting.OutdoorAmbient = Color3.fromRGB(data.outR or 127, data.outG or 127, data.outB or 127)
            Lighting.Brightness     = data.bright or 1
            if data.time then Lighting.TimeOfDay = data.time end
        end)
    end)
end

--==================================================
-- CONFIG SAVE / LOAD BUTTONS (bottom of World tab)
--==================================================
MakeSectionLabel(pages.World, "CONFIG")

MakeButton(pages.World, "💾  Save Config", function(btn)
    saveConfig()
    local old = btn.Text
    btn.Text = "SAVED ✓"
    btn.BackgroundColor3 = theme.success
    task.wait(1.5)
    btn.Text = old
    btn.BackgroundColor3 = theme.accent
end)

MakeButton(pages.World, "♻️  Reset Config", function(btn)
    for k, v in pairs(defaultConfig) do cfg[k] = v end
    saveConfig()
    local old = btn.Text
    btn.Text = "RESET ✓"
    btn.BackgroundColor3 = theme.danger
    task.wait(1.5)
    btn.Text = old
    btn.BackgroundColor3 = theme.accent
end)

--==================================================
-- FONTS TAB (Scrollable)
--==================================================
local selectedFont = Enum.Font.Montserrat

local fMsg = Instance.new("TextLabel", pages.Fonts)
fMsg.Size  = UDim2.new(1, 0, 0, 25)
fMsg.BackgroundTransparency = 1
fMsg.Text  = "-Global font changer-"
fMsg.TextColor3 = theme.accent
fMsg.Font  = Enum.Font.GothamBold
fMsg.TextSize = 13
fMsg.TextXAlignment = Enum.TextXAlignment.Center

local function applySafe(obj)
    if not obj or not obj:IsDescendantOf(game) then return end
    pcall(function()
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
            obj.Font = selectedFont
        end
    end)
end

local function runGlobalUpdate()
    for _, v in pairs(player.PlayerGui:GetDescendants()) do
        if v ~= gui then applySafe(v) end
    end
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("SurfaceGui") or v:IsA("BillboardGui") then
            for _, textObj in pairs(v:GetDescendants()) do applySafe(textObj) end
        end
    end
end

workspace.DescendantAdded:Connect(function(newObj)
    if newObj:IsA("SurfaceGui") or newObj:IsA("BillboardGui") then
        task.wait(0.1)
        for _, textObj in pairs(newObj:GetDescendants()) do applySafe(textObj) end
    end
end)

local fontsList = {
    "Arial","ArialBold","SourceSans","SourceSansBold","SourceSansSemibold",
    "CourierNew","Garamond","Georgia","Helvetica","Gothic",
    "Arcade","Highway","SciFi","IndieFlower","PlayfairDisplay",
    "SpecialElite","Merriweather","Nunito","Oswald","PermanentMarker",
    "Creepster","Jura","PressStart2P","Ubuntu","FredokaOne",
    "LuckiestGuy","Bangers","Gotham","GothamMedium","GothamBold",
    "GothamBlack","Montserrat","Roboto","RobotoMono",
    "BurbankBigCondensed"
}

local fGrid = Instance.new("Frame", pages.Fonts)
fGrid.Size  = UDim2.new(1, 0, 0, 0)
fGrid.Position = UDim2.new(0, 0, 0, 35)
fGrid.BackgroundTransparency = 1
fGrid.AutomaticSize = Enum.AutomaticSize.Y

fGl = Instance.new("UIGridLayout", fGrid)
fGl.CellSize    = UDim2.new(0.48, 0, 0, 32)
fGl.CellPadding = UDim2.new(0, 8, 0, 6)

fGl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    fGrid.Size = UDim2.new(1, 0, 0, fGl.AbsoluteContentSize.Y + 10)
end)

for _, fname in ipairs(fontsList) do
    local b = Instance.new("TextButton", fGrid)
    b.Text = fname
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = theme.bg_card
    b.Font = Enum.Font.SourceSansBold
    b.TextSize = 12
    b.AutoButtonColor = false
    local bStroke = applyStyle(b, 6, true)
    bStroke.Transparency = 0.5
    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent}):Play()
        TweenService:Create(bStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.2), {BackgroundColor3 = theme.bg_card}):Play()
        TweenService:Create(bStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
    end)
    b.MouseButton1Click:Connect(function()
        local success, fontEnum = pcall(function() return Enum.Font[fname] end)
        if success and fontEnum then
            selectedFont = fontEnum
            runGlobalUpdate()
        else
            warn("Font "..fname.." not found, using SourceSans")
            selectedFont = Enum.Font.SourceSans
            runGlobalUpdate()
        end
    end)
end

--==================================================
-- UNI MOD TAB (Scrollable)
--==================================================
local uniCard = Instance.new("Frame", pages["Uni mod"])
uniCard.Size  = UDim2.new(1, 0, 0, 220)
uniCard.BackgroundColor3 = Color3.new(1,1,1)
uniCard.BackgroundTransparency = 0.45
applyStyle(uniCard, 10, true)
applyGradient(uniCard, theme.bg_card, Color3.fromRGB(18,38,25))

local uniHeader = Instance.new("TextLabel", uniCard)
uniHeader.Size  = UDim2.new(1,-30, 0, 30)
uniHeader.Position = UDim2.new(0,15, 0, 10)
uniHeader.BackgroundTransparency = 1
uniHeader.Text  = "Locker Room Customizer"
uniHeader.TextColor3 = Color3.new(1,1,1)
uniHeader.Font  = Enum.Font.GothamBold
uniHeader.TextSize = 15
uniHeader.TextXAlignment = Enum.TextXAlignment.Left

local nameLabel = Instance.new("TextLabel", uniCard)
nameLabel.Size  = UDim2.new(1,-30, 0, 15)
nameLabel.Position = UDim2.new(0,15, 0, 42)
nameLabel.BackgroundTransparency = 1
nameLabel.Text  = "Jersey name"
nameLabel.TextColor3 = theme.text_dim
nameLabel.Font  = Enum.Font.GothamBold
nameLabel.TextSize = 9
nameLabel.TextXAlignment = Enum.TextXAlignment.Left

local jName = Instance.new("TextBox", uniCard)
jName.Size  = UDim2.new(1,-30, 0, 32)
jName.Position = UDim2.new(0,15, 0, 56)
jName.BackgroundColor3 = theme.bg_sidebar
jName.Text  = ""
jName.PlaceholderText = "Enter Name..."
jName.PlaceholderColor3 = theme.outline
jName.TextColor3 = theme.accent
jName.Font  = Enum.Font.GothamMedium
jName.TextSize = 12
applyStyle(jName, 6, true)

local numLabel = Instance.new("TextLabel", uniCard)
numLabel.Size  = UDim2.new(1,-30, 0, 15)
numLabel.Position = UDim2.new(0,15, 0, 96)
numLabel.BackgroundTransparency = 1
numLabel.Text  = "Jersey number"
numLabel.TextColor3 = theme.text_dim
numLabel.Font  = Enum.Font.GothamBold
numLabel.TextSize = 9
numLabel.TextXAlignment = Enum.TextXAlignment.Left

local jNum = Instance.new("TextBox", uniCard)
jNum.Size  = UDim2.new(1,-30, 0, 32)
jNum.Position = UDim2.new(0,15, 0, 110)
jNum.BackgroundColor3 = theme.bg_sidebar
jNum.Text  = ""
jNum.PlaceholderText = "00"
jNum.PlaceholderColor3 = theme.outline
jNum.TextColor3 = theme.accent
jNum.Font  = Enum.Font.GothamMedium
jNum.TextSize = 12
applyStyle(jNum, 6, true)

local applyJ = Instance.new("TextButton", uniCard)
applyJ.Size  = UDim2.new(1,-30, 0, 36)
applyJ.Position = UDim2.new(0,15, 0, 160)
applyJ.BackgroundColor3 = theme.accent
applyJ.Text  = "Update uniform"
applyJ.Font  = Enum.Font.GothamBold
applyJ.TextColor3 = Color3.new(1,1,1)
applyJ.TextSize = 12
applyJ.AutoButtonColor = false
local btnStroke = applyStyle(applyJ, 6, true, theme.accent_light)
btnStroke.Transparency = 0.5

applyJ.MouseEnter:Connect(function()
    TweenService:Create(applyJ, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent_light}):Play()
    TweenService:Create(btnStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
end)
applyJ.MouseLeave:Connect(function()
    TweenService:Create(applyJ, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent}):Play()
    TweenService:Create(btnStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
end)

applyJ.MouseButton1Click:Connect(function()
    local ch = player.Character
    if ch and ch:FindFirstChild("Uniform") then
        local remotes = ReplicatedStorage:WaitForChild("Remotes", 3)
        local ev = remotes and remotes:WaitForChild("CharacterSoundEvent", 3)
        if ev then
            ev:FireServer("Game","Customization","Name", jName.Text)
            ev:FireServer("Game","Customization","Number", jNum.Text)
            applyJ.BackgroundColor3 = theme.success
            applyJ.Text = "Identity applied"
            task.wait(1.5)
            applyJ.BackgroundColor3 = theme.accent
            applyJ.Text = "Update uniform"
        else
            applyJ.Text = "Remotes not found"
            applyJ.BackgroundColor3 = theme.danger
            task.wait(1.5)
            applyJ.BackgroundColor3 = theme.accent
            applyJ.Text = "Update uniform"
        end
    else
        applyJ.Text = "Uniform not found"
        applyJ.BackgroundColor3 = theme.danger
        task.wait(1.5)
        applyJ.BackgroundColor3 = theme.accent
        applyJ.Text = "Update uniform"
    end
end)

local uniConn
createToggle(pages["Uni mod"], "Uni Script", function(state)
    if not state then if uniConn then pcall(function() uniConn:Disconnect() end) uniConn = nil end return end
    local remotes = ReplicatedStorage:WaitForChild("Remotes", 5)
    local event = remotes and remotes:WaitForChild("CharacterSoundEvent", 5)
    if not event then return end
    local function apply()
        local ch = player.Character
        if not ch then return end
        task.wait(0.5)
        if ch and ch.Parent and ch:FindFirstChild("Uniform") then
            event:FireServer("Game","Customization","Toggle","LeftGlove")
            event:FireServer("Game","Customization","Toggle","RightGlove")
            event:FireServer("Game","Customization","Number","0")
            event:FireServer("Game","Customization","Name","                                ")
        end
    end
    apply()
    if uniConn then pcall(function() uniConn:Disconnect() end) end
    uniConn = player.CharacterAdded:Connect(apply)
end)

--==================================================
-- SLEEVE ENGINE
--==================================================
local leftSleeveEnabled = false
local rightSleeveEnabled = false
local leftSleeveColor = Color3.fromRGB(255, 255, 255)
local rightSleeveColor = Color3.fromRGB(255, 255, 255)
local sleeveCharConn = nil

local function removeSleeveFromChar(char, side)
    if not char then return end
    local name = "RealSleeve_" .. side
    local existing = char:FindFirstChild(name)
    if existing then pcall(function() existing:Destroy() end) end
end

local function applyOneSleeve(char, side, color)
    if not char or not char.Parent then return end
    local arm = char:FindFirstChild(side)
    if not arm or not arm:IsA("BasePart") then return end
    removeSleeveFromChar(char, side)
    
    local sleeve = Instance.new("Part")
    sleeve.Name = "RealSleeve_" .. side
    sleeve.Size = Vector3.new(arm.Size.X + 0.03, 1.75, arm.Size.Z + 0.03)
    sleeve.Color = color
    sleeve.Material = Enum.Material.Sand
    sleeve.Reflectance = 0
    sleeve.CanCollide = false
    sleeve.Massless = true
    sleeve.CastShadow = false
    sleeve.CFrame = arm.CFrame * CFrame.new(0, 0.125, 0)
    sleeve.Parent = char
    
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = arm
    weld.Part1 = sleeve
    weld.Parent = sleeve
end

local function refreshSleeves(char)
    if not char then return end
    task.wait(1.5)
    if leftSleeveEnabled then
        applyOneSleeve(char, "Left Arm", leftSleeveColor)
    else
        removeSleeveFromChar(char, "Left Arm")
    end
    if rightSleeveEnabled then
        applyOneSleeve(char, "Right Arm", rightSleeveColor)
    else
        removeSleeveFromChar(char, "Right Arm")
    end
end

local function updateSleeveConn()
    if sleeveCharConn then pcall(function() sleeveCharConn:Disconnect() end) sleeveCharConn = nil end
    if leftSleeveEnabled or rightSleeveEnabled then
        sleeveCharConn = player.CharacterAdded:Connect(refreshSleeves)
        if player.Character then task.spawn(refreshSleeves, player.Character) end
    end
end

local function updateActiveSleeves()
    local char = player.Character
    if char then
        if leftSleeveEnabled then applyOneSleeve(char, "Left Arm", leftSleeveColor) end
        if rightSleeveEnabled then applyOneSleeve(char, "Right Arm", rightSleeveColor) end
    end
end

-- Sleeve Customizer Card UI
local sleeveCard = Instance.new("Frame", pages["Sleeves"])
sleeveCard.AutomaticSize = Enum.AutomaticSize.Y
sleeveCard.Size = UDim2.new(1, 0, 0, 0)
sleeveCard.BackgroundColor3 = Color3.new(1, 1, 1)
sleeveCard.BackgroundTransparency = 0.45
applyStyle(sleeveCard, 10, true)
applyGradient(sleeveCard, theme.bg_card, Color3.fromRGB(18, 38, 25))

local sleeveListLayout = Instance.new("UIListLayout", sleeveCard)
sleeveListLayout.Padding = UDim.new(0, 8)
sleeveListLayout.SortOrder = Enum.SortOrder.LayoutOrder

local sleevePadding = Instance.new("UIPadding", sleeveCard)
sleevePadding.PaddingLeft = UDim.new(0, 12)
sleevePadding.PaddingRight = UDim.new(0, 12)
sleevePadding.PaddingTop = UDim.new(0, 12)
sleevePadding.PaddingBottom = UDim.new(0, 12)

local sleeveHeader = Instance.new("TextLabel", sleeveCard)
sleeveHeader.Size = UDim2.new(1, 0, 0, 25)
sleeveHeader.BackgroundTransparency = 1
sleeveHeader.Text = "Custom Sleeve Designer"
sleeveHeader.TextColor3 = Color3.new(1, 1, 1)
sleeveHeader.Font = Enum.Font.GothamBold
sleeveHeader.TextSize = 15
sleeveHeader.TextXAlignment = Enum.TextXAlignment.Left
sleeveHeader.LayoutOrder = 1

local leftSleeveToggle = createToggle(sleeveCard, "Left Sleeve Enabled", function(s)
    leftSleeveEnabled = s
    if s then
        updateActiveSleeves()
    else
        if player.Character then removeSleeveFromChar(player.Character, "Left Arm") end
    end
    updateSleeveConn()
end, false)
leftSleeveToggle.Button.Parent.LayoutOrder = 2

local leftColorRow = MakeColorRow(sleeveCard, "Left Sleeve Color (RGB)", 255, 255, 255, function(r, g, b)
    leftSleeveColor = Color3.fromRGB(r, g, b)
    if leftSleeveEnabled and player.Character then
        applyOneSleeve(player.Character, "Left Arm", leftSleeveColor)
    end
end)
leftColorRow.LayoutOrder = 3

local rightSleeveToggle = createToggle(sleeveCard, "Right Sleeve Enabled", function(s)
    rightSleeveEnabled = s
    if s then
        updateActiveSleeves()
    else
        if player.Character then removeSleeveFromChar(player.Character, "Right Arm") end
    end
    updateSleeveConn()
end, false)
rightSleeveToggle.Button.Parent.LayoutOrder = 4

local rightColorRow = MakeColorRow(sleeveCard, "Right Sleeve Color (RGB)", 255, 255, 255, function(r, g, b)
    rightSleeveColor = Color3.fromRGB(r, g, b)
    if rightSleeveEnabled and player.Character then
        applyOneSleeve(player.Character, "Right Arm", rightSleeveColor)
    end
end)
rightColorRow.LayoutOrder = 5

local removeAllBtn = MakeButton(sleeveCard, "❌  Remove All Sleeves", function()
    leftSleeveEnabled = false
    rightSleeveEnabled = false
    leftSleeveToggle.SetState(false)
    rightSleeveToggle.SetState(false)
    if player.Character then
        removeSleeveFromChar(player.Character, "Left Arm")
        removeSleeveFromChar(player.Character, "Right Arm")
    end
    updateSleeveConn()
end)
removeAllBtn.LayoutOrder = 6

--==================================================
-- BALL ENGINE
--==================================================
local ballCard = Instance.new("Frame", pages["Ball mod"])
ballCard.Size  = UDim2.new(1, 0, 0, 220)
ballCard.BackgroundColor3 = Color3.new(1,1,1)
ballCard.BackgroundTransparency = 0.45
applyStyle(ballCard, 10, true)
applyGradient(ballCard, theme.bg_card, Color3.fromRGB(18,38,25))

local ballHeader = Instance.new("TextLabel", ballCard)
ballHeader.Size  = UDim2.new(1,-30, 0, 30)
ballHeader.Position = UDim2.new(0,15, 0, 10)
ballHeader.BackgroundTransparency = 1
ballHeader.Text  = "Custom Game Ball Engine"
ballHeader.TextColor3 = Color3.new(1,1,1)
ballHeader.Font  = Enum.Font.GothamBold
ballHeader.TextSize = 15
ballHeader.TextXAlignment = Enum.TextXAlignment.Left

local assetLabel = Instance.new("TextLabel", ballCard)
assetLabel.Size  = UDim2.new(1,-30, 0, 15)
assetLabel.Position = UDim2.new(0,15, 0, 42)
assetLabel.BackgroundTransparency = 1
assetLabel.Text  = "Ball skin asset id"
assetLabel.TextColor3 = theme.text_dim
assetLabel.Font  = Enum.Font.GothamBold
assetLabel.TextSize = 9
assetLabel.TextXAlignment = Enum.TextXAlignment.Left

local ballNameInput = Instance.new("TextBox", ballCard)
ballNameInput.Size  = UDim2.new(1,-30, 0, 32)
ballNameInput.Position = UDim2.new(0,15, 0, 56)
ballNameInput.BackgroundColor3 = theme.bg_sidebar
ballNameInput.Text  = ""
ballNameInput.PlaceholderText = "Paste Texture Asset ID..."
ballNameInput.PlaceholderColor3 = theme.outline
ballNameInput.TextColor3 = theme.accent
ballNameInput.Font  = Enum.Font.GothamMedium
ballNameInput.TextSize = 12
applyStyle(ballNameInput, 6, true)

local colorLabel = Instance.new("TextLabel", ballCard)
colorLabel.Size  = UDim2.new(1,-30, 0, 15)
colorLabel.Position = UDim2.new(0,15, 0, 96)
colorLabel.BackgroundTransparency = 1
colorLabel.Text  = "Local solid color"
colorLabel.TextColor3 = theme.text_dim
colorLabel.Font  = Enum.Font.GothamBold
colorLabel.TextSize = 9
colorLabel.TextXAlignment = Enum.TextXAlignment.Left

local ballColorInput = Instance.new("TextBox", ballCard)
ballColorInput.Size  = UDim2.new(1,-30, 0, 32)
ballColorInput.Position = UDim2.new(0,15, 0, 110)
ballColorInput.BackgroundColor3 = theme.bg_sidebar
ballColorInput.Text  = ""
ballColorInput.PlaceholderText = "e.g., Red, 255,0,0, or #FF0000"
ballColorInput.PlaceholderColor3 = theme.outline
ballColorInput.TextColor3 = theme.accent
ballColorInput.Font  = Enum.Font.GothamMedium
ballColorInput.TextSize = 12
applyStyle(ballColorInput, 6, true)

local applyBall = Instance.new("TextButton", ballCard)
applyBall.Size  = UDim2.new(1,-30, 0, 36)
applyBall.Position = UDim2.new(0,15, 0, 160)
applyBall.BackgroundColor3 = theme.accent
applyBall.Text  = "Apply to ball"
applyBall.Font  = Enum.Font.GothamBold
applyBall.TextColor3 = Color3.new(1,1,1)
applyBall.TextSize = 12
applyBall.AutoButtonColor = false
local ballBtnStroke = applyStyle(applyBall, 6, true, theme.accent_light)
ballBtnStroke.Transparency = 0.5

applyBall.MouseEnter:Connect(function()
    TweenService:Create(applyBall, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent_light}):Play()
    TweenService:Create(ballBtnStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
end)
applyBall.MouseLeave:Connect(function()
    TweenService:Create(applyBall, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent}):Play()
    TweenService:Create(ballBtnStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
end)

local ballPresets = {
    white  = Color3.fromRGB(255,255,255), black = Color3.fromRGB(0,0,0),
    red    = Color3.fromRGB(255,0,0),     blue  = Color3.fromRGB(0,0,255),
    green  = Color3.fromRGB(0,255,0),     yellow= Color3.fromRGB(255,255,0),
    purple = Color3.fromRGB(150,0,255),   pink  = Color3.fromRGB(255,100,200),
    orange = Color3.fromRGB(255,127,0),   cyan  = Color3.fromRGB(0,255,255),
    grey   = Color3.fromRGB(127,127,127), gray  = Color3.fromRGB(127,127,127),
    brown  = Color3.fromRGB(139,69,19),    lime  = Color3.fromRGB(50,205,50),
    gold   = Color3.fromRGB(255,215,0),   silver= Color3.fromRGB(192,192,192),
    magenta= Color3.fromRGB(255,0,255)
}

local function parseColor(str)
    str = str:lower():gsub("%s+", "")
    if ballPresets[str] then
        return ballPresets[str]
    end
    -- Try matching R,G,B format
    local r, g, b = str:match("^(%d+),(%d+),(%d+)$")
    if r and g and b then
        r, g, b = tonumber(r), tonumber(g), tonumber(b)
        if r and g and b then
            return Color3.fromRGB(math.clamp(r, 0, 255), math.clamp(g, 0, 255), math.clamp(b, 0, 255))
        end
    end
    -- Try matching Hex format
    local hexClean = str:gsub("^#", "")
    if #hexClean == 6 and hexClean:match("^[%da-f]+$") then
        local hr = tonumber(hexClean:sub(1, 2), 16)
        local hg = tonumber(hexClean:sub(3, 4), 16)
        local hb = tonumber(hexClean:sub(5, 6), 16)
        if hr and hg and hb then
            return Color3.fromRGB(hr, hg, hb)
        end
    end
    return nil
end

local currentBallAssetId = ""
local currentBallColor   = nil
local ballUpdateConnection = nil

local function getBallPart(obj)
    if not obj then return nil end
    if obj:IsA("BasePart") then
        if obj.Name == "Ball" or obj.Name == "Football" or obj.Name == "TPSBall" then
            return obj
        end
        local parent = obj.Parent
        if parent and (parent.Name == "Ball" or parent.Name == "Football" or parent.Name == "TPSBall" or parent:FindFirstChild("BallScript")) then
            return obj
        end
    elseif obj:IsA("Model") then
        if obj.Name == "Ball" or obj.Name == "Football" or obj.Name == "TPSBall" or obj:FindFirstChild("BallScript") then
            return obj:FindFirstChildOfClass("BasePart")
        end
    end
    return nil
end

local function forceBallSkin(part)
    if not part or not part:IsA("BasePart") then return end
    pcall(function()
        if currentBallColor then
            for _, subObj in ipairs(part:GetDescendants()) do
                if subObj:IsA("Decal") or subObj:IsA("Texture") then
                    subObj.Texture = ""
                    subObj.Transparency = 1
                elseif subObj:IsA("SurfaceAppearance") then
                    subObj:Destroy()
                elseif subObj:IsA("SpecialMesh") then
                    subObj.TextureId = ""
                end
            end
            if part:IsA("MeshPart") then part.TextureID = "" end
            part.Color    = currentBallColor
            part.Material = Enum.Material.SmoothPlastic
        elseif currentBallAssetId ~= "" then
            local textureUrl = "rbxassetid://"..currentBallAssetId
            if part:IsA("MeshPart") then
                part.TextureID = textureUrl
            elseif part:FindFirstChildOfClass("SpecialMesh") then
                part:FindFirstChildOfClass("SpecialMesh").TextureId = textureUrl
            else
                local d = part:FindFirstChildOfClass("Decal") or Instance.new("Decal", part)
                d.Texture = textureUrl
                d.Transparency = 0
            end
            for _, child in pairs(part:GetDescendants()) do
                if child:IsA("Decal") or child:IsA("Texture") then
                    child.Texture = textureUrl
                    child.Transparency = 0
                end
            end
        end
    end)
end

local cachedBalls = {}
local function updateCachedBalls()
    cachedBalls = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        local part = getBallPart(v)
        if part and not table.find(cachedBalls, part) then
            table.insert(cachedBalls, part)
        end
    end
    for _, v in ipairs(game.Workspace.CurrentCamera:GetDescendants()) do
        local part = getBallPart(v)
        if part and not table.find(cachedBalls, part) then
            table.insert(cachedBalls, part)
        end
    end
end

applyBall.MouseButton1Click:Connect(function()
    local cleanAssetId = ballNameInput.Text:match("%d+")
    local parsedColor = parseColor(ballColorInput.Text)
    if parsedColor then
        currentBallColor   = parsedColor
        currentBallAssetId = ""
    elseif cleanAssetId then
        currentBallAssetId = cleanAssetId
        currentBallColor   = nil
    else
        currentBallColor   = nil
        currentBallAssetId = ""
    end
    if currentBallColor or currentBallAssetId ~= "" then
        if currentBallAssetId ~= "" then
            pcall(function()
                local remotes = ReplicatedStorage:WaitForChild("Remotes", 5)
                local ev = remotes and remotes:WaitForChild("CharacterSoundEvent", 5)
                if ev then ev:FireServer("Game","Customization","BallTexture", ballNameInput.Text) end
            end)
        end
        applyBall.BackgroundColor3 = theme.success
        applyBall.Text = "Ball mod applied"
        
        updateCachedBalls()
        for _, v in ipairs(cachedBalls) do forceBallSkin(v) end
        
        if not ballUpdateConnection then
            ballUpdateConnection = game.Workspace.DescendantAdded:Connect(function(newObj)
                task.wait(0.05)
                local part = getBallPart(newObj)
                if part and not table.find(cachedBalls, part) then
                    table.insert(cachedBalls, part)
                    forceBallSkin(part)
                end
            end)
            task.spawn(function()
                while currentBallColor or currentBallAssetId ~= "" do
                    for i = #cachedBalls, 1, -1 do
                        local v = cachedBalls[i]
                        if v and v.Parent then
                            forceBallSkin(v)
                        else
                            table.remove(cachedBalls, i)
                        end
                    end
                    task.wait(0.5)
                end
                -- If we exit the loop, clean up the connection so it can be re-established if toggled again
                if ballUpdateConnection then
                    pcall(function() ballUpdateConnection:Disconnect() end)
                    ballUpdateConnection = nil
                end
            end)
        end
        task.wait(1.5)
        applyBall.BackgroundColor3 = theme.accent
        applyBall.Text = "Apply to ball"
    else
        applyBall.Text = "Ball not found / invalid"
        applyBall.BackgroundColor3 = theme.danger
        task.wait(1.5)
        applyBall.BackgroundColor3 = theme.accent
        applyBall.Text = "Apply to ball"
    end
end)

--==================================================
-- FPS TAB (Scrollable)
--==================================================
local fpsCard = Instance.new("Frame", pages.FPS)
fpsCard.Size  = UDim2.new(1, 0, 0, 130)
fpsCard.BackgroundColor3 = Color3.new(1,1,1)
fpsCard.BackgroundTransparency = 0.45
applyStyle(fpsCard, 10, true)
applyGradient(fpsCard, theme.bg_card, Color3.fromRGB(18,38,25))

local fpsHeader = Instance.new("TextLabel", fpsCard)
fpsHeader.Size  = UDim2.new(1,-30, 0, 30)
fpsHeader.Position = UDim2.new(0,15, 0, 10)
fpsHeader.BackgroundTransparency = 1
fpsHeader.Text  = "Performance Engine"
fpsHeader.TextColor3 = Color3.new(1,1,1)
fpsHeader.Font  = Enum.Font.GothamBold
fpsHeader.TextSize = 15
fpsHeader.TextXAlignment = Enum.TextXAlignment.Left

local fpsInputFrame = Instance.new("Frame", fpsCard)
fpsInputFrame.Size  = UDim2.new(1,-30, 0, 36)
fpsInputFrame.Position = UDim2.new(0,15, 0, 42)
fpsInputFrame.BackgroundColor3 = theme.bg_sidebar
applyStyle(fpsInputFrame, 6, true)

local fpsBox = Instance.new("TextBox", fpsInputFrame)
fpsBox.Size  = UDim2.new(0.7,-10, 1, 0)
fpsBox.Position = UDim2.new(0,10, 0, 0)
fpsBox.BackgroundTransparency = 1
fpsBox.PlaceholderText = "Enter custom limit..."
fpsBox.PlaceholderColor3 = theme.outline
fpsBox.Text  = ""
fpsBox.TextColor3 = theme.accent
fpsBox.Font  = Enum.Font.GothamMedium
fpsBox.TextSize = 13
fpsBox.TextXAlignment = Enum.TextXAlignment.Left

local setFpsBtn = Instance.new("TextButton", fpsInputFrame)
setFpsBtn.Size  = UDim2.new(0.25, 0, 0.8, 0)
setFpsBtn.Position = UDim2.new(1,-5, 0.5, 0)
setFpsBtn.AnchorPoint = Vector2.new(1, 0.5)
setFpsBtn.BackgroundColor3 = theme.accent
setFpsBtn.Text  = "SET"
setFpsBtn.Font  = Enum.Font.GothamBold
setFpsBtn.TextColor3 = Color3.new(1,1,1)
setFpsBtn.TextSize = 12
setFpsBtn.AutoButtonColor = false
local setFpsStroke = applyStyle(setFpsBtn, 4, true, theme.accent_light)
setFpsStroke.Transparency = 0.5

setFpsBtn.MouseEnter:Connect(function()
    TweenService:Create(setFpsBtn, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent_light}):Play()
    TweenService:Create(setFpsStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
end)
setFpsBtn.MouseLeave:Connect(function()
    TweenService:Create(setFpsBtn, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent}):Play()
    TweenService:Create(setFpsStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
end)

local function updateFPS(val)
    local n = tonumber(val)
    if setfpscap and n then
        setfpscap(n)
        fpsBox.Text = tostring(n)
    end
end
setFpsBtn.MouseButton1Click:Connect(function() updateFPS(fpsBox.Text) end)
fpsBox.FocusLost:Connect(function(enter) if enter then updateFPS(fpsBox.Text) end end)

local fpsBtnRow = Instance.new("Frame", pages.FPS)
fpsBtnRow.Size  = UDim2.new(1, 0, 0, 28)
fpsBtnRow.BackgroundTransparency = 1

local fpsValues = {60, 144, 360, 999}
for idx, v in ipairs(fpsValues) do
    local pb = Instance.new("TextButton", fpsBtnRow)
    pb.Size  = UDim2.new(0.22, 0, 1, 0)
    pb.Position = UDim2.new((idx-1)*0.26, 0, 0, 0)
    pb.BackgroundColor3 = theme.bg_sidebar
    pb.Text  = v == 999 and "Uncap" or tostring(v)
    pb.Font  = Enum.Font.GothamMedium
    pb.TextColor3 = theme.text_dim
    pb.TextSize = 12
    pb.AutoButtonColor = false
    local pbStroke = applyStyle(pb, 4, true)
    pbStroke.Transparency = 0.5
    pb.MouseEnter:Connect(function()
        TweenService:Create(pb, TweenInfo.new(0.2), {BackgroundColor3 = theme.accent, TextColor3 = Color3.new(1,1,1)}):Play()
        TweenService:Create(pbStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
    end)
    pb.MouseLeave:Connect(function()
        TweenService:Create(pb, TweenInfo.new(0.2), {BackgroundColor3 = theme.bg_sidebar, TextColor3 = theme.text_dim}):Play()
        TweenService:Create(pbStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
    end)
    pb.MouseButton1Click:Connect(function() updateFPS(v) end)
end

--==================================================
-- FAST FLAGS TAB (Scrollable)
--==================================================
local FF_PATH = "zaystrap_flags.json"

-- Pre-load saved flags from file
local ffSavedText = '{\n\n}'
pcall(function()
    if isfile and isfile(FF_PATH) then
        local s = readfile(FF_PATH)
        if s and s ~= "" then ffSavedText = s end
    end
end)

local ffBox = Instance.new("TextBox", pages["Fast Flags"])
ffBox.Size          = UDim2.new(1, 0, 0, 140)
ffBox.BackgroundColor3 = theme.bg_sidebar
ffBox.BackgroundTransparency = 0.2
ffBox.TextColor3    = theme.text
ffBox.PlaceholderColor3 = theme.text_dim
ffBox.PlaceholderText = '{"FFlagName": "Value"}'
ffBox.Font          = Enum.Font.Code
ffBox.TextSize      = 13
ffBox.MultiLine     = true
ffBox.ClearTextOnFocus = false
ffBox.TextXAlignment = Enum.TextXAlignment.Left
ffBox.TextYAlignment = Enum.TextYAlignment.Top
ffBox.TextWrapped   = true
ffBox.Text          = ffSavedText
applyStyle(ffBox, 6, true)

local ffStatus = Instance.new("TextLabel", pages["Fast Flags"])
ffStatus.Size  = UDim2.new(1, 0, 0, 22)
ffStatus.BackgroundTransparency = 1
ffStatus.Text  = "Ready to inject..."
ffStatus.TextColor3 = theme.text_dim
ffStatus.Font  = Enum.Font.GothamMedium
ffStatus.TextSize = 12
ffStatus.TextXAlignment = Enum.TextXAlignment.Left

MakeButton(pages["Fast Flags"], "Inject Flags", function(btn)
    local text = ffBox.Text
    local ok, data = pcall(function() return HttpService:JSONDecode(text) end)
    if ok then
        -- Save to file
        pcall(function() if writefile then writefile(FF_PATH, text) end end)
        local jumpTotal = 0
        for k, val in pairs(data) do
            -- Strip all known prefixes (Fluidstrap approach)
            local clean = k
                :gsub("DFInt",    "")
                :gsub("DFFlag",   "")
                :gsub("FFlag",    "")
                :gsub("FInt",     "")
                :gsub("DFString", "")
                :gsub("FString",  "")
            pcall(setfflag, clean, tostring(val))
            -- Track jump flags → wire into stealthPhysics
            if k:lower():find("jump") then jumpTotal = jumpTotal + 3 end
        end
        -- Update stealthPhysics if jump flags were found
        if jumpTotal > 0 then
            stealthPhysics.active = true
            stealthPhysics.jumpVelocity = jumpTotal
        end
        ffStatus.Text = "Injected " .. tostring(#data and (function() local n=0 for _ in pairs(data) do n=n+1 end return n end)() or 0) .. " flags"
        ffStatus.TextColor3 = theme.success
        btn.Text = "Injected ✓"
        task.delay(2, function() btn.Text = "Inject Flags" end)
    else
        ffStatus.Text = "Invalid JSON — check your format"
        ffStatus.TextColor3 = theme.danger
        btn.Text = "JSON Error"
        task.delay(2, function() btn.Text = "Inject Flags" end)
    end
end)

MakeButton(pages["Fast Flags"], "Clear Saved Flags", function()
    ffBox.Text = '{\n\n}'
    ffStatus.Text = "Cleared."
    ffStatus.TextColor3 = theme.text_dim
    stealthPhysics.active = false
    stealthPhysics.jumpVelocity = 0
    pcall(function() if writefile then writefile(FF_PATH, "") end end)
end)

--==================================================
-- DISCORD TAB
--==================================================
local dLabel = Instance.new("TextLabel", pages["Discord"])
dLabel.Size  = UDim2.new(1, 0, 0, 25)
dLabel.BackgroundTransparency = 1
dLabel.Text  = "Official discord community"
dLabel.TextColor3 = theme.accent
dLabel.Font  = Enum.Font.GothamBold
dLabel.TextSize = 11
dLabel.TextXAlignment = Enum.TextXAlignment.Left

MakeButton(pages["Discord"], "Copy Discord Invite Link", function(btn)
    if setclipboard then setclipboard("https://discord.gg/XkVHKVfTju") end
    local oldText = btn.Text
    btn.Text = "Invite copied!"
    btn.BackgroundColor3 = theme.success
    task.wait(1.5)
    btn.Text = oldText
    btn.BackgroundColor3 = theme.accent
end)

--==================================================
-- PROFILE TAB
--==================================================
MakeSectionLabel(pages.Profile, "User profile")

local profileCard = Instance.new("Frame", pages.Profile)
profileCard.Size  = UDim2.new(1, 0, 0, 120)
profileCard.BackgroundColor3 = theme.bg_card
profileCard.BackgroundTransparency = 0.3
applyStyle(profileCard, 10, true)

local pfpImage = Instance.new("ImageLabel", profileCard)
pfpImage.Size  = UDim2.new(0, 90, 0, 90)
pfpImage.Position = UDim2.new(0, 15, 0.5, -45)
pfpImage.BackgroundColor3 = theme.bg_sidebar
pfpImage.Image = cfg.profilePic or "rbxassetid://6033788226"
applyStyle(pfpImage, 45, true) -- circular

local profileUser = Instance.new("TextLabel", profileCard)
profileUser.Size  = UDim2.new(1, -130, 0, 25)
profileUser.Position = UDim2.new(0, 120, 0, 20)
profileUser.BackgroundTransparency = 1
profileUser.Text  = "User: " .. (cfg.username or "Zay")
profileUser.TextColor3 = theme.accent
profileUser.Font  = Enum.Font.GothamBold
profileUser.TextSize = 16
profileUser.TextXAlignment = Enum.TextXAlignment.Left

local profileStatus = Instance.new("TextLabel", profileCard)
profileStatus.Size  = UDim2.new(1, -130, 0, 20)
profileStatus.Position = UDim2.new(0, 120, 0, 45)
profileStatus.BackgroundTransparency = 1
profileStatus.Text  = "Rank: Premium Client"
profileStatus.TextColor3 = theme.text
profileStatus.Font  = Enum.Font.GothamMedium
profileStatus.TextSize = 12
profileStatus.TextXAlignment = Enum.TextXAlignment.Left

local profileStatus2 = Instance.new("TextLabel", profileCard)
profileStatus2.Size  = UDim2.new(1, -130, 0, 20)
profileStatus2.Position = UDim2.new(0, 120, 0, 65)
profileStatus2.BackgroundTransparency = 1
profileStatus2.Text  = "Joined: 2026"
profileStatus2.TextColor3 = theme.text_dim
profileStatus2.Font  = Enum.Font.GothamMedium
profileStatus2.TextSize = 11
profileStatus2.TextXAlignment = Enum.TextXAlignment.Left

MakeSectionLabel(pages.Profile, "Edit profile")

local editUsernameBox = MakeInput(pages.Profile, "Change Username", cfg.username or "Zay", function(val)
    if val ~= "" then
        cfg.username = val
        profileUser.Text = "User: " .. val
        saveConfig()
    end
end)

local editPasswordBox = MakeInput(pages.Profile, "Change Password", cfg.password or "revampv4", function(val)
    if val ~= "" then
        cfg.password = val
        saveConfig()
    end
end)

local editKeyBox = MakeInput(pages.Profile, "Change Key", cfg.key or "Zaystrap2026", function(val)
    if val ~= "" then
        cfg.key = val
        saveConfig()
    end
end)

local editPfpBox = MakeInput(pages.Profile, "Profile Picture Asset ID", cfg.profilePic or "rbxassetid://6033788226", function(val)
    if val ~= "" then
        local pfpUrl = val:match("%d+") and ("rbxassetid://" .. val:match("%d+")) or val
        cfg.profilePic = pfpUrl
        pfpImage.Image = pfpUrl
        saveConfig()
    end
end)

--==================================================
-- SETTINGS TAB
--==================================================
MakeSectionLabel(pages.Settings, "Theme customizer")

MakeColorRow(pages.Settings, "UI Accent Color (RGB)", math.round(theme.accent.R * 255), math.round(theme.accent.G * 255), math.round(theme.accent.B * 255), function(r, g, b)
    updateThemeColors(Color3.fromRGB(r, g, b))
end)

MakeSectionLabel(pages.Settings, "Interface customization")

local toggleBar, toggleZ

toggleBar = createToggle(pages.Settings, "Minimize to Floating Bar (- and X)", function(s)
    if s then
        cfg.minimizedType = "bar"
        saveConfig()
        if toggleZ then toggleZ.SetState(false) end
    else
        cfg.minimizedType = "z"
        saveConfig()
        if toggleZ then toggleZ.SetState(true) end
    end
end, cfg.minimizedType ~= "z")

toggleZ = createToggle(pages.Settings, "Minimize to Z Button Widget", function(s)
    if s then
        cfg.minimizedType = "z"
        saveConfig()
        if toggleBar then toggleBar.SetState(false) end
    else
        cfg.minimizedType = "bar"
        saveConfig()
        if toggleBar then toggleBar.SetState(true) end
    end
end, cfg.minimizedType == "z")

createToggle(pages.Settings, "Show Top Bar Close/Minimize Buttons", function(s)
    cfg.showTopBarControls = s
    saveConfig()
    updateControlsVisibility()
end, cfg.showTopBarControls == true)

MakeSectionLabel(pages.Settings, "Z button")

createToggle(pages.Settings, "Show Z Overlay While UI Is Open", function(s)
    cfg.zButtonVisible = s
    saveConfig()
    -- Show or hide the floating Z button immediately (only while main is visible)
    if main.Visible then
        zButton.Visible = s
    end
end, cfg.zButtonVisible == true)


--==================================================
-- PHYSICS & LOGIC LOOPS
--==================================================
RunService.RenderStepped:Connect(function(dt)
    if screenFps.Visible then
        screenFps.Text = "FPS: "..math.round(1/dt)
    end
end)

RunService.RenderStepped:Connect(function()
    if not jpEnabled then return end
    if not hrp or not hrp.Parent or not hum or not hum.Parent then return end
    
    local state = hum:GetState()
    if not (state == Enum.HumanoidStateType.Freefall or state == Enum.HumanoidStateType.Jumping) then
        return
    end

    local myPos = hrp.Position
    for _, targetPlayer in ipairs(Players:GetPlayers()) do
        if targetPlayer ~= localPlayer and targetPlayer.Character then
            local head = targetPlayer.Character:FindFirstChild("Head")
            if head then
                local distance = (myPos - head.Position).Magnitude
                if distance < 5 and not hasBoosted then
                    hrp.AssemblyLinearVelocity = Vector3.new(
                        hrp.AssemblyLinearVelocity.X, 54, hrp.AssemblyLinearVelocity.Z)
                    hasBoosted = true
                    task.delay(0.1, function()
                        hasBoosted = false
                    end)
                    break
                end
            end
        end
    end
end)

local function runAntiAce()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid", 5)
    if not humanoid then return end
    local lastState = humanoid:GetState()
    while true do
        task.wait(0.01)
        if antiAceEnabled and player.Character == character and humanoid and humanoid.Parent then
            pcall(function()
                local state = humanoid:GetState()
                if state == Enum.HumanoidStateType.Jumping and
                   (lastState == Enum.HumanoidStateType.Running or lastState == Enum.HumanoidStateType.Landed) then
                    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
                    task.wait(0.01)
                    if humanoid and humanoid.Parent then
                        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                    end
                end
                lastState = state
            end)
        elseif player.Character ~= character then 
            break 
        end
    end
end
task.spawn(runAntiAce)
player.CharacterAdded:Connect(function() task.spawn(runAntiAce) end)

local macroConnection = nil
local function setupMacros(ch)
    if macroConnection then
        pcall(function() macroConnection:Disconnect() end)
        macroConnection = nil
    end
    local humObj = ch:WaitForChild("Humanoid", 5)
    local hrpObj = ch:WaitForChild("HumanoidRootPart", 5)
    if not humObj or not hrpObj then return end
    macroConnection = humObj.Jumping:Connect(function()
        if angleEnabled and hrpObj and hrpObj.Parent then
            task.wait(0.001)
            if hrpObj and hrpObj.Parent then
                hrpObj.AssemblyLinearVelocity = Vector3.new(
                    hrpObj.AssemblyLinearVelocity.X, 44, hrpObj.AssemblyLinearVelocity.Z)
            end
        end
    end)
end
if player.Character then setupMacros(player.Character) end
player.CharacterAdded:Connect(setupMacros)

--==================================================
-- INIT
--==================================================
layoutUI()
updateControlsVisibility()

-- Initialize zButton visibility from saved config
zButton.Visible = false -- always hidden at start; only shown when minimized

if keyVerified then
    -- Key already saved — skip key screen
    keyFrame:Destroy()
    main.Visible = true
    toggleUI("restore")
else
    keyFrame.Visible = true
    main.Visible     = false
    topBar.Visible   = false
    zButton.Visible  = false
end
