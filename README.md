local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações do Aimbot
local fov = 100 -- Tamanho do FOV
local aimbotEnabled = false -- Aimbot começa desativado
local aimPart = "Head" -- Parte do corpo para mirar (Head ou Torso)
local smoothing = 0.5 -- Porcentagem de "grude" (0 a 1)

-- Função para criar o FOV do Aimbot
local FOVring = Drawing.new("Circle")
FOVring.Visible = false -- FOV invisível por padrão
FOVring.Thickness = 2
FOVring.Color = Color3.fromRGB(255, 0, 0) -- Cor do FOV
FOVring.Filled = false
FOVring.Radius = fov
FOVring.Position = Camera.ViewportSize / 2

-- Função para atualizar o FOV
local function updateFOV()
    FOVring.Position = Camera.ViewportSize / 2
    FOVring.Radius = fov -- Atualiza o tamanho do FOV
end

-- Função para mirar no alvo com suavização (smoothing)
local function lookAt(target)
    local lookVector = (target - Camera.CFrame.Position).unit
    local newCFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + lookVector), smoothing)
    Camera.CFrame = newCFrame
end

-- Função para encontrar o jogador mais próximo dentro do FOV
local function getClosestPlayerInFOV()
    local nearest = nil
    local lastDistance = math.huge
    local playerMousePos = Camera.ViewportSize / 2

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(aimPart) then
            local part = player.Character[aimPart]
            local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
            local distance = (Vector2.new(screenPos.X, screenPos.Y) - playerMousePos).Magnitude

            if onScreen and distance < lastDistance and distance < fov then
                lastDistance = distance
                nearest = player
            end
        end
    end

    return nearest
end

-- Função para ativar/desativar o Aimbot
local function toggleAimbot()
    aimbotEnabled = not aimbotEnabled

    if aimbotEnabled then
        RunService:BindToRenderStep("AimbotUpdate", Enum.RenderPriority.Camera.Value, function()
            local closest = getClosestPlayerInFOV()
            if closest and closest.Character:FindFirstChild(aimPart) then
                lookAt(closest.Character[aimPart].Position)
            end
        end)
    else
        RunService:UnbindFromRenderStep("AimbotUpdate")
    end
end

-- Integração com o tab "Aimbot-Mobile"
local WindUI = loadstring(game:HttpGet("https://tree-hub.vercel.app/api/UI/WindUI"))()

local Window = WindUI:CreateWindow({
    Title = "🚀Eclipse Menu🚀",
    Icon = "door-open",
    Author = "🚀Mobile E Pc🚀",
    Folder = "Bk9biel",
    Size = UDim2.fromOffset(580, 460),
    Transparent = true,
    Theme = "Dark",
    SideBarWidth = 200,
    HasOutline = false,
})

local Tabs = {
    PlayersTab = Window:Tab({ Title = "Players", Icon = "users", Desc = "Player-related functionalities." }),
    JogadorTab = Window:Tab({ Title = "Jogador", Icon = "user", Desc = "Ajustes e funções para o jogador." }),
    ArmasTab = Window:Tab({ Title = "Armas", Icon = "target", Desc = "Funções relacionadas a armas." }),
    PcTab = Window:Tab({ Title = "Aimbot-PC", Icon = "target", Desc = "Funções relacionadas a armas." }),
    MobileTab = Window:Tab({ Title = "Aimbot-Mobile", Icon = "target", Desc = "Funções relacionadas a armas." }),
    TrollTab = Window:Tab({ Title = "Troll", Icon = "smile", Desc = "Funções de trollagem." }),
    VeiculosTab = Window:Tab({ Title = "Veículos", Icon = "car", Desc = "Funções relacionadas a veículos." }),
    AutoRevistarTab = Window:Tab({ Title = "Mini City", Icon = "search", Desc = "Funções de revistar automático." }),
    AutoFarmTab = Window:Tab({ Title = "Auto Farms", Icon = "tractor", Desc = "Farms automáticos." }),
    AutoFarmsTab = Window:Tab({ Title = "Teleportes", Icon = "tractor", Desc = "Farms automáticos." }),
    MenusTab = Window:Tab({ Title = "Menus", Icon = "list", Desc = "Menus de scripts populares." }),
    ConfiguracoesTab = Window:Tab({ Title = "Configurações", Icon = "settings", Desc = "Configurações e utilitários." }),
}

