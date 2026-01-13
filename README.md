local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local Settings = {
    TP_Key = Enum.KeyCode.Q,
    Menu_Key = Enum.KeyCode.Insert,
    Noclip_Key = Enum.KeyCode.N,
    Fly_Key = Enum.KeyCode.F,
    Aim_Key = Enum.UserInputType.MouseButton2, -- Yeni: Aim tuşu ayarı
    ESP_Enabled = false,
    Team_Check = true,
    Names_Enabled = false,
    InfJump = false,
    Noclip = false,
    Fly = false,
    FullBright = false,
    FlySpeed = 101,
    Max_Dist = 2500,
    Binding = false,
    Running = true,
    AimVisible = false,
    Aimbot_Enabled = false,
    AimPart = "Head",
    AimFOV = 150,
    ShowFOV = true, -- Yeni: FOV Görünürlüğü
    Smoothness = 1,
    TargetLock = nil -- Yeni: Rivals için kilit sabitleyici
}

local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "OcTunaFinal"
ScreenGui.ResetOnSpawn = false

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1
FOVCircle.Color = Color3.new(1, 1, 1)
FOVCircle.Transparency = 0.7
FOVCircle.Visible = false

local NotifyFrame = Instance.new("Frame", ScreenGui)
NotifyFrame.Size = UDim2.new(0, 160, 0, 35)
NotifyFrame.Position = UDim2.new(1, -180, 1, -85)
NotifyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
NotifyFrame.BackgroundTransparency = 0.4
NotifyFrame.Visible = false
Instance.new("UICorner", NotifyFrame)

local NotifyLabel = Instance.new("TextLabel", NotifyFrame)
NotifyLabel.Size = UDim2.new(1, 0, 1, 0)
NotifyLabel.BackgroundTransparency = 1
NotifyLabel.Font = Enum.Font.SourceSansBold
NotifyLabel.TextSize = 18

local function ShowNotify(feature, state)
    if not Settings.Running then return end
    NotifyFrame.Visible = true
    NotifyLabel.Text = feature .. ": " .. (state and "ON" or "OFF")
    NotifyLabel.TextColor3 = state and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    task.spawn(function()
        task.wait(2)
        if NotifyLabel.Text == (feature .. ": " .. (state and "ON" or "OFF")) then
            NotifyFrame.Visible = false
        end
    end)
end

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 300, 0, 540)
MainFrame.Position = UDim2.new(0.5, -350, 0.5, -270)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Instance.new("UICorner", MainFrame)

local Title = Instance.new("TextButton", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Title.Text = "OçTunaV1.1"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Instance.new("UICorner", Title)

local dragToggle, dragStart, startPos
Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragToggle = true dragStart = input.Position startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragToggle then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragToggle = false end end)

local function CreateRow(parent, text, yPos, hasKey)
    local btn = Instance.new("TextButton", parent)
    btn.Size = hasKey and UDim2.new(0, 200, 0, 32) or UDim2.new(0, 270, 0, 32)
    btn.Position = UDim2.new(0.05, 0, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    Instance.new("UICorner", btn)
    local kBtn = nil
    if hasKey then
        kBtn = Instance.new("TextButton", parent)
        kBtn.Size = UDim2.new(0, 50, 0, 32)
        kBtn.Position = UDim2.new(0, 225, 0, yPos)
        kBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        kBtn.TextColor3 = Color3.fromRGB(0, 255, 120)
        Instance.new("UICorner", kBtn)
    end
    return btn, kBtn
end

local TPBtn = CreateRow(MainFrame, "TP Key: " .. Settings.TP_Key.Name, 55, false)
local ESPBtn = CreateRow(MainFrame, "ESP: OFF", 95, false)
local NameBtn = CreateRow(MainFrame, "Names: OFF", 135, false)
local TeamBtn = CreateRow(MainFrame, "Team Check: ON", 175, false)
local JumpBtn = CreateRow(MainFrame, "Inf Jump: OFF", 215, false)
local NocBtn, NocKey = CreateRow(MainFrame, "Noclip: OFF", 255, true)
local FlyBtn, FlyKey = CreateRow(MainFrame, "Fly: OFF", 295, true)
local FBBtn = CreateRow(MainFrame, "FullBright: OFF", 430, false)
local OpenAimBtn = CreateRow(MainFrame, "AIMBOT MENU >>", 390, false)

OpenAimBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
NocKey.Text = Settings.Noclip_Key.Name FlyKey.Text = Settings.Fly_Key.Name

local AimFrame = Instance.new("Frame", ScreenGui)
AimFrame.Size = UDim2.new(0, 380, 0, 400)
AimFrame.Position = UDim2.new(0.5, 10, 0.5, -200)
AimFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
AimFrame.Visible = false
Instance.new("UICorner", AimFrame)

local AimTitle = Instance.new("TextButton", AimFrame)
AimTitle.Size = UDim2.new(1, 0, 0, 45)
AimTitle.BackgroundColor3 = Color3.fromRGB(0, 80, 150)
AimTitle.Text = "AimBot Tuna Oç"
AimTitle.TextColor3 = Color3.new(1, 1, 1)
AimTitle.Font = Enum.Font.SourceSansBold
AimTitle.TextSize = 18
Instance.new("UICorner", AimTitle)

local aDragToggle, aDragStart, aStartPos
AimTitle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        aDragToggle = true aDragStart = input.Position aStartPos = AimFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and aDragToggle then
        local delta = input.Position - aDragStart
        AimFrame.Position = UDim2.new(aStartPos.X.Scale, aStartPos.X.Offset + delta.X, aStartPos.Y.Scale, aStartPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then aDragToggle = false end end)

local AimTog = CreateRow(AimFrame, "Enable Aimbot: OFF", 60, false)
local AimPartBtn = CreateRow(AimFrame, "Target: Head", 100, false)
local AimKeyBtn, AimKeyLabel = CreateRow(AimFrame, "Aim Key", 140, true) -- Yeni Satır
local FOVVisibleBtn = CreateRow(AimFrame, "Show FOV: ON", 180, false) -- Yeni Satır
AimKeyLabel.Text = "MB2"

local function GetClosestPlayer()
    local target = nil
    local dist = Settings.AimFOV
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Settings.AimPart) then
            if Settings.Team_Check and v.Team == LocalPlayer.Team then continue end
            local hum = v.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health <= 0 then continue end
            local pos, vis = Camera:WorldToViewportPoint(v.Character[Settings.AimPart].Position)
            if vis then
                local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if mag < dist then
                    target = v.Character[Settings.AimPart]
                    dist = mag
                end
            end
        end
    end
    return target
