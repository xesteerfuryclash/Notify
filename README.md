-- Xesteer Hub Notify - Versão Instantânea e Confiável
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

-- Cor fixa
local EMBED_COLOR = 5111808

-- Webhooks
local WEBHOOKS = {
    FullMoon = "https://discord.com/api/webhooks/1255213642795810957/oycEkJx4F68sgQtSpt-xFPG0JmA8V_g_NueY9eQdQ6LQ3N2vhK00Nln7x4W9H1z2e-Dn",
    NearFullMoon = "https://discord.com/api/webhooks/1255213790905589820/v7tO6pUBuvROi6eM5lRnkV5p_Yl3nObHXms1NGKjx-xSssg60hhTPzXFlf9w9m3NGp3G",
    MirageIsland = "https://discord.com/api/webhooks/1255213851578224700/oayOWv7guA6pQ8j8do0nqdmS5r9oIklANBlbkx94tiO8sGMi9F-6Z0ltdGVJCe1SL7Ko",
    KitsuneIsland = "https://discord.com/api/webhooks/1255213903376498728/I46b1JJKNPHNGUu2MY17o2yJwEo9A_lw7cGZAT5yqMmwbdkbZRHQNgpToKeYrDgqLm14",
    PrehistoricIsland = "https://discord.com/api/webhooks/1255213947688558652/dK6Cbo7onAuLEzM4VgNOoGjqk_KY52wQvOE6PPTay4U5P-KS7IZiFJziJw4AlCPhSEwf",
    Fruit = "https://discord.com/api/webhooks/1255213991898105918/gCkbCnyVx_ZtpoIWtGkEmoYxvtR6xHiPZr1MR3JSfRexqq2OQ1bRQ0Kv11uuxpfRYc4X",
    LegendHaki = "https://discord.com/api/webhooks/1255214041033936956/sHd1x-Rj22TbkMhtW4x3qqmb7bcQheZcY08m3aVY0o1N6RFLpNsZ_JN6FKnIkLgQ8ejQ",
    LegendSword = "https://discord.com/api/webhooks/1255214088047290449/8rhlRGWmylNE4TQ-P5t0YAk8ufc-dxssldVVBgM5lf9gDUpZTJepD7M1XY9JrvxE-hFl",
    RareBoss = "https://discord.com/api/webhooks/1255214138137593916/o9qfQzx5B_nyBv98ZTyB-Pze7KhA5oqZPHOj_L9jP3rvhTL0zHzkQTVsLs1VjFsX4o0m"
}

-- Listas
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

-- Funções utilitárias
local function getPlayersCount() return tostring(#Players:GetPlayers()) end
local function getJobId() return tostring(game.JobId) end
local function makeJoinScript(jobId)
    return string.format('game:GetService("TeleportService"):TeleportToPlaceInstance(%d,"%s")', game.PlaceId, jobId)
end

-- Monta embed
local function buildEmbed(eventType)
    local jobId = getJobId()
    local joinScript = makeJoinScript(jobId)
    local now = os.date("!%d/%m/%Y - %H:%M:%S")
    return {
        ["title"] = eventType .. " - Xesteer Hub Notify",
        ["color"] = EMBED_COLOR,
        ["fields"] = {
            {["name"]="Type :",["value"]=string.format("```%s [Spawn]```",eventType),["inline"]=false},
            {["name"]="Players In Server :",["value"]=string.format("```%s```",getPlayersCount()),["inline"]=false},
            {["name"]="Sea :",["value"]=string.format("```%s```",DEFAULT_SEA),["inline"]=false},
            {["name"]="Job ID (Pc Copy):",["value"]=string.format("```%s```",jobId),["inline"]=false},
            {["name"]="Join Script (Pc Copy):",["value"]=string.format("```lua\n%s\n```",joinScript),["inline"]=false},
            {["name"]="Job ID (Mobile Copy):",["value"]=string.format("```%s```",jobId),["inline"]=false},
            {["name"]="Join Script (Mobile Copy):",["value"]=string.format("```lua\n%s\n```",joinScript),["inline"]=false}
        },
        ["footer"] = {["text"]="Made by vitorzz07 • Time : "..now}
    }
end

-- Envia embed
local function sendEmbed(eventType, webhookKey)
    local webhookUrl = WEBHOOKS[webhookKey]
    if not webhookUrl or webhookUrl == "" then return end
    local payload = HttpService:JSONEncode({embeds={buildEmbed(eventType)}})
    local req = (syn and syn.request) or (http and http.request) or request
    if req then
        req({
            Url = webhookUrl,
            Method = "POST",
            Headers = {["Content-Type"]="application/json"},
            Body = payload
        })
    end
end

-- Anti-spam
local function sendOnce(tag, eventName, webhookKey)
    local key = tag.."::"..getJobId()
    if sentCache[key] then return end
    sentCache[key] = true
    sendEmbed(eventName, webhookKey)
end

-- Função para varrer e detectar eventos
local function scanWorkspace()
    for _, inst in ipairs(Workspace:GetDescendants()) do
        -- Fruits
        if inst:IsA("Tool") or inst:IsA("Model") or inst:IsA("Folder") then
            for _, fname in ipairs(FRUITS_NAMES) do
                if inst.Name:lower():find(fname:lower(),1,true) then
                    sendOnce("Fruit_"..fname,"Fruit: "..fname,"Fruit")
                end
            end
        end

        -- Ilhas
        if inst.Name:find("Mirage") then
            sendOnce("Mirage","Mirage Island","MirageIsland")
        elseif inst.Name:find("Kitsune") then
            sendOnce("Kitsune","Kitsune Island","KitsuneIsland")
        elseif inst.Name:find("Prehistoric") then
            sendOnce("Prehistoric","Prehistoric Island","PrehistoricIsland")
        end

        -- Rare Bosses
        for _, boss in ipairs(RARE_BOSSES) do
            if inst.Name == boss then
                sendOnce("Boss_"..boss,"Boss: "..boss,"RareBoss")
            end
        end

        -- Legend Swords
        for _, sword in ipairs(LEGEND_SWORDS) do
            if inst.Name == sword then
                sendOnce("Sword_"..sword,"Sword: "..sword,"LegendSword")
            end
        end
    end

    -- Legend Haki
    for _, inst in ipairs(ReplicatedStorage:GetDescendants()) do
        if inst:IsA("StringValue") then
            if inst.Name:lower():find("haki") and inst.Value ~= "" then
                sendOnce("Haki_"..inst.Value,"Haki: "..inst.Value,"LegendHaki")
            end
        end
    end
end

-- Lua Cheia / Quase Cheia
local function checkMoon()
    if Lighting:FindFirstChild("MoonPhase") then
        local v = tostring(Lighting.MoonPhase.Value or ""):lower()
        if v:find("full") then
            sendOnce("Moon_Full","Full Moon","FullMoon")
        elseif v:find("waxing") or v:find("waning") or v:find("gibbous") then
            sendOnce("Moon_NearFull","Near Full Moon","NearFullMoon")
        end
    end
end
Lighting:GetPropertyChangedSignal("MoonPhase"):Connect(checkMoon)
checkMoon()

-- Loop contínuo para scan instantâneo
RunService.Heartbeat:Connect(scanWorkspace)

print("✅ Xesteer Hub Notify (Instant) iniciado com sucesso!")
