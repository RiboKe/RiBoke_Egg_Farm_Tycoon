-- Функция для рекурсивного поиска всех папок с именем ItemDebris
local function findItemDebrisFolders(startFolder)
    local debrisFolders = {}
    
    for _, child in ipairs(startFolder:GetChildren()) do
        if child:IsA("Folder") and child.Name == "ItemDebris" then
            table.insert(debrisFolders, child)
        elseif child:IsA("Model") or child:IsA("Folder") then
            local subFolders = findItemDebrisFolders(child)
            for _, folder in ipairs(subFolders) do
                table.insert(debrisFolders, folder)
            end
        end
    end
    
    return debrisFolders
end

-- Создаем переменную для отслеживания состояния скрипта (включен/выключен)
local scriptEnabled = false

-- Функция для перемещения объекта к игроку, отключения CanCollide
local function processObject(object)
    local player = game.Players.LocalPlayer
    if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = player.Character.HumanoidRootPart.Position + Vector3.new(0, 10, 0) -- Перемещаем объект на X единиц выше головы игрока
        
        if object:IsA("Model") and object.PrimaryPart then
            object:SetPrimaryPartCFrame(CFrame.new(targetPosition))
            -- Обрабатываем все части внутри модели
            for _, part in ipairs(object:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        elseif object:IsA("BasePart") then
            object.CFrame = CFrame.new(targetPosition)
            object.CanCollide = false
        end
    end
end

-- Функция для обработки всех объектов в папках ItemDebris
local function handleObjects()
    local allDebrisFolders = findItemDebrisFolders(workspace)
    for _, debrisFolder in ipairs(allDebrisFolders) do
        for _, child in ipairs(debrisFolder:GetChildren()) do
            if child:IsA("Model") or child:IsA("BasePart") then
                wait(0.1)
                if scriptEnabled then
                    processObject(child)
                end
            end
        end
    end
end

-- Подключаемся к событию добавления нового объекта в папку ItemDebris
local function connectChildAddedEvents(debrisFolder)
    debrisFolder.ChildAdded:Connect(function(child)
        if child:IsA("Model") or child:IsA("BasePart") then
            wait(0.1)
            if scriptEnabled then
                processObject(child)
            end
        end
    end)
end

-- Находим все папки ItemDebris и подключаем события добавления новых объектов
local allDebrisFolders = findItemDebrisFolders(workspace)
for _, debrisFolder in ipairs(allDebrisFolders) do
    connectChildAddedEvents(debrisFolder)
end

-- Создаем GUI элементы
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 140)
mainFrame.Position = UDim2.new(0.5, -110, 0.5, -70) -- Позиция по центру экрана
mainFrame.BackgroundTransparency = 0.5
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Черный фон
mainFrame.BorderSizePixel = 0
mainFrame.Draggable = true
mainFrame.Active = true
mainFrame.Visible = true -- Изначально видимое окно
mainFrame.Parent = screenGui

local titleFrame = Instance.new("Frame")
titleFrame.Size = UDim2.new(1, 0, 0, 20)
titleFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
titleFrame.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.Text = "Hi, exploiter :)"
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Parent = titleFrame

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 30)
button.Position = UDim2.new(0.5, -50, 0, 30)
button.Text = "Off"
button.Parent = mainFrame

local valueLabel = Instance.new("TextLabel")
valueLabel.Size = UDim2.new(0.8, 0, 0, 20)
valueLabel.Position = UDim2.new(0.1, 0, 0, 100)
valueLabel.Text = "Status: Off"
valueLabel.BackgroundTransparency = 1
valueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
valueLabel.Parent = mainFrame

-- Функция для переключения состояния скрипта
local function toggleScript()
    scriptEnabled = not scriptEnabled
    if scriptEnabled then
        button.Text = "On"
        valueLabel.Text = "Status: On"
        handleObjects() -- Обрабатываем все существующие объекты при включении скрипта
        coroutine.wrap(function()
            while scriptEnabled do
                local time = tick() * 2 -- Увеличиваем скорость анимации
                local red = math.sin(time) * 0.5 + 0.5
                local green = math.sin(time + 2 * math.pi / 3) * 0.5 + 0.5
                local blue = math.sin(time + 4 * math.pi / 3) * 0.5 + 0.5
                button.BackgroundColor3 = Color3.fromRGB(red * 255, green * 255, blue * 255)
                wait(0.05)
            end
            button.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- Возвращаем белый цвет после завершения эффекта
        end)()
    else
        button.Text = "Off"
        valueLabel.Text = "Status: Off"
        button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    end
end

-- Привязываем функцию к нажатию кнопки
button.MouseButton1Click:Connect(toggleScript)

-- Функция для сворачивания/разворачивания GUI по нажатию клавиши "K"
local function toggleGUIVisibility()
    if mainFrame.Visible then
        mainFrame.Visible = false
        mainFrame.BackgroundTransparency = 1 -- Делаем фон полностью прозрачным
    else
        mainFrame.Visible = true
        mainFrame.BackgroundTransparency = 0.5 -- Восстанавливаем полупрозрачность фона
    end
end

game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent then
        if input.KeyCode == Enum.KeyCode.K then
            toggleGUIVisibility()
        end
    end
end)

-- Добавляем эффект тени через изменение цвета и прозрачности основного окна
local shadowEffect = Instance.new("ImageLabel")
shadowEffect.Size = UDim2.new(1, 20, 1, 20) -- Размер больше основного окна
shadowEffect.Position = UDim2.new(0, -10, 0, -10) -- Смещение для создания эффекта тени
shadowEffect.BackgroundTransparency = 1 -- Полностью прозрачный фон
shadowEffect.Image = "" -- Без изображения
shadowEffect.ImageColor3 = Color3.fromRGB(0, 0, 0) -- Черный цвет для эффекта тени
shadowEffect.ImageTransparency = 0.8 -- Частичная прозрачность для эффекта тени
shadowEffect.ZIndex = mainFrame.ZIndex - 1 -- Задаем Z-индекс ниже основного окна
shadowEffect.Parent = mainFrame

-- Обновление положения и размера эффекта тени при изменении основного окна
mainFrame:GetPropertyChangedSignal("Position"):Connect(function()
    shadowEffect.Position = mainFrame.Position - UDim2.new(0, 10, 0, 10)
end)

mainFrame:GetPropertyChangedSignal("Size"):Connect(function()
    shadowEffect.Size = mainFrame.Size + UDim2.new(0, 20, 0, 20)
end)
