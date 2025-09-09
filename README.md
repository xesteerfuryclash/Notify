-- Xesteer Hub Notificador Instantâneo (0s)
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

local EMBED_COLOR = 5111808

-- Webhooks
local WEBHOOKS = {
    Fruits = "https://discord.com/api/webhooks/SEU_WEBHOOK_FRUTAS",
    Bosses = "https://discord.com/api/webhooks/SEU_WEBHOOK_BOSSES",
    LegendSwords = "https://discord.com/api/webhooks/SEU_WEBHOOK_SWORDS",
    LegendHaki = "https://discord.com/api/webhooks/SEU_WEBHOOK_HAKI",
    FullMoon = "https://discord.com/api/webhooks/SEU_WEBHOOK_FULLMOON",
    NearFullMoon = "https://discord.com/api/webhooks/SEU_WEBHOOK_NEARFULLMOON",
    Islands = "https://discord.com/api/webhooks/SEU_WEBHOOK_ISLANDS"
}

-- Place IDs por Sea
local SEAS = {
    Sea1 = 7449420000,
    Sea2 = 7449421000,
    Sea3 = 7449423635
}

-- Cache anti-spam
local sentCache = {}

-- Funções utilitárias
local function getPlayersCount() return tostring(#Players:GetPlayers()) end
local function getJobId() return tostring(game.JobId) end
local function makeJoinScript(jobId)
    return string.format('game:GetService("TeleportService"):TeleportToPlaceInstance(%d,"%s")', game.PlaceId, jobId)
end

local function buildEmbed(eventType)
    local jobId = getJobId()
    local joinScript = makeJoinScript(jobId)
    local now = os.date("!%d/%m/%Y - %H:%M:%S")

    return {
        ["title"] = eventType.." - Xesteer Hub Notify",
        ["color"] = EMBED_COLOR,
        ["fields"] = {
            {["name"]="Type :",["value"]=string.format("```%s [Spawn]```",eventType),["inline"]=false},
            {["name"]="Players In Server :",["value"]=string.format("```%s```",getPlayersCount()),["inline"]=false},
            {["name"]="Job ID (Pc Copy):",["value"]=string.format("```%s```",jobId),["inline"]=false},
            {["name"]="Join Script (Pc Copy):",["value"]=string.format("```lua\n%s\n```",joinScript),["inline"]=false},
            {["name"]="Job ID (Mobile Copy):",["value"]=string.format("```%s```",jobId),["inline"]=false},
            {["name"]="Join Script (Mobile Copy):",["value"]=string.format("```lua\n%s\n```",joinScript),["inline"]=false}
        },
        ["footer"] = {["text"]="Made by vitorzz07 • Time : "..now}
    }
end

local function sendEmbed(eventType, webhookKey)
    local webhookUrl = WEBHOOKS[webhookKey]
    if not webhookUrl or webhookUrl == "" then return end

    local payload = HttpService:JSONEncode({embeds={buildEmbed(eventType)}})
    local req = (syn and syn.request) or (http and http.request) or request
    if req then
        req({
            Url = webhookUrl,
            Method = "POST",
            Headers = {["Content-Type"] = "application/json"},
            Body = payload
        })
    end
end

local function sendOnce(tag,eventName,webhookKey)
    local key = tag.."::"..getJobId()
    if sentCache[key] then return end
    sentCache[key] = true
    sendEmbed(eventName,webhookKey)
end

local function notifyObject(obj, webhookKey)
    local name = obj.Name or "Unknown"
    sendOnce(name, name, webhookKey)
end

-- Scan de eventos já existentes
local function scanAllEvents()
    -- Fruits todos Seas
    for _, fruit in pairs(Workspace:FindFirstChild("Fruits") or {}) do
        notifyObject(fruit, "Fruits")
    end

    -- Rare Boss todos Seas
    for _, boss in pairs(Workspace:FindFirstChild("Bosses") or {}) do
        notifyObject(boss, "Bosses")
    end

    -- Legend Sword (Sea2)
    if game.PlaceId == SEAS.Sea2 then
        for _, sword in pairs(Workspace:FindFirstChild("LegendSwords") or {}) do
            notifyObject(sword, "LegendSwords")
        end
    end

    -- Legend Haki (Sea2 e Sea3)
    if game.PlaceId == SEAS.Sea2 or game.PlaceId == SEAS.Sea3 then
        for _, haki in pairs(Workspace:FindFirstChild("LegendHaki") or {}) do
            notifyObject(haki, "LegendHaki")
        end
    end

    -- Miragem / Kitsune / Prehistoric / Islands (Sea3)
    if game.PlaceId == SEAS.Sea3 then
        for _, island in pairs(Workspace:FindFirstChild("Islands") or {}) do
            notifyObject(island, "Islands")
        end
        for _, boss in pairs(Workspace:FindFirstChild("Bosses") or {}) do
            local n = boss.Name:lower()
            if n:find("kitsune") or n:find("prehistoric") or n:find("miragem") then
                notifyObject(boss,"Bosses")
            end
        end
    end

    -- Full Moon / Near Full Moon (Sea3)
    if game.PlaceId == SEAS.Sea3 then
        if Lighting:FindFirstChild("MoonPhase") then
            local v = tostring(Lighting.MoonPhase.Value or ""):lower()
            if v:find("full") then
                sendOnce("Moon_Full","Full Moon","FullMoon")
            elseif v:find("waxing") or v:find("waning") or v:find("gibbous") then
                sendOnce("Moon_NearFull","Near Full Moon","NearFullMoon")
            end
        end
    end
end

-- Listener instantâneo para spawn futuro
local function listenDescendants()
    Workspace.DescendantAdded:Connect(function(obj)
        local n = obj.Name:lower()
        -- Fruits
        if obj.Parent == Workspace:FindFirstChild("Fruits") then
            notifyObject(obj, "Fruits")
        end
        -- Bosses
        if obj.Parent == Workspace:FindFirstChild("Bosses") then
            if n:find("kitsune") or n:find("prehistoric") or n:find("miragem") then
                notifyObject(obj,"Bosses")
            else
                notifyObject(obj,"Bosses")
            end
        end
        -- Legend Sword
        if obj.Parent == Workspace:FindFirstChild("LegendSwords") and game.PlaceId == SEAS.Sea2 then
            notifyObject(obj, "LegendSwords")
        end
        -- Legend Haki
        if obj.Parent == Workspace:FindFirstChild("LegendHaki") and (game.PlaceId == SEAS.Sea2 or game.PlaceId == SEAS.Sea3) then
            notifyObject(obj, "LegendHaki")
        end
        -- Islands
        if obj.Parent == Workspace:FindFirstChild("Islands") and game.PlaceId == SEAS.Sea3 then
            notifyObject(obj,"Islands")
        end
    end)
end

-- Rodar scan inicial e listener instantâneo
scanAllEvents()
listenDescendants()

print("Xesteer Hub Notify Instantâneo iniciado com sucesso! 🚀")
