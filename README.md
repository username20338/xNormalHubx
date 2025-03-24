-- Requer a biblioteca Orion
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

-- Criar a janela principal
local Window = OrionLib:MakeWindow({
    Name = xNormalx",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "MurderMysteryConfig"
})

-- Aba de Configuração
local ConfigTab = Window:MakeTab({
    Name = "Configurações",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Aba de Funções do Jogo
local GameTab = Window:MakeTab({
    Name = "Funções do Jogo",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Variáveis do jogo
local players = game:GetService("Players")
local chosenMurderer, chosenSheriff, chosenInnocent
local espEnabled = false

-- Função para atribuir papéis aos jogadores
function assignRoles()
    local playerList = players:GetPlayers()

    -- Escolher aleatoriamente Murderer, Sheriff e Innocent
    chosenMurderer = playerList[math.random(1, #playerList)]
    local chosenSheriff
    repeat
        chosenSheriff = playerList[math.random(1, #playerList)]
    until chosenSheriff ~= chosenMurderer

    -- Definir os papéis
    for _, player in pairs(playerList) do
        if player == chosenMurderer then
            player:SetAttribute("Role", "Murderer")
        elseif player == chosenSheriff then
            player:SetAttribute("Role", "Sheriff")
        else
            player:SetAttribute("Role", "Innocent")
        end
    end

    -- Notificar os jogadores sobre seus papéis
    OrionLib:MakeNotification({
        Name = "Role Assignment",
        Content = chosenMurderer.Name .. " é o Murderer! " .. chosenSheriff.Name .. " é o Sheriff!",
        Image = "rbxassetid://4483345998",
        Time = 5
    })
end

-- Função para ativar/desativar o ESP
ConfigTab:AddToggle({
    Name = "Ativar ESP",
    Default = false,
    Callback = function(Value)
        espEnabled = Value
        if espEnabled then
            enableESP()
        else
            disableESP()
        end
    end
})

-- Função para criar o ESP
function enableESP()
    for _, player in pairs(players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            -- Criar uma caixa ao redor do jogador (ESP Box)
            local espBox = Instance.new("BoxHandleAdornment")
            espBox.Parent = player.Character
            espBox.Adornee = player.Character.HumanoidRootPart
            espBox.Size = Vector3.new(4, 6, 4) -- Tamanho da caixa
            espBox.Color3 = Color3.fromRGB(0, 255, 0)  -- Cor padrão do Murderer (Verde)
            espBox.Transparency = 0.5
            espBox.Visible = true

            -- Adicionar nome acima da cabeça
            local nameTag = Instance.new("BillboardGui")
            nameTag.Parent = player.Character:WaitForChild("Head")
            nameTag.Adornee = player.Character:WaitForChild("Head")
            nameTag.Size = UDim2.new(0, 200, 0, 50)
            nameTag.StudsOffset = Vector3.new(0, 3, 0)
            nameTag.AlwaysOnTop = true

            local label = Instance.new("TextLabel")
            label.Parent = nameTag
            label.Text = player.Name
            label.TextColor3 = Color3.fromRGB(255, 255, 255)
            label.TextStrokeTransparency = 0.8
            label.BackgroundTransparency = 1
            label.Size = UDim2.new(1, 0, 1, 0)

            -- Altere a cor do nome de acordo com o papel
            if player:GetAttribute("Role") == "Murderer" then
                espBox.Color3 = Color3.fromRGB(0, 255, 0)  -- Verde para Murderer
                label.TextColor3 = Color3.fromRGB(0, 255, 0)  -- Verde para Murderer
            elseif player:GetAttribute("Role") == "Sheriff" then
                espBox.Color3 = Color3.fromRGB(0, 0, 255)  -- Azul para Sheriff
                label.TextColor3 = Color3.fromRGB(0, 0, 255)  -- Azul para Sheriff
            elseif player:GetAttribute("Role") == "Innocent" then
                espBox.Color3 = Color3.fromRGB(0, 255, 255)  -- Ciano para Innocent
                label.TextColor3 = Color3.fromRGB(0, 255, 255)  -- Ciano para Innocent
            end
        end
    end
end

-- Função para desativar o ESP
function disableESP()
    for _, player in pairs(players:GetPlayers()) do
        if player.Character then
            -- Remover a caixa ESP
            for _, obj in pairs(player.Character:GetChildren()) do
                if obj:IsA("BoxHandleAdornment") then
                    obj:Destroy()
                end
            end

            -- Remover os nomes acima das cabeças
            for _, obj in pairs(player.Character:GetChildren()) do
                if obj:IsA("BillboardGui") then
                    obj:Destroy()
                end
            end
        end
    end
end

-- Aba para controle do início do jogo
GameTab:AddButton({
    Name = "Iniciar Jogo",
    Callback = function()
        assignRoles()
    end
})

-- Inicializar a interface Orion
OrionLib:Init()
