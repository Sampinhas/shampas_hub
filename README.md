-- Verifica se o jogo está carregado
if not game:IsLoaded() then
    game.Loaded:Wait() -- Espera o jogo carregar
end
-- Carregar a Fluent UI e os addons
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local P = game:GetService("Players");
local selff = P.LocalPlayer;
local PSG = selff:WaitForChild("PlayerGui");
Fluent:Notify({
    Title = "Shampas HUB v1.0",
    Content = "Carregando em alguns segundos...",
    Duration = 8
})

-- Serviços necessários
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer

-- Função para escolher o lado automaticamente
local function chooseTeam(team)
    if team == "Pirates" or team == "Marines" then
        ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", team) -- Escolhe o time
        print("Você escolheu automaticamente o lado dos " .. team .. "!")
    else
        warn("Time inválido! Escolha 'Pirates' ou 'Marines'.")
    end
end

wait(1)

-- Escolhe automaticamente os Piratas
chooseTeam("Pirates")

wait(7)


-- Criar a janela principal
local Window = Fluent:CreateWindow({
    Title = "Shampas HUB v1.0",
    SubTitle = "feito por shampas 😍",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true, -- The blur may be detectable, setting this to false disables blur entirely
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl -- Used when theres no MinimizeKeybind
})

-- Variáveis globais
local World1, World2, World3
local MyLevel = game:GetService("Players").LocalPlayer.Data.Level.Value
local equiparEspadasAtivo = false

-- Verifica o mundo atual
if game.PlaceId == 2753915549 then
    World1 = true
elseif game.PlaceId == 4442272183 then
    World2 = true
elseif game.PlaceId == 7449423635 then
    World3 = true
else
    game:GetService("Players").LocalPlayer:Kick("Jogo não suportado. Por favor, aguarde...")
end

-- Função Hop
function Hop()
    local PlaceID = game.PlaceId
    local AllIDs = {}
    local foundAnything = ""
    local actualHour = os.date("!*t").hour
    local Deleted = false

    function TPReturner()
        local Site;
        if foundAnything == "" then
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
        else
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
        end
        local ID = ""
        if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
            foundAnything = Site.nextPageCursor
        end
        local num = 0;
        for i, v in pairs(Site.data) do
            local Possible = true
            ID = tostring(v.id)
            if tonumber(v.maxPlayers) > tonumber(v.playing) then
                for _, Existing in pairs(AllIDs) do
                    if num ~= 0 then
                        if ID == tostring(Existing) then
                            Possible = false
                        end
                    else
                        if tonumber(actualHour) ~= tonumber(Existing) then
                            local delFile = pcall(function()
                                AllIDs = {}
                                table.insert(AllIDs, actualHour)
                            end)
                        end
                    end
                    num = num + 1
                end
                if Possible == true then
                    table.insert(AllIDs, ID)
                    wait()
                    pcall(function()
                        wait()
                        game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                    end)
                    wait(4)
                end
            end
        end
    end

    function Teleport()
        while wait() do
            pcall(function()
                TPReturner()
                if foundAnything ~= "" then
                    TPReturner()
                end
            end)
        end
    end

    Teleport()
end

-- Função CheckQuest
function CheckQuest()
    if World1 then
        if MyLevel == 1 or MyLevel <= 9 then
            Mon = "Bandit"
            LevelQuest = 1
            NameQuest = "BanditQuest1"
            NameMon = "Bandit"
            CFrameQuest = CFrame.new(1059.37195, 15.4495068, 1550.4231, 0.939700544, -0, -0.341998369, 0, 1, -0, 0.341998369, 0, 0.939700544)
            CFrameMon = CFrame.new(1045.962646484375, 27.00250816345215, 1560.8203125)
        elseif MyLevel == 10 or MyLevel <= 14 then
            Mon = "Monkey"
            LevelQuest = 1
            NameQuest = "JungleQuest"
            NameMon = "Monkey"
            CFrameQuest = CFrame.new(-1598.08911, 35.5501175, 153.377838, 0, 0, 1, 0, 1, -0, -1, 0, 0)
            CFrameMon = CFrame.new(-1448.51806640625, 67.85301208496094, 11.46579647064209)
        -- Continua com as outras condições do CheckQuest...
        end
    elseif World2 then
        -- Condições para o World2...
    elseif World3 then
        -- Condições para o World3...
    end
