--[[
  BLOX FRUITS NOTIFIER (VERSÃO COM WEBHOOKS INDIVIDUAIS)
  - Cada evento possui seu próprio webhook para configuração máxima.
  - Detecções: Mirage, Kitsune, Prehistoric, Full/Near Moon, Rare Boss, Dealers, Fruits.
  - Cor do Embed: #4e0000 (fixa)
  - Script real e funcional para executores.
]]--

-- ============================ CONFIGURAÇÃO ESSENCIAL ============================
-- É A ÚNICA PARTE QUE VOCÊ PRECISA EDITAR.
local CONFIG = {
  -- Cole aqui os seus webhooks do Discord PARA CADA EVENTO.
  Webhooks = {
    -- Ilhas
    KitsuneIsland     = "https://discord.com/api/webhooks/1407701317652185110/sEBbyWCmxqlpTHrKSszL_5oI2SwTY3Clra2gBbK3OlgNyuw3wbKZ6ISrV7m_GxWU11xL",
    MirageIsland      = "COLOQUE_SEU_WEBHOOK_DA_MIRAGE_ISLAND_AQUI",
    PrehistoricIsland = "https://discord.com/api/webhooks/1407701730556248116/IgZR5hYZwLtOBCQdIWkGSl4U_r1sycp7CfdJWOY3PaDCRypxqMkS7WBFcAn7Be3rCb1r",

    -- Fases da Lua
    FullMoon          = "https://discord.com/api/webhooks/1407700239045099540/8gEglTjK5b2KePmrtCTiapNl8r19pekDFDrnuplAw1GrE9zS_pRVKFYRZV-QykX3pVbZ",
    NearFullMoon      = "https://discord.com/api/webhooks/1407700622316408872/YKDTHXD_Hx10ABACHWZrI7vQwaPVHzfl6zzdNkYJEHFzr6cNoYhTbI2qxU-tuk943eQ8",

    -- Bosses
    RareBoss          = "https://discord.com/api/webhooks/1407701191483195416/KdqDN0ZytLCIGm2HXqgZygm9tc3gfTJSXahoGdCJYEL7FoBD5Npv2EOLoLVOn-jODQKm",

    -- Vendedores Lendários
    LegendSword       = "https://discord.com/api/webhooks/1407701082930675812/tglxqyGteLn18BBqMrFkLH5jfxU2GGsitQfWWzvncLgNf0Vnpa5_7AFaV-sBWTDOEZhD",
    LegendHaki        = "https://discord.com/api/webhooks/1407700947945259078/cJqI0tG82tXiYUn2jowpH5MhbwmbPnr0PocoYYV6Z3N8gr1CDk6cY4sRXYSzSq_Fjik9",
    
    -- Frutas
    FruitSpawned      = "https://discord.com/api/webhooks/1407700809084305408/w35VXujuPp80AIhAd4lYmAt9jhRMIizmvgcGDYS0O7AyT6NmhTY9jwcYay0pWZ4tm7aT"
  },

  -- Ative ou desative os notificadores que desejar (opcional).
  Enabled = {
    MirageIsland      = true,
    KitsuneIsland     = true,
    PrehistoricIsland = true,
    FullMoon          = true,
    NearFullMoon      = true,
    RareBoss          = true,
    LegendHaki        = true,
    LegendSword       = true,
    Fruit             = true
  },

  -- Cooldowns (em segundos) para não spammar o mesmo evento.
  Cooldowns = {
    Island = 300, -- 5 minutos
    Moon   = 900, -- 15 minutos
    Boss   = 120, -- 2 minutos
    Dealer = 300, -- 5 minutos
    Fruit  = 60   -- 1 minuto
  },

  -- Lista de bosses raros que serão monitorados.
  RareBosses = {
    ["Don Swan"] = true, ["Cursed Captain"] = true, ["Beautiful Pirate"] = true,
    ["rip_indra"] = true, ["Cake Queen"] = true, ["Soul Reaper"] = true
  }
}
-- ========================== FIM DA CONFIGURAÇÃO ===========================


-- ========================== NÚCLEO DO SCRIPT (NÃO EDITAR) ==========================
local Notifier = { _connections = {}, _sentCache = {} }
local Players, Workspace, ReplicatedStorage, HttpService, Lighting =
  game:GetService("Players"), game:GetService("Workspace"), game:GetService("ReplicatedStorage"),
  game:GetService("HttpService"), game:GetService("Lighting")