end

local SliderFrame = Instance.new("Frame", MainFrame)
SliderFrame.Size = UDim2.new(0, 270, 0, 40)
SliderFrame.Position = UDim2.new(0.05, 0, 0, 340)
SliderFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Instance.new("UICorner", SliderFrame)
local SliderLabel = Instance.new("TextLabel", SliderFrame)
SliderLabel.Size = UDim2.new(1, 0, 0, 15)
SliderLabel.Text = "Fly Speed: 101"
SliderLabel.TextColor3 = Color3.new(1, 1, 1)
SliderLabel.BackgroundTransparency = 1
local SliderBar = Instance.new("Frame", SliderFrame)
SliderBar.Size = UDim2.new(0.8, 0, 0, 5)
SliderBar.Position = UDim2.new(0.1, 0, 0.65, 0)
SliderBar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
local SliderFill = Instance.new("Frame", SliderBar)
SliderFill.Size = UDim2.new(0.33, 0, 1, 0)
SliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 120)

local function ToggleFly()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if Settings.Fly then
        hum.PlatformStand = true
        if hrp then
            local bv = Instance.new("BodyVelocity", hrp)
            bv.Name = "FlyVel" bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge) bv.Velocity = Vector3.new(0,0,0)
            local bg = Instance.new("BodyGyro", hrp)
            bg.Name = "FlyGyro" bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge) bg.P = 90000 bg.CFrame = hrp.CFrame
        end
    else
        hum.PlatformStand = false
        if hrp then
            if hrp:FindFirstChild("FlyVel") then hrp.FlyVel:Destroy() end
            if hrp:FindFirstChild("FlyGyro") then hrp.FlyGyro:Destroy() end
        end
    end
end

local function UpdateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local dist = (hrp.Position - Camera.CFrame.Position).Magnitude
                local shouldShow = Settings.ESP_Enabled and dist <= Settings.Max_Dist
                if Settings.Team_Check and player.Team == LocalPlayer.Team then shouldShow = false end
                local hl = char:FindFirstChild("OcTunaHL")
                if shouldShow then
                    if not hl then hl = Instance.new("Highlight", char) hl.Name = "OcTunaHL" end
                    hl.FillColor = player.TeamColor.Color hl.FillTransparency = 0.5
                else if hl then hl:Destroy() end end
                local tag = char:FindFirstChild("OcTunaTag")
                if shouldShow and Settings.Names_Enabled then
                    if not tag then
                        tag = Instance.new("BillboardGui", char)
                        tag.Name = "OcTunaTag" tag.Size = UDim2.new(0, 150, 0, 60)
                        tag.AlwaysOnTop = true tag.ExtentsOffset = Vector3.new(0, 3, 0)
                        local l = Instance.new("TextLabel", tag)
                        l.Size = UDim2.new(1, 0, 1, 0) l.BackgroundTransparency = 1 l.TextColor3 = player.TeamColor.Color l.TextSize = 14 l.Font = Enum.Font.RobotoCondensed l.TextStrokeTransparency = 0 l.TextStrokeColor3 = Color3.new(0, 0, 0) l.Parent = tag
                    end
                    tag.TextLabel.Text = player.Name .. "\n" .. math.floor(dist) .. "m"
                else if tag then tag:Destroy() end end
            end
        end
    end
