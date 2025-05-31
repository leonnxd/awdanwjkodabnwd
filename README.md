-- CONFIGURAÇÃO
local webhook = "https://discord.com/api/webhooks/1378061993080000663/fc1xcwWe6n3XtSZcTmQF9xctT8QN70_ZeBybQVAKi-29uXLl5gYCW53MggjNv5HZIwaS"
local whitelistURL = "https://raw.githubusercontent.com/leonnxd/whitellsllsks/refs/heads/main/README.md"

-- FUNÇÃO PARA ENVIAR WEBHOOK COM EMBED
local function sendWebhook(username, autorizado)
    local HttpService = game:GetService("HttpService")

    local data = {
        ["embeds"] = {{
            ["title"] = autorizado and "Player verificado na whitelist!" or "Tentativa de acesso negada!",
            ["description"] = autorizado and "Um jogador entrou no script!" or "Um jogador tentou usar o script sem estar na whitelist!",
            ["fields"] = {{
                ["name"] = "Player User",
                ["value"] = username,
                ["inline"] = true
            }},
            ["color"] = autorizado and 65280 or 16711680 -- Verde se autorizado, vermelho se negado
        }},
        ["username"] = "Whitelist Logger"
    }

    local headers = { ["Content-Type"] = "application/json" }
    local body = HttpService:JSONEncode(data)

    local request = (syn and syn.request) or (http and http.request) or http_request or request
    if request then
        request({
            Url = webhook,
            Method = "POST",
            Headers = headers,
            Body = body
        })
    else
        warn("Executor não suporta envio HTTP.")
    end
end

-- WHITELIST ONLINE
local function isPlayerWhitelisted(playerName)
    local HttpService = game:GetService("HttpService")
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet(whitelistURL))
    end)
    if success and typeof(result) == "table" then
        for _, name in ipairs(result) do
            if name == playerName then
                return true
            end
        end
    end
    return false
end

-- GUI E VERIFICAÇÃO
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui

local inputBox = Instance.new("TextBox")
inputBox.Parent = screenGui
inputBox.Size = UDim2.new(0, 400, 0, 50)
inputBox.Position = UDim2.new(0.5, -200, 0.5, -25)
inputBox.PlaceholderText = "Verificando sua whitelist..."
inputBox.Text = ""
inputBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
inputBox.TextSize = 20
inputBox.ClearTextOnFocus = false

-- Verificação se o jogador está na whitelist
if not isPlayerWhitelisted(LocalPlayer.Name) then
    inputBox.Text = "❌ Você não está na whitelist!"
    sendWebhook(LocalPlayer.Name, false) -- Envia webhook informando acesso negado
    task.wait(2)
    LocalPlayer:Kick("Você não está na whitelist!")
    return
end

-- Se o jogador estiver na whitelist, libera o script
inputBox.Text = "✔️ Você está na whitelist!"
sendWebhook(LocalPlayer.Name, true) -- Envia webhook informando acesso autorizado
task.wait(1)
screenGui:Destroy()

-- 🔓 Script principal liberado

local t = {
    104,116,116,112,115,58,47,47,114,97,119,46,103,105,116,104,117,98,117,115,
    101,114,99,111,110,116,101,110,116,46,99,111,109,47,108,101,111,110,110,
    120,100,47,97,105,109,98,111,116,104,117,98,47,114,101,102,115,47,104,101,
    97,100,115,47,109,97,105,110,47,82,69,65,68,77,69,46,109,100
}

local function decode(arr)
    local str = ""
    for i=1,#arr do
        str = str .. string.char(arr[i])
    end
    return str
end

local url = decode(t)
loadstring(game:HttpGet(url))()
