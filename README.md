-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer

-- Skybox
local skybox = Instance.new("Sky")
skybox.SkyboxBk = "rbxassetid://511726139"
skybox.SkyboxDn = "rbxassetid://511726139"
skybox.SkyboxFt = "rbxassetid://511726139"
skybox.SkyboxLf = "rbxassetid://511726139"
skybox.SkyboxRt = "rbxassetid://511726139"
skybox.SkyboxUp = "rbxassetid://511726139"
skybox.Parent = Lighting

-- Notificação
local StarterGui = game:GetService("StarterGui")
StarterGui:SetCore("SendNotification", {
    Title = "By FELIPEHUB",
    Text = "Obrigado por usar FELIPEHUB\nhttps://discord.gg/3k6BNHCV",
    Icon = "rbxassetid://511726139",
    Duration = 5
})

-- RGB arco-íris
local function getRainbowColor()
    return Color3.fromHSV(tick() % 5 / 5, 1, 1)
end

-- Cor da equipe ou RGB
local function getTeamColor(player)
    return player.Team and player.TeamColor.Color or getRainbowColor()
end

-- Armazenamento dos desenhos
local drawings = {}

-- Criar ESP
local function createESP(player)
    if player == LocalPlayer then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Filled = false
    box.Visible = false

    local name = Drawing.new("Text")
    name.Size = 13
    name.Center = true
    name.Outline = true
    name.Color = Color3.new(0, 1, 0)
    name.Visible = false

    local distance = Drawing.new("Text")
    distance.Size = 13
    distance.Center = true
    distance.Outline = true
    distance.Color = Color3.new(0, 1, 0)
    distance.Visible = false

    local line = Drawing.new("Line")
    line.Thickness = 1
    line.Visible = false

    local healthBar = Drawing.new("Line")
    healthBar.Thickness = 2
    healthBar.Visible = false

    local healthText = Drawing.new("Text")
    healthText.Size = 13
    healthText.Center = false
    healthText.Outline = true
    healthText.Color = Color3.new(1, 1, 1)
    healthText.Visible = false

    drawings[player] = {
        Box = box,
        Name = name,
        Distance = distance,
        Line = line,
        HealthBar = healthBar,
        HealthText = healthText,
        Highlight = nil
    }

    if player.Character then
        local highlight = Instance.new("Highlight")
        highlight.Adornee = player.Character
        highlight.FillTransparency = 1
        highlight.OutlineTransparency = 0
        highlight.OutlineColor = getTeamColor(player)
        highlight.Parent = player.Character
        drawings[player].Highlight = highlight
    end

    player.CharacterAdded:Connect(function(char)
        task.wait(0.1)
        if drawings[player].Highlight then
            drawings[player].Highlight:Destroy()
        end
        local highlight = Instance.new("Highlight")
        highlight.Adornee = char
        highlight.FillTransparency = 1
        highlight.OutlineTransparency = 0
        highlight.OutlineColor = getTeamColor(player)
        highlight.Parent = char
        drawings[player].Highlight = highlight
    end)
end

-- Criar ESP para todos os jogadores
for _, player in pairs(Players:GetPlayers()) do
    createESP(player)
end
Players.PlayerAdded:Connect(createESP)

-- Atualizar ESP
RunService.RenderStepped:Connect(function()
    local screenCenter = Camera.ViewportSize / 2

    for player, data in pairs(drawings) do
        local character = player.Character
        local box, name, distance, line, healthBar, healthText, highlight = data.Box, data.Name, data.Distance, data.Line, data.HealthBar, data.HealthText, data.Highlight

        if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("Head") and character:FindFirstChildOfClass("Humanoid") then
            local hrp = character.HumanoidRootPart
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            local head = character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
            local footPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 2.5, 0))

            if onScreen then
                local height = math.abs(headPos.Y - footPos.Y)
                local width = height / 2

                box.Size = Vector2.new(width, height)
                box.Position = Vector2.new(pos.X - width / 2, pos.Y - height / 2)
                box.Color = getTeamColor(player)
                box.Visible = true

                name.Text = player.Name
                name.Position = Vector2.new(pos.X, pos.Y - height / 2 - 15)
                name.Visible = true

                local dist = math.floor((LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) and (LocalPlayer.Character.HumanoidRootPart.Position - hrp.Position).Magnitude or 0)
                distance.Text = dist .. "m"
                distance.Position = Vector2.new(pos.X, pos.Y - height / 2 - 30)
                distance.Visible = true

                line.From = Vector2.new(Camera.ViewportSize.X / 2, 0)
                line.To = Vector2.new(pos.X, pos.Y - height / 2)
                line.Color = getTeamColor(player)
                line.Visible = true

                -- Barra de Vida (vertical)
                local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                local barHeight = height
                local barY1 = pos.Y + height / 2
                local barY2 = barY1 - (barHeight * healthPercent)
                local barX = pos.X - width / 2 - 6

                local green = Color3.fromRGB(0, 255, 0)
                local red = Color3.fromRGB(255, 0, 0)
                healthBar.Color = green:lerp(red, 1 - healthPercent)

                healthBar.From = Vector2.new(barX, barY1)
                healthBar.To = Vector2.new(barX, barY2)
                healthBar.Visible = true

                -- Texto da Vida
                healthText.Text = "Vida: " .. math.floor(humanoid.Health)
                healthText.Position = Vector2.new(barX - 55, barY2 - 5)
                healthText.Visible = true

                if highlight and highlight.Parent then
                    highlight.OutlineColor = getTeamColor(player)
                end
            else
                box.Visible = false
                name.Visible = false
                distance.Visible = false
                line.Visible = false
                healthBar.Visible = false
                healthText.Visible = false
            end
        else
            box.Visible = false
            name.Visible = false
            distance.Visible = false
            line.Visible = false
            healthBar.Visible = false
            healthText.Visible = false
        end
    end
end)

-- FPS
local fpsText = Drawing.new("Text")
fpsText.Size = 20
fpsText.Position = Vector2.new(10, Camera.ViewportSize.Y / 2)
fpsText.Color = Color3.new(1, 1, 1)
fpsText.Outline = true
fpsText.Visible = true

local last = tick()
local frames = 0

RunService.RenderStepped:Connect(function()
    frames += 1
    if tick() - last >= 1 then
        fpsText.Text = "FPS: " .. frames
        frames = 0
        last = tick()
    end
end)