local LP = Players.LocalPlayer
local httpRequest = (syn and syn.request) or request or http_request or (http and http.request)
assert(httpRequest, "[Notifier] Erro fatal: Seu executor não possui uma função de request HTTP. O script não pode funcionar.")

function Notifier:CanSend(key, ttl)
  local now = os.time()
  if not self._sentCache[key] or (now - self._sentCache[key]) >= ttl then
    self._sentCache[key] = now
    return true
  end
  return false
end

function Notifier:GetServerInfo()
  local playerCount = #Players:GetPlayers()
  local serverLink = ("roblox://games/launch?placeId=2753915549&gameId=%s"):format(game.JobId)
  return ("%d/12"):format(playerCount), serverLink
end

function Notifier:SendEmbed(webhookKey, title, fields)
  local webhookUrl = CONFIG.Webhooks[webhookKey]
  if not webhookUrl or webhookUrl == "" or webhookUrl:find("COLOQUE_SEU_WEBHOOK") then return end

  local playerCount, serverLink = self:GetServerInfo()
  
  local finalFields = {}
  for _, field in ipairs(fields) do table.insert(finalFields, field) end
  table.insert(finalFields, { name = "Players no Servidor", value = "```" .. playerCount .. "```", inline = true })
  table.insert(finalFields, { name = "🔗 Link do Servidor", value = "[Clique aqui para entrar](" .. serverLink .. ")" })

  local payload = {
    embeds = {{
      title = title,
      fields = finalFields,
      color = 5111808, -- Cor fixa #4e0000
      timestamp = os.date("!%Y-%m-%dT%H:%M:%S.000Z"),
      footer = { text = "Blox Fruits Notifier" }
    }}
  }
  
  pcall(function()
    local body = HttpService:JSONEncode(payload)
    httpRequest({ Url = webhookUrl, Method = "POST", Headers = { ["Content-Type"] = "application/json" }, Body = body })
  end)
end

function Notifier:InitIslandDetector()
  if not (CONFIG.Enabled.MirageIsland or CONFIG.Enabled.KitsuneIsland or CONFIG.Enabled.PrehistoricIsland) then return end

  local function checkIsland(instance)
    if not (instance and instance:IsA("Model")) then return end
    local name = instance.Name:lower()
    
    if CONFIG.Enabled.KitsuneIsland and name:find("kitsune") and self:CanSend("island:kitsune", CONFIG.Cooldowns.Island) then
      print("[Notifier] Ilha Detectada: Kitsune Island")
      self:SendEmbed("KitsuneIsland", "🏝️ Kitsune Island Spawned!", {{ name = "Status", value = "A ilha apareceu no servidor!", inline = true }})
    elseif CONFIG.Enabled.MirageIsland and name:find("mirage") and self:CanSend("island:mirage", CONFIG.Cooldowns.Island) then
      print("[Notifier] Ilha Detectada: Mirage Island")
      self:SendEmbed("MirageIsland", "🏝️ Mirage Island Spawned!", {{ name = "Status", value = "A ilha apareceu no servidor!", inline = true }})
    elseif CONFIG.Enabled.PrehistoricIsland and name:find("prehistoric") and self:CanSend("island:prehistoric", CONFIG.Cooldowns.Island) then
      print("[Notifier] Ilha Detectada: Prehistoric Island")
      self:SendEmbed("PrehistoricIsland", "🏝️ Prehistoric Island Spawned!", {{ name = "Status", value = "A ilha (Mar 6) apareceu no servidor!", inline = true }})
    end
  end

  for _, child in ipairs(Workspace:GetChildren()) do checkIsland(child) end
  table.insert(self._connections, Workspace.ChildAdded:Connect(checkIsland))
end