-- Adicionando as funções ao tab "Aimbot-Mobile"
Tabs.MobileTab:Button({
    Title = "Ativar Aimbot",
    Desc = "Ativa ou desativa o Aimbot.",
    Callback = function()
        toggleAimbot()
        Tabs.MobileTab:UpdateButton("Ativar Aimbot", { Title = aimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot" })
    end
})

Tabs.MobileTab:Button({
    Title = "Mira em: Cabeça",
    Desc = "Mira na cabeça do jogador.",
    Callback = function()
        aimPart = "Head"
        Tabs.MobileTab:UpdateButton("Mira em: Cabeça", { Title = "Mira em: Cabeça" })
        Tabs.MobileTab:UpdateButton("Mira em: Peito", { Title = "Mira em: Peito" })
    end
})

Tabs.MobileTab:Button({
    Title = "Mira em: Peito",
    Desc = "Mira no peito do jogador.",
    Callback = function()
        aimPart = "Torso"
        Tabs.MobileTab:UpdateButton("Mira em: Cabeça", { Title = "Mira em: Cabeça" })
        Tabs.MobileTab:UpdateButton("Mira em: Peito", { Title = "Mira em: Peito" })
    end
})

Tabs.MobileTab:Slider({
    Title = "Tamanho do FOV",
    Desc = "Ajusta o tamanho do FOV.",
    Value = {
        Min = 50,
        Max = 500,
        Default = 100,
    },
    Callback = function(value)
        fov = value
    end
})

Tabs.MobileTab:Slider({
    Title = "Suavização (%)",
    Desc = "Ajusta a suavidade do Aimbot.",
    Value = {
        Min = 0,
        Max = 100,
        Default = 50,
    },
    Callback = function(value)
        smoothing = value / 100 -- Converte para valor entre 0 e 1
    end
})

Tabs.MobileTab:Toggle({
    Title = "Exibir FOV",
    Desc = "Ativa ou desativa a exibição do FOV.",
    Callback = function(value)
        FOVring.Visible = value
    end
})

-- Selecionar o tab "Aimbot-Mobile" por padrão
Window:SelectTab(1)


-- Aimbot-PC Tab
local teamCheck = false
local fov = 150
local smoothing = 1
local aimbotEnabled = false
local fovVisible = false
local aimPart = "Head" -- Padrão: Cabeça

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Função para buscar a melhor parte do corpo para mirar
local function getBestAimPart(character)
    if aimPart == "Head" then
        if character:FindFirstChild("Head") then
            return character.Head
        end
    elseif aimPart == "Torso" then
        if character:FindFirstChild("UpperTorso") then
            return character.UpperTorso
        elseif character:FindFirstChild("Torso") then
            return character.Torso
        elseif character:FindFirstChild("HumanoidRootPart") then
            return character.HumanoidRootPart
        end
    end
    return nil
end

local FOVring = Drawing.new("Circle")
FOVring.Visible = fovVisible
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = workspace.CurrentCamera.ViewportSize / 2

local function getClosestTarget()
    local closestTarget = nil
    local closestDist = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
            local bestPart = getBestAimPart(player.Character)
            if bestPart then
                if not teamCheck or (player.Team ~= LocalPlayer.Team) then
                    local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(bestPart.Position)
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - workspace.CurrentCamera.ViewportSize / 2).Magnitude

                    if onScreen and dist < closestDist and dist <= fov then
                        closestDist = dist
                        closestTarget = player
                    end
                end
            end
        end
    end
    return closestTarget
end

local function aimAt(target)
    if aimbotEnabled and target and target.Character then
        local bestPart = getBestAimPart(target.Character)
        if bestPart then
            local aimPosition = bestPart.Position
            workspace.CurrentCamera.CFrame = workspace.CurrentCamera.CFrame:Lerp(CFrame.new(workspace.CurrentCamera.CFrame.Position, aimPosition), smoothing)
        end
    end
end

RunService.RenderStepped:Connect(function()
    FOVring.Radius = fov
    FOVring.Position = workspace.CurrentCamera.ViewportSize / 2
    FOVring.Visible = fovVisible

    if aimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestTarget()
        aimAt(target)
    end
end)

-- Adicionando as funções ao menu
Tabs.PcTab:Toggle({
    Title = "Ativar Aimbot",
    Desc = "Ativa ou desativa o aimbot.",
    Callback = function(value)
        aimbotEnabled = value
    end
})

Tabs.PcTab:Toggle({
    Title = "Ativar FOV",
    Desc = "Ativa ou desativa o FOV.",
    Callback = function(value)
        fovVisible = value
    end
})

Tabs.PcTab:Slider({
    Title = "Smoothing (%)",
    Desc = "Ajusta o 'grude' do aimbot.",
    Value = {
        Min = 0,
        Max = 100,
        Default = 50,
    },
    Callback = function(value)
        smoothing = value / 100
    end
})

Tabs.PcTab:Slider({
    Title = "Tamanho do FOV",
    Desc = "Ajusta o tamanho do FOV.",
    Value = {
        Min = 50,
        Max = 500,
        Default = 150,
    },
    Callback = function(value)
        fov = value
    end
})

-- Botão para mirar na cabeça
Tabs.PcTab:Button({
    Title = "Grudar Cabeça",
    Desc = "Mira na cabeça do jogador.",
    Callback = function()
        aimPart = "Head"
        WindUI:Notify({
            Title = "Aimbot",
            Content = "Agora mirando na Cabeça.",
            Duration = 3,
            Icon = "check-circle",
        })
    end
})

-- Botão para mirar no peito
Tabs.PcTab:Button({
    Title = "Grudar Peito",
    Desc = "Mira no peito do jogador.",
    Callback = function()
        aimPart = "Torso"
        WindUI:Notify({
            Title = "Aimbot",
            Content = "Agora mirando no Peito.",
            Duration = 3,
            Icon = "check-circle",
        })
    end
})

-- Restante do código...

-- Select the first tab by default
Window:SelectTab(1)
-- Veículos Tab
local velocityMult = 0.025
local velocityMult2 = 150e-3
local flightSpeed = 1
local flightEnabled = false

Tabs.VeiculosTab:Slider({
    Title = "Velocidade (Veículo)",
    Desc = "Ajuste a velocidade do veículo",
    Value = {
        Min = 0,
        Max = 50,
        Default = 25,
    },
    Callback = function(value)
        velocityMult = value / 1000
    end
})

Tabs.VeiculosTab:Slider({
    Title = "Velocidade do Freio",
    Desc = "Ajuste a velocidade do freio",
    Value = {
        Min = 0,
        Max = 300,
        Default = 150,
    },
    Callback = function(value)
        velocityMult2 = value / 1000
    end
})

Tabs.VeiculosTab:Slider({
    Title = "Velocidade de Voo",
    Desc = "Ajuste a velocidade de voo",
    Value = {
        Min = 0,
        Max = 800,
        Default = 100,
    },
    Callback = function(value)
        flightSpeed = value / 100
    end
})

Tabs.VeiculosTab:Toggle({
    Title = "Ativar Voo",
    Desc = "Ative ou desativa o voo do veículo",
    Callback = function(value)
        flightEnabled = value
    end
})

Tabs.VeiculosTab:Toggle({
    Title = "Lista de Carros",
    Desc = "Não funcione em todos os Mapas",
    Callback = function(value)
        loadstring(game:HttpGet("https://pastebin.com/raw/KcXqPnbk"))()

    end
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetVehicleFromDescendant(Descendant)
    return
        Descendant:FindFirstAncestor(LocalPlayer.Name .. "\'s Car") or
        (Descendant:FindFirstAncestor("Body") and Descendant:FindFirstAncestor("Body").Parent) or
        (Descendant:FindFirstAncestor("Misc") and Descendant:FindFirstAncestor("Misc").Parent) or
        Descendant:FindFirstAncestorWhichIsA("Model")
end

local defaultCharacterParent

RunService.Stepped:Connect(function()
    local Character = LocalPlayer.Character
    if flightEnabled == true then
        if Character and typeof(Character) == "Instance" then
            local Humanoid = Character:FindFirstChildWhichIsA("Humanoid")
            if Humanoid and typeof(Humanoid) == "Instance" then
                local SeatPart = Humanoid.SeatPart
                if SeatPart and typeof(SeatPart) == "Instance" and SeatPart:IsA("VehicleSeat") then
                    local Vehicle = GetVehicleFromDescendant(SeatPart)
                    if Vehicle and Vehicle:IsA("Model") then
                        Character.Parent = Vehicle
                        if not Vehicle.PrimaryPart then
                            if SeatPart.Parent == Vehicle then
                                Vehicle.PrimaryPart = SeatPart
                            else
                                Vehicle.PrimaryPart = Vehicle:FindFirstChildWhichIsA("BasePart")
                            end
                        end
                        local PrimaryPartCFrame = Vehicle:GetPrimaryPartCFrame()
                        Vehicle:SetPrimaryPartCFrame(CFrame.new(PrimaryPartCFrame.Position, PrimaryPartCFrame.Position + workspace.CurrentCamera.CFrame.LookVector) * (UserInputService:GetFocusedTextBox() and CFrame.new(0, 0, 0) or CFrame.new((UserInputService:IsKeyDown(Enum.KeyCode.D) and flightSpeed) or (UserInputService:IsKeyDown(Enum.KeyCode.A) and -flightSpeed) or 0, (UserInputService:IsKeyDown(Enum.KeyCode.E) and flightSpeed / 2) or (UserInputService:IsKeyDown(Enum.KeyCode.Q) and -flightSpeed / 2) or 0, (UserInputService:IsKeyDown(Enum.KeyCode.S) and flightSpeed) or (UserInputService:IsKeyDown(Enum.KeyCode.W) and -flightSpeed) or 0)))
                        SeatPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        SeatPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                    end
                end
            end
        end
    else
        if Character and typeof(Character) == "Instance" then
            Character.Parent = defaultCharacterParent or Character.Parent
            defaultCharacterParent = Character.Parent
        end
    end
end)

local velocityEnabledKeyCode = Enum.KeyCode.W
local qbEnabledKeyCode = Enum.KeyCode.S

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == velocityEnabledKeyCode then
        while UserInputService:IsKeyDown(velocityEnabledKeyCode) do
            task.wait(0)
            local Character = LocalPlayer.Character
            if Character and typeof(Character) == "Instance" then
                local Humanoid = Character:FindFirstChildWhichIsA("Humanoid")
                if Humanoid and typeof(Humanoid) == "Instance" then
                    local SeatPart = Humanoid.SeatPart
                    if SeatPart and typeof(SeatPart) == "Instance" and SeatPart:IsA("VehicleSeat") then
                        SeatPart.AssemblyLinearVelocity *= Vector3.new(1 + velocityMult, 1, 1 + velocityMult)
                    end
                end
            end
        end
    elseif input.KeyCode == qbEnabledKeyCode then
        while UserInputService:IsKeyDown(qbEnabledKeyCode) do
            task.wait(0)
            local Character = LocalPlayer.Character
            if Character and typeof(Character) == "Instance" then
                local Humanoid = Character:FindFirstChildWhichIsA("Humanoid")
                if Humanoid and typeof(Humanoid) == "Instance" then
                    local SeatPart = Humanoid.SeatPart
                    if SeatPart and typeof(SeatPart) == "Instance" and SeatPart:IsA("VehicleSeat") then
                        SeatPart.AssemblyLinearVelocity *= Vector3.new(1 - velocityMult2, 1, 1 - velocityMult2)
                    end
                end
            end
        end
    elseif input.KeyCode == Enum.KeyCode.P then
        local Character = LocalPlayer.Character
        if Character and typeof(Character) == "Instance" then
            local Humanoid = Character:FindFirstChildWhichIsA("Humanoid")
            if Humanoid and typeof(Humanoid) == "Instance" then
                local SeatPart = Humanoid.SeatPart
                if SeatPart and typeof(SeatPart) == "Instance" and SeatPart:IsA("VehicleSeat") then
                    SeatPart.AssemblyLinearVelocity *= Vector3.new(0, 0, 0)
                    SeatPart.AssemblyAngularVelocity *= Vector3.new(0, 0, 0)
                end
            end
        end
    end
end)

-- Auto Revistar Tab
Tabs.AutoRevistarTab:Button({
    Title = "Puxar itens do inv 1 click por vez",
    Desc = "Puxa itens do inventário automaticamente.",
    Callback = function()
        -- Adicione o código para puxar itens aqui
    end
})

Tabs.AutoRevistarTab:Toggle({
    Title = "Mandar revistar (TECLA E)",
    Desc = "Ativa/desativa o revistar automático com a tecla E.",
    Callback = function(Value)
        getgenv().Enabled = Value
    end
})

Tabs.AutoRevistarTab:Button({
    Title = "Mandar revistar UI (Mobile)",
    Desc = "Cria uma interface para revistar no mobile.",
    Callback = function()
        -- Adicione o código para a interface móvel aqui
    end
})

-- Botão "Ver itens UI (irreversivel)"
Tabs.AutoRevistarTab:Button({
   Name = "Ver itens UI (irreversivel)",
   Callback = function()
    -- Verifica se a UI já foi criada
        if not createdUI then
            -- Executa o loadstring para carregar o script
            loadScript()

            -- Aqui você pode adicionar o código para criar a UI que o script cria.
            -- Caso o script já crie a UI, pode ser necessário ajustar isso:
            createdUI = Instance.new("BillboardGui")
            createdUI.Name = "PlayerUI"
            createdUI.Adornee = game.Players.LocalPlayer.Character:FindFirstChild("Head")
            createdUI.Size = UDim2.new(0, 200, 0, 70)
            createdUI.StudsOffset = Vector3.new(0, -3, 0)
            createdUI.AlwaysOnTop = true
            createdUI.Parent = game.Players.LocalPlayer.Character:FindFirstChild("Head")
        else
            -- Se a UI já existe, remove-a
            removeUI()
        end
    end
})

-- Restante do código...

-- Select the first tab by default
Window:SelectTab(1)








































-- Players Tab
Tabs.PlayersTab:Button({
    Title = "FOV",
    Desc = "Change FOV settings",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/we2jmfhH"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Auto Revistar (Mini City)",
    Desc = "Enable auto revistar feature",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/Yfbb4X65"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Lista de Players",
    Desc = "List all players",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/kuf7b3Ls"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Espectar Players",
    Desc = "Spectate players",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/XigxmBYm"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Teleporte (Indetected)",
    Desc = "Teleport to players (undetected)",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/3TpvDsdC"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Click Teleport",
    Desc = "Teleport by clicking",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/1dj6hVeq"))()
    end
})

Tabs.PlayersTab:Button({
    Title = "Bring Player",
    Desc = "Bring a player to you",
    Callback = function()
        loadstring("loadstring(game:HttpGet(('https://pastebin.com/raw/mFfRnPsw'),true))()")()
    end
})

Tabs.PlayersTab:Button({
    Title = "Loop Teleport",
    Desc = "Loop teleport to a player",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/iLanlu/Roblox-TP-Script/refs/heads/main/TProblox.lua"))()
    end
})

-- Jogador Tab
Tabs.JogadorTab:Slider({
    Title = "WalkSpeed",
    Desc = "Ajuste a velocidade de caminhada",
    Value = {
        Min = 16,
        Max = 1000,
        Default = 16,
    },
    Callback = function(value)
        game:GetService("Players").LocalPlayer.Character.Humanoid.WalkSpeed = value
    end
})

Tabs.JogadorTab:Slider({
    Title = "Jump Power",
    Desc = "Ajuste o poder de pulo",
    Value = {
        Min = 50,
        Max = 1000,
        Default = 50,
    },
    Callback = function(value)
        game:GetService("Players").LocalPlayer.Character.Humanoid.JumpPower = value
    end
})

Tabs.JogadorTab:Button({
    Title = "God Mod",
    Desc = "Ative o modo Deus",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/skroopzW/god-mode-script/refs/heads/main/god%20mode.lua"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Reiniciar Personagem",
    Desc = "Reinicia o personagem",
    Callback = function()
        game.Players.LocalPlayer.Character.Humanoid.Health = 0
    end
})

Tabs.JogadorTab:Button({
    Title = "Fly",
    Desc = "Ative o modo de voar",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/5vdjY2tt"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Noclip pra Mobile",
    Desc = "Ative o noclip para dispositivos móveis",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/qZnLZrif"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Noclip pra Pc",
    Desc = "Ative o noclip para PC",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/Cp1pJCbq"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Invisibilidade",
    Desc = "Ative a invisibilidade",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/3Rnd9rHf"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Anti Afk",
    Desc = "Ative o anti-AFK",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/7a9LPTjp"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Anti Kick",
    Desc = "Ative o anti-kick",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/Anti-Kick/main/Anti-Kick.lua"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Anti Lag",
    Desc = "Ative o anti-lag",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/ZgDNY1Re"))()
    end
})

Tabs.JogadorTab:Button({
    Title = "Fling",
    Desc = "Ative o fling (não funciona em todos os mapas)",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/P12sgf8B"))()
    end
})

-- Auto Farms Tab
Tabs.AutoFarmsTab:Button({
    Title = "Policia Militar RP Farm dinheiro",
    Desc = "Farm automático de dinheiro no Policia Militar RP",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/n1sPfbu1"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Auto Farm Desastres Naturais",
    Desc = "Farm automático em Desastres Naturais",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/uBXtcasU"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir Mecanica (Mini City)",
    Desc = "Teleportar para a mecânica no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/3EBcZPaB"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir Praça (Mini City)",
    Desc = "Teleportar para a praça no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/nM0xCewd"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir lojinha (Mini City)",
    Desc = "Teleportar para a lojinha no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/2zRghdMT"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir loja De Carro (Mini City)",
    Desc = "Teleportar para a loja de carros no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/nM0xCewd"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir Farm De Maconha (Mini City)",
    Desc = "Teleportar para o farm de maconha no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/Y8DQhFnu"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir Pra Lavagem de Dinheiro (Mini City)",
    Desc = "Teleportar para a lavagem de dinheiro no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Bk9biel/SSSSSDAS/refs/heads/main/README.md"))()
    end
})

