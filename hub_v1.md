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

-- Переменная для хранения текущего значения прозрачности
local currentTransparency = 0

-- Функция для перемещения объекта к игроку, отключения CanCollide и установки Transparency
local function processObject(object)
    local player = game.Players.LocalPlayer
    if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = player.Character.HumanoidRootPart.Position + Vector3.new(0, 5, 0) -- Перемещаем объект на 5 единиц выше головы игрока
        
        if object:IsA("Model") and object.PrimaryPart then
            object:SetPrimaryPartCFrame(CFrame.new(targetPosition))
            -- Обрабатываем все части внутри модели
            for _, part in ipairs(object:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                    part.Transparency = currentTransparency
                end
            end
        elseif object:IsA("BasePart") then
            object.CFrame = CFrame.new(targetPosition)
            object.CanCollide = false
            object.Transparency = currentTransparency
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

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0.5, -100, 0, 10) -- Позиция по центру сверху
frame.BackgroundTransparency = 0.5
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.Parent = screenGui

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 50)
button.Position = UDim2.new(0, 0, 0, 0)
button.Text = "Off"
button.Parent = frame

local slider = Instance.new("Slider")
slider.Size = UDim2.new(0, 200, 0, 50)
slider.Position = UDim2.new(0, 0, 0, 50)
slider.Min = 0
slider.Max = 1
slider.Value = 0
slider.ShowDecimalValue = true
slider.Parent = frame

local valueLabel = Instance.new("TextLabel")
valueLabel.Size = UDim2.new(0, 100, 0, 20)
valueLabel.Position = UDim2.new(0.5, -50, 1, 0)
valueLabel.Text = "Transparency: 0"
valueLabel.BackgroundTransparency = 1
valueLabel.Parent = slider

-- Функция для переключения состояния скрипта
local function toggleScript()
    scriptEnabled = not scriptEnabled
    if scriptEnabled then
        button.Text = "On"
        handleObjects() -- Обрабатываем все существующие объекты при включении скрипта
    else
        button.Text = "Off"
    end
end

-- Привязываем функцию к нажатию кнопки
button.MouseButton1Click:Connect(toggleScript)

-- Обновляем значение прозрачности при изменении положения ползунка
slider.Changed:Connect(function(property)
    if property == "Value" then
        currentTransparency = slider.Value
        valueLabel.Text = string.format("Transparency: %.2f", currentTransparency)
        if scriptEnabled then
            handleObjects() -- Обрабатываем все существующие объекты при изменении прозрачности
        end
    end
end)
