local lp = game.Players.LocalPlayer
local cam = workspace.CurrentCamera
local rs = game:GetService("RunService")

-- Aimbot simples
local function getClosestEnemy()
    local closest = nil
    local shortest = math.huge

    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= lp and p.Team ~= lp.Team and p.Character and p.Character:FindFirstChild("Head") then
            local pos, onScreen = cam:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - cam.ViewportSize / 2).Magnitude
                if dist < shortest then
                    closest = p
                    shortest = dist
                end
            end
        end
    end

    return closest
end

-- ESP simples (linhas)
local function createESP(part)
    local line = Drawing.new("Line")
    line.Visible = true
    line.Color = Color3.new(0, 1, 0)
    line.Thickness = 2
    line.Transparency = 1
    line.ZIndex = 2
    return line
end

local espTable = {}

-- Loop principal
rs.RenderStepped:Connect(function()
    -- Aimbot
    local target = getClosestEnemy()
    if target and target.Character and target.Character:FindFirstChild("Head") then
        cam.CFrame = CFrame.new(cam.CFrame.Position, target.Character.Head.Position)
    end

    -- ESP
    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= lp and p.Team ~= lp.Team and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if not espTable[p] then
                espTable[p] = createESP(p.Character.HumanoidRootPart)
            end

            local screenPos, onScreen = cam:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
            if onScreen then
                espTable[p].From = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y)
                espTable[p].To = Vector2.new(screenPos.X, screenPos.Y)
                espTable[p].Visible = true
            else
                espTable[p].Visible = false
            end
        end
    end
end)