end

-- Função Auto Buy Legendary Sword
function AutoBuyLegendarySword()
    spawn(function()
        while wait() do
            if _G.AutoBuyLegendarySword then
                pcall(function()
                    local args1 = { [1] = "LegendarySwordDealer", [2] = "1" }
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args1))

                    local args2 = { [1] = "LegendarySwordDealer", [2] = "2" }
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args2))

                    local args3 = { [1] = "LegendarySwordDealer", [2] = "3" }
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args3))

                    if _G.AutoBuyLegendarySword_Hop and World2 then
                        wait(10)
                        Hop()
                    end
                end)
            end
        end
    end)
end

-- Função AutoHaki
function AutoHaki()
    if not game:GetService("Players").LocalPlayer.Character:FindFirstChild("HasBuso") then
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
    end
end


local player = game.Players.LocalPlayer
local replicatedStorage = game:GetService("ReplicatedStorage")
local equiparEspadasAtivo = false

-- Função para solicitar a espada ao NPC/jogo
local function claimSword(swordName)
    local remote = replicatedStorage:FindFirstChild("Remotes"):FindFirstChild("CommF_")
    if remote then
        print("Tentando pegar a espada " .. swordName .. "...")
        local success, response = pcall(function()
            return remote:InvokeServer("LoadItem", swordName)
        end)

        if success then
            print("Espada " .. swordName .. " retirada do sistema!")
        else
            print("Falha ao pegar " .. swordName .. ":", response)
        end
    else
        print("Não foi possível encontrar a função de retirada!")
    end
end

-- Função para equipar a espada
local function equipSword(swordName)
    local character = player.Character or player.CharacterAdded:Wait()
    local backpack = player.Backpack

    -- Procura a espada na mochila ou no personagem
    local sword = backpack:FindFirstChild(swordName) or character:FindFirstChild(swordName)

    if sword then
        print("Equipando " .. swordName .. "...")
        character.Humanoid:EquipTool(sword)
        print(swordName .. " equipada com sucesso!")
    else
        print(swordName .. " não encontrada no inventário!")
    end
end

-- Função para verificar a maestria
local function verificarMaestria()
    local playerGui = player:WaitForChild("PlayerGui")
    local allTextLabels = playerGui:GetDescendants()

    for _, object in pairs(allTextLabels) do
        if object:IsA("TextLabel") then
            if string.find(object.Text, "Mastery") or string.find(object.Text, "Maestria") then
                local numeroMaestria = object.Text:match("%d+")
                if numeroMaestria then
                    return tonumber(numeroMaestria)
                end
            end
        end
    end
    return 0
end

-- Função principal para equipar as espadas e verificar a maestria
local function equiparEspadas()
    local espadas = {"Oroshi", "Saishi", "Shizu"}

    while equiparEspadasAtivo do
        for _, espada in ipairs(espadas) do
            if not equiparEspadasAtivo then return end -- Para o loop se o usuário desligar

            claimSword(espada)
            wait(1)
            equipSword(espada)

            local maestria = 0
            while maestria < 300 and equiparEspadasAtivo do
                wait(2)
                maestria = verificarMaestria()
                print("Maestria atual de " .. espada .. ": " .. maestria)

                -- Garantir que a espada continue equipada
                local character = player.Character or player.CharacterAdded:Wait()
                local currentTool = character.Humanoid:FindFirstChildOfClass("Tool")
                if currentTool and currentTool.Name ~= espada then
                    print("Reequipando " .. espada .. "...")
                    equipSword(espada)
                end
            end

            print("Maestria de " .. espada .. " atingiu 300. Equipando próxima espada.")
        end

        print("Todas as espadas equipadas e maestrias atingidas!")
        wait(5) -- Delay antes de reiniciar o processo
    end
end

print("Criando abas...")
local Tabs = {
    principal = Window:AddTab({ Title = "Principal", Icon = "" }),
    Teleporte = Window:AddTab({ Title = "Teleporte", Icon = "" }),
    Outros = Window:AddTab({ Title = "Outros", Icon = "" })
}
print("Abas criadas!")

