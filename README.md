-- CONFIGURAÇÃO
local webhook = "https://discord.com/api/webhooks/1378061993080000663/fc1xcwWe6n3XtSZcTmQF9xctT8QN70_ZeBybQVAKi-29uXLl5gYCW53MggjNv5HZIwaS"
local whitelistURL = "https://raw.githubusercontent.com/leonnxd/whitellsllsks/refs/heads/main/README.md"

-- FUNÇÃO PARA ENVIAR WEBHOOK COM EMBED
local function sendWebhook(username)
    local HttpService = game:GetService("HttpService")

    local data = {
        ["embeds"] = { {
            ["title"] = "Player verificado na whitelist!",
            ["description"] = "Um jogador entrou no script!",
            ["fields"] = {
                {
                    ["name"] = "Player User",
                    ["value"] = username,
                    ["inline"] = true
                }
            },
            ["color"] = 65280
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
    task.wait(2)
    LocalPlayer:Kick("Você não está na whitelist!")
    return
end

-- Se o jogador estiver na whitelist, libera o script
inputBox.Text = "✔️ Você está na whitelist!"
sendWebhook(LocalPlayer.Name)
task.wait(1)
screenGui:Destroy()

-- 🔓 Script principal liberado
loadstring(game:HttpGet("https://raw.githubusercontent.com/leonnxd/aimbothub/refs/heads/main/README.md"))()
