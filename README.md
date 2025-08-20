--[[ 
  BLOX FRUITS SERVER NOTIFIER — Xesteer Community - Crypton
  ✅ Compat executores + anti-spam + detecção prática por eventos/nomes
  ⚠️ Rode DENTRO do jogo (executor). Não funciona em Studio.
]]--

-------------------- CONFIG --------------------
-------------------- CONFIG --------------------
local Webhooks = {
    FullMoon = "https://discord.com/api/webhooks/1407700239045099540/8gEglTjK5b2KePmrtCTiapNl8r19pekDFDrnuplAw1GrE9zS_pRVKFYRZV-QykX3pVbZ",
    NearFullMoon = "https://discord.com/api/webhooks/1407700622316408872/YKDTHXD_Hx10ABACHWZrI7vQwaPVHzfl6zzdNkYJEHFzr6cNoYhTbI2qxU-tuk943eQ8",
    FruitSpawned = "https://discord.com/api/webhooks/1407700809084305408/w35VXujuPp80AIhAd4lYmAt9jhRMIizmvgcGDYS0O7AyT6NmhTY9jwcYay0pWZ4tm7aT",
    LegendHaki = "https://discord.com/api/webhooks/1407700947945259078/cJqI0tG82tXiYUn2jowpH5MhbwmbPnr0PocoYYV6Z3N8gr1CDk6cY4sRXYSzSq_Fjik9",
    LegendSword = "https://discord.com/api/webhooks/1407701082930675812/tglxqyGteLn18BBqMrFkLH5jfxU2GGsitQfWWzvncLgNf0Vnpa5_7AFaV-sBWTDOEZhD",
    RareBoss = "https://discord.com/api/webhooks/1407701191483195416/KdqDN0ZytLCIGm2HXqgZygm9tc3gfTJSXahoGdCJYEL7FoBD5Npv2EOLoLVOn-jODQKm",
    KitsuneIsland = "https://discord.com/api/webhooks/1407701317652185110/sEBbyWCmxqlpTHrKSszL_5oI2SwTY3Clra2gBbK3OlgNyuw3wbKZ6ISrV7m_GxWU11xL",
    PrehistoricIsland = "https://discord.com/api/webhooks/1407701730556248116/IgZR5hYZwLtOBCQdIWkGSl4U_r1sycp7CfdJWOY3PaDCRypxqMkS7WBFcAn7Be3rCb1r"
}

-- Quanto tempo (segundos) esperar antes de notificar o MESMO item novamente
local COOLDOWN = {
  Fruit = 60,           -- mesma fruta
  Boss  = 120,          -- mesmo boss
  Dealer = 300,         -- sword/haki dealer
  Island = 300,         -- kitsune/prehistoric
  Moon = 900            -- full/near full
}

-- Bosses que queremos notificar (edite à vontade)
local RARE_BOSSES = {
  ["Don Swan"] = true,
  ["Cursed Captain"] = true,
  ["Beautiful Pirate"] = true,
  ["rip_indra"] = true,
  ["Cake Queen"] = true,
  ["Soul Reaper"] = true
}

----------------- DEPENDÊNCIAS ----------------
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LP = Players.LocalPlayer

-- Compat: função HTTP do executor
local httpRequest = (syn and syn.request) or request or http_request or (http and http.request)
assert(httpRequest, "Seu executor não fornece request/http_request. Use um com suporte HTTP.")

----------------- FUNÇÕES BASE ----------------
local function serverInfo()
  local count = #Players:GetPlayers()
  local link = ("https://www.roblox.com/games/2753915549/Blox-Fruits?server=%s"):format(game.JobId)
  return count, link
end

local function sendEmbed(webhook, notifyName, fields)
  if not webhook or webhook == "" then return end
  local data = {
    embeds = {{
      title = notifyName .. " Notify",
      fields = fields,
      footer = { text = "Xesteer Community - Crypton" }
    }}
  }
  local ok, body = pcall(HttpService.JSONEncode, HttpService, data)
  if not ok then return end

  httpRequest({
    Url = webhook,
    Method = "POST",
    Headers = { ["Content-Type"] = "application/json" },
    Body = body
  })
end

-- Anti-spam cache
local sentCache = {} -- key -> lastTime
local function canSend(key, ttl)
  local now = os.time()
  local last = sentCache[key]
  if not last or (now - last) >= ttl then
    sentCache[key] = now
    return true
  end
  return false
