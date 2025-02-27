task.wait(10)
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
    AutoBuyLegendarySword = false,
    Auto_Bone = false,
    BringMode = 375
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
    
    -- Verifica se o arquivo existe
    if isfile(caminho) then
        local jsonDados = readfile(caminho)
        local dados = game:GetService("HttpService"):JSONDecode(jsonDados)
        print("Dados carregados com sucesso de " .. caminho)
        return dados
    else
        print("Nenhum dado salvo encontrado. Criando base de dados padrão...")
        
        -- Caso o arquivo não exista, cria uma base de dados padrão
        local dadosPadrao = {
            EquiparEspadasAtivo = false,
            AutoBuyLegendarySword = false,
            Auto_Bone = false
        }
        
        -- Salva os dados padrão
        salvarDados(dadosPadrao)
        return dadosPadrao
    end
end

local dados = carregarDados()

-- Verifica se a chave EquiparEspadasAtivo existe
if dados.EquiparEspadasAtivo == nil then
    dados.EquiparEspadasAtivo = false  -- Define o valor padrão se não existir
end
equiparEspadasAtivo = dados.EquiparEspadasAtivo

-- Verifica se a chave EquiparEspadasAtivo existe
if dados.Auto_Bone == nil then
    dados.Auto_Bone = false  -- Define o valor padrão se não existir
end
Auto_Bone = dados.Auto_Bone

-- Verifica se a chave EquiparEspadasAtivo existe
if dados.AutoBuyLegendarySword == nil then
    dados.AutoBuyLegendarySword = false  -- Define o valor padrão se não existir
end
_G.AutoBuyLegendarySword = dados.AutoBuyLegendarySword


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

local replicatedStorage = game:GetService("ReplicatedStorage")

-- Função para pegar o dinheiro (Beli)
local function pegarBeli()
    local dataFolder = player:FindFirstChild("Data")
    if dataFolder then
        local money = dataFolder:FindFirstChild("Beli")
        if money then
            return money.Value
        end
    end
    return 0
end

local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")

-- Função para verificar se o jogador está no Second Sea (Game ID: 4442272183)
local function verificarSecondSea()
    local placeId = game.PlaceId -- Obtém o ID do jogo atual
    return placeId == 4442272183 -- Retorna true se o jogador estiver no Second Sea
end

-- Função para ativar noclip
local function ativarNoClip()
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end

-- Função para mover o jogador até o NPC
local function moverParaNPC(posicaoNPC)
    ativarNoClip()
    humanoidRootPart.CFrame = CFrame.new(posicaoNPC)
end

-- Função para viajar para o Second Sea
local function viajarParaSecondSea()
    print("🛳️ Verificando se " .. player.Name .. " pode viajar para Second Sea...")
    local money = pegarBeli()

    if money >= 2000000 then
        print("💰 Você tem " .. money .. " Beli! Indo até o NPC...")

        -- Defina a posição do NPC aqui
        local posicaoNPC = Vector3.new(1234, 100, 5678) -- Substitua pela posição real do NPC

        moverParaNPC(posicaoNPC)
        wait(1) -- Pequena espera para garantir que chegou ao NPC

        print("🚢 Chegou ao NPC! Viajando para Second Sea...")
        replicatedStorage.Remotes.CommF_:InvokeServer("TravelDressrosa")
    else
        print("❌ Você tem " .. money .. " Beli. Precisa de pelo menos 2.000.000.")
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

    local palavra1 = "Mastery"
    local palavra2 = "Maestria"

    for _, object in pairs(allTextLabels) do
        if object:IsA("TextLabel") then
            if string.find(object.Text, palavra1) or string.find(object.Text, palavra2) then
                local numeroMaestria = object.Text:match("%d+")
                if numeroMaestria then
                    return tonumber(numeroMaestria)
                end
            end
        end
    end

    return 0
end

-- Função para equipar as espadas e verificar a maestria
local function equiparEspadas()
    local espadas = {"Oroshi", "Saishi", "Shizu"}

    for _, espada in ipairs(espadas) do
        claimSword(espada)
        wait(1)
        equipSword(espada)

        local maestria = 0
        while maestria < 300 do
            wait(2)
            maestria = verificarMaestria()
            print("Maestria atual de " .. espada .. ": " .. maestria)

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

    -- Agora verifica o dinheiro para viajar
    viajarParaSecondSea()
