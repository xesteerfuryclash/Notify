local HttpService = game:GetService("HttpService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")

local Webhooks = {
    FullMoon = "https://discord.com/api/webhooks/SEU_WEBHOOK_FULLMOON",
    NearFullMoon = "https://discord.com/api/webhooks/SEU_WEBHOOK_NEARFULLMOON",
    Fruits = "https://discord.com/api/webhooks/SEU_WEBHOOK_FRUITS",
    Miragem = "https://discord.com/api/webhooks/SEU_WEBHOOK_MIRAGEM",
    Kitsune = "https://discord.com/api/webhooks/SEU_WEBHOOK_KITSUNE",
    Prehistoric = "https://discord.com/api/webhooks/SEU_WEBHOOK_PREHISTORIC",
    RareBoss = "https://discord.com/api/webhooks/SEU_WEBHOOK_RAREBOSS",
    LegendSword = "https://discord.com/api/webhooks/SEU_WEBHOOK_LEGENDSWORD",
    LegendHaki = "https://discord.com/api/webhooks/SEU_WEBHOOK_LEGENDHAKI"
}

local SEA_BY_PLACEID = {
    [2753915549] = "First Sea",
    [4442272183] = "Second Sea",
    [7449423635] = "Third Sea"
}

local currentSea = SEA_BY_PLACEID[game.PlaceId] or "Unknown Sea"
local sentCache = {}

local function sendWebhook(webhookUrl, title, description)
    if sentCache[title..description] then return end
    sentCache[title..description] = true
    local data = {
        ["content"] = "",
        ["embeds"] = {{
            ["title"] = title,
            ["description"] = description,
            ["color"] = 0x7000ff
        }}
    }
    local success, err = pcall(function()
        HttpService:PostAsync(webhookUrl, HttpService:JSONEncode(data), Enum.HttpContentType.ApplicationJson)
    end)
    if not success then
        warn("Falha ao enviar webhook: "..err)
    end
end

local function checkEvents()
    local moonPhase = Lighting:FindFirstChild("MoonPhase")
    if moonPhase then
        local phase = moonPhase.Value:lower()
        if phase == "full moon" and currentSea == "Third Sea" then
            sendWebhook(Webhooks.FullMoon, "Full Moon", "Full Moon ativo no Third Sea!")
        elseif phase == "near full moon" and currentSea == "Third Sea" then
            sendWebhook(Webhooks.NearFullMoon, "Near Full Moon", "Near Full Moon ativo no Third Sea!")
        end
    end

    for _, item in ipairs(Workspace:GetDescendants()) do
        if item:IsA("Tool") and item.Name:lower():find("fruit") then
            sendWebhook(Webhooks.Fruits, "Fruta Spawnou", "Fruta encontrada: "..item.Name.." ("..currentSea..")")
        end
        if item:IsA("Tool") and item.Name:lower():find("legend sword") and currentSea == "Second Sea" then
            sendWebhook(Webhooks.LegendSword, "Legend Sword", item.Name)
        end
        if item:IsA("Tool") and item.Name:lower():find("legend haki") and (currentSea == "Second Sea" or currentSea == "Third Sea") then
            sendWebhook(Webhooks.LegendHaki, "Legend Haki", item.Name)
        end
        if item:IsA("Model") then
            local nameLower = item.Name:lower()
            if nameLower:find("miragem") and currentSea == "Third Sea" then
                sendWebhook(Webhooks.Miragem, "Miragem Spawnou", item.Name)
            elseif nameLower:find("kitsune") and currentSea == "Third Sea" then
                sendWebhook(Webhooks.Kitsune, "Kitsune Spawnou", item.Name)
            elseif nameLower:find("prehistoric") and currentSea == "Third Sea" then
                sendWebhook(Webhooks.Prehistoric, "Prehistoric Spawnou", item.Name)
            elseif nameLower:find("rare") then
                sendWebhook(Webhooks.RareBoss, "Rare Boss Spawnou", item.Name.." ("..currentSea..")")
            end
        end
    end
end

Workspace.DescendantAdded:Connect(function(inst)
    if inst:IsA("Tool") or inst:IsA("Model") then
        checkEvents()
    end
end)

checkEvents()