end

-- Facilidade pra montar fields padrão
local function baseFields(extraName, extraValue)
  local players, link = serverInfo()
  local fields = {}
  if extraName and extraValue then
    table.insert(fields, { name = extraName, value = extraValue, inline = true })
  end
  table.insert(fields, { name = "Players Count", value = ("%d/12"):format(players), inline = true })
  table.insert(fields, { name = "Link server", value = link })
  return fields
end

------------------ AUTOTESTE -------------------
-- Envia um embed de teste (ajuda a validar os webhooks)
task.delay(2, function()
  local fields = baseFields("Self-Test", "```Notificador iniciado com sucesso```")
  for name, hook in pairs(Webhooks) do
    sendEmbed(hook, "Boot (" .. name .. ")", fields)
    task.wait(0.3)
  end
end)

------------------ DETECTORES ------------------

-- 1) FRUITS NO CHÃO (Tools com "Fruit" no nome)
local function isFruitTool(inst)
  if not inst then return false end
  if not inst:IsA("Tool") then return false end
  local n = tostring(inst.Name):lower()
  return n:find("fruit") ~= nil
end

local function notifyFruit(tool)
  local key = "fruit:" .. tool:GetDebugId(0)
  if not canSend(key, COOLDOWN.Fruit) then return end
  local fname = "```" .. tool.Name .. "```"
  local fields = baseFields("Fruit Spawned", fname)
  sendEmbed(Webhooks.FruitSpawned, "Fruit Spawned", fields)
end

-- Escaneia existentes e conecta ChildAdded
task.spawn(function()
  -- varredura inicial
  for _, inst in ipairs(Workspace:GetDescendants()) do
    if isFruitTool(inst) then notifyFruit(inst) end
  end
  -- notifica quando surgirem novas
  Workspace.DescendantAdded:Connect(function(inst)
    if isFruitTool(inst) then
      -- aguarda nome carregar por segurança
      task.delay(0.2, function()
        if inst and inst.Parent then notifyFruit(inst) end
      end)
    end
  end)
end)

-- 2) RARE BOSSES (Workspace.Enemies)
local Enemies
task.spawn(function()
  Enemies = Workspace:FindFirstChild("Enemies")
  if not Enemies then
    -- alguns servidores criam depois
    Workspace.ChildAdded:Connect(function(c)
      if c.Name == "Enemies" then Enemies = c end
    end)
  end
  -- Varredura inicial + listeners
  local function checkBoss(model)
    if model:IsA("Model") and model:FindFirstChildOfClass("Humanoid") then
      if RARE_BOSSES[model.Name] then
        local key = "boss:" .. model.Name
        if canSend(key, COOLDOWN.Boss) then
          local fields = baseFields("Rare Boss", "```" .. model.Name .. "```")
          sendEmbed(Webhooks.RareBoss, "Rare Boss", fields)
        end
      end
    end
  end

  -- inicial (se já existir Enemies)
  task.delay(1, function()
    if Enemies then
      for _, m in ipairs(Enemies:GetChildren()) do
        checkBoss(m)
      end
      Enemies.ChildAdded:Connect(checkBoss)
    end
  end)

  -- fallback: se Enemies não existir, monitora Workspace inteiro (mais pesado)
  Workspace.ChildAdded:Connect(function(c)
    if c:IsA("Model") and c:FindFirstChildOfClass("Humanoid") then
      if RARE_BOSSES[c.Name] then
        local key = "boss:" .. c.Name
        if canSend(key, COOLDOWN.Boss) then
          local fields = baseFields("Rare Boss", "```" .. c.Name .. "```")
          sendEmbed(Webhooks.RareBoss, "Rare Boss", fields)
        end
      end
    end
  end)
end)

-- 3) LEGENDARY SWORD DEALER (padrões por nome)
local function isLegendSwordInstance(inst)
  local n = tostring(inst.Name):lower()
  return n:find("legend") and n:find("sword") and n:find("dealer")  -- "Legendary Sword Dealer"
end

local function watchLegendSword(inst)
  if isLegendSwordInstance(inst) or (inst:IsA("Model") and isLegendSwordInstance(inst)) then
    local key = "dealer:sword"
    if canSend(key, COOLDOWN.Dealer) then
      local fields = baseFields("Legendary Sword Dealer", "⚔️ Dealer is Available!")
      sendEmbed(Webhooks.LegendSword, "Legend Sword", fields)
    end
  end