end

-- Função para transportar o jogador para as coordenadas especificadas no Second Sea
local function transportarParaCoordenadas()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

    -- Coordenadas de destino
    local destinoFinal = Vector3.new(-2568.31, 1623.49, -3740.51)

    -- Criar a plataforma flutuante
    local plataforma = Instance.new("Part")
    plataforma.Size = Vector3.new(6, 1, 6)  -- Tamanho da plataforma
    plataforma.Position = humanoidRootPart.Position - Vector3.new(0, 3, 0) -- Abaixo do personagem
    plataforma.Anchored = true
    plataforma.Transparency = 1  -- Deixa invisível
    plataforma.CanCollide = true
    plataforma.Parent = workspace

    -- Criar um Tween para mover a plataforma suavemente (agora mais rápido)
    local tweenService = game:GetService("TweenService")
    local tempoDeVoo = (humanoidRootPart.Position - destinoFinal).Magnitude / 150  -- Aumentamos a velocidade!
    local info = TweenInfo.new(math.max(tempoDeVoo, 2), Enum.EasingStyle.Linear)  -- Tempo mínimo de 2 segundos
    local objetivo = {Position = destinoFinal - Vector3.new(0, 3, 0)}

    local tween = tweenService:Create(plataforma, info, objetivo)
    tween:Play()

    -- Manter o personagem sobre a plataforma
    local function acompanharPlataforma()
        while tween.PlaybackState == Enum.PlaybackState.Playing do
            humanoidRootPart.CFrame = plataforma.CFrame + Vector3.new(0, 3, 0) -- Mantém o player em cima
            wait()
        end
        print("Chegamos ao destino!")

        -- Verifica se o jogador está nas coordenadas corretas
        local posicaoAtual = humanoidRootPart.Position
        local distancia = (posicaoAtual - destinoFinal).Magnitude

        if distancia <= 5 then -- Se estiver próximo das coordenadas (tolerância de 5 unidades)
        else
            print("O jogador não chegou às coordenadas corretas.")
        end
    end

    -- Iniciar voo
    acompanharPlataforma()
end

-- Função para verificar e executar equiparEspadas quando ativo
local function verificarEquiparEspadas()
    while true do
        wait(1) -- Verifica a cada 1 segundo
        if equiparEspadasAtivo then
            print("Equipar espadas ativado! Iniciando processo...")
            equiparEspadas()
        end
    end
end

-- Inicia a verificação em uma thread separada
spawn(verificarEquiparEspadas)

