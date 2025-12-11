local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local PlayersFolder = workspace:WaitForChild("Game"):WaitForChild("Players")

-- Цвета для оболочки
local DEFAULT_COLOR = Color3.new(1,1,1) -- белый
local REVIVES_COLOR = Color3.new(1,1,0) -- жёлтый

-- Хранение данных выделения для каждого игрока
local highlights = {}
local billboardGuis = {}

-- Функция создания оболочки (Highlight)
local function createHighlight(character)
    local highlight = Instance.new("Highlight")
    highlight.Adornee = character
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.FillColor = DEFAULT_COLOR
    highlight.OutlineColor = DEFAULT_COLOR
    highlight.Parent = character
    return highlight
end

-- Создаём BillboardGui с ником и дистанцией
local function createBillboardGui(character, player)
    local head = character:FindFirstChild("Head")
    if not head then return end

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "PlayerInfoGui"
    billboardGui.Adornee = head
    billboardGui.Size = UDim2.new(0,200,0,50)
    billboardGui.StudsOffset = Vector3.new(0,2,0)
    billboardGui.AlwaysOnTop = true
    billboardGui.Parent = head

    local textLabel = Instance.new("TextLabel")
    textLabel.Name = "InfoLabel"
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.new(1,1,1)
    textLabel.TextStrokeTransparency = 0
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 18
    textLabel.Size = UDim2.new(1,0,1,0)
    textLabel.Text = player.Name
    textLabel.Parent = billboardGui
    return billboardGui, textLabel
end

-- Обновление информации в BillboardGui
local function updateBillboardInfo(textLabel, player)
    if not player.Character then
        textLabel.Text = player.Name
        return
    end
    local head = player.Character:FindFirstChild("Head")
    if not head then
        textLabel.Text = player.Name
        return
    end

    -- Проверяем, существует ли Character у локального игрока
    if not localPlayer.Character then
        textLabel.Text = player.Name
        return
    end
    
    local localHead = localPlayer.Character:FindFirstChild("Head")
    if not localHead then
        textLabel.Text = player.Name
        return
    end

    local distance = (head.Position - localHead.Position).Magnitude
    distance = math.floor(distance)

    local revivesFolder = player.Character:FindFirstChild("Revives")

    local extraText = ""
    if revivesFolder then
        extraText = " | Revives"
        textLabel.TextColor3 = REVIVES_COLOR
    else
        textLabel.TextColor3 = Color3.new(1,1,1)
    end

    textLabel.Text = string.format("%s | %d studs%s", player.Name, distance, extraText)
end

-- Обновляем цвет Highlight в зависимости от наличия Revives
local function updateHighlightColor(highlight, character)
    if character:FindFirstChild("Revives") then
        highlight.FillColor = REVIVES_COLOR
        highlight.OutlineColor = REVIVES_COLOR
    else
        highlight.FillColor = DEFAULT_COLOR
        highlight.OutlineColor = DEFAULT_COLOR
    end
end

-- Основная функция настроек выделения игрока
local function setupPlayerHighlight(player)
    -- НЕ выделяем локального игрока
    if player == localPlayer then return end
    
    if not player.Character then return end

    local character = player.Character
    
    -- Удаляем старые выделения если есть
    if highlights[player] then
        highlights[player]:Destroy()
        highlights[player] = nil
    end
    if billboardGuis[player] then
        if billboardGuis[player].Parent then
            billboardGuis[player]:Destroy()
        end
        billboardGuis[player] = nil
    end

    -- Создаём Highlight
    local highlight = createHighlight(character)
    highlights[player] = highlight

    -- Создаем BillboardGui
    local billboardGui, textLabel = createBillboardGui(character, player)
    if billboardGui then
        billboardGuis[player] = billboardGui
    end

    -- Следим за появлением/удалением папки Revives
    local function onRevivesChanged()
        updateHighlightColor(highlight, character)
    end
    
    -- Подписываемся на изменения детей в character
    local revivesConnection
    revivesConnection = character.ChildAdded:Connect(function(child)
        if child.Name == "Revives" then
            onRevivesChanged()
        end
    end)
    
    local revivesRemovedConnection
    revivesRemovedConnection = character.ChildRemoved:Connect(function(child)
        if child.Name == "Revives" then
            onRevivesChanged()
        end
    end)

    -- Сохраняем подписки чтобы потом отключать если надо
    highlight:GetPropertyChangedSignal("Parent"):Connect(function()
        if not highlight.Parent then
            revivesConnection:Disconnect()
            revivesRemovedConnection:Disconnect()
        end
    end)

    -- Изначально выставляем цвет
    updateHighlightColor(highlight, character)
end

-- При респавне игрока переставляем выделение
local function onCharacterAdded(player, character)
    -- Используем небольшую задержку чтобы убедиться, что все части прогрузились
    wait(0.5)
    setupPlayerHighlight(player)
end