end

FBBtn.MouseButton1Click:Connect(function()
    Settings.FullBright = not Settings.FullBright
    FBBtn.Text = "FullBright: " .. (Settings.FullBright and "ON" or "OFF")
    if Settings.FullBright then
        Lighting.Ambient = Color3.new(1, 1, 1)
        Lighting.ColorShift_Bottom = Color3.new(1, 1, 1)
        Lighting.ColorShift_Top = Color3.new(1, 1, 1)
    else
        Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
        Lighting.ColorShift_Bottom = Color3.new(0, 0, 0)
        Lighting.ColorShift_Top = Color3.new(0, 0, 0)
    end
end)

NocBtn.MouseButton1Click:Connect(function() 
    Settings.Noclip = not Settings.Noclip 
    NocBtn.Text = "Noclip: " .. (Settings.Noclip and "ON" or "OFF") 
    ShowNotify("Noclip", Settings.Noclip)
    if not Settings.Noclip and LocalPlayer.Character then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = true end end
    end
end)
FlyBtn.MouseButton1Click:Connect(function() Settings.Fly = not Settings.Fly FlyBtn.Text = "Fly: " .. (Settings.Fly and "ON" or "OFF") ShowNotify("Fly", Settings.Fly) ToggleFly() end)
JumpBtn.MouseButton1Click:Connect(function() Settings.InfJump = not Settings.InfJump JumpBtn.Text = "Inf Jump: " .. (Settings.InfJump and "ON" or "OFF") end)
ESPBtn.MouseButton1Click:Connect(function() Settings.ESP_Enabled = not Settings.ESP_Enabled ESPBtn.Text = "ESP: " .. (Settings.ESP_Enabled and "ON" or "OFF") end)
NameBtn.MouseButton1Click:Connect(function() Settings.Names_Enabled = not Settings.Names_Enabled NameBtn.Text = "Names: " .. (Settings.Names_Enabled and "ON" or "OFF") end)
TeamBtn.MouseButton1Click:Connect(function() Settings.Team_Check = not Settings.Team_Check TeamBtn.Text = "Team Check: " .. (Settings.Team_Check and "ON" or "OFF") end)
OpenAimBtn.MouseButton1Click:Connect(function() Settings.AimVisible = not Settings.AimVisible AimFrame.Visible = Settings.AimVisible end)
AimTog.MouseButton1Click:Connect(function() Settings.Aimbot_Enabled = not Settings.Aimbot_Enabled AimTog.Text = "Enable Aimbot: " .. (Settings.Aimbot_Enabled and "ON" or "OFF") end)
AimPartBtn.MouseButton1Click:Connect(function() local p = {"Head", "UpperTorso", "HumanoidRootPart"} local i = table.find(p, Settings.AimPart) or 1 Settings.AimPart = p[i % #p + 1] AimPartBtn.Text = "Target: " .. Settings.AimPart end)
FOVVisibleBtn.MouseButton1Click:Connect(function() Settings.ShowFOV = not Settings.ShowFOV FOVVisibleBtn.Text = "Show FOV: " .. (Settings.ShowFOV and "ON" or "OFF") end)

TPBtn.MouseButton1Click:Connect(function() Settings.Binding = "TP" TPBtn.Text = "..." end)
AimKeyLabel.MouseButton1Click:Connect(function() Settings.Binding = "Aim" AimKeyLabel.Text = "..." end)

local InputConn = UserInputService.InputBegan:Connect(function(input, p)
    if not Settings.Running then return end
    if Settings.Binding then
        local key = (input.UserInputType == Enum.UserInputType.Keyboard) and input.KeyCode or input.UserInputType
        if Settings.Binding == "TP" then
            Settings.TP_Key = key
            TPBtn.Text = "TP Key: " .. (input.UserInputType == Enum.UserInputType.Keyboard and key.Name or key.Name:gsub("MouseButton", "MB"))
        elseif Settings.Binding == "Aim" then
            Settings.Aim_Key = key
            AimKeyLabel.Text = (input.UserInputType == Enum.UserInputType.Keyboard and key.Name or key.Name:gsub("MouseButton", "MB"))
        end
        Settings.Binding = false
        return
    end
    if p then return end
    if input.KeyCode == Settings.Noclip_Key then 
        Settings.Noclip = not Settings.Noclip 
        NocBtn.Text = "Noclip: " .. (Settings.Noclip and "ON" or "OFF") 
        ShowNotify("Noclip", Settings.Noclip)
        if not Settings.Noclip and LocalPlayer.Character then
            for _, v in pairs(LocalPlayer.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = true end end
        end
    elseif input.KeyCode == Settings.Fly_Key then Settings.Fly = not Settings.Fly FlyBtn.Text = "Fly: " .. (Settings.Fly and "ON" or "OFF") ShowNotify("Fly", Settings.Fly) ToggleFly()
    elseif input.KeyCode == Settings.Menu_Key then MainFrame.Visible = not MainFrame.Visible AimFrame.Visible = false
    elseif input.KeyCode == Enum.KeyCode.Space and Settings.InfJump then if LocalPlayer.Character then LocalPlayer.Character.Humanoid:ChangeState(3) end
    elseif input.KeyCode == Settings.TP_Key or input.UserInputType == Settings.TP_Key then
        local m = LocalPlayer:GetMouse()
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(m.Hit.p + Vector3.new(0,3,0)) hrp.Velocity = Vector3.new(0,0,0) hrp.RotVelocity = Vector3.new(0,0,0) end
    end
end)

local SteppedConn = RunService.Stepped:Connect(function(_, dt)
    if not Settings.Running then return end
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp and Settings.Fly then
        local bv = hrp:FindFirstChild("FlyVel")
        local bg = hrp:FindFirstChild("FlyGyro")
        if bv and bg then
            local moveDir = Vector3.new(0,0,0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Camera.CFrame.RightVector end
            bv.Velocity = moveDir * Settings.FlySpeed
            bg.CFrame = Camera.CFrame
        end
    end
    if Settings.Noclip and char then
        for _, v in pairs(char:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end
    end
end)

local draggingSlider = false
SliderBar.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then draggingSlider = true end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then draggingSlider = false end end)

local RenderLoop = RunService.RenderStepped:Connect(function()
    if not Settings.Running then return end
    UpdateESP()
    
    FOVCircle.Visible = Settings.Aimbot_Enabled and Settings.ShowFOV
    FOVCircle.Position = UserInputService:GetMouseLocation()
    FOVCircle.Radius = Settings.AimFOV

    local isPressed = (Settings.Aim_Key.EnumType == Enum.KeyCode) and UserInputService:IsKeyDown(Settings.Aim_Key) or UserInputService:IsMouseButtonPressed(Settings.Aim_Key)

    if Settings.Aimbot_Enabled and isPressed then
        if not Settings.TargetLock or not Settings.TargetLock.Parent or Settings.TargetLock.Parent.Humanoid.Health <= 0 then
            Settings.TargetLock = GetClosestPlayer()
        end
        if Settings.TargetLock then
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Settings.TargetLock.Position), 1/Settings.Smoothness)
        end
    else
        Settings.TargetLock = nil
    end

    if draggingSlider then
        local percent = math.clamp((UserInputService:GetMouseLocation().X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
        SliderFill.Size = UDim2.new(percent, 0, 1, 0) Settings.FlySpeed = math.floor(percent * 300) SliderLabel.Text = "Fly Speed: " .. Settings.FlySpeed
    end
end)

local Credit = Instance.new("TextLabel", MainFrame)
Credit.Size = UDim2.new(1, 0, 0, 20)
Credit.Position = UDim2.new(0, 0, 1, -25)
Credit.BackgroundTransparency = 1
Credit.Text = "Made By Ataturk"
Credit.TextColor3 = Color3.new(1, 1, 1)
Credit.Font = Enum.Font.SourceSansItalic
Credit.TextSize = 14

local Unload = CreateRow(MainFrame, "UNLOAD SCRIPT", 480, false)
Unload.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
Unload.MouseButton1Click:Connect(function()
    Settings.Running = false Settings.Fly = false Settings.Noclip = false Settings.ESP_Enabled = false
    SteppedConn:Disconnect() RenderLoop:Disconnect() InputConn:Disconnect() FOVCircle:Remove()
    Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character then
            local hl = p.Character:FindFirstChild("OcTunaHL")
            local tag = p.Character:FindFirstChild("OcTunaTag")
            if hl then hl:Destroy() end if tag then tag:Destroy() end
        end
    end
    if LocalPlayer.Character then 
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = true end end 
        LocalPlayer.Character.Humanoid.PlatformStand = false 
        local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then if hrp:FindFirstChild("FlyVel") then hrp.FlyVel:Destroy() end if hrp:FindFirstChild("FlyGyro") then hrp.FlyGyro:Destroy() end end
    end
    ScreenGui:Destroy()
end)