spawn(function()
    game:GetService("RunService").Heartbeat:Connect(function()
        if _G.AutoVampire or AutoFarmChest or _G.AutoAdvanceDungeon or _G.AutoDoughtBoss or _G.Auto_DungeonMobAura or _G.AutoFarmChest or _G.AutoFactory or _G.AutoFarmBossHallow or _G.AutoFarmSwanGlasses or _G.AutoLongSword or _G.AutoBlackSpikeycoat or _G.AutoElectricClaw or _G.AutoFarmGunMastery or _G.AutoHolyTorch or _G.AutoLawRaid or _G.AutoFarmBoss or _G.AutoTwinHooks or _G.AutoOpenSwanDoor or _G.AutoDragon_Trident or _G.AutoSaber or _G.NOCLIP or _G.AutoFarmFruitMastery or _G.AutoFarmGunMastery or _G.TeleportIsland or _G.Auto_EvoRace or _G.AutoFarmAllMsBypassType or _G.AutoObservationv2 or _G.AutoMusketeerHat or _G.AutoEctoplasm or _G.AutoRengoku or _G.Auto_Rainbow_Haki or _G.AutoObservation or _G.AutoDarkDagger or _G.Safe_Mode or _G.MasteryFruit or _G.AutoBudySword or _G.AutoOderSword or _G.AutoBounty or _G.AutoAllBoss or _G.Auto_Bounty or _G.AutoSharkman or _G.Auto_Mastery_Fruit or _G.Auto_Mastery_Gun or _G.Auto_Dungeon or _G.Auto_Cavender or _G.Auto_Pole or _G.Auto_Kill_Ply or _G.Auto_Factory or _G.AutoSecondSea or _G.TeleportPly or _G.AutoBartilo or _G.Auto_DarkBoss or _G.GrabChest or _G.AutoFarmBounty or _G.Holy_Torch or _G.AutoFarm or _G.Clip or _G.AutoElitehunter or _G.AutoThirdSea or _G.Auto_Bone or _G.Autoheart or _G.Autodoughking or _G.AutoFarmMaterial or _G.AutoNevaSoulGuitar or _G.Auto_Dragon_Trident or _G.Autotushita or _G.d or _G.Autowaden or _G.Autogay or _G.Autopole or _G.Autosaw or _G.AutoObservationHakiV2 or _G.AutoFarmNearest or AutoFarmChest or _G.AutoCarvender or _G.AutoTwinHook or AutoMobAura or _G.Tweenfruit or _G.AutoKai or _G.TeleportNPC or _G.Leather or _G.Auto_Wing or _G.Umm or _G.Makori_gay or Radioactive or Fish or Gunpowder or Dragon_Scale or Cocoafarm or Scrap or MiniHee or _G.AutoFarmSeabaest or Auto_Cursed_Dual_Katana or _G.AutoFarmMob or _G.AutoMysticIsland or _G.AutoFarmDungeon or _G.AutoRaidPirate or _G.AutoQuestRace or _G.TweenMGear or getgenv().AutoFarm or _G.AutoPlayerHunter or _G.AutoFactory or Grab_Chest or _G.Namfon or _G.AutoSwordMastery or _G.AutoSeaBest or _G.AutoKillTial or _G.Auto_Saber or _G.Position_Spawn or _G.Farmfast or _G.AutoRace or _G.RaidPirate then
            if not game:GetService("Workspace"):FindFirstChild("LOL") then
                local LOL = Instance.new("Part")
                LOL.Name = "LOL"
                LOL.Parent = game.Workspace
                LOL.Anchored = true
                LOL.Transparency = 1
                LOL.Size = Vector3.new(30,-0.5,30)
            elseif game:GetService("Workspace"):FindFirstChild("LOL") then
                game.Workspace["LOL"].CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(0, -3.6, 0)
            end
        else
            if game:GetService("Workspace"):FindFirstChild("LOL") then
                game:GetService("Workspace"):FindFirstChild("LOL"):Destroy()
            end
        end
    end)
end)
    
