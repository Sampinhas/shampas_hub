-- Verifica se o jogo está carregado
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Carregar a Fluent UI e os addons
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Serviços necessários
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer


local function esperarObjeto(pasta, nome)
    while not pasta:FindFirstChild(nome) do
        task.wait()
    end
    return pasta:FindFirstChild(nome)
end

local Effect = esperarObjeto(ReplicatedStorage, "Effect")
local Container = esperarObjeto(Effect, "Container")
local Anchor = esperarObjeto(Container, "Anchor")

-- Verifica se Anchor tem a propriedade correta antes de acessá-la
if Anchor and Anchor:IsA("BasePart") then
    print("Z Position:", Anchor.Position.Z) -- Certifique-se de acessar a propriedade correta
else
    warn("Anchor não encontrado ou não é uma peça válida!")
end


-- Definir a pasta de configuração para ShampasHub
SaveManager:SetFolder("ShampasHub")

-- Função para escolher o lado automaticamente
local function chooseTeam(team)
    if team == "Pirates" or team == "Marines" then
        ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", team) -- Escolhe o time
        print("Você escolheu automaticamente o lado dos " .. team .. "!")
    else
        warn("Time inválido! Escolha 'Pirates' ou 'Marines'.")
    end
end

-- Escolhe automaticamente os Piratas
chooseTeam("Pirates")

-- Variáveis globais
local equiparEspadasAtivo = false
_G = {
    AutoBuyLegendarySword = false
}

-- Função para salvar dados em um arquivo JSON
local function salvarDados(dados)
    local nomeConta = player.Name
    local caminho = "ShampasHub/Temp/" .. nomeConta .. "_data_config.json"
    
    local jsonDados = game:GetService("HttpService"):JSONEncode(dados)
    
    writefile(caminho, jsonDados)
    print("Dados salvos com sucesso em " .. caminho)
end

-- Função para carregar dados de um arquivo JSON
local function carregarDados()
    local nomeConta  = player.Name
    local caminho = "ShampasHub/Temp/" .. nomeConta .. "_data_config.json"
    
    if isfile(caminho) then
        local jsonDados = readfile(caminho)
        local dados = game:GetService("HttpService"):JSONDecode(jsonDados)
        print("Dados carregados com sucesso de " .. caminho)
        return dados
    else
        print("Nenhum dado salvo encontrado.")
        return {}
    end
end

local dados = carregarDados()  -- Chama a função para carregar os dados
    
equiparEspadasAtivo = dados.EquiparEspadasAtivo or false  -- Define valores padrões, caso os dados estejam vazios
_G.AutoBuyLegendarySword = dados.AutoBuyLegendarySword or false

-- Criar a janela principal
local Window = Fluent:CreateWindow({
    Title = "Shampas HUB v1.0",
    SubTitle = "feito por shampas 😍",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- Variáveis globais
local World1, World2, World3
local MyLevel = game:GetService("Players").LocalPlayer.Data.Level.Value

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
    -- Acessando a GUI do jogador e a parte que pode conter as informações de maestria
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    local allTextLabels = playerGui:GetDescendants()

    -- Variáveis para armazenar a palavra "Mastery" ou "Maestria"
    local palavra1 = "Mastery"
    local palavra2 = "Maestria"

    -- Percorrer todos os TextLabels na GUI para procurar pelas palavras
    for _, object in pairs(allTextLabels) do
        if object:IsA("TextLabel") then
            -- Verificar se o texto contém "Mastery" ou "Maestria"
            if string.find(object.Text, palavra1) or string.find(object.Text, palavra2) then
                -- Extrair o número (maestria) ao lado do texto
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

-- Criar abas
local Tabs = {
    principal = Window:AddTab({ Title = "Principal", Icon = "" }),
    Teleporte = Window:AddTab({ Title = "Teleporte", Icon = "" }),
    Outros = Window:AddTab({ Title = "Outros", Icon = "" })
}

-- Função de Toggle com salvamento automático
Tabs.principal:AddToggle("Equipar Espadas", {
    Title = "Equipar Espadas",
    Description = "Liga/desliga o equipar espadas",
    Default = equiparEspadasAtivo,
    Callback = function(Value)
        equiparEspadasAtivo = Value
        if Value then
            print("Equipar espadas ativado!")
            task.spawn(equiparEspadas)
        else
            print("Equipar espadas desativado!")
        end
        salvarDados(dados)
    end
})

Tabs.principal:AddToggle("Auto Buy Legendary Sword", {
    Title = "Auto Buy Legendary Sword",
    Description = "Liga/desliga a compra automática de espadas lendárias",
    Default = _G.AutoBuyLegendarySword,
    Callback = function(Value)
        _G.AutoBuyLegendarySword = Value
        if Value then
            print("Auto Buy Legendary Sword ativado!")
            AutoBuyLegendarySword()
        else
            print("Auto Buy Legendary Sword desativado!")
        end
        local dados = {
            EquiparEspadasAtivo = equiparEspadasAtivo,
            AutoBuyLegendarySword = _G.AutoBuyLegendarySword
        }
        salvarDados(dados)
    end
})

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
SaveManager:SetFolder("ShampasHub")

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
