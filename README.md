-- Saber Simulator Ultimate Mobile v5.0
-- Interface Premium com todas as funcionalidades
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local VirtualInputManager = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Configurações globais
local states = {
    AutoSwing = false,
    AutoSell = false,
    AutoClass = false,
    AutoDNA = false,
    AutoSword = false,
    AutoTokens = false,
    AutoCrowns = false,
    AutoRollPets = false,
    AutoCraftPets = false,
    AutoEvent = false,
    AntiAFK = true,
    RollSpeed = 1, -- 1 = normal, 2 = rápida, 3 = ultra
    EventSpeed = 1 -- Velocidade no evento
}

local inEvent = false
local eventStartTime = 0
local eventDuration = 15 * 60 -- 15 minutos em segundos
local lastEventCheck = 0

-- Criar janela premium
local Window = Rayfield:CreateWindow({
   Name = "SABER SIMULATOR ULTIMATE",
   LoadingTitle = "Carregando Interface Premium...",
   LoadingSubtitle = "por SaberMaster",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "SaberSimConfig",
      FileName = "ConfigUltra"
   },
   Discord = {
      Enabled = false,
      Invite = "noinvite",
      RememberJoins = true
   },
   KeySystem = false
})

-- Aba principal
local MainTab = Window:CreateTab("Principal", 4483362458)
local AutoFarmTab = Window:CreateTab("Auto Farm", 4483362458)
local PetsTab = Window:CreateTab("Pets", 4483362458)
local EventTab = Window:CreateTab("Eventos", 4483362458)
local SettingsTab = Window:CreateTab("Configurações", 4483362458)

-- Funções principais
local function autoSwing()
    while states.AutoSwing and Rayfield:GetWindow().Enabled do
        if not inEvent then
            game:GetService("ReplicatedStorage").Remotes.Swing:FireServer()
        end
        task.wait(0.1)
    end
end

local function autoSell()
    while states.AutoSell and Rayfield:GetWindow().Enabled do
        if not inEvent then
            game:GetService("ReplicatedStorage").Remotes.Sell:FireServer()
        end
        task.wait(5)
    end
end

local function autoUpgrade(upgradeType)
    while states["Auto"..upgradeType] and Rayfield:GetWindow().Enabled do
        if not inEvent then
            game:GetService("ReplicatedStorage").Remotes.Upgrade:InvokeServer(upgradeType)
        end
        task.wait(1)
    end
end

local function collectTokens()
    for _, token in ipairs(game:GetService("Workspace").Tokens:GetChildren()) do
        if token:IsA("BasePart") and Rayfield:GetWindow().Enabled then
            LocalPlayer.Character.HumanoidRootPart.CFrame = token.CFrame
            task.wait(0.2)
        end
    end
end

local function collectCrowns()
    for _, crown in ipairs(game:GetService("Workspace").Crowns:GetChildren()) do
        if crown:IsA("BasePart") and Rayfield:GetWindow().Enabled then
            LocalPlayer.Character.HumanoidRootPart.CFrame = crown.CFrame
            task.wait(0.2)
        end
    end
end

-- Sistema de Rolagem Turbo
local function autoRollPets()
    while states.AutoRollPets and Rayfield:GetWindow().Enabled do
        if not inEvent then
            local waitTime = ({1, 0.5, 0.1})[states.RollSpeed]
            game:GetService("ReplicatedStorage").Remotes.RollPet:InvokeServer()
            task.wait(waitTime)
        else
            task.wait(1)
        end
    end
end

local function autoCraftPets()
    while states.AutoCraftPets and Rayfield:GetWindow().Enabled do
        if not inEvent then
            game:GetService("ReplicatedStorage").Remotes.CraftPet:InvokeServer()
        end
        task.wait(5)
    end
end

-- Sistema Inteligente de Eventos
local function findEvent()
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name:find("Event") and obj:FindFirstChild("Circle") then
            if obj.Circle:IsA("Part") and (obj.Circle.BrickColor == BrickColor.new("Royal purple") or obj.Circle.Color == Color3.fromRGB(0.4, 0, 0.8)) then
                return obj.Circle
            end
        end
    end
    return nil
end

local function enterEvent(circle)
    if not circle then return false end
    
    LocalPlayer.Character.HumanoidRootPart.CFrame = circle.CFrame
    task.wait(2)
    
    local gui = LocalPlayer.PlayerGui:FindFirstChild("EventPopup")
    if gui and gui:FindFirstChild("ConfirmButton") then
        for _, v in ipairs(gui:GetDescendants()) do
            if v:IsA("TextButton") and v.Text:lower():find("sim") then
                VirtualInputManager:SendMouseButtonEvent(v.AbsolutePosition.X + v.AbsoluteSize.X/2, 
                                                         v.AbsolutePosition.Y + v.AbsoluteSize.Y/2, 
                                                         0, true, game, 1)
                task.wait()
                VirtualInputManager:SendMouseButtonEvent(v.AbsolutePosition.X + v.AbsoluteSize.X/2, 
                                                         v.AbsolutePosition.Y + v.AbsoluteSize.Y/2, 
                                                         0, false, game, 1)
                return true
            end
        end
    end
    return false