Tabs.AutoFarmsTab:Button({
    Title = "Ir Pro Banco (Mini City)",
    Desc = "Teleportar para o banco no Mini City",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Bk9biel/ssssdsasdas/refs/heads/main/README.md"))()
    end
})


Tabs.ArmasTab:Button({
    Title = "Puxar Armas",
    Desc = "Puxe armas automaticamente",
    Callback = function()
        loadstring(game:HttpGetAsync("https://pastebin.com/raw/eCtEKcMU"))()
    end
})

Tabs.ArmasTab:Button({
    Title = "Esp 1 (Para ativar e desativar tecla Y)",
    Desc = "Ative o ESP 1 (tecla Y)",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Bk9biel/ss/refs/heads/main/README.md"))()
    end
})

Tabs.ArmasTab:Button({
    Title = "Esp 2",
    Desc = "Ative o ESP 2",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/hJ1n0kbP"))()
    end
})

Tabs.ArmasTab:Button({
    Title = "Tracers",
    Desc = "Ative os tracers",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/pUYfcA9H"))()
    end
})

-- Menus Tab
Tabs.MenusTab:Button({
    Title = "Infinity Yield",
    Desc = "Carregar o menu Infinity Yield",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/DarkNetworks/Infinite-Yield/main/latest.lua"))()
    end
})