spawn(function()
    pcall(function()
        while wait() do
            if _G.AutoVampire or AutoFarmChest or _G.AutoAdvanceDungeon or _G.AutoDoughtBoss or _G.Auto_DungeonMobAura or _G.AutoFarmChest or _G.AutoFactory or _G.AutoFarmBossHallow or _G.AutoFarmSwanGlasses or _G.AutoLongSword or _G.AutoBlackSpikeycoat or _G.AutoElectricClaw or _G.AutoFarmGunMastery or _G.AutoHolyTorch or _G.AutoLawRaid or _G.AutoFarmBoss or _G.AutoTwinHooks or _G.AutoOpenSwanDoor or _G.AutoDragon_Trident or _G.AutoSaber or _G.AutoFarmFruitMastery or _G.AutoFarmGunMastery or _G.TeleportIsland or _G.Auto_EvoRace or _G.AutoFarmAllMsBypassType or _G.AutoObservationv2 or _G.AutoMusketeerHat or _G.AutoEctoplasm or _G.AutoRengoku or _G.Auto_Rainbow_Haki or _G.AutoObservation or _G.AutoDarkDagger or _G.Safe_Mode or _G.MasteryFruit or _G.AutoBudySword or _G.AutoOderSword or _G.AutoBounty or _G.AutoAllBoss or _G.Auto_Bounty or _G.AutoSharkman or _G.Auto_Mastery_Fruit or _G.Auto_Mastery_Gun or _G.Auto_Dungeon or _G.Auto_Cavender or _G.Auto_Pole or _G.Auto_Kill_Ply or _G.Auto_Factory or _G.AutoSecondSea or _G.TeleportPly or _G.AutoBartilo or _G.Auto_DarkBoss or _G.GrabChest or _G.AutoFarmBounty or _G.Holy_Torch or _G.AutoFarm or _G.Clip or FarmBoss or _G.AutoElitehunter or _G.AutoThirdSea or _G.Auto_Bone or _G.Autoheart or _G.Autodoughking or _G.AutoFarmMaterial or _G.AutoNevaSoulGuitar or _G.Auto_Dragon_Trident or _G.Autotushita or _G.d or _G.Autowaden or _G.Autogay or _G.Autopole or _G.Autosaw or _G.AutoObservationHakiV2 or _G.AutoFarmNearest or AutoFarmChest or _G.AutoCarvender or _G.AutoTwinHook or AutoMobAura or _G.Tweenfruit or _G.TeleportNPC or _G.Leather or _G.Auto_Wing or _G.Umm or _G.Makori_gay or Radioactive or Fish or Gunpowder or Dragon_Scale or Cocoafarm or Scrap or MiniHee or _G.AutoFarmSeabaest or Auto_Cursed_Dual_Katana or _G.AutoFarmMob or _G.AutoMysticIsland or _G.AutoFarmDungeon or _G.AutoRaidPirate or _G.AutoQuestRace or _G.TweenMGear or getgenv().AutoFarm or _G.AutoPlayerHunter or _G.AutoFactory or Grab_Chest or _G.Namfon or _G.AutoSwordMastery or _G.Auto_Seabest or _G.AutoSeaBest or _G.AutoKillTial or _G.Auto_Saber or _G.Position_Spawn or _G.Farmfast or _G.AutoRace or _G.RaidPirate == true then
                if not game:GetService("Players").LocalPlayer.Character.HumanoidRootPart:FindFirstChild("BodyClip") then
                    local Noclip = Instance.new("BodyVelocity")
                    Noclip.Name = "BodyClip"
                    Noclip.Parent = game:GetService("Players").LocalPlayer.Character.HumanoidRootPart
                    Noclip.MaxForce = Vector3.new(100000,100000,100000)
                    Noclip.Velocity = Vector3.new(0,0,0)
                end
            end
        end
    end)
end)
    