end

local function doEventRolls()
    inEvent = true
    eventStartTime = os.time()
    
    Rayfield:Notify({
        Title = "Evento Iniciado!",
        Content = "Participando do evento por 15 minutos",
        Duration = 6.5,
        Image = 4483362458
    })
    
    while inEvent and (os.time() - eventStartTime) < eventDuration and Rayfield:GetWindow().Enabled do
        local eventWaitTime = ({0.5, 0.2, 0.05})[states.EventSpeed]
        
        -- Rolagem no evento
        game:GetService("ReplicatedStorage").Remotes.EventRoll:InvokeServer()
        
        -- Continuar farmando durante o evento
        if states.AutoSwing then
            game:GetService("ReplicatedStorage").Remotes.Swing:FireServer()
        end
        
        task.wait(eventWaitTime)
    end
    
    inEvent = false
    Rayfield:Notify({
        Title = "Evento Concluído",
        Content = "Retomando atividades normais",
        Duration = 6.5,
        Image = 4483362458
    })
end

local function eventManager()
    while Rayfield:GetWindow().Enabled do
        if states.AutoEvent and not inEvent then
            local eventCircle = findEvent()
            if eventCircle then
                if enterEvent(eventCircle) then
                    coroutine.wrap(doEventRolls)()
                end
            end
        end
        task.wait(60)
    end
end

-- Sistema Anti-AFK
local function antiAfk()
    while Rayfield:GetWindow().Enabled do
        if states.AntiAFK then
            LocalPlayer.Idled:Connect(function()
                game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(1)
                game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            end)
        end
        task.wait(30)
    end
end

-- Elementos da interface
MainTab:CreateSection("Controles Gerais")

MainTab:CreateToggle({
    Name = "Ativar Tudo",
    CurrentValue = false,
    Flag = "AllToggle",
    Callback = function(value)
        states.AutoSwing = value
        states.AutoSell = value
        states.AutoClass = value
        states.AutoDNA = value
        states.AutoSword = value
        states.AutoTokens = value
        states.AutoCrowns = value
        states.AutoRollPets = value
        states.AutoCraftPets = value
        states.AutoEvent = value
        
        Rayfield:ChangeToggle("AutoSwing", value)
        Rayfield:ChangeToggle("AutoSell", value)
        Rayfield:ChangeToggle("AutoClass", value)
        Rayfield:ChangeToggle("AutoDNA", value)
        Rayfield:ChangeToggle("AutoSword", value)
        Rayfield:ChangeToggle("AutoTokens", value)
        Rayfield:ChangeToggle("AutoCrowns", value)
        Rayfield:ChangeToggle("AutoRollPets", value)
        Rayfield:ChangeToggle("AutoCraftPets", value)
        Rayfield:ChangeToggle("AutoEvent", value)
        
        if value then
            coroutine.wrap(autoSwing)()
            coroutine.wrap(autoSell)()
            coroutine.wrap(function() autoUpgrade("Class") end)()
            coroutine.wrap(function() autoUpgrade("DNA") end)()
            coroutine.wrap(function() autoUpgrade("Sword") end)()
            coroutine.wrap(collectTokens)()
            coroutine.wrap(collectCrowns)()
            coroutine.wrap(autoRollPets)()
            coroutine.wrap(autoCraftPets)()
            coroutine.wrap(eventManager)()
        end
    end
})

MainTab:CreateButton({
    Name = "Coletar Tudo (Tokens + Coroas)",
    Callback = function()
        coroutine.wrap(collectTokens)()
        coroutine.wrap(collectCrowns)()
    end
})

-- Auto Farm
AutoFarmTab:CreateSection("Auto Farm Básico")

AutoFarmTab:CreateToggle({
    Name = "Auto Swing Espada",
    CurrentValue = states.AutoSwing,
    Flag = "AutoSwing",
    Callback = function(value)
        states.AutoSwing = value
        if value then coroutine.wrap(autoSwing)() end
    end
})

AutoFarmTab:CreateToggle({
    Name = "Auto Vender",
    CurrentValue = states.AutoSell,
    Flag = "AutoSell",
    Callback = function(value)
        states.AutoSell = value
        if value then coroutine.wrap(autoSell)() end
    end
})

AutoFarmTab:CreateSection("Auto Upgrade")

AutoFarmTab:CreateToggle({
    Name = "Auto Upgrade Classe",
    CurrentValue = states.AutoClass,
    Flag = "AutoClass",
    Callback = function(value)
        states.AutoClass = value
        if value then coroutine.wrap(function() autoUpgrade("Class") end)() end
    end
})