Tabs.MenusTab:Button({
    Title = "Orca Menu",
    Desc = "Carregar o menu Orca",
    Callback = function()
        loadstring(game:HttpGetAsync("https://raw.githubusercontent.com/richie0866/orca/master/public/snapshot.lua"))()
    end
})

Tabs.MenusTab:Button({
    Title = "Eclipse Menu v1",
    Desc = "Carregar o menu Eclipse v1",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/WdA1Qi33"))()
    end
})

-- Configurações Tab
Tabs.ConfiguracoesTab:Button({
    Title = "Copiar Discord",
    Desc = "Copiar o link do Discord para a área de transferência",
    Callback = function()
        local discordLink = "https://discord.gg/6hanbJwwCD"
        setclipboard(discordLink) -- Copia o link para a área de transferência
        WindUI:Notify({
            Title = "Discord Copiado!",
            Content = "O link do Discord foi copiado: " .. discordLink,
            Duration = 5,
            Icon = "check-circle",
        })
    end
})

-- Troll Tab
Tabs.TrollTab:Button({
    Title = "Script De Reset All Mapa Da Nyviii (Só Clicar 'F')",
    Desc = "Créditos: cold9999999999999999999999999999",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/zeeywashere/SongMapID/refs/heads/main/rustcc.lua"))()
    end
})

