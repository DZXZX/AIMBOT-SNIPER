local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- // CONFIGURAÇÕES GLOBAIS //
_G.AimbotEnabled = false
_G.AimbotFOV = 150
_G.AimbotSmoothness = 0.08
_G.AimbotPart = "Head"
_G.WallCheck = true 
_G.ShowFOV = true
_G.SpeedEnabled = false
_G.SpeedMultiplier = 1 -- Agora controlado pelo Slider
_G.InfJumpEnabled = false
_G.JumpHeight = 50
_G.ESP_Boxes = false
_G.ESP_Tracers = false
_G.Whitelist = {}

local Camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // 1. FOV DZXZX //
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(255, 80, 0)
FOVCircle.Filled = false
FOVCircle.Visible = true

-- // 2. FIX BOTÃO DE PULO (ANTI-HIDE) //
RunService.Heartbeat:Connect(function()
    if _G.InfJumpEnabled then
        local pGui = LocalPlayer:FindFirstChildOfClass("PlayerGui")
        local touchGui = pGui and pGui:FindFirstChild("TouchGui")
        local touchFrame = touchGui and touchGui:FindFirstChild("TouchControlFrame")
        local jumpButton = touchFrame and touchFrame:FindFirstChild("JumpButton")
        if jumpButton then jumpButton.Visible = true jumpButton.Active = true end
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if _G.InfJumpEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.Velocity = Vector3.new(LocalPlayer.Character.HumanoidRootPart.Velocity.X, _G.JumpHeight, LocalPlayer.Character.HumanoidRootPart.Velocity.Z)
    end
end)

-- // 3. LÓGICA DE VISIBILIDADE & AIMBOT //
local function IsVisible(TargetPart)
    if not _G.WallCheck then return true end
    local RayParams = RaycastParams.new()
    RayParams.FilterType = Enum.RaycastFilterType.Exclude
    RayParams.FilterDescendantsInstances = {LocalPlayer.Character, TargetPart.Parent}
    local RayResult = workspace:Raycast(Camera.CFrame.Position, (TargetPart.Position - Camera.CFrame.Position), RayParams)
    return RayResult == nil
end

local function GetTarget()
    local Target = nil
    local MaxDist = _G.AimbotFOV
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and not table.find(_G.Whitelist, v.Name) then
            local Part = v.Character:FindFirstChild(_G.AimbotPart) or v.Character:FindFirstChild("HumanoidRootPart")
            if Part then
                local ScreenPos, OnScreen = Camera:WorldToViewportPoint(Part.Position)
                if OnScreen and IsVisible(Part) then
                    local MouseDist = (Vector2.new(ScreenPos.X, ScreenPos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
                    if MouseDist < MaxDist then Target = v MaxDist = MouseDist end
                end
            end
        end
    end
    return Target
end

-- // 4. RENDER LOOP (SPEED + AIM) //
RunService.RenderStepped:Connect(function()
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Radius = _G.AimbotFOV
    FOVCircle.Visible = _G.ShowFOV
    
    if _G.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 16 * _G.SpeedMultiplier
    elseif not _G.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end
    
    if _G.AimbotEnabled then
        local Target = GetTarget()
        if Target and Target.Character then 
            local Part = Target.Character:FindFirstChild(_G.AimbotPart) or Target.Character:FindFirstChild("HumanoidRootPart")
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Part.Position), _G.AimbotSmoothness) 
        end
    end
end)

-- // 5. INTERFACE DZXZX SNIPER V30 //
local Window = Rayfield:CreateWindow({
   Name = "DZXZX SNIPER | V30 SPEED FIX",
   LoadingTitle = "Injetando DZXZX System...",
   LoadingSubtitle = "Gemini Gostoso Edition",
   ConfigurationSaving = { Enabled = true, FolderName = "DZXZX_Configs", FileName = "SniperConfig" },
   Keybind = Enum.KeyCode.RightControl
})

local CombatTab = Window:CreateTab("🎯 Combate", 4483362458)
local MoveTab = Window:CreateTab("🏃 Speed/Jump", 4483362458)
local VisualTab = Window:CreateTab("👁️ Visual", 4483362458)
local WhiteTab = Window:CreateTab("🛡️ Whitelist", 4483362458)
local TeleTab = Window:CreateTab("🚀 Teleports", 4483362458)

-- 🏃 MOVIMENTAÇÃO (COM SLIDER DE VELOCIDADE)
MoveTab:CreateToggle({Name = "Ativar Speed Hack [ON/OFF]", CurrentValue = false, Flag = "SpeedToggle", Callback = function(v) _G.SpeedEnabled = v end})
MoveTab:CreateSlider({
    Name = "Controle de Velocidade",
    Range = {1, 15},
    Increment = 1,
    CurrentValue = 1,
    Flag = "SpeedSlider",
    Callback = function(v) _G.SpeedMultiplier = v end
})
MoveTab:CreateSection("Pulo")
MoveTab:CreateToggle({Name = "Infinite Jump [ON/OFF]", CurrentValue = false, Flag = "JumpToggle", Callback = function(v) _G.InfJumpEnabled = v end})
MoveTab:CreateSlider({Name = "Altura do Pulo", Range = {30, 150}, Increment = 5, CurrentValue = 50, Flag = "JumpHeight", Callback = function(v) _G.JumpHeight = v end})

-- 🎯 COMBATE
CombatTab:CreateToggle({Name = "Aimbot Legit [ON/OFF]", CurrentValue = false, Flag = "AimToggle", Callback = function(v) _G.AimbotEnabled = v end})
CombatTab:CreateToggle({Name = "Wall Check", CurrentValue = true, Flag = "WallToggle", Callback = function(v) _G.WallCheck = v end})
CombatTab:CreateDropdown({Name = "Alvo", Options = {"Head", "UpperTorso"}, CurrentOption = "Head", Flag = "PartDrop", Callback = function(Option) _G.AimbotPart = Option end})
CombatTab:CreateSlider({Name = "Grude", Range = {1, 50}, Increment = 1, CurrentValue = 8, Flag = "SmoothSlider", Callback = function(v) _G.AimbotSmoothness = v/100 end})
CombatTab:CreateSlider({Name = "Raio FOV", Range = {50, 600}, Increment = 10, CurrentValue = 150, Flag = "FOVSlider", Callback = function(v) _G.AimbotFOV = v end})

-- 👁️ VISUAL & OUTROS
VisualTab:CreateToggle({Name = "ESP Box", CurrentValue = false, Flag = "BoxToggle", Callback = function(v) _G.ESP_Boxes = v end})
VisualTab:CreateToggle({Name = "ESP Tracers", CurrentValue = false, Flag = "TracerToggle", Callback = function(v) _G.ESP_Tracers = v end})
VisualTab:CreateToggle({Name = "Ver FOV", CurrentValue = true, Flag = "ShowFOV", Callback = function(v) _G.ShowFOV = v end})

-- 🚀 TELEPORTES
TeleTab:CreateButton({Name = "⭐ FINAL (Coords)", Callback = function() if LocalPlayer.Character then LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-54.33042907714844, 5.094512462615967, 3642.275390625) end end})
TeleTab:CreateButton({Name = "🔄 Listar Players p/ Teleport", Callback = function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then TeleTab:CreateButton({Name = "Ir até: "..p.Name, Callback = function() if p.Character then LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame end end}) end
    end
end})

Rayfield:LoadConfiguration()
Rayfield:Notify({Title = "DZXZX V30", Content = "Slider de Velocidade Ativo!", Duration = 5})
