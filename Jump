local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Создаем ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CarryToggleGui"
screenGui.Parent = CoreGui  -- Внимание! Добавляем в CoreGui

-- Создаем кнопку
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 120, 0, 50)
button.Position = UDim2.new(0, 20, 0, 20)
button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Красный
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 24
button.Text = "Jump: Вкл"
button.BackgroundTransparency = 0.5 -- Полупрозрачная кнопка
button.Parent = screenGui

local isRunning = false
local spamDelay = 0  -- Задержка в секундах между вызовами BindableEvent

-- Заменяем carryAll и spawn на вызовы BindableEvents
local function fireJumpEvents()
    local LocalPlayer = Players.LocalPlayer
    if not LocalPlayer then return end

    local PlayerScripts = LocalPlayer:WaitForChild("PlayerScripts", 10)
    if not PlayerScripts then return end

    local Events = PlayerScripts:WaitForChild("Events", 10)
    if not Events then return end

    local TemporaryEvents = Events:WaitForChild("temporary_events", 10)
    if not TemporaryEvents then return end

    local JumpReact = TemporaryEvents:WaitForChild("JumpReact", 10)
    if JumpReact then
        JumpReact:Fire("Hello from Carry Button!")
    end

    local EndJump = TemporaryEvents:WaitForChild("EndJump", 10)
    if EndJump then
        EndJump:Fire("Jump Ended from Carry Button!")
    end
end

-- Перетаскивание для мобильных и ПК
local dragging = false
local startTouchPosition
local startButtonPosition

button.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        startTouchPosition = input.Position
        startButtonPosition = button.Position

        input.Changed:Connect(function(change)
            if change == "UserInputState.End" then
                dragging = false
            end
        end)
    end
end)

button.InputChanged:Connect(function(input)
    if (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) and dragging then
        local delta = input.Position - startTouchPosition
        local newPos = startButtonPosition + UDim2.new(0, delta.X, 0, delta.Y)

        -- Ограничим внутри границ экрана
        local parentSize = screenGui.AbsoluteSize
        local btnSize = button.Size

        local newX = math.clamp(newPos.X.Offset, 0, parentSize.X - btnSize.X.Offset)
        local newY = math.clamp(newPos.Y.Offset, 0, parentSize.Y - btnSize.Y.Offset)

        button.Position = UDim2.new(0, newX, 0, newY)
    end
end)

button.MouseButton1Click:Connect(function()
    if not isRunning then
        isRunning = true
        button.Text = "Jump: Выкл"
        button.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Зеленый

        spawn(function()
            while isRunning do
                fireJumpEvents()
                wait(spamDelay)  -- Задержка перед следующим вызовом
            end
        end)
    else
        isRunning = false
        button.Text = "Jump: Вкл"
        button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Красный
    end
end)