spawn(function()
    pcall(function()
        game:GetService("RunService").Stepped:Connect(function()
            if _G.AutoVampire or AutoFarmChest or _G.AutoAdvanceDungeon or _G.AutoDoughtBoss or _G.Auto_DungeonMobAura or _G.AutoFarmChest or _G.AutoFactory or _G.AutoFarmBossHallow or _G.AutoFarmSwanGlasses or _G.AutoLongSword or _G.AutoBlackSpikeycoat or _G.AutoElectricClaw or _G.AutoFarmGunMastery or _G.AutoHolyTorch or _G.AutoLawRaid or _G.AutoFarmBoss or _G.AutoTwinHooks or _G.AutoOpenSwanDoor or _G.AutoDragon_Trident or _G.AutoSaber or _G.NOCLIP or _G.AutoFarmFruitMastery or _G.AutoFarmGunMastery or _G.TeleportIsland or _G.Auto_EvoRace or _G.AutoFarmAllMsBypassType or _G.AutoObservationv2 or _G.AutoMusketeerHat or _G.AutoEctoplasm or _G.AutoRengoku or _G.Auto_Rainbow_Haki or _G.AutoObservation or _G.AutoDarkDagger or _G.Safe_Mode or _G.MasteryFruit or _G.AutoBudySword or _G.AutoOderSword or _G.AutoBounty or _G.AutoAllBoss or _G.Auto_Bounty or _G.AutoSharkman or _G.Auto_Mastery_Fruit or _G.Auto_Mastery_Gun or _G.Auto_Dungeon or _G.Auto_Cavender or _G.Auto_Pole or _G.Auto_Kill_Ply or _G.Auto_Factory or _G.AutoSecondSea or _G.TeleportPly or _G.AutoBartilo or _G.Auto_DarkBoss or _G.GrabChest or _G.AutoFarmBounty or _G.Holy_Torch or _G.AutoFarm or _G.Clip or _G.AutoElitehunter or _G.AutoThirdSea or _G.Auto_Bone or _G.Autoheart or _G.Autodoughking or _G.AutoFarmMaterial or _G.AutoNevaSoulGuitar or _G.Auto_Dragon_Trident or _G.Autotushita or _G.Autowaden or _G.Autogay or _G.Autopole or _G.Autosaw or _G.AutoObservationHakiV2 or _G.AutoFarmNearest or _G.AutoCarvender or _G.AutoTwinHook or AutoMobAura or _G.Tweenfruit or _G.TeleportNPC or _G.AutoKai or _G.Leather or _G.Auto_Wing or _G.Umm or _G.Makori_gay or Radioactive or Fish or Gunpowder or Dragon_Scale or Cocoafarm or Scrap or MiniHee or _G.AutoFarmSeabaest or Auto_Cursed_Dual_Katana or _G.AutoFarmMob or _G.AutoMysticIsland or _G.AutoFarmDungeon or _G.AutoRaidPirate or _G.AutoQuestRace or _G.TweenMGear or getgenv().AutoFarm or _G.AutoPlayerHunter or _G.AutoFactory or _G.Namfon or _G.AutoSwordMastery or _G.Auto_Seabest or _G.AutoSeaBest or _G.AutoKillTial or _G.Auto_Saber or _G.Position_Spawn or _G.TPB or _G.Farmfast or _G.AutoRace or _G.RaidPirate == true then
                for _, v in pairs(game:GetService("Players").LocalPlayer.Character:GetDescendants()) do
                    if v:IsA("BasePart") then
                        v.CanCollide = false    
                    end
                end
            end
        end)
    end)
end)

spawn(function()
    game:GetService("RunService").RenderStepped:Connect(function()
        if _G.AutoClick or Fastattack then
             pcall(function()
                game:GetService'VirtualUser':CaptureController()
                game:GetService'VirtualUser':Button1Down(Vector2.new(0,1,0,1))
            end)
        end
    end)
end)