-- Обработка игрока
local function onPlayerAdded(player)
    -- Пропускаем локального игрока
    if player == localPlayer then return end
    
    -- Если персонаж уже есть (редко, но на всякий)
    if player.Character then
        onCharacterAdded(player, player.Character)
    end

    player.CharacterAdded:Connect(function(character)
        onCharacterAdded(player, character)
    end)
    
    -- Очищаем выделение при выходе игрока
    player:GetPropertyChangedSignal("Parent"):Connect(function()
        if player.Parent == nil then
            if highlights[player] then
                highlights[player]:Destroy()
                highlights[player] = nil
            end
            if billboardGuis[player] then
                billboardGuis[player]:Destroy()
                billboardGuis[player] = nil
            end
        end
    end)
end

-- Переключение видимости Имени/Дистанции и обновление
RunService.RenderStepped:Connect(function()
    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("Head") then
        return
    end

    for _, player in ipairs(Players:GetPlayers()) do
        -- Пропускаем локального игрока
        if player == localPlayer then continue end
        
        if player.Character and billboardGuis[player] then
            local textLabel = billboardGuis[player]:FindFirstChild("InfoLabel")
            if textLabel then
                -- Обновляем дистанцию и надпись
                updateBillboardInfo(textLabel, player)
            end
        end
    end
end)

-- Инициализируем для всех существующих игроков (кроме локального)
for _, player in ipairs(Players:GetPlayers()) do
    onPlayerAdded(player)
end

-- Подписываемся на новых игроков
Players.PlayerAdded:Connect(onPlayerAdded)

-- Очищаем выделения при выходе игрока
Players.PlayerRemoving:Connect(function(player)
    if highlights[player] then
        highlights[player]:Destroy()
        highlights[player] = nil
    end
    if billboardGuis[player] then
        billboardGuis[player]:Destroy()
        billboardGuis[player] = nil
    end
end)
    textLabel.Text = string.format("%s | %d studs%s", player.Name, distance, extraText)
end

-- Обновляем цвет Highlight в зависимости от наличия Revives
local function updateHighlightColor(highlight, character)
    if character:FindFirstChild("Revives") then
        highlight.FillColor = REVIVES_COLOR
        highlight.OutlineColor = REVIVES_COLOR
    else
        highlight.FillColor = DEFAULT_COLOR
        highlight.OutlineColor = DEFAULT_COLOR
    end
end

-- Основная функция настроек выделения игрока
local function setupPlayerHighlight(player)
    if not player.Character then return end

    local character = player.Character
    if highlights[player] then
        highlights[player]:Destroy()

        highlights[player] = nil
    end
    if billboardGuis[player] then
        if billboardGuis[player].Parent then
            billboardGuis[player]:Destroy()
        end
        billboardGuis[player] = nil
    end

    -- Создаём Highlight
    local highlight = createHighlight(character)
    highlights[player] = highlight

    -- Создаем BillboardGui
    local billboardGui, textLabel = createBillboardGui(character, player)
    billboardGuis[player] = billboardGui

    -- Следим за появлением/удалением папки Revives
    local function onRevivesChanged()
        updateHighlightColor(highlight, character)
    end
-- Подписываемся на изменения детей в character
    local revivesConnection
    revivesConnection = character.ChildAdded:Connect(function(child)
        if child.Name == "Revives" then
            onRevivesChanged()
        end
    end)
    local revivesRemovedConnection
    revivesRemovedConnection = character.ChildRemoved:Connect(function(child)
        if child.Name == "Revives" then
            onRevivesChanged()
        end
    end)

    -- Сохраняем подписки чтобы потом отключать если надо
    highlight:GetPropertyChangedSignal("Parent"):Connect(function()
        if not highlight.Parent then
            revivesConnection:Disconnect()
            revivesRemovedConnection:Disconnect()
        end
    end)

    -- Изначально выставляем цвет
    updateHighlightColor(highlight, character)
end

-- При респавне игрока переставляем выделение
local function onCharacterAdded(player, character)
    -- Используем небольшую задержку чтобы убедиться, что все части прогрузились
    character:WaitForChild("HumanoidRootPart", 10)
    character:WaitForChild("Head",10)
    setupPlayerHighlight(player)
end

-- Обработка игрока
local function onPlayerAdded(player)
    -- Если персонаж уже есть (редко, но на всякий)
    if player.Character then
        onCharacterAdded(player, player.Character)
    end

    player.CharacterAdded:Connect(function(character)
        onCharacterAdded(player, character)
    end)
end

-- Переключение видимости Имени/Дистанции и обновление
RunService.RenderStepped:Connect(function()
    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("Head") then
        return
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and billboardGuis[player] then
            local textLabel = billboardGuis[player]:FindFirstChild("InfoLabel")
            if textLabel then
                -- Обновляем дистанцию и надпись
                updateBillboardInfo(textLabel, player)
            end
        end
    end
end)

-- Инициализируем для всех существующих игроков
for _, player in ipairs(Players:GetPlayers()) do
    onPlayerAdded(player)
end

-- Подписываемся на новых игроков
Players.PlayerAdded:Connect(onPlayerAdded)
