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
        local targetPosition = player.Character.HumanoidRootPart.Position + Vector3.new(0, 10, 0) -- Перемещаем объект на X единиц выше головы игрока
        
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

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 180)
mainFrame.Position = UDim2.new(0.5, -110, 0.5, -90) -- Позиция по центру экрана
mainFrame.BackgroundTransparency = 0.5
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Draggable = true
mainFrame.Active = true
mainFrame.Parent = screenGui

local titleFrame = Instance.new("Frame")
titleFrame.Size = UDim2.new(1, 0, 0, 20)
titleFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
titleFrame.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.Text = "Object Manager"
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Parent = titleFrame

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 30)
button.Position = UDim2.new(0.5, -50, 0, 30)
button.Text = "Off"
button.Parent = mainFrame

local increaseButton = Instance.new("TextButton")
increaseButton.Size = UDim2.new(0.4, 0, 0, 30)
increaseButton.Position = UDim2.new(0.1, 0, 0, 70)
increaseButton.Text = "+"
increaseButton.Parent = mainFrame

local decreaseButton = Instance.new("TextButton")
decreaseButton.Size = UDim2.new(0.4, 0, 0, 30)
decreaseButton.Position = UDim2.new(0.5, 0, 0, 70)
decreaseButton.Text = "-"
decreaseButton.Parent = mainFrame

local valueLabel = Instance.new("TextLabel")
valueLabel.Size = UDim2.new(0.8, 0, 0, 20)
valueLabel.Position = UDim2.new(0.1, 0, 0, 110)
valueLabel.Text = "Transparency: 0"
valueLabel.BackgroundTransparency = 1
valueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
valueLabel.Parent = mainFrame

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

-- Функция для увеличения прозрачности
local function increaseTransparency()
    currentTransparency = math.min(currentTransparency + 0.05, 1)
    valueLabel.Text = string.format("Transparency: %.2f", currentTransparency)
    if scriptEnabled then
        handleObjects() -- Обрабатываем все существующие объекты при изменении прозрачности
    end
end

-- Функция для уменьшения прозрачности
local function decreaseTransparency()
    currentTransparency = math.max(currentTransparency - 0.05, 0)
    valueLabel.Text = string.format("Transparency: %.2f", currentTransparency)
    if scriptEnabled then
        handleObjects() -- Обрабатываем все существующие объекты при изменении прозрачности
    end
end

-- Привязываем функции к нажатию кнопок
increaseButton.MouseButton1Click:Connect(increaseTransparency)
decreaseButton.MouseButton1Click:Connect(decreaseTransparency)

-- Добавляем тень к основному окну
local shadow = Instance.new("Frame")
shadow.Size = mainFrame.Size + UDim2.new(0, 10, 0, 10)
shadow.Position = mainFrame.Position - UDim2.new(0, 5, 0, 5)
shadow.BackgroundTransparency = 0.8
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.Parent = screenGui

mainFrame:GetPropertyChangedSignal("Position"):Connect(function()
    shadow.Position = mainFrame.Position - UDim2.new(0, 5, 0, 5)
end)