function topos(Pos)
    Distance = (Pos.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
    if Distance < 25 then
        Speed = 5000
    elseif Distance < 50 then
        Speed = 2000
    elseif Distance < 150 then
        Speed = 800
    elseif Distance < 250 then
        Speed = 600
    elseif Distance < 500 then
        Speed = 300
    elseif Distance < 750 then
        Speed = 250
    elseif Distance >= 1000 then
        Speed = 200
    end
    game:GetService("TweenService"):Create(
        game:GetService("Players").LocalPlayer.Character.HumanoidRootPart,
        TweenInfo.new(Distance/Speed, Enum.EasingStyle.Linear),
        {CFrame = Pos}
    ):Play()
end

function StopTween(target)
    if not target then
        _G.StopTween = true
        wait()
        topos(game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame)
        wait()
        if game:GetService("Players").LocalPlayer.Character.HumanoidRootPart:FindFirstChild("BodyClip") then
            game:GetService("Players").LocalPlayer.Character.HumanoidRootPart:FindFirstChild("BodyClip"):Destroy()
        end
        _G.StopTween = false
        _G.Clip = false
    end
end
task.spawn(function()
    while task.wait() do
        pcall(function()
            for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
                if not string.find(v.Name,"Boss") and (v.HumanoidRootPart.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= _G.BringMode then
                    if InMyNetWork(v.HumanoidRootPart) then
                        v.HumanoidRootPart.CFrame = PosMon
                        v.Humanoid.JumpPower = 0
                        v.Humanoid.WalkSpeed = 0
                        v.HumanoidRootPart.Size = Vector3.new(60,60,60)
                        v.HumanoidRootPart.Transparency = 1
                        v.HumanoidRootPart.CanCollide = false
                        v.Head.CanCollide = false
                        if v.Humanoid:FindFirstChild("Animator") then
                            v.Humanoid.Animator:Destroy()
                        end
                        v.Humanoid:ChangeState(11)
                        v.Humanoid:ChangeState(14)
                    end
                end
            end
        end)
    end
end)
spawn(function()
    while task.wait() do
        pcall(function()
            for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                if _G.Auto_Bone and StartMagnetBoneMon then
                    if (v.Name == "Reborn Skeleton" or v.Name == "Living Zombie" or v.Name == "Demonic Soul" or v.Name == "Posessed Mummy") and (v.HumanoidRootPart.Position - PosMonBone.Position).Magnitude <= _G.BringMode and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 then
                        v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                        v.Humanoid:ChangeState(14)
                        v.HumanoidRootPart.CanCollide = false
                        v.Head.CanCollide = false
                        v.HumanoidRootPart.CFrame = PosMonBone
                        if v.Humanoid:FindFirstChild("Animator") then
                            v.Humanoid.Animator:Destroy()
                        end
                        sethiddenproperty(game:GetService("Players").LocalPlayer, "SimulationRadius", math.huge)
                    end
                end
            end
        end)
    end
end)

spawn(function()
    while task.wait() do
        pcall(function()
            CheckQuest()
            for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                if _G.AutoFarm and StartMagnet and v.Name == Mon and (Mon == "Factory Staff" or Mon == "Monkey" or Mon == "Dragon Crew Warrior" or Mon == "Dragon Crew Archer") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and (v.HumanoidRootPart.Position - game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= 220 then
                    v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                    v.HumanoidRootPart.CFrame = PosMon
                    v.Humanoid:ChangeState(14)
                    v.HumanoidRootPart.CanCollide = false
                    v.Head.CanCollide = false
                    if v.Humanoid:FindFirstChild("Animator") then
                        v.Humanoid.Animator:Destroy()
                    end
                    sethiddenproperty(game:GetService("Players").LocalPlayer,"SimulationRadius",math.huge)
                elseif _G.AutoFarm and StartMagnet and v.Name == Mon and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and (v.HumanoidRootPart.Position - game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= _G.BringMode then
                    v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                    v.HumanoidRootPart.CFrame = PosMon
                    v.Humanoid:ChangeState(14)
                    v.HumanoidRootPart.CanCollide = false
                    v.Head.CanCollide = false
                    if v.Humanoid:FindFirstChild("Animator") then
                        v.Humanoid.Animator:Destroy()
                    end
                    sethiddenproperty(game:GetService("Players").LocalPlayer,"SimulationRadius",math.huge)
                end
            end
        end)
    end
end)

spawn(function()
    while wait() do
        if sethiddenproperty then
            sethiddenproperty(game.Players.LocalPlayer,"SimulationRadius",100)
        end
        if setscriptable then
            setscriptable(game.Players.LocalPlayer, "SimulationRadius", true)
            game.Players.LocalPlayer.SimulationRadius = math.huge * math.huge, math.huge * math.huge * 0 / 0 * 0 / 0 * 0 / 0 * 0 / 0 * 0 / 0
        end
    end
end)

function TP1(Pos)
    Distance = (Pos.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
    if Distance < 25 then
        Speed = 5000
    elseif Distance < 50 then
        Speed = 2000
    elseif Distance < 150 then
        Speed = 800
    elseif Distance < 250 then
        Speed = 600
    elseif Distance < 500 then
        Speed = 300
    elseif Distance < 750 then
        Speed = 250
    elseif Distance >= 1000 then
        Speed = 200
    end
    game:GetService("TweenService"):Create(
        game:GetService("Players").LocalPlayer.Character.HumanoidRootPart,
        TweenInfo.new(Distance/Speed, Enum.EasingStyle.Linear),
        {CFrame = Pos}
    ):Play()
end

function BTP(P)
	repeat wait(1)
		game.Players.LocalPlayer.Character.Humanoid:ChangeState(15)
		game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = P
		task.wait()
		game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = P
	until (P.Position-game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= 1500
end

spawn(function()
    while wait() do 
      local boneframe = CFrame.new(-9508.5673828125, 142.1398468017578, 5737.3603515625)
        if _G.Auto_Bone and World3 then
            pcall(function()
                  if BypassTP then
                  if (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - boneframe.Position).Magnitude > 2000 then
                  BTP(boneframe)
                  wait(.1)
                  game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-9515.3720703125, 164.00624084473, 5786.0610351562)
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("SetSpawnPoint")			
                  wait(.1)
                  game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-9515.3720703125, 164.00624084473, 5786.0610351562)
          game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("SetSpawnPoint")					
                  wait(.1)
                  game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-9515.3720703125, 164.00624084473, 5786.0610351562)
                  wait(.1)
                  game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-9515.3720703125, 164.00624084473, 5786.0610351562)
                  wait(3)
                  elseif (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - boneframe.Position).Magnitude < 2000 then
                  TP1(boneframe)
                  end
                  else
                    TP1(boneframe)
                  end
                if game:GetService("Workspace").Enemies:FindFirstChild("Reborn Skeleton") or game:GetService("Workspace").Enemies:FindFirstChild("Living Zombie") or game:GetService("Workspace").Enemies:FindFirstChild("Demonic Soul") or game:GetService("Workspace").Enemies:FindFirstChild("Posessed Mummy") then
                    for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Reborn Skeleton" or v.Name == "Living Zombie" or v.Name == "Demonic Soul" or v.Name == "Posessed Mummy" then
                           if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 then
                               repeat task.wait()
                                    AutoHaki()
                                    v.HumanoidRootPart.CanCollide = false
                                    v.Humanoid.WalkSpeed = 0
                                    v.Head.CanCollide = false 
                                    StartMagnetBoneMon = true
                                    PosMonBone = v.HumanoidRootPart.CFrame
                                    topos(v.HumanoidRootPart.CFrame * CFrame.new(PosX,PosY,PosZ))
                                    game:GetService("VirtualUser"):CaptureController()
                                    game:GetService("VirtualUser"):Button1Down(Vector2.new(1280,672))
                                until not _G.Auto_Bone or not v.Parent or v.Humanoid.Health <= 0
                            end
                        end
                    end
                else
                    StartMagnetBoneMon = false
                    topos(CFrame.new(-9506.234375, 172.130615234375, 6117.0771484375))
                    for i,v in pairs(game:GetService("ReplicatedStorage"):GetChildren()) do 
                        if v.Name == "Reborn Skeleton" then
                            topos(v.HumanoidRootPart.CFrame * CFrame.new(2,20,2))
                        elseif v.Name == "Living Zombie" then
                            topos(v.HumanoidRootPart.CFrame * CFrame.new(2,20,2))
                        elseif v.Name == "Demonic Soul" then
                            topos(v.HumanoidRootPart.CFrame * CFrame.new(2,20,2))
                        elseif v.Name == "Posessed Mummy" then
                            topos(v.HumanoidRootPart.CFrame * CFrame.new(2,20,2))
                        end
                    end
                end
            end)
        end
    end
end)

-- Criar abas
local Tabs = {
    principal = Window:AddTab({ Title = "Principal", Icon = "" }),
    Teleporte = Window:AddTab({ Title = "Teleporte", Icon = "" }),
    Outros = Window:AddTab({ Title = "Outros", Icon = "" })
}

local dados = {
    EquiparEspadasAtivo = equiparEspadasAtivo,
    AutoBuyLegendarySword = _G.AutoBuyLegendarySword,
    Auto_Bone = _G.Auto_Bone
}

Tabs.principal:AddToggle("Auto Bones", {
    Title = "Auto Bones",
    Description = "Ele farma ossos muahmmuah",
    Default = _G.Auto_Bone,
    Callback = function(Value)
        _G.Auto_Bone = Value
        _G.
        StopTween(_G.Auto_Bone)
        local dados = {
            EquiparEspadasAtivo = equiparEspadasAtivo,
            AutoBuyLegendarySword = _G.AutoBuyLegendarySword,
            Auto_Bone = _G.Auto_Bone
        }
        salvarDados(dados)
    end
})

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
        local dados = {
            EquiparEspadasAtivo = equiparEspadasAtivo,
            AutoBuyLegendarySword = _G.AutoBuyLegendarySword,
            Auto_Bone = _G.Auto_Bone
        }
        
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