-- Função para salvar as configurações
local function saveSettings()
    local settings = {
        EquiparEspadasAtivo = equiparEspadasAtivo,
        AutoBuyLegendarySword = _G.AutoBuyLegendarySword
    }
    SaveManager:SaveConfiguration(player.Name, settings) -- Salva as configurações no SaveManager
end

-- Função para carregar as configurações
local function loadSettings()
    local settings = SaveManager:LoadConfiguration(player.Name) -- Carrega as configurações do SaveManager
    if settings then
        equiparEspadasAtivo = settings.EquiparEspadasAtivo
        _G.AutoBuyLegendarySword = settings.AutoBuyLegendarySword
        -- Atualiza a interface com os valores carregados
        Tabs.principal:SetToggle("Equipar Espadas", equiparEspadasAtivo)
        Tabs.principal:SetToggle("Auto Buy Legendary Sword", _G.AutoBuyLegendarySword)
    end
end

-- Função de Toggle sem salvar configurações
Tabs.principal:AddToggle("Auto Farm Bone", {
    Title = "Auto Farm Bone",
    Description = "Liga/desliga o AutoFarm de ossos no Sea 3",
    Default = false, -- Inicializa sem depender de um valor carregado
    Callback = function(Value)
        Configs.Farming.Sea3.AutoFarmBone = Value  -- A variável é alterada diretamente
        if Value then
            print("AutoFarm Bone ativado!")
            AutoFarmBone()  -- Inicia o loop de AutoFarm
        else
            print("AutoFarm Bone desativado!")
        end
    end
})

-- Função de Toggle com salvamento automático
Tabs.principal:AddToggle("Equipar Espadas", {
    Title = "Equipar Espadas",
    Description = "Liga/desliga o equipar espadas",
    Default = false,
    Callback = function(Value)
        equiparEspadasAtivo = Value
        if Value then
            print("Equipar espadas ativado!")
            task.spawn(equiparEspadas)
        else
            print("Equipar espadas desativado!")
        end
        saveSettings() -- Salva as configurações ao alterar o toggle
    end
})

Tabs.principal:AddToggle("Auto Buy Legendary Sword", {
    Title = "Auto Buy Legendary Sword",
    Description = "Liga/desliga a compra automática de espadas lendárias",
    Default = false,
    Callback = function(Value)
        _G.AutoBuyLegendarySword = Value
        if Value then
            print("Auto Buy Legendary Sword ativado!")
            AutoBuyLegendarySword()
        else
            print("Auto Buy Legendary Sword desativado!")
        end
        saveSettings() -- Salva as configurações ao alterar o toggle
    end
})

-- Carregar configurações ao iniciar o script
loadSettings()

-- Outros botões e funções
Tabs.Teleporte:AddButton({
    Title = "Teleportar para Sea 1",
    Description = "Clique para teleportar para o Sea 1",
    Callback = function()
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("TravelMain")
    end
})

Tabs.Teleporte:AddButton({
    Title = "Teleportar para Sea 2",
    Description = "Clique para teleportar para o Sea 2",
    Callback = function()
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("TravelDressrosa")
    end
})

Tabs.Teleporte:AddButton({
    Title = "Teleportar para Sea 3",
    Description = "Clique para teleportar para o Sea 3",
    Callback = function()
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("TravelZou")
    end
})

Tabs.Teleporte:AddButton({
    Title = "Hop para outro Server",
    Description = "Clique para trocar de server",
    Callback = function()
        Hop()
    end
})

Tabs.Outros:AddButton({
    Title = "Verificar Quest",
    Description = "Clique para verificar a quest atual",
    Callback = function()
        CheckQuest()
    end
})

-- Inicializar SaveManager e InterfaceManager
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)

SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})

InterfaceManager:SetFolder("ShampasHub")
SaveManager:SetFolder("ShampasHub/specific-game")

InterfaceManager:BuildInterfaceSection(Tabs.Outros)
SaveManager:BuildConfigSection(Tabs.Outros)

-- Selecionar a primeira aba
Window:SelectTab(1)

-- Notificação de carregamento
Fluent:Notify({
    Title = "Shampas HUB v1.0",
    Content = "A interface foi carregada com sucesso!",
    Duration = 8
})

if not Fluent then
    print("Erro ao carregar Fluent!")
else
    print("Fluent carregado com sucesso!")
end