function Notifier:InitMoonDetector()
  if not (CONFIG.Enabled.FullMoon or CONFIG.Enabled.NearFullMoon) then return end

  local dayCycle = Lighting:WaitForChild("DayCycle")

  local function checkMoonPhase()
    local moonPhase = dayCycle.MoonPhase.Value

    if CONFIG.Enabled.FullMoon and moonPhase == 7 and self:CanSend("moon:full", CONFIG.Cooldowns.Moon) then
      print("[Notifier] Full Moon Detectada!")
      self:SendEmbed("FullMoon", "🌕 Full Moon!", {{ name = "Status", value = "A Lua Cheia está ativa!", inline = true }})
    elseif CONFIG.Enabled.NearFullMoon and moonPhase == 6 and self:CanSend("moon:near", CONFIG.Cooldowns.Moon) then
      print("[Notifier] Near Full Moon Detectada!")
      self:SendEmbed("NearFullMoon", "🌖 Near Full Moon!", {{ name = "Status", value = "A noite está quase em Lua Cheia!", inline = true }})
    end
  end
  
  checkMoonPhase()
  table.insert(self._connections, dayCycle.MoonPhase.Changed:Connect(checkMoonPhase))
end

function Notifier:InitBossDetector()
  if not CONFIG.Enabled.RareBoss then return end

  local function checkBoss(model)
    if not (model and model:IsA("Model") and model:FindFirstChildOfClass("Humanoid")) then return end
    if CONFIG.RareBosses[model.Name] and self:CanSend("boss:" .. model.Name, CONFIG.Cooldowns.Boss) then
      print("[Notifier] Boss Raro Detectado:", model.Name)
      self:SendEmbed("RareBoss", "🔥 Rare Boss Spawned!", {{ name = "Boss", value = "```" .. model.Name .. "```", inline = true }})
    end
  end
  
  for _, child in ipairs(Workspace.Enemies:GetChildren()) do checkBoss(child) end
  table.insert(self._connections, Workspace.Enemies.ChildAdded:Connect(checkBoss))
end

function Notifier:InitLegendaryDealerDetector()
  if not (CONFIG.Enabled.LegendHaki or CONFIG.Enabled.LegendSword) then return end

  local function checkDealer(instance)
    if not (instance and instance.Parent) then return end
    local name = instance.Name:lower()
    
    if CONFIG.Enabled.LegendSword and name:find("legend") and name:find("sword") and name:find("dealer") and self:CanSend("dealer:sword", CONFIG.Cooldowns.Dealer) then
      print("[Notifier] Legendary Sword Dealer Detectado!")
      self:SendEmbed("LegendSword", "⚔️ Legendary Sword Dealer", {{ name = "Status", value = "O vendedor apareceu! (Second Sea)", inline = true }})
    elseif CONFIG.Enabled.LegendHaki and (name:find("master of auras") or (name:find("legend") and name:find("haki"))) and self:CanSend("dealer:haki", CONFIG.Cooldowns.Dealer) then
      print("[Notifier] Legendary Haki Dealer Detectado!")
      self:SendEmbed("LegendHaki", "💥 Legendary Haki Dealer", {{ name = "Status", value = "O vendedor apareceu! (Second/Third Sea)", inline = true }})
    end
  end

  for _, item in ipairs(Workspace:GetDescendants()) do checkDealer(item) end
  table.insert(self._connections, Workspace.DescendantAdded:Connect(checkDealer))
end

function Notifier:InitFruitDetector()
  if not CONFIG.Enabled.Fruit then return end
  
  local function checkFruit(tool)
    if not (tool and tool:IsA("Tool") and tool.Name:lower():find("fruit")) then return end
    if not tool.Parent or not tool.Parent:IsA("Workspace") then return end
    
    if self:CanSend("fruit:" .. tool.Name, CONFIG.Cooldowns.Fruit) then
      print("[Notifier] Fruta Detectada:", tool.Name)
      self:SendEmbed("FruitSpawned", "🍉 Fruit Spawned!", {{ name = "Fruta", value = "```" .. tool.Name .. "```", inline = true }})
    end
  end

  for _, item in ipairs(Workspace:GetDescendants()) do checkFruit(item) end
  table.insert(self._connections, Workspace.DescendantAdded:Connect(checkFruit))
end

function Notifier:Start()
  print("[Notifier] Iniciando script com webhooks individuais...")
  self:InitIslandDetector()
  self:InitMoonDetector()
  self:InitBossDetector()
  self:InitLegendaryDealerDetector()
  self:InitFruitDetector()
  print("[Notifier] Todos os módulos foram carregados. Monitorando o servidor.")
end

Notifier:Start()
