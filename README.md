local UILibrary = loadstring(game:HttpGet("https://pastebin.com/raw/YOUR_PASTEBIN_ID"))()
local win = UILibrary:CreateWindow("Meu Hub Legal")

local aba1 = win:CreateTab("Farm")
aba1:AddButton("aimbot", function(local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local UIS = game:GetService("UserInputService")
    local Camera = workspace.CurrentCamera
    local LocalPlayer = Players.LocalPlayer
    
    -- CONFIG
    local AimPart = "Head"
    local AimFov = 120
    local AimSmoothness = 0.15
    local AimKey = Enum.UserInputType.MouseButton2
    
    -- ESTADO
    local isAiming = false
    local espBoxes = {}
    local espNames = {}
    
    -- CÍRCULO FOV
    local fovCircle = Drawing.new("Circle")
    fovCircle.Radius = AimFov
    fovCircle.Thickness = 2
    fovCircle.Transparency = 1
    fovCircle.Color = Color3.fromRGB(255, 0, 0)
    fovCircle.Filled = false
    fovCircle.Visible = true
    
    -- PEGAR JOGADOR MAIS PRÓXIMO
    local function getClosestPlayer()
        local closest = nil
        local shortestDist = AimFov
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AimPart) then
                local part = player.Character[AimPart]
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
                    if dist < shortestDist then
                        shortestDist = dist
                        closest = part
                    end
                end
            end
        end
        return closest
    end
    
    -- ATUALIZAR ESP
    local function updateESP()
        for _, box in pairs(espBoxes) do box.Visible = false end
        for _, name in pairs(espNames) do name.Visible = false end
    
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
                    local scale = 1 / distance * 100
                    local width = 40 * scale
                    local height = 60 * scale
    
                    local box = espBoxes[player]
                    if not box then
                        box = Drawing.new("Square")
                        box.Thickness = 1
                        box.Color = Color3.fromRGB(0, 255, 0)
                        box.Transparency = 1
                        box.Filled = false
                        espBoxes[player] = box
                    end
                    box.Size = Vector2.new(width, height)
                    box.Position = Vector2.new(pos.X - width / 2, pos.Y - height / 2)
                    box.Visible = true
    
                    local nameText = espNames[player]
                    if not nameText then
                        nameText = Drawing.new("Text")
                        nameText.Color = Color3.fromRGB(0, 255, 0)
                        nameText.Size = 14
                        nameText.Center = true
                        nameText.Outline = true
                        nameText.Font = 2
                        espNames[player] = nameText
                    end
                    nameText.Text = player.Name
                    nameText.Position = Vector2.new(pos.X, pos.Y - height / 2 - 15)
                    nameText.Visible = true
                end
            end
        end
    end
    
    -- INPUT DE MOUSE PARA PC
    UIS.InputBegan:Connect(function(input)
        if input.UserInputType == AimKey then
            isAiming = true
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == AimKey then
            isAiming = false
        end
    end)
    
    -- BOTÃO PARA MOBILE E PC
    local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
    ScreenGui.Name = "AimbotUI"
    
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0, 140, 0, 40)
    toggleButton.Position = UDim2.new(1, -150, 0, 50)
    toggleButton.AnchorPoint = Vector2.new(0, 0)
    toggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.Font = Enum.Font.GothamBold
    toggleButton.TextSize = 16
    toggleButton.Text = "Ativar Aimbot"
    toggleButton.Parent = ScreenGui
    toggleButton.BorderSizePixel = 0
    toggleButton.BackgroundTransparency = 0.1
    
    local isButtonAiming = false
    toggleButton.MouseButton1Click:Connect(function()
        isButtonAiming = not isButtonAiming
        isAiming = isButtonAiming
        toggleButton.Text = isButtonAiming and "Desativar Aimbot" or "Ativar Aimbot"
    end)
    
    -- LOOP PRINCIPAL
    RunService.RenderStepped:Connect(function()
        fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        updateESP()
    
        if isAiming then
            local targetPart = getClosestPlayer()
            if targetPart then
                local camPos = Camera.CFrame.Position
                local newCFrame = CFrame.new(camPos, targetPart.Position)
                Camera.CFrame = Camera.CFrame:Lerp(newCFrame, AimSmoothness)
            end
        end
    end))
    print("aimbot ativado!")
    -- Adicione seu código aqui
end)

local aba2 = win:CreateTab("aimbot")
aba2:AddButton("esp", function(local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LocalPlayer = Players.LocalPlayer
    local Camera = workspace.CurrentCamera
    
    local tracers = {}
    
    -- Cores por time (com nomes corrigidos)
    local cores = {
        police = Color3.fromRGB(0, 0, 255),        -- Azul
        criminals = Color3.fromRGB(255, 0, 0),     -- Vermelho
        prisoners = Color3.fromRGB(255, 140, 0),   -- Laranja
        heroes = Color3.fromRGB(255, 255, 0)       -- Amarelo
    }
    
    -- Cria uma linha (tracer) entre o jogador local e outro jogador
    local function criarTracer(player)
        local line = Drawing.new("Line")
        line.Thickness = 2
        line.Transparency = 1
        line.Visible = true
        line.ZIndex = 1
        tracers[player] = line
    
        RunService.RenderStepped:Connect(function()
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local targetPos = player.Character.HumanoidRootPart.Position
                local myPos = LocalPlayer.Character.HumanoidRootPart.Position
    
                local screenPos1, onScreen1 = Camera:WorldToViewportPoint(myPos)
                local screenPos2, onScreen2 = Camera:WorldToViewportPoint(targetPos)
    
                if onScreen1 and onScreen2 then
                    line.From = Vector2.new(screenPos1.X, screenPos1.Y)
                    line.To = Vector2.new(screenPos2.X, screenPos2.Y)
    
                    local teamName = player.Team and player.Team.Name:lower() or ""
                    line.Color = cores[teamName] or Color3.fromRGB(255, 255, 255)
                    line.Visible = true
                else
                    line.Visible = false
                end
            else
                line.Visible = false
            end
        end)
    end
    
    -- Cria os tracers para jogadores existentes
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            criarTracer(player)
        end
    end
    
    -- Aplica aos novos jogadores
    Players.PlayerAdded:Connect(function(player)
        if player ~= LocalPlayer then
            player.CharacterAdded:Connect(function()
                criarTracer(player)
            end)
        end
    end)
end)
    print("detectando jogador...")
    -- Adicione seu código aqui
end)

AddButton(abaOutros, "Ativar Fly", function(fly)
    -- Carregar o script do Fly a partir do link externo
    loadstring(game:HttpGet("\108\111\97\100\115\116\114\105\110\103\40\103\97\109\101\58\72\116\116\112\71\101\116\40\40\39\104\116\116\112\115\58\47\47\103\105\115\116\46\103\105\116\104\117\98\117\115\101\114\99\111\110\116\101\110\116\46\99\111\109\47\109\101\111\122\111\110\101\89\84\47\98\102\48\51\55\100\102\102\57\102\48\97\55\48\48\49\55\51\48\52\100\100\100\54\55\102\100\99\100\51\55\48\47\114\97\119\47\101\49\52\101\55\52\102\52\50\53\98\48\54\48\100\102\53\50\51\51\52\51\99\102\51\48\98\55\56\55\48\55\52\101\98\51\99\53\100\50\47\97\114\99\101\117\115\37\50\53\50\48\120\37\50\53\50\48\102\108\121\37\50\53\50\48\50\37\50\53\50\48\111\98\102\108\117\99\97\116\111\114\39\41\44\116\114\117\101\41\41\40\41\10\10", true))()
end)