Tabs.TrollTab:Button({
    Title = "Kill Aura Mapa Do Danone v2",
    Desc = "Só Chegar Perto Que mata",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/TijZmQWj"))()
    end
})

Tabs.TrollTab:Button({
    Title = "Kill Aura Mapa Do Danone v3",
    Desc = "Só Chegar Perto Que mata",
    Callback = function()
        loadstring(game:HttpGet("https://pastebin.com/raw/KHBWcYX0"))()
    end
})

Tabs.TrollTab:Button({
    Title = "Script de bater uma",
    Desc = "Ativar/desativar script de bater uma",
    Callback = function(state)
        if state then
            loadstring(game:HttpGet("https://pastefy.app/YZoglOyJ/raw"))()
        else
            -- Adicione aqui o código para desativar o script, se necessário.
        end
    end
})

Tabs.TrollTab:Button({
    Title = "Script de dar mamada",
    Desc = "Ativar/desativar script de dar mamada",
    Callback = function(state)
        if state then
            loadstring(game:HttpGet("https://pastebin.com/raw/yrTkMx54"))()
        else
            -- Adicione aqui o código para desativar o script, se necessário.
        end
    end
})

Tabs.TrollTab:Button({
    Title = "Script de dar",
    Desc = "Ativar/desativar script de dar",
    Callback = function(state)
        if state then
            loadstring(game:HttpGet("https://pastebin.com/raw/nrcDHmnV"))()
        else
            -- Adicione aqui o código para desativar o script, se necessário.
        end
    end
})

Tabs.TrollTab:Button({
    Title = "Bang",
    Desc = "Ativar/desativar script Bang",
    Callback = function(state)
        if state then
            loadstring(game:HttpGet("https://pastebin.com/raw/RMpdUcHH"))()
        else
            -- Adicione aqui o código para desativar o script, se necessário.
        end
    end
})

-- Select the first tab by default
Window:SelectTab(1)
