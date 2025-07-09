-- Serviços
local uis = game:GetService("UserInputService")
local rs = game:GetService("RunService")
local plrs = game:GetService("Players")
local lp = plrs.LocalPlayer
local cam = workspace.CurrentCamera
local ws = game:GetService("Workspace")

-- Variáveis
local aimbot = false
local esp = false
local aimbotBind = "MouseButton2"
local segurandoAimbot = false
local mouseTravado = true
local aguardandoTecla = false
local fpsPlus = false

-- Espera PlayerGui
repeat task.wait() until lp:FindFirstChild("PlayerGui")

-- GUI principal
local gui = Instance.new("ScreenGui", lp:WaitForChild("PlayerGui"))
gui.Name = "CBX_UI"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Bola flutuante
local bola = Instance.new("ImageButton")
bola.Size = UDim2.new(0, 80, 0, 80)
bola.Position = UDim2.new(0, 20, 0, 130)
bola.BackgroundTransparency = 1
bola.Image = "rbxassetid://81168570238976"
bola.ZIndex = 999
bola.Parent = gui

-- Menu
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 230, 0, 240)
frame.Position = UDim2.new(0.5, -115, 0.5, -120)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Visible = false
frame.ZIndex = 999
frame.Parent = gui

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.Text = "Painel de Hack"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextScaled = true
title.ZIndex = 999

-- Toggle com botão de bind
local function criarToggleComBind(nome, y, callback, mostrarConfig)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, mostrarConfig and 170 or 210, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextScaled = true
    btn.ZIndex = 999
    local estado = false
    btn.Text = nome.." [OFF]"..(mostrarConfig and (" ("..aimbotBind..")") or "")

    btn.MouseButton1Click:Connect(function()
        estado = not estado
        btn.Text = nome.." ["..(estado and "ON" or "OFF").."]"..(mostrarConfig and (" ("..aimbotBind..")") or "")
        callback(estado)
    end)

    if mostrarConfig then
        local configBtn = Instance.new("TextButton", frame)
        configBtn.Size = UDim2.new(0, 30, 0, 30)
        configBtn.Position = UDim2.new(0, 190, 0, y)
        configBtn.Text = "⚙"
        configBtn.TextScaled = true
        configBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        configBtn.TextColor3 = Color3.new(1, 1, 1)
        configBtn.ZIndex = 999
        configBtn.Parent = frame

        configBtn.MouseButton1Click:Connect(function()
            if aguardandoTecla then return end
            aguardandoTecla = true
            btn.Text = nome.." [..] (pressione tecla/mouse)"
            local conn
            conn = uis.InputBegan:Connect(function(input, gp)
                if not gp then
                    if input.UserInputType == Enum.UserInputType.Keyboard then
                        aimbotBind = input.KeyCode.Name
                    elseif input.UserInputType == Enum.UserInputType.MouseButton then
                        aimbotBind = "MouseButton"..tostring(input.UserInputIndex.Value - 0)
                    end
                    btn.Text = nome.." ["..(estado and "ON" or "OFF").."] ("..aimbotBind..")"
                    aguardandoTecla = false
                    conn:Disconnect()
                end
            end)
        end)
    end

    btn.Parent = frame
end

-- Abrir menu
bola.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

-- Botões
criarToggleComBind("ESP", 40, function(v) esp = v end, false)
criarToggleComBind("Aimbot", 80, function(v) aimbot = v end, true)
criarToggleComBind("FPS Plus", 120, function(v)
    fpsPlus = v
    for _, obj in pairs(ws:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire")
        or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") or obj:IsA("PointLight") then
            obj.Enabled = not v
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = v and 1 or 0
        end
    end
end, false)

-- ESP
local function criarESP(player)
    if not player.Character then return end
    if player.Character:FindFirstChild("ESP") then return end
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESP"
    box.Adornee = player.Character:WaitForChild("HumanoidRootPart")
    box.Size = Vector3.new(4, 6, 2)
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Color3 = Color3.new(0, 1, 0)
    box.Transparency = 0.3
    box.Parent = player.Character
end

local function inimigo(p)
    return p ~= lp and p.Team ~= lp.Team
end

local function alvoMaisProximo()
    local closest = nil
    local minDist = math.huge
    for _, p in ipairs(plrs:GetPlayers()) do
        if inimigo(p) and p.Character and p.Character:FindFirstChild("Head") then
            local headPos, onScreen = cam:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(headPos.X, headPos.Y) - cam.ViewportSize/2).Magnitude
                if dist < 1000 and dist < minDist then
                    closest = p
                    minDist = dist
                end
            end
        end
    end
    return closest
end

-- Aimbot bind
uis.InputBegan:Connect(function(input)
    local tecla = input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode.Name
        or input.UserInputType == Enum.UserInputType.MouseButton and ("MouseButton"..tostring(input.UserInputIndex.Value - 0))
    if tecla == aimbotBind then
        segurandoAimbot = true
    end
end)
uis.InputEnded:Connect(function(input)
    local tecla = input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode.Name
        or input.UserInputType == Enum.UserInputType.MouseButton and ("MouseButton"..tostring(input.UserInputIndex.Value - 0))
    if tecla == aimbotBind then
        segurandoAimbot = false
    end
end)

-- Loop de funcionamento
rs.RenderStepped:Connect(function()
    for _, p in ipairs(plrs:GetPlayers()) do
        if inimigo(p) and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if esp then
                if not p.Character:FindFirstChild("ESP") then
                    criarESP(p)
                end
                p.Character.ESP.Visible = true
            elseif p.Character:FindFirstChild("ESP") then
                p.Character.ESP.Visible = false
            end
        end
    end
    if aimbot and segurandoAimbot then
        local alvo = alvoMaisProximo()
        if alvo and alvo.Character and alvo.Character:FindFirstChild("Head") then
            cam.CFrame = CFrame.new(cam.CFrame.Position, alvo.Character.Head.Position)
        end
    end
end)

-- ALT destrava mouse
uis.InputBegan:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.LeftAlt and not gp then
        mouseTravado = not mouseTravado
        uis.MouseBehavior = mouseTravado and Enum.MouseBehavior.LockCenter or Enum.MouseBehavior.Default
        uis.MouseIconEnabled = not mouseTravado
    end
end)