AutoFarmTab:CreateToggle({
    Name = "Auto Upgrade DNA",
    CurrentValue = states.AutoDNA,
    Flag = "AutoDNA",
    Callback = function(value)
        states.AutoDNA = value
        if value then coroutine.wrap(function() autoUpgrade("DNA") end)() end
    end
})

AutoFarmTab:CreateToggle({
    Name = "Auto Upgrade Espada",
    CurrentValue = states.AutoSword,
    Flag = "AutoSword",
    Callback = function(value)
        states.AutoSword = value
        if value then coroutine.wrap(function() autoUpgrade("Sword") end)() end
    end
})

-- Pets
PetsTab:CreateSection("Gerenciamento de Pets")

PetsTab:CreateToggle({
    Name = "Auto Rolagem de Pets",
    CurrentValue = states.AutoRollPets,
    Flag = "AutoRollPets",
    Callback = function(value)
        states.AutoRollPets = value
        if value then coroutine.wrap(autoRollPets)() end
    end
})

PetsTab:CreateToggle({
    Name = "Auto Craft de Pets",
    CurrentValue = states.AutoCraftPets,
    Flag = "AutoCraftPets",
    Callback = function(value)
        states.AutoCraftPets = value
        if value then coroutine.wrap(autoCraftPets)() end
    end
})

PetsTab:CreateDropdown({
    Name = "Velocidade de Rolagem",
    Options = {"Normal", "Rápida", "Ultra"},
    CurrentOption = "Normal",
    Flag = "RollSpeed",
    Callback = function(option)
        states.RollSpeed = ({"Normal", "Rápida", "Ultra"})[option]
        Rayfield:Notify({
            Title = "Velocidade Alterada",
            Content = "Rolagem em: "..option,
            Duration = 3,
            Image = 4483362458
        })
    end
})

-- Eventos
EventTab:CreateSection("Gerenciamento de Eventos")

EventTab:CreateToggle({
    Name = "Participação Automática em Eventos",
    CurrentValue = states.AutoEvent,
    Flag = "AutoEvent",
    Callback = function(value)
        states.AutoEvent = value
        if value then 
            coroutine.wrap(eventManager)()
            coroutine.wrap(antiAfk)()
        end
    end
})

EventTab:CreateButton({
    Name = "Encontrar Evento AGORA",
    Callback = function()
        local eventCircle = findEvent()
        if eventCircle then
            enterEvent(eventCircle)
        else
            Rayfield:Notify({
                Title = "Evento Não Encontrado",
                Content = "O evento não está ativo neste servidor",
                Duration = 6.5,
                Image = 4483362458
            })
        end
    end
})

EventTab:CreateDropdown({
    Name = "Velocidade no Evento",
    Options = {"Normal", "Rápida", "Ultra"},
    CurrentOption = "Normal",
    Flag = "EventSpeed",
    Callback = function(option)
        states.EventSpeed = ({"Normal", "Rápida", "Ultra"})[option]
        Rayfield:Notify({
            Title = "Velocidade no Evento",
            Content = "Definida para: "..option,
            Duration = 3,
            Image = 4483362458
        })
    end
})

-- Configurações
SettingsTab:CreateSection("Configurações Gerais")

SettingsTab:CreateToggle({
    Name = "Sistema Anti-AFK",
    CurrentValue = states.AntiAFK,
    Flag = "AntiAFK",
    Callback = function(value)
        states.AntiAFK = value
        if value then coroutine.wrap(antiAfk)() end
    end
})

SettingsTab:CreateButton({
    Name = "Recarregar Interface",
    Callback = function()
        Rayfield:Destroy()
        loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    end
})

SettingsTab:CreateButton({
    Name = "Copiar Link do Jogo",
    Callback = function()
        setclipboard("https://www.roblox.com/games/5780309044/Saber-Simulator")
        Rayfield:Notify({
            Title = "Link Copiado!",
            Content = "Cole no navegador para compartilhar",
            Duration = 3,
            Image = 4483362458
        })
    end
})

SettingsTab:CreateKeybind({
    Name = "Mostrar/Esconder Interface",
    CurrentKeybind = "RightControl",
    HoldToInteract = false,
    Flag = "UIKeybind",
    Callback = function()
        Rayfield:GetWindow():Toggle()
    end
})

SettingsTab:CreateSection("Suporte")
SettingsTab:CreateLabel("Criado especialmente para você!")
SettingsTab:CreateLabel("Script atualizado: 20/06/2025")

-- Iniciar sistemas
coroutine.wrap(antiAfk)()
coroutine.wrap(eventManager)()

-- Notificação inicial
Rayfield:Notify({
    Title = "SABER SIMULATOR ULTIMATE",
    Content = "Bem-vindo ao script premium!",
    Duration = 8,
    Image = 4483362458
})