end

-- 4) LEGENDARY HAKI (Master of Auras / Haki keywords)
local function isLegendHakiInstance(inst)
  local n = tostring(inst.Name):lower()
  return n:find("master of auras") or (n:find("legend") and n:find("haki")) or n:find("aura")
end

local function watchLegendHaki(inst)
  if isLegendHakiInstance(inst) then
    local key = "dealer:haki"
    if canSend(key, COOLDOWN.Dealer) then
      local fields = baseFields("Legendary Haki Dealer", "💥 Dealer is Available!")
      sendEmbed(Webhooks.LegendHaki, "Legend Haki", fields)
    end
  end
end

-- Varre Workspace e ReplicatedStorage por segurança + listeners
task.spawn(function()
  local function sweep()
    for _, d in ipairs(Workspace:GetDescendants()) do
      watchLegendSword(d)
      watchLegendHaki(d)
    end
    for _, d in ipairs(ReplicatedStorage:GetDescendants()) do
      watchLegendSword(d)
      watchLegendHaki(d)
    end
  end
  sweep()
  Workspace.DescendantAdded:Connect(function(i)
    watchLegendSword(i)
    watchLegendHaki(i)
  end)
  ReplicatedStorage.DescendantAdded:Connect(function(i)
    watchLegendSword(i)
    watchLegendHaki(i)
  end)
end)

-- 5) ILHAS ESPECIAIS (Kitsune / Prehistoric) — por nome
local function checkIslandByName(namePart, hook, title)
  local nameLower = namePart:lower()
  local function sweep()
    for _, c in ipairs(Workspace:GetChildren()) do
      local n = c.Name:lower()
      if n:find(nameLower) then
        local key = "island:" .. nameLower
        if canSend(key, COOLDOWN.Island) then
          local fields = baseFields(title, "```" .. c.Name .. "```")
          sendEmbed(hook, title, fields)
        end
      end
    end
  end
  -- inicial + listeners
  sweep()
  Workspace.ChildAdded:Connect(function(c)
    if tostring(c.Name):lower():find(nameLower) then
      local key = "island:" .. nameLower
      if canSend(key, COOLDOWN.Island) then
        local fields = baseFields(title, "```" .. c.Name .. "```")
        sendEmbed(hook, title, fields)
      end
    end
  end)
  -- periódica (garante caso crie com delay profundo)
  task.spawn(function()
    while task.wait(60) do
      sweep()
    end
  end)
end

checkIslandByName("Kitsune", Webhooks.KitsuneIsland, "Kitsune Island")
checkIslandByName("Prehistoric", Webhooks.PrehistoricIsland, "Prehistoric Island")

-- 6) FULL MOON / NEAR FULL MOON
-- Muitos servidores mostram textos/labels na tela quando a lua está cheia ou quase cheia.
-- Vamos vigiar o PlayerGui e procurar labels com esse texto.
local function watchMoonInGui(pg)
  local function scan(obj)
    if obj:IsA("TextLabel") or obj:IsA("TextButton") then
      local t = (obj.Text or ""):lower()
      if t:find("full moon") then
        local key = "moon:full"
        if canSend(key, COOLDOWN.Moon) then
          local fields = baseFields("Status", "🌕 **Full Moon Active!**")
          sendEmbed(Webhooks.FullMoon, "Full Moon", fields)
        end
      elseif t:find("near full moon") or t:find("almost full moon") or t:find("peaks through the clouds") then
        local key = "moon:near"
        if canSend(key, COOLDOWN.Moon) then
          local fields = baseFields("Status", "🌖 **Near Full Moon!**")
          sendEmbed(Webhooks.NearFullMoon, "Near Full Moon", fields)
        end
      end
    end
  end

  -- escaneia tudo que já existe
  for _, d in ipairs(pg:GetDescendants()) do
    scan(d)
  end
  -- escuta novas labels
  pg.DescendantAdded:Connect(scan)
end

task.spawn(function()
  local pg = LP:FindFirstChildOfClass("PlayerGui") or LP:WaitForChild("PlayerGui")
  watchMoonInGui(pg)
end)

-- FIM
