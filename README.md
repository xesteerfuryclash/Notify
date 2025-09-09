-- Notify Script Update 27 - Instantâneo
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- ---------- CONFIGURAÇÕES ----------
local SERVER_ICON = "https://cdn.discordapp.com/attachments/1407790618125668559/1414907782422855691/PNG_03.png"

local WEBHOOKS = {
    FullMoon = "https://discord.com/api/webhooks/1407700239045099540/8gEglTjK5b2KePmrtCTiapNl8r19pekDFDrnuplAw1GrE9zS_pRVKFYRZV-QykX3pVbZ",
    NearFullMoon = "https://discord.com/api/webhooks/1407700622316408872/YKDTHXD_Hx10ABACHWZrI7vQwaPVHzfl6zzdNkYJEHFzr6cNoYhTbI2qxU-tuk943eQ8",
    MirageIsland = "https://discord.com/api/webhooks/1413078808478613565/7Q2-Lo626ESJZyFukt75xLZJeVoZuGPUG4GIpatq62EqFpNQYARCUU0eo5QmI6Ott3v5",
    KitsuneIsland = "https://discord.com/api/webhooks/1407701317652185110/sEBbyWCmxqlpTHrKSszL_5oI2SwTY3Clra2gBbK3OlgNyuw3wbKZ6ISrV7m_GxWU11xL",
    PrehistoricIsland = "https://discord.com/api/webhooks/1407701730556248116/IgZR5hYZwLtOBCQdIWkGSl4U_r1sycp7CfdJWOY3PaDCRypxqMkS7WBFcAn7Be3rCb1r",
    Fruit = "https://discord.com/api/webhooks/1407700809084305408/w35VXujuPp80AIhAd4lYmAt9jhRMIizmvgcGDYS0O7AyT6NmhTY9jwcYay0pWZ4tm7aT",
    LegendHaki = "https://discord.com/api/webhooks/1407700947945259078/cJqI0tG82tXiYUn2jowpH5MhbwmbPnr0PocoYYV6Z3N8gr1CDk6cY4sRXYSzSq_Fjik9",
    LegendSword = "https://discord.com/api/webhooks/1407701082930675812/tglxqyGteLn18BBqMrFkLH5jfxU2GGsitQfWWzvncLgNf0Vnpa5_7AFaV-sBWTDOEZhD",
    RareBoss = "https://discord.com/api/webhooks/1407701191483195416/KdqDN0ZytLCIGm2HXqgZygm9tc3gfTJSXahoGdCJYEL7FoBD5Npv2EOLoLVOn-jODQKm"
}

local FRUITS_NAMES = {
    "Rocket","Spin","Chop","Spring","Bomb","Smoke","Spike","Flame","Eagle","Ice","Sand",
    "Dark","Diamond","Light","Rubber","Barrier","Magma","Quake","Buddha","Love","Spider",
    "Phoenix","Portal","Rumble","Pain","Blizzard","Gravity","Mammoth","Venom","Shadow",
    "Control","Spirit","Dough","T-Rex","Leopard","Sound","Kitsune","Dragon","Lightning"
}

local LEGEND_SWORDS = {"Oroshi","Saishi","Shizu"}
local RARE_BOSSES = {
    "Rip Indra","Dough King","Cake Prince","Tyrant of the Skies",
    "Darkbeard","Soul Reaper","Cursed Captain"
}

local SEA_BY_PLACEID = {
    [2753915549] = "First Sea",
    [4442272183] = "Second Sea",
    [7449423635] = "Third Sea"
}
local DEFAULT_SEA = SEA_BY_PLACEID[game.PlaceId] or "Unknown"

-- Cache anti-spam
local sentCache = {}

-- ---------- FUNÇÕES ----------
local function nowISO() return os.date("%d/%m/%Y - %H:%M:%S") end
local function playersCount() return #Players:GetPlayers() end
local function getJobId() return tostring(game.JobId or "unknown-job") end
local function makeJoinScript()
    return string.format("```lua\ngame:GetService('TeleportService'):TeleportToPlaceInstance(%d,'%s',game.Players.LocalPlayer)\n```", game.PlaceId, getJobId())
end

local function sendEmbed(title,key,webhookUrl)
    if not webhookUrl or webhookUrl=="" then return end
    local embed = {
        title = title,
        color = 0x5865F2,
        fields = {
            {name="Tipo :", value="`"..key.."`", inline=false},
            {name="Jogadores no Servidor :", value=tostring(playersCount()), inline=false},
            {name="Sea :", value=DEFAULT_SEA, inline=false},
            {name="Job ID (Copiar PC):", value="`"..getJobId().."`", inline=false},
            {name="Script de Entrada (Copiar PC):", value=makeJoinScript(), inline=false},
            {name="Job ID (Copiar Mobile):", value="`"..getJobId().."`", inline=false},
            {name="Script de Entrada (Copiar Mobile):", value=makeJoinScript(), inline=false}
        },
        footer={text="Made by vitorzz07 • Time : "..nowISO(),icon_url=SERVER_ICON}
    }
    local payload={username="Server Notify",avatar_url=SERVER_ICON,embeds={embed}}
    local body=HttpService:JSONEncode(payload)

    local ok,res
    if syn and syn.request then
        ok,res=pcall(function() return syn.request({Url=webhookUrl,Method="POST",Headers={["Content-Type"]="application/json"},Body=body}) end)
    elseif http and http.request then
        ok,res=pcall(function() return http.request({Url=webhookUrl,Method="POST",Headers={["Content-Type"]="application/json"},Body=body}) end)
    elseif request then
        ok,res=pcall(function() return request({Url=webhookUrl,Method="POST",Headers={["Content-Type"]="application/json"},Body=body}) end)
    else
        ok,res=pcall(function() return HttpService:RequestInternal({Url=webhookUrl,Method="POST",Body=body,Headers={["Content-Type"]="application/json"}}) end)
    end
    if not ok then warn("Falha ao enviar webhook:",res) end
end

local function sendOnce(key,title,webhookKey)
    local tag=key.."::"..getJobId()
    if sentCache[tag] then return end
    sentCache[tag]=true
    sendEmbed(title,key,WEBHOOKS[webhookKey])
end

-- ---------- EVENTOS INSTANTÂNEOS ----------

-- Frutas
Workspace.DescendantAdded:Connect(function(inst)
    if inst:IsA("Tool") or inst:IsA("Model") or inst:IsA("Folder") then
        local name=inst.Name:lower()
        for _,fname in ipairs(FRUITS_NAMES) do
            if name:find(fname:lower(),1,true) then
                sendOnce("Fruit_"..fname,"Fruta Detectada: "..fname,"Fruit")
            end
        end
    end
end)

-- Ilhas
local function checkIsland(inst)
    local islands={["MirageIsland"]="MirageIsland",["Mirage Island"]="MirageIsland",
                   ["KitsuneIsland"]="KitsuneIsland",["Kitsune Island"]="KitsuneIsland",
                   ["PrehistoricIsland"]="PrehistoricIsland",["Prehistoric Island"]="PrehistoricIsland"}
    for k,v in pairs(islands) do
        if inst.Name==k then sendOnce(v,"Ilha Detectada: "..k,v) end
    end
end
Workspace.DescendantAdded:Connect(checkIsland)

-- Rare Bosses
Workspace.DescendantAdded:Connect(function(inst)
    for _,boss in ipairs(RARE_BOSSES) do
        if inst.Name==boss then
            sendOnce("RareBoss_"..boss,"Boss Raro Detectado: "..boss,"RareBoss")
        end
    end
end)

-- Legend Swords
Workspace.DescendantAdded:Connect(function(inst)
    for _,sword in ipairs(LEGEND_SWORDS) do
        if inst.Name==sword then
            sendOnce("LegendSword_"..sword,"Legend Sword Disponível: "..sword,"LegendSword")
        end
    end
end)

-- Legend Haki
ReplicatedStorage.DescendantAdded:Connect(function(inst)
    local names={"LegendHaki","HakiColor","Haki","LegendaryHaki"}
    for _,n in ipairs(names) do
        if inst.Name==n then
            local val=tostring(inst.Value or ""):lower()
            if val~="" then sendOnce("LegendHaki_"..val,"Legend Haki Disponível: "..inst.Value,"LegendHaki") end
        end
    end
end)

-- Lua Cheia / Quase Cheia
local function checkMoon()
    if Lighting:FindFirstChild("MoonPhase") and Lighting.MoonPhase.Value then
        local v=tostring(Lighting.MoonPhase.Value):lower()
        if v:find("full") then sendOnce("FullMoon","Lua Cheia","FullMoon") end
        if v:find("waxing") or v:find("waning") or v:find("gibbous") then
            sendOnce("NearFullMoon","Lua Quase Cheia","NearFullMoon")
        end
    end
end
Lighting:GetPropertyChangedSignal("MoonPhase"):Connect(checkMoon)
checkMoon()

print("Notify Script Instantâneo iniciado! Eventos detectados em tempo real.")
