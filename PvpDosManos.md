local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Name = "ExtraHitGUI"
gui.Parent = player:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- ==================== TEMAS ====================
local THEMES = {
    Vinho = {
        Background = Color3.fromRGB(10, 10, 10),
        Surface = Color3.fromRGB(20, 20, 20),
        Primary = Color3.fromRGB(128, 0, 32),
        PrimaryLight = Color3.fromRGB(180, 40, 60),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(200, 50, 70),
    },
    Azul = {
        Background = Color3.fromRGB(10, 10, 20),
        Surface = Color3.fromRGB(20, 20, 40),
        Primary = Color3.fromRGB(0, 100, 200),
        PrimaryLight = Color3.fromRGB(50, 150, 255),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(180, 180, 200),
        Accent = Color3.fromRGB(0, 150, 255),
    },
    Verde = {
        Background = Color3.fromRGB(10, 20, 10),
        Surface = Color3.fromRGB(20, 40, 20),
        Primary = Color3.fromRGB(0, 150, 0),
        PrimaryLight = Color3.fromRGB(50, 200, 50),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(180, 200, 180),
        Accent = Color3.fromRGB(0, 200, 0),
    },
    Roxo = {
        Background = Color3.fromRGB(20, 10, 20),
        Surface = Color3.fromRGB(40, 20, 40),
        Primary = Color3.fromRGB(150, 0, 255),
        PrimaryLight = Color3.fromRGB(200, 50, 255),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(200, 180, 200),
        Accent = Color3.fromRGB(180, 0, 255),
    },
    Branco = {
        Background = Color3.fromRGB(240, 240, 240),
        Surface = Color3.fromRGB(255, 255, 255),
        Primary = Color3.fromRGB(80, 80, 80),
        PrimaryLight = Color3.fromRGB(120, 120, 120),
        Text = Color3.fromRGB(0, 0, 0),
        TextDim = Color3.fromRGB(80, 80, 80),
        Accent = Color3.fromRGB(180, 180, 180),
    },
}

local currentTheme = "Vinho"
local COLORS = THEMES[currentTheme]

-- ==================== CONFIGURAÇÕES ====================
local CONFIG = {
    ToggleKey = Enum.KeyCode.P,
    
    ExtraHit = {
        Event = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Punch") or nil,
        Args = {0, 0.1, 1},
        Cooldown = 0,
    },
    
    MetalSkin = {
        Enabled = false,
        Key = Enum.KeyCode.R,
        Event = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Transform") or nil,
        Cooldown = 2,
        LastUse = 0,
    },
    
    RealJump = {
        Enabled = false,
        Key = Enum.KeyCode.LeftAlt,  -- Alt esquerdo
        Cooldown = 0.5,
        LastUse = 0,
    },
    
    ESP = {
        Box = {
            Enabled = false,
            Color = Color3.fromRGB(255, 50, 50),
        },
        Text = {
            Enabled = false,
            Color = Color3.fromRGB(255, 255, 255),
            ShowDistance = true,
        },
    },
    
    AllowMouseButtons = true,
}

-- ==================== VARIÁVEIS GLOBAIS ====================
local uiOpen = true
local selectedKey = nil
local keyChangeModeExtra = false
local lastExtraHitTime = 0
local uiElements = {}
local espBoxHighlights = {}
local espTextLabels = {}

-- ==================== FUNÇÕES AUXILIARES ====================
local function getFriendlyInputName(input)
    if type(input) == "table" then
        if input.KeyCode then
            return input.KeyCode.Name
        elseif input.UserInputType then
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                return "Clique Esquerdo"
            elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
                return "Clique Direito"
            elseif input.UserInputType == Enum.UserInputType.MouseButton3 then
                return "Botão do Meio"
            elseif input.UserInputType == Enum.UserInputType.MouseButton4 then
                return "Botão Lateral 1"
            elseif input.UserInputType == Enum.UserInputType.MouseButton5 then
                return "Botão Lateral 2"
            end
        end
        return "Desconhecido"
    end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        return "Clique Esquerdo"
    elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
        return "Clique Direito"
    elseif input.UserInputType == Enum.UserInputType.MouseButton3 then
        return "Botão do Meio"
    elseif input.UserInputType == Enum.UserInputType.MouseButton4 then
        return "Botão Lateral 1"
    elseif input.UserInputType == Enum.UserInputType.MouseButton5 then
        return "Botão Lateral 2"
    elseif input.UserInputType == Enum.UserInputType.Keyboard then
        return input.KeyCode.Name
    end
    return "Desconhecido"
end

local function updateKeyDisplay()
    if not uiElements or not uiElements.KeyValue then return end
    if selectedKey then
        local displayTable = {}
        if typeof(selectedKey) == "EnumItem" then
            if selectedKey.EnumType == Enum.KeyCode then
                displayTable.KeyCode = selectedKey
            elseif selectedKey.EnumType == Enum.UserInputType then
                displayTable.UserInputType = selectedKey
            end
        end
        uiElements.KeyValue.Text = getFriendlyInputName(displayTable)
    else
        uiElements.KeyValue.Text = "NENHUMA"
    end
end

local function sendExtraHit()
    if not selectedKey or not CONFIG.ExtraHit.Event then return end
    local now = tick()
    if now - lastExtraHitTime < CONFIG.ExtraHit.Cooldown then return end
    lastExtraHitTime = now
    pcall(function()
        CONFIG.ExtraHit.Event:FireServer(unpack(CONFIG.ExtraHit.Args))
    end)
end

local function toggleMetalSkin()
    if not CONFIG.MetalSkin.Event then return end
    local now = tick()
    if now - CONFIG.MetalSkin.LastUse < CONFIG.MetalSkin.Cooldown then return end
    CONFIG.MetalSkin.LastUse = now
    CONFIG.MetalSkin.Enabled = not CONFIG.MetalSkin.Enabled
    -- Atualiza o toggle na interface
    if uiElements.MetalToggleBtn then
        uiElements.MetalToggleBtn.Text = CONFIG.MetalSkin.Enabled and "ON" or "OFF"
        uiElements.MetalToggleBtn.BackgroundColor3 = CONFIG.MetalSkin.Enabled and COLORS.Accent or COLORS.Surface
    end
    pcall(function()
        CONFIG.MetalSkin.Event:FireServer("metalSkin", CONFIG.MetalSkin.Enabled)
    end)
end

local function doRealJump()
    if not CONFIG.RealJump.Enabled then return end
    local now = tick()
    if now - CONFIG.RealJump.LastUse < CONFIG.RealJump.Cooldown then return end
    CONFIG.RealJump.LastUse = now
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end

-- ==================== FUNÇÃO DE TROCA DE TECLA (EXTRA HIT) ====================
local function startKeyChange()
    if keyChangeModeExtra then return end
    keyChangeModeExtra = true
    uiElements.KeyValue.Text = "Pressione uma tecla..."
    uiElements.KeyValue.TextColor3 = COLORS.Accent

    local connection
    connection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed or not keyChangeModeExtra then return end
        if input.KeyCode == CONFIG.ToggleKey then return end

        if input.UserInputType == Enum.UserInputType.Keyboard then
            selectedKey = input.KeyCode
            uiElements.KeyValue.Text = getFriendlyInputName(input)
        elseif CONFIG.AllowMouseButtons and (
            input.UserInputType == Enum.UserInputType.MouseButton1 or
            input.UserInputType == Enum.UserInputType.MouseButton2 or
            input.UserInputType == Enum.UserInputType.MouseButton3 or
            input.UserInputType == Enum.UserInputType.MouseButton4 or
            input.UserInputType == Enum.UserInputType.MouseButton5) then
            selectedKey = input.UserInputType
            uiElements.KeyValue.Text = getFriendlyInputName(input)
        else
            return
        end

        uiElements.KeyValue.TextColor3 = COLORS.Text
        keyChangeModeExtra = false
        connection:Disconnect()
    end)
end

-- ==================== APLICAÇÃO DE TEMA ====================
local function applyTheme(themeName)
    currentTheme = themeName
    COLORS = THEMES[themeName]
    
    if not uiElements or not uiElements.MainFrame then return end
    
    local main = uiElements.MainFrame
    main.BackgroundColor3 = COLORS.Background
    
    local title = main:FindFirstChild("TitleLabel")
    if title then title.TextColor3 = COLORS.Primary end
    
    for _, child in ipairs(main:GetChildren()) do
        if child:IsA("Frame") and child.Name == "Line" then
            child.BackgroundColor3 = COLORS.Primary
        end
    end
    
    local extraCard = main:FindFirstChild("ExtraCard")
    if extraCard then
        extraCard.BackgroundColor3 = COLORS.Surface
        local extraTitle = extraCard:FindFirstChild("ExtraTitle")
        if extraTitle then extraTitle.TextColor3 = COLORS.Primary end
    end
    
    local metalCard = main:FindFirstChild("MetalCard")
    if metalCard then
        metalCard.BackgroundColor3 = COLORS.Surface
        local metalTitle = metalCard:FindFirstChild("MetalTitle")
        if metalTitle then metalTitle.TextColor3 = COLORS.Primary end
        local metalInfo = metalCard:FindFirstChild("MetalInfo")
        if metalInfo then metalInfo.TextColor3 = COLORS.TextDim end
    end
    
    local realJumpCard = main:FindFirstChild("RealJumpCard")
    if realJumpCard then
        realJumpCard.BackgroundColor3 = COLORS.Surface
        local realJumpTitle = realJumpCard:FindFirstChild("RealJumpTitle")
        if realJumpTitle then realJumpTitle.TextColor3 = COLORS.Primary end
        local realJumpInfo = realJumpCard:FindFirstChild("RealJumpInfo")
        if realJumpInfo then realJumpInfo.TextColor3 = COLORS.TextDim end
    end
    
    local espCard = main:FindFirstChild("ESPCard")
    if espCard then
        espCard.BackgroundColor3 = COLORS.Surface
        local espTitle = espCard:FindFirstChild("ESPTitle")
        if espTitle then espTitle.TextColor3 = COLORS.Primary end
    end
    
    local themeCard = main:FindFirstChild("ThemeCard")
    if themeCard then
        themeCard.BackgroundColor3 = COLORS.Surface
        local themeTitle = themeCard:FindFirstChild("ThemeTitle")
        if themeTitle then themeTitle.TextColor3 = COLORS.Primary end
        local themeDropdown = themeCard:FindFirstChild("ThemeDropdown")
        if themeDropdown then
            themeDropdown.BackgroundColor3 = COLORS.Primary
            themeDropdown.TextColor3 = COLORS.Text
        end
        for _, btn in ipairs(themeCard:GetChildren()) do
            if btn:IsA("TextButton") and btn.Name:find("ThemeBtn_") then
                local opt = btn.Name:gsub("ThemeBtn_", "")
                if THEMES[opt] then
                    btn.BackgroundColor3 = THEMES[opt].Primary
                    btn.TextColor3 = COLORS.Text
                end
            end
        end
    end
    
    if uiElements.ChangeBtn then uiElements.ChangeBtn.BackgroundColor3 = COLORS.Primary end
    if uiElements.ResetBtn then
        uiElements.ResetBtn.BorderColor3 = COLORS.Primary
        uiElements.ResetBtn.BackgroundColor3 = COLORS.Surface
        uiElements.ResetBtn.TextColor3 = COLORS.Text
    end
    if uiElements.ActivateBtn then uiElements.ActivateBtn.BackgroundColor3 = COLORS.Primary end
    if uiElements.MetalToggleBtn then
        uiElements.MetalToggleBtn.BorderColor3 = COLORS.Primary
        uiElements.MetalToggleBtn.BackgroundColor3 = CONFIG.MetalSkin.Enabled and COLORS.Accent or COLORS.Surface
        uiElements.MetalToggleBtn.TextColor3 = COLORS.Text
    end
    if uiElements.RealJumpToggleBtn then
        uiElements.RealJumpToggleBtn.BorderColor3 = COLORS.Primary
        uiElements.RealJumpToggleBtn.BackgroundColor3 = CONFIG.RealJump.Enabled and COLORS.Accent or COLORS.Surface
        uiElements.RealJumpToggleBtn.TextColor3 = COLORS.Text
    end
    if uiElements.CooldownLabel then uiElements.CooldownLabel.TextColor3 = COLORS.TextDim end
    if uiElements.CooldownValue then uiElements.CooldownValue.TextColor3 = COLORS.Text end
    if uiElements.CooldownBtns then
        for _, btn in ipairs(uiElements.CooldownBtns) do
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
        end
    end
    if uiElements.RealJumpCooldownLabel then uiElements.RealJumpCooldownLabel.TextColor3 = COLORS.TextDim end
    if uiElements.RealJumpCooldownValue then uiElements.RealJumpCooldownValue.TextColor3 = COLORS.Text end
    if uiElements.RealJumpCooldownBtns then
        for _, btn in ipairs(uiElements.RealJumpCooldownBtns) do
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
        end
    end
    if uiElements.CloseBtn then uiElements.CloseBtn.BackgroundColor3 = COLORS.Primary end
    
    if uiElements.BoxToggleBtn then
        uiElements.BoxToggleBtn.BorderColor3 = COLORS.Primary
        uiElements.BoxToggleBtn.BackgroundColor3 = CONFIG.ESP.Box.Enabled and COLORS.Accent or COLORS.Surface
        uiElements.BoxToggleBtn.TextColor3 = COLORS.Text
    end
    if uiElements.TextToggleBtn then
        uiElements.TextToggleBtn.BorderColor3 = COLORS.Primary
        uiElements.TextToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.Enabled and COLORS.Accent or COLORS.Surface
        uiElements.TextToggleBtn.TextColor3 = COLORS.Text
    end
    if uiElements.DistToggleBtn then
        uiElements.DistToggleBtn.BorderColor3 = COLORS.Primary
        uiElements.DistToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.ShowDistance and COLORS.Accent or COLORS.Surface
        uiElements.DistToggleBtn.TextColor3 = COLORS.Text
    end
end

-- ==================== SISTEMA DE ESP ====================
local function createTextESP(plr)
    if not plr.Character then return false end
    local head = plr.Character:FindFirstChild("Head")
    if not head then return false end
    if espTextLabels[plr] and espTextLabels[plr].billboard then
        pcall(function() espTextLabels[plr].billboard:Destroy() end)
    end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Text"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Enabled = CONFIG.ESP.Text.Enabled
    billboard.Parent = head
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = plr.Name
    nameLabel.TextColor3 = CONFIG.ESP.Text.Color
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = billboard
    local distLabel = Instance.new("TextLabel")
    distLabel.Size = UDim2.new(1, 0, 0.5, 0)
    distLabel.Position = UDim2.new(0, 0, 0.5, 0)
    distLabel.BackgroundTransparency = 1
    distLabel.Text = ""
    distLabel.TextColor3 = CONFIG.ESP.Text.Color
    distLabel.TextScaled = true
    distLabel.Font = Enum.Font.Gotham
    distLabel.Parent = billboard
    espTextLabels[plr] = {billboard = billboard, nameLabel = nameLabel, distLabel = distLabel}
    return true
end

local function createBoxESP(plr)
    if not plr.Character then return false end
    if not plr.Character:FindFirstChild("HumanoidRootPart") then return false end
    if espBoxHighlights[plr] then
        pcall(function() espBoxHighlights[plr]:Destroy() end)
    end
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Box"
    highlight.FillColor = CONFIG.ESP.Box.Color
    highlight.OutlineColor = CONFIG.ESP.Box.Color
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = plr.Character
    espBoxHighlights[plr] = highlight
    return true
end

local function removePlayerESP(plr)
    if espBoxHighlights[plr] then
        pcall(function() espBoxHighlights[plr]:Destroy() end)
        espBoxHighlights[plr] = nil
    end
    if espTextLabels[plr] then
        if espTextLabels[plr].billboard then
            pcall(function() espTextLabels[plr].billboard:Destroy() end)
        end
        espTextLabels[plr] = nil
    end
end

local function updateESPColors()
    for _, highlight in pairs(espBoxHighlights) do
        highlight.FillColor = CONFIG.ESP.Box.Color
        highlight.OutlineColor = CONFIG.ESP.Box.Color
    end
    for _, data in pairs(espTextLabels) do
        if data.nameLabel then
            data.nameLabel.TextColor3 = CONFIG.ESP.Text.Color
            data.distLabel.TextColor3 = CONFIG.ESP.Text.Color
        end
    end
end

local function updateTextDistances()
    if not CONFIG.ESP.Text.Enabled or not CONFIG.ESP.Text.ShowDistance then return end
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    local myPos = player.Character.HumanoidRootPart.Position
    for plr, data in pairs(espTextLabels) do
        if plr and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            local dist = math.floor((myPos - head.Position).Magnitude)
            data.distLabel.Text = dist .. "m"
            data.distLabel.Visible = true
        end
    end
end

RunService.RenderStepped:Connect(function()
    if CONFIG.ESP.Text.Enabled and CONFIG.ESP.Text.ShowDistance then
        updateTextDistances()
    end
end)

Players.PlayerAdded:Connect(function(plr)
    if plr == player then return end
    plr.CharacterAdded:Connect(function()
        task.wait(0.5)
        if CONFIG.ESP.Box.Enabled then createBoxESP(plr) end
        if CONFIG.ESP.Text.Enabled then createTextESP(plr) end
    end)
    if plr.Character then
        task.wait(0.5)
        if CONFIG.ESP.Box.Enabled then createBoxESP(plr) end
        if CONFIG.ESP.Text.Enabled then createTextESP(plr) end
    end
end)

Players.PlayerRemoving:Connect(removePlayerESP)

task.wait(1)
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= player then
        if CONFIG.ESP.Box.Enabled then createBoxESP(plr) end
        if CONFIG.ESP.Text.Enabled then createTextESP(plr) end
    end
end

-- ==================== CONSTRUÇÃO DA INTERFACE ====================
local function buildUI()
    local main = Instance.new("Frame")
    main.Name = "MainFrame"
    main.Size = UDim2.new(0, 550, 0, 750)
    main.Position = UDim2.new(0.5, -275, 0.5, -375)
    main.BackgroundColor3 = COLORS.Background
    main.BorderSizePixel = 0
    main.Visible = uiOpen
    main.Parent = gui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = main

    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 20, 1, 20)
    shadow.Position = UDim2.new(0, -10, 0, -10)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6015897843"
    shadow.ImageColor3 = Color3.new(0,0,0)
    shadow.ImageTransparency = 0.6
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(10,10,10,10)
    shadow.Parent = main

    local title = Instance.new("TextLabel")
    title.Name = "TitleLabel"
    title.Size = UDim2.new(1, -20, 0, 40)
    title.Position = UDim2.new(0, 10, 0, 10)
    title.BackgroundTransparency = 1
    title.Text = "# SCRIPT"
    title.TextColor3 = COLORS.Primary
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = main

    local line1 = Instance.new("Frame")
    line1.Name = "Line"
    line1.Size = UDim2.new(0.9, 0, 0, 1)
    line1.Position = UDim2.new(0.05, 0, 0, 50)
    line1.BackgroundColor3 = COLORS.Primary
    line1.BorderSizePixel = 0
    line1.Parent = main

    -- CARD EXTRA HIT
    local extraCard = Instance.new("Frame")
    extraCard.Name = "ExtraCard"
    extraCard.Size = UDim2.new(0.9, 0, 0, 180)
    extraCard.Position = UDim2.new(0.05, 0, 0, 60)
    extraCard.BackgroundColor3 = COLORS.Surface
    extraCard.BorderSizePixel = 0
    local extraCorner = Instance.new("UICorner")
    extraCorner.CornerRadius = UDim.new(0, 10)
    extraCorner.Parent = extraCard
    extraCard.Parent = main

    local extraTitle = Instance.new("TextLabel")
    extraTitle.Name = "ExtraTitle"
    extraTitle.Size = UDim2.new(1, -15, 0, 25)
    extraTitle.Position = UDim2.new(0, 8, 0, 5)
    extraTitle.BackgroundTransparency = 1
    extraTitle.Text = "## EXTRA HIT"
    extraTitle.TextColor3 = COLORS.Primary
    extraTitle.TextXAlignment = Enum.TextXAlignment.Left
    extraTitle.TextScaled = true
    extraTitle.Font = Enum.Font.GothamBold
    extraTitle.Parent = extraCard

    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(0.5, -5, 0, 20)
    keyLabel.Position = UDim2.new(0, 8, 0, 32)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA CONFIGURADA"
    keyLabel.TextColor3 = COLORS.TextDim
    keyLabel.TextXAlignment = Enum.TextXAlignment.Left
    keyLabel.TextScaled = true
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = extraCard

    local keyValue = Instance.new("TextLabel")
    keyValue.Name = "KeyValue"
    keyValue.Size = UDim2.new(0.5, -5, 0, 25)
    keyValue.Position = UDim2.new(0, 8, 0, 52)
    keyValue.BackgroundTransparency = 1
    keyValue.Text = "NENHUMA"
    keyValue.TextColor3 = COLORS.Text
    keyValue.TextXAlignment = Enum.TextXAlignment.Left
    keyValue.TextScaled = true
    keyValue.Font = Enum.Font.GothamBold
    keyValue.Parent = extraCard

    local btnRow1 = Instance.new("Frame")
    btnRow1.Name = "BtnRow1"
    btnRow1.Size = UDim2.new(1, -20, 0, 35)
    btnRow1.Position = UDim2.new(0, 10, 0, 82)
    btnRow1.BackgroundTransparency = 1
    btnRow1.Parent = extraCard

    local changeBtn = Instance.new("TextButton")
    changeBtn.Name = "ChangeBtn"
    changeBtn.Size = UDim2.new(0, 80, 0, 30)
    changeBtn.Position = UDim2.new(0, 0, 0, 0)
    changeBtn.BackgroundColor3 = COLORS.Primary
    changeBtn.Text = "TROCAR"
    changeBtn.TextColor3 = COLORS.Text
    changeBtn.TextScaled = true
    changeBtn.Font = Enum.Font.GothamBold
    changeBtn.BorderSizePixel = 0
    local changeCorner = Instance.new("UICorner")
    changeCorner.CornerRadius = UDim.new(0, 6)
    changeCorner.Parent = changeBtn
    changeBtn.Parent = btnRow1

    changeBtn.MouseEnter:Connect(function()
        TweenService:Create(changeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    changeBtn.MouseLeave:Connect(function()
        TweenService:Create(changeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    local resetBtn = Instance.new("TextButton")
    resetBtn.Name = "ResetBtn"
    resetBtn.Size = UDim2.new(0, 80, 0, 30)
    resetBtn.Position = UDim2.new(0, 90, 0, 0)
    resetBtn.BackgroundColor3 = COLORS.Surface
    resetBtn.Text = "RESETAR"
    resetBtn.TextColor3 = COLORS.Text
    resetBtn.TextScaled = true
    resetBtn.Font = Enum.Font.GothamBold
    resetBtn.BorderSizePixel = 1
    resetBtn.BorderColor3 = COLORS.Primary
    local resetCorner = Instance.new("UICorner")
    resetCorner.CornerRadius = UDim.new(0, 6)
    resetCorner.Parent = resetBtn
    resetBtn.Parent = btnRow1

    resetBtn.MouseEnter:Connect(function()
        TweenService:Create(resetBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight, TextColor3 = COLORS.Text}):Play()
    end)
    resetBtn.MouseLeave:Connect(function()
        TweenService:Create(resetBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Surface, TextColor3 = COLORS.Text}):Play()
    end)

    local activateBtn = Instance.new("TextButton")
    activateBtn.Name = "ActivateBtn"
    activateBtn.Size = UDim2.new(0, 80, 0, 30)
    activateBtn.Position = UDim2.new(0, 180, 0, 0)
    activateBtn.BackgroundColor3 = COLORS.Primary
    activateBtn.Text = "⚡ ATIVAR"
    activateBtn.TextColor3 = COLORS.Text
    activateBtn.TextScaled = true
    activateBtn.Font = Enum.Font.GothamBold
    activateBtn.BorderSizePixel = 0
    local activateCorner = Instance.new("UICorner")
    activateCorner.CornerRadius = UDim.new(0, 6)
    activateCorner.Parent = activateBtn
    activateBtn.Parent = btnRow1

    activateBtn.MouseEnter:Connect(function()
        TweenService:Create(activateBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    activateBtn.MouseLeave:Connect(function()
        TweenService:Create(activateBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    -- Área de cooldown do Extra Hit
    local cooldownArea = Instance.new("Frame")
    cooldownArea.Name = "CooldownArea"
    cooldownArea.Size = UDim2.new(1, -20, 0, 40)
    cooldownArea.Position = UDim2.new(0, 10, 0, 132)
    cooldownArea.BackgroundTransparency = 1
    cooldownArea.Parent = extraCard

    local cooldownLabel = Instance.new("TextLabel")
    cooldownLabel.Name = "CooldownLabel"
    cooldownLabel.Size = UDim2.new(0, 80, 0, 30)
    cooldownLabel.Position = UDim2.new(0, 0, 0, 5)
    cooldownLabel.BackgroundTransparency = 1
    cooldownLabel.Text = "Cooldown:"
    cooldownLabel.TextColor3 = COLORS.TextDim
    cooldownLabel.TextScaled = true
    cooldownLabel.Font = Enum.Font.Gotham
    cooldownLabel.Parent = cooldownArea

    local cooldownValue = Instance.new("TextLabel")
    cooldownValue.Name = "CooldownValue"
    cooldownValue.Size = UDim2.new(0, 50, 0, 30)
    cooldownValue.Position = UDim2.new(0, 260, 0, 5)
    cooldownValue.BackgroundTransparency = 1
    cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    cooldownValue.TextColor3 = COLORS.Text
    cooldownValue.TextScaled = true
    cooldownValue.Font = Enum.Font.GothamBold
    cooldownValue.Parent = cooldownArea

    local btnMinus = Instance.new("TextButton")
    btnMinus.Size = UDim2.new(0, 30, 0, 30)
    btnMinus.Position = UDim2.new(0, 220, 0, 5)
    btnMinus.BackgroundColor3 = COLORS.Primary
    btnMinus.Text = "-"
    btnMinus.TextColor3 = COLORS.Text
    btnMinus.TextScaled = true
    btnMinus.Font = Enum.Font.GothamBold
    btnMinus.BorderSizePixel = 0
    local minusCorner = Instance.new("UICorner")
    minusCorner.CornerRadius = UDim.new(0, 5)
    minusCorner.Parent = btnMinus
    btnMinus.Parent = cooldownArea

    btnMinus.MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.ExtraHit.Cooldown - 0.1)
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)

    local btnPlus = Instance.new("TextButton")
    btnPlus.Size = UDim2.new(0, 30, 0, 30)
    btnPlus.Position = UDim2.new(0, 300, 0, 5)
    btnPlus.BackgroundColor3 = COLORS.Primary
    btnPlus.Text = "+"
    btnPlus.TextColor3 = COLORS.Text
    btnPlus.TextScaled = true
    btnPlus.Font = Enum.Font.GothamBold
    btnPlus.BorderSizePixel = 0
    local plusCorner = Instance.new("UICorner")
    plusCorner.CornerRadius = UDim.new(0, 5)
    plusCorner.Parent = btnPlus
    btnPlus.Parent = cooldownArea

    btnPlus.MouseButton1Click:Connect(function()
        local new = CONFIG.ExtraHit.Cooldown + 0.1
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)

    -- CARD METAL SKIN
    local metalCard = Instance.new("Frame")
    metalCard.Name = "MetalCard"
    metalCard.Size = UDim2.new(0.9, 0, 0, 80)
    metalCard.Position = UDim2.new(0.05, 0, 0, 250)
    metalCard.BackgroundColor3 = COLORS.Surface
    metalCard.BorderSizePixel = 0
    local metalCorner = Instance.new("UICorner")
    metalCorner.CornerRadius = UDim.new(0, 10)
    metalCorner.Parent = metalCard
    metalCard.Parent = main

    local metalTitle = Instance.new("TextLabel")
    metalTitle.Name = "MetalTitle"
    metalTitle.Size = UDim2.new(1, -15, 0, 25)
    metalTitle.Position = UDim2.new(0, 8, 0, 5)
    metalTitle.BackgroundTransparency = 1
    metalTitle.Text = "### METAL SKIN"
    metalTitle.TextColor3 = COLORS.Primary
    metalTitle.TextXAlignment = Enum.TextXAlignment.Left
    metalTitle.TextScaled = true
    metalTitle.Font = Enum.Font.GothamBold
    metalTitle.Parent = metalCard

    local metalInfo = Instance.new("TextLabel")
    metalInfo.Name = "MetalInfo"
    metalInfo.Size = UDim2.new(0.5, -5, 0, 20)
    metalInfo.Position = UDim2.new(0, 8, 0, 32)
    metalInfo.BackgroundTransparency = 1
    metalInfo.Text = "Tecla R (cooldown 2s)"
    metalInfo.TextColor3 = COLORS.TextDim
    metalInfo.TextXAlignment = Enum.TextXAlignment.Left
    metalInfo.TextScaled = true
    metalInfo.Font = Enum.Font.Gotham
    metalInfo.Parent = metalCard

    local metalToggleBtn = Instance.new("TextButton")
    metalToggleBtn.Name = "MetalToggleBtn"
    metalToggleBtn.Size = UDim2.new(0, 50, 0, 30)
    metalToggleBtn.Position = UDim2.new(1, -60, 0, 30)
    metalToggleBtn.BackgroundColor3 = CONFIG.MetalSkin.Enabled and COLORS.Accent or COLORS.Surface
    metalToggleBtn.Text = CONFIG.MetalSkin.Enabled and "ON" or "OFF"
    metalToggleBtn.TextColor3 = COLORS.Text
    metalToggleBtn.TextScaled = true
    metalToggleBtn.Font = Enum.Font.GothamBold
    metalToggleBtn.BorderSizePixel = 1
    metalToggleBtn.BorderColor3 = COLORS.Primary
    local metalToggleCorner = Instance.new("UICorner")
    metalToggleCorner.CornerRadius = UDim.new(0, 5)
    metalToggleCorner.Parent = metalToggleBtn
    metalToggleBtn.Parent = metalCard

    -- CARD REAL JUMP (agora com Alt)
    local realJumpCard = Instance.new("Frame")
    realJumpCard.Name = "RealJumpCard"
    realJumpCard.Size = UDim2.new(0.9, 0, 0, 110)
    realJumpCard.Position = UDim2.new(0.05, 0, 0, 340)
    realJumpCard.BackgroundColor3 = COLORS.Surface
    realJumpCard.BorderSizePixel = 0
    local realJumpCorner = Instance.new("UICorner")
    realJumpCorner.CornerRadius = UDim.new(0, 10)
    realJumpCorner.Parent = realJumpCard
    realJumpCard.Parent = main

    local realJumpTitle = Instance.new("TextLabel")
    realJumpTitle.Name = "RealJumpTitle"
    realJumpTitle.Size = UDim2.new(1, -15, 0, 25)
    realJumpTitle.Position = UDim2.new(0, 8, 0, 5)
    realJumpTitle.BackgroundTransparency = 1
    realJumpTitle.Text = "#### REAL JUMP (Infinito)"
    realJumpTitle.TextColor3 = COLORS.Primary
    realJumpTitle.TextXAlignment = Enum.TextXAlignment.Left
    realJumpTitle.TextScaled = true
    realJumpTitle.Font = Enum.Font.GothamBold
    realJumpTitle.Parent = realJumpCard

    local realJumpInfo = Instance.new("TextLabel")
    realJumpInfo.Name = "RealJumpInfo"
    realJumpInfo.Size = UDim2.new(0.5, -5, 0, 20)
    realJumpInfo.Position = UDim2.new(0, 8, 0, 32)
    realJumpInfo.BackgroundTransparency = 1
    realJumpInfo.Text = "Tecla: ALT"
    realJumpInfo.TextColor3 = COLORS.TextDim
    realJumpInfo.TextXAlignment = Enum.TextXAlignment.Left
    realJumpInfo.TextScaled = true
    realJumpInfo.Font = Enum.Font.Gotham
    realJumpInfo.Parent = realJumpCard

    local realJumpToggleBtn = Instance.new("TextButton")
    realJumpToggleBtn.Name = "RealJumpToggleBtn"
    realJumpToggleBtn.Size = UDim2.new(0, 50, 0, 30)
    realJumpToggleBtn.Position = UDim2.new(1, -60, 0, 30)
    realJumpToggleBtn.BackgroundColor3 = CONFIG.RealJump.Enabled and COLORS.Accent or COLORS.Surface
    realJumpToggleBtn.Text = CONFIG.RealJump.Enabled and "ON" or "OFF"
    realJumpToggleBtn.TextColor3 = COLORS.Text
    realJumpToggleBtn.TextScaled = true
    realJumpToggleBtn.Font = Enum.Font.GothamBold
    realJumpToggleBtn.BorderSizePixel = 1
    realJumpToggleBtn.BorderColor3 = COLORS.Primary
    local realJumpToggleCorner = Instance.new("UICorner")
    realJumpToggleCorner.CornerRadius = UDim.new(0, 5)
    realJumpToggleCorner.Parent = realJumpToggleBtn
    realJumpToggleBtn.Parent = realJumpCard

    -- Área de cooldown do Real Jump
    local realJumpCooldownArea = Instance.new("Frame")
    realJumpCooldownArea.Name = "RealJumpCooldownArea"
    realJumpCooldownArea.Size = UDim2.new(1, -20, 0, 30)
    realJumpCooldownArea.Position = UDim2.new(0, 10, 0, 70)
    realJumpCooldownArea.BackgroundTransparency = 1
    realJumpCooldownArea.Parent = realJumpCard

    local realJumpCooldownLabel = Instance.new("TextLabel")
    realJumpCooldownLabel.Name = "RealJumpCooldownLabel"
    realJumpCooldownLabel.Size = UDim2.new(0, 80, 0, 30)
    realJumpCooldownLabel.Position = UDim2.new(0, 0, 0, 0)
    realJumpCooldownLabel.BackgroundTransparency = 1
    realJumpCooldownLabel.Text = "Cooldown:"
    realJumpCooldownLabel.TextColor3 = COLORS.TextDim
    realJumpCooldownLabel.TextScaled = true
    realJumpCooldownLabel.Font = Enum.Font.Gotham
    realJumpCooldownLabel.Parent = realJumpCooldownArea

    local realJumpCooldownValue = Instance.new("TextLabel")
    realJumpCooldownValue.Name = "RealJumpCooldownValue"
    realJumpCooldownValue.Size = UDim2.new(0, 50, 0, 30)
    realJumpCooldownValue.Position = UDim2.new(0, 260, 0, 0)
    realJumpCooldownValue.BackgroundTransparency = 1
    realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    realJumpCooldownValue.TextColor3 = COLORS.Text
    realJumpCooldownValue.TextScaled = true
    realJumpCooldownValue.Font = Enum.Font.GothamBold
    realJumpCooldownValue.Parent = realJumpCooldownArea

    local realJumpBtnMinus = Instance.new("TextButton")
    realJumpBtnMinus.Size = UDim2.new(0, 30, 0, 30)
    realJumpBtnMinus.Position = UDim2.new(0, 220, 0, 0)
    realJumpBtnMinus.BackgroundColor3 = COLORS.Primary
    realJumpBtnMinus.Text = "-"
    realJumpBtnMinus.TextColor3 = COLORS.Text
    realJumpBtnMinus.TextScaled = true
    realJumpBtnMinus.Font = Enum.Font.GothamBold
    realJumpBtnMinus.BorderSizePixel = 0
    local realJumpMinusCorner = Instance.new("UICorner")
    realJumpMinusCorner.CornerRadius = UDim.new(0, 5)
    realJumpMinusCorner.Parent = realJumpBtnMinus
    realJumpBtnMinus.Parent = realJumpCooldownArea

    realJumpBtnMinus.MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.RealJump.Cooldown - 0.1)
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)

    local realJumpBtnPlus = Instance.new("TextButton")
    realJumpBtnPlus.Size = UDim2.new(0, 30, 0, 30)
    realJumpBtnPlus.Position = UDim2.new(0, 300, 0, 0)
    realJumpBtnPlus.BackgroundColor3 = COLORS.Primary
    realJumpBtnPlus.Text = "+"
    realJumpBtnPlus.TextColor3 = COLORS.Text
    realJumpBtnPlus.TextScaled = true
    realJumpBtnPlus.Font = Enum.Font.GothamBold
    realJumpBtnPlus.BorderSizePixel = 0
    local realJumpPlusCorner = Instance.new("UICorner")
    realJumpPlusCorner.CornerRadius = UDim.new(0, 5)
    realJumpPlusCorner.Parent = realJumpBtnPlus
    realJumpBtnPlus.Parent = realJumpCooldownArea

    realJumpBtnPlus.MouseButton1Click:Connect(function()
        local new = CONFIG.RealJump.Cooldown + 0.1
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)

    -- CARD ESP
    local espCard = Instance.new("Frame")
    espCard.Name = "ESPCard"
    espCard.Size = UDim2.new(0.9, 0, 0, 190)
    espCard.Position = UDim2.new(0.05, 0, 0, 460)
    espCard.BackgroundColor3 = COLORS.Surface
    espCard.BorderSizePixel = 0
    local espCorner = Instance.new("UICorner")
    espCorner.CornerRadius = UDim.new(0, 10)
    espCorner.Parent = espCard
    espCard.Parent = main

    local espTitle = Instance.new("TextLabel")
    espTitle.Name = "ESPTitle"
    espTitle.Size = UDim2.new(1, -15, 0, 25)
    espTitle.Position = UDim2.new(0, 8, 0, 5)
    espTitle.BackgroundTransparency = 1
    espTitle.Text = "##### ESP"
    espTitle.TextColor3 = COLORS.Primary
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.TextScaled = true
    espTitle.Font = Enum.Font.GothamBold
    espTitle.Parent = espCard

    -- Caixa colorida
    local boxLabel = Instance.new("TextLabel")
    boxLabel.Name = "BoxLabel"
    boxLabel.Size = UDim2.new(0.5, -5, 0, 25)
    boxLabel.Position = UDim2.new(0, 8, 0, 35)
    boxLabel.BackgroundTransparency = 1
    boxLabel.Text = "Caixa colorida"
    boxLabel.TextColor3 = COLORS.TextDim
    boxLabel.TextXAlignment = Enum.TextXAlignment.Left
    boxLabel.TextScaled = true
    boxLabel.Font = Enum.Font.Gotham
    boxLabel.Parent = espCard

    local boxToggleBtn = Instance.new("TextButton")
    boxToggleBtn.Name = "BoxToggleBtn"
    boxToggleBtn.Size = UDim2.new(0, 45, 0, 25)
    boxToggleBtn.Position = UDim2.new(1, -110, 0, 35)
    boxToggleBtn.BackgroundColor3 = CONFIG.ESP.Box.Enabled and COLORS.Accent or COLORS.Surface
    boxToggleBtn.Text = CONFIG.ESP.Box.Enabled and "ON" or "OFF"
    boxToggleBtn.TextColor3 = COLORS.Text
    boxToggleBtn.TextScaled = true
    boxToggleBtn.Font = Enum.Font.GothamBold
    boxToggleBtn.BorderSizePixel = 1
    boxToggleBtn.BorderColor3 = COLORS.Primary
    local boxToggleCorner = Instance.new("UICorner")
    boxToggleCorner.CornerRadius = UDim.new(0, 5)
    boxToggleCorner.Parent = boxToggleBtn
    boxToggleBtn.Parent = espCard

    local boxColorFrame = Instance.new("Frame")
    boxColorFrame.Name = "BoxColorFrame"
    boxColorFrame.Size = UDim2.new(0, 25, 0, 25)
    boxColorFrame.Position = UDim2.new(1, -55, 0, 35)
    boxColorFrame.BackgroundColor3 = CONFIG.ESP.Box.Color
    boxColorFrame.BorderSizePixel = 0
    local boxColorCorner = Instance.new("UICorner")
    boxColorCorner.CornerRadius = UDim.new(0, 5)
    boxColorCorner.Parent = boxColorFrame
    boxColorFrame.Parent = espCard

    local boxColorBtn = Instance.new("TextButton")
    boxColorBtn.Name = "BoxColorBtn"
    boxColorBtn.Size = UDim2.new(0, 25, 0, 25)
    boxColorBtn.Position = UDim2.new(1, -55, 0, 35)
    boxColorBtn.BackgroundTransparency = 1
    boxColorBtn.Text = "✎"
    boxColorBtn.TextColor3 = COLORS.Text
    boxColorBtn.TextScaled = true
    boxColorBtn.Font = Enum.Font.GothamBold
    boxColorBtn.BorderSizePixel = 0
    boxColorBtn.Parent = espCard

    -- Texto (nome)
    local textLabel = Instance.new("TextLabel")
    textLabel.Name = "TextLabel"
    textLabel.Size = UDim2.new(0.5, -5, 0, 25)
    textLabel.Position = UDim2.new(0, 8, 0, 70)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = "Texto (nome)"
    textLabel.TextColor3 = COLORS.TextDim
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.Gotham
    textLabel.Parent = espCard

    local textToggleBtn = Instance.new("TextButton")
    textToggleBtn.Name = "TextToggleBtn"
    textToggleBtn.Size = UDim2.new(0, 45, 0, 25)
    textToggleBtn.Position = UDim2.new(1, -110, 0, 70)
    textToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.Enabled and COLORS.Accent or COLORS.Surface
    textToggleBtn.Text = CONFIG.ESP.Text.Enabled and "ON" or "OFF"
    textToggleBtn.TextColor3 = COLORS.Text
    textToggleBtn.TextScaled = true
    textToggleBtn.Font = Enum.Font.GothamBold
    textToggleBtn.BorderSizePixel = 1
    textToggleBtn.BorderColor3 = COLORS.Primary
    local textToggleCorner = Instance.new("UICorner")
    textToggleCorner.CornerRadius = UDim.new(0, 5)
    textToggleCorner.Parent = textToggleBtn
    textToggleBtn.Parent = espCard

    local textColorFrame = Instance.new("Frame")
    textColorFrame.Name = "TextColorFrame"
    textColorFrame.Size = UDim2.new(0, 25, 0, 25)
    textColorFrame.Position = UDim2.new(1, -55, 0, 70)
    textColorFrame.BackgroundColor3 = CONFIG.ESP.Text.Color
    textColorFrame.BorderSizePixel = 0
    local textColorCorner = Instance.new("UICorner")
    textColorCorner.CornerRadius = UDim.new(0, 5)
    textColorCorner.Parent = textColorFrame
    textColorFrame.Parent = espCard

    local textColorBtn = Instance.new("TextButton")
    textColorBtn.Name = "TextColorBtn"
    textColorBtn.Size = UDim2.new(0, 25, 0, 25)
    textColorBtn.Position = UDim2.new(1, -55, 0, 70)
    textColorBtn.BackgroundTransparency = 1
    textColorBtn.Text = "✎"
    textColorBtn.TextColor3 = COLORS.Text
    textColorBtn.TextScaled = true
    textColorBtn.Font = Enum.Font.GothamBold
    textColorBtn.BorderSizePixel = 0
    textColorBtn.Parent = espCard

    -- Mostrar distância
    local distLabel = Instance.new("TextLabel")
    distLabel.Name = "DistLabel"
    distLabel.Size = UDim2.new(0.5, -5, 0, 25)
    distLabel.Position = UDim2.new(0, 8, 0, 105)
    distLabel.BackgroundTransparency = 1
    distLabel.Text = "Mostrar distância"
    distLabel.TextColor3 = COLORS.TextDim
    distLabel.TextXAlignment = Enum.TextXAlignment.Left
    distLabel.TextScaled = true
    distLabel.Font = Enum.Font.Gotham
    distLabel.Parent = espCard

    local distToggleBtn = Instance.new("TextButton")
    distToggleBtn.Name = "DistToggleBtn"
    distToggleBtn.Size = UDim2.new(0, 45, 0, 25)
    distToggleBtn.Position = UDim2.new(1, -55, 0, 105)
    distToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.ShowDistance and COLORS.Accent or COLORS.Surface
    distToggleBtn.Text = CONFIG.ESP.Text.ShowDistance and "ON" or "OFF"
    distToggleBtn.TextColor3 = COLORS.Text
    distToggleBtn.TextScaled = true
    distToggleBtn.Font = Enum.Font.GothamBold
    distToggleBtn.BorderSizePixel = 1
    distToggleBtn.BorderColor3 = COLORS.Primary
    local distToggleCorner = Instance.new("UICorner")
    distToggleCorner.CornerRadius = UDim.new(0, 5)
    distToggleCorner.Parent = distToggleBtn
    distToggleBtn.Parent = espCard

    -- CARD TEMA
    local themeCard = Instance.new("Frame")
    themeCard.Name = "ThemeCard"
    themeCard.Size = UDim2.new(0.9, 0, 0, 70)
    themeCard.Position = UDim2.new(0.05, 0, 0, 660)
    themeCard.BackgroundColor3 = COLORS.Surface
    themeCard.BorderSizePixel = 0
    local themeCorner = Instance.new("UICorner")
    themeCorner.CornerRadius = UDim.new(0, 10)
    themeCorner.Parent = themeCard
    themeCard.Parent = main

    local themeTitle = Instance.new("TextLabel")
    themeTitle.Name = "ThemeTitle"
    themeTitle.Size = UDim2.new(1, -15, 0, 25)
    themeTitle.Position = UDim2.new(0, 8, 0, 5)
    themeTitle.BackgroundTransparency = 1
    themeTitle.Text = "###### TEMA"
    themeTitle.TextColor3 = COLORS.Primary
    themeTitle.TextXAlignment = Enum.TextXAlignment.Left
    themeTitle.TextScaled = true
    themeTitle.Font = Enum.Font.GothamBold
    themeTitle.Parent = themeCard

    local themeDropdown = Instance.new("TextButton")
    themeDropdown.Name = "ThemeDropdown"
    themeDropdown.Size = UDim2.new(0, 100, 0, 30)
    themeDropdown.Position = UDim2.new(1, -110, 0, 30)
    themeDropdown.BackgroundColor3 = COLORS.Primary
    themeDropdown.Text = currentTheme
    themeDropdown.TextColor3 = COLORS.Text
    themeDropdown.TextScaled = true
    themeDropdown.Font = Enum.Font.GothamBold
    themeDropdown.BorderSizePixel = 0
    local dropdownCorner = Instance.new("UICorner")
    dropdownCorner.CornerRadius = UDim.new(0, 5)
    dropdownCorner.Parent = themeDropdown
    themeDropdown.Parent = themeCard

    local themeOptions = {}
    local options = {"Vinho", "Azul", "Verde", "Roxo", "Branco"}
    for i, opt in ipairs(options) do
        local btn = Instance.new("TextButton")
        btn.Name = "ThemeBtn_"..opt
        btn.Size = UDim2.new(0, 70, 0, 30)
        btn.Position = UDim2.new(0, 10 + (i-1)*75, 0, 30)
        btn.BackgroundColor3 = THEMES[opt].Primary
        btn.Text = opt
        btn.TextColor3 = COLORS.Text
        btn.TextScaled = true
        btn.Font = Enum.Font.GothamBold
        btn.BorderSizePixel = 0
        btn.Visible = false
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 5)
        btnCorner.Parent = btn
        btn.Parent = themeCard
        table.insert(themeOptions, btn)
        
        btn.MouseButton1Click:Connect(function()
            applyTheme(opt)
            themeDropdown.Text = opt
            for _, b in ipairs(themeOptions) do b.Visible = false end
        end)
    end

    themeDropdown.MouseButton1Click:Connect(function()
        for _, btn in ipairs(themeOptions) do
            btn.Visible = not btn.Visible
        end
    end)

    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseBtn"
    closeBtn.Size = UDim2.new(0, 25, 0, 25)
    closeBtn.Position = UDim2.new(1, -35, 0, 10)
    closeBtn.BackgroundColor3 = COLORS.Primary
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = COLORS.Text
    closeBtn.TextScaled = true
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 5)
    closeCorner.Parent = closeBtn
    closeBtn.Parent = main

    closeBtn.MouseEnter:Connect(function()
        TweenService:Create(closeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    closeBtn.MouseLeave:Connect(function()
        TweenService:Create(closeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    return {
        MainFrame = main,
        KeyValue = keyValue,
        ChangeBtn = changeBtn,
        ResetBtn = resetBtn,
        ActivateBtn = activateBtn,
        MetalToggleBtn = metalToggleBtn,
        RealJumpToggleBtn = realJumpToggleBtn,
        CooldownLabel = cooldownLabel,
        CooldownValue = cooldownValue,
        CooldownBtns = {btnMinus, btnPlus},
        RealJumpCooldownLabel = realJumpCooldownLabel,
        RealJumpCooldownValue = realJumpCooldownValue,
        RealJumpCooldownBtns = {realJumpBtnMinus, realJumpBtnPlus},
        BoxToggleBtn = boxToggleBtn,
        BoxColorBtn = boxColorBtn,
        BoxColorFrame = boxColorFrame,
        TextToggleBtn = textToggleBtn,
        TextColorBtn = textColorBtn,
        TextColorFrame = textColorFrame,
        DistToggleBtn = distToggleBtn,
        CloseBtn = closeBtn,
        ThemeDropdown = themeDropdown,
        ThemeOptions = themeOptions,
    }
end

-- ==================== CONEXÕES DA UI ====================
local function setupUIEvents()
    uiElements.ChangeBtn.MouseButton1Click:Connect(startKeyChange)
    uiElements.ResetBtn.MouseButton1Click:Connect(function()
        selectedKey = nil
        updateKeyDisplay()
    end)
    uiElements.ActivateBtn.MouseButton1Click:Connect(sendExtraHit)

    uiElements.MetalToggleBtn.MouseButton1Click:Connect(function()
        toggleMetalSkin()
    end)

    uiElements.RealJumpToggleBtn.MouseButton1Click:Connect(function()
        CONFIG.RealJump.Enabled = not CONFIG.RealJump.Enabled
        uiElements.RealJumpToggleBtn.Text = CONFIG.RealJump.Enabled and "ON" or "OFF"
        uiElements.RealJumpToggleBtn.BackgroundColor3 = CONFIG.RealJump.Enabled and COLORS.Accent or COLORS.Surface
    end)

    uiElements.BoxToggleBtn.MouseButton1Click:Connect(function()
        CONFIG.ESP.Box.Enabled = not CONFIG.ESP.Box.Enabled
        uiElements.BoxToggleBtn.Text = CONFIG.ESP.Box.Enabled and "ON" or "OFF"
        uiElements.BoxToggleBtn.BackgroundColor3 = CONFIG.ESP.Box.Enabled and COLORS.Accent or COLORS.Surface
        if CONFIG.ESP.Box.Enabled then
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player then createBoxESP(plr) end
            end
        else
            for _, highlight in pairs(espBoxHighlights) do
                pcall(function() highlight:Destroy() end)
            end
            espBoxHighlights = {}
        end
    end)

    local boxColors = {
        Color3.fromRGB(255, 50, 50),
        Color3.fromRGB(50, 255, 50),
        Color3.fromRGB(50, 50, 255),
        Color3.fromRGB(255, 255, 50),
        Color3.fromRGB(150, 0, 255),
    }
    local boxColorIndex = 1
    for i, color in ipairs(boxColors) do
        if color == CONFIG.ESP.Box.Color then boxColorIndex = i; break end
    end
    uiElements.BoxColorBtn.MouseButton1Click:Connect(function()
        boxColorIndex = boxColorIndex % #boxColors + 1
        CONFIG.ESP.Box.Color = boxColors[boxColorIndex]
        uiElements.BoxColorFrame.BackgroundColor3 = CONFIG.ESP.Box.Color
        updateESPColors()
    end)

    uiElements.TextToggleBtn.MouseButton1Click:Connect(function()
        CONFIG.ESP.Text.Enabled = not CONFIG.ESP.Text.Enabled
        uiElements.TextToggleBtn.Text = CONFIG.ESP.Text.Enabled and "ON" or "OFF"
        uiElements.TextToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.Enabled and COLORS.Accent or COLORS.Surface
        if CONFIG.ESP.Text.Enabled then
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player then createTextESP(plr) end
            end
        else
            for _, data in pairs(espTextLabels) do
                if data.billboard then data.billboard.Enabled = false end
            end
        end
    end)

    local textColors = {
        Color3.fromRGB(255, 255, 255),
        Color3.fromRGB(255, 50, 50),
        Color3.fromRGB(50, 255, 50),
        Color3.fromRGB(50, 50, 255),
        Color3.fromRGB(150, 0, 255),
    }
    local textColorIndex = 1
    for i, color in ipairs(textColors) do
        if color == CONFIG.ESP.Text.Color then textColorIndex = i; break end
    end
    uiElements.TextColorBtn.MouseButton1Click:Connect(function()
        textColorIndex = textColorIndex % #textColors + 1
        CONFIG.ESP.Text.Color = textColors[textColorIndex]
        uiElements.TextColorFrame.BackgroundColor3 = CONFIG.ESP.Text.Color
        updateESPColors()
    end)

    uiElements.DistToggleBtn.MouseButton1Click:Connect(function()
        CONFIG.ESP.Text.ShowDistance = not CONFIG.ESP.Text.ShowDistance
        uiElements.DistToggleBtn.Text = CONFIG.ESP.Text.ShowDistance and "ON" or "OFF"
        uiElements.DistToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.ShowDistance and COLORS.Accent or COLORS.Surface
    end)

    uiElements.CloseBtn.MouseButton1Click:Connect(function()
        uiOpen = false
        uiElements.MainFrame.Visible = false
    end)

    -- Arrastar a interface
    local dragging = false
    local dragInput, dragStart, startPos
    uiElements.MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = uiElements.MainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    uiElements.MainFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            uiElements.MainFrame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Criar UI
uiElements = buildUI()
setupUIEvents()
updateKeyDisplay()

-- Input global
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Tecla P para toggle
    if input.KeyCode == CONFIG.ToggleKey then
        uiOpen = not uiOpen
        if uiElements.MainFrame then uiElements.MainFrame.Visible = uiOpen end
        return
    end
    
    -- Prioridade: modo de troca do extra hit
    if keyChangeModeExtra then
        return
    end
    
    -- Metal Skin (tecla R)
    if input.KeyCode == CONFIG.MetalSkin.Key then
        toggleMetalSkin()
        return
    end
    
    -- Extra hit
    if selectedKey then
        local match = false
        if typeof(selectedKey) == "EnumItem" then
            if selectedKey.EnumType == Enum.KeyCode then
                match = (input.KeyCode == selectedKey)
            elseif selectedKey.EnumType == Enum.UserInputType then
                match = (input.UserInputType == selectedKey)
            end
        end
        if match then sendExtraHit() end
    end
    
    -- Real Jump (tecla Alt)
    if CONFIG.RealJump.Enabled and (input.KeyCode == Enum.KeyCode.LeftAlt or input.KeyCode == Enum.KeyCode.RightAlt) then
        doRealJump()
    end
end)

-- Respawn do próprio jogador
player.CharacterAdded:Connect(function()
    task.wait(1)
    if not gui or not gui.Parent then
        gui = Instance.new("ScreenGui")
        gui.Name = "ExtraHitGUI"
        gui.Parent = player:WaitForChild("PlayerGui")
        gui.ResetOnSpawn = false
        uiElements = buildUI()
        setupUIEvents()
        updateKeyDisplay()
        applyTheme(currentTheme)
    else
        updateKeyDisplay()
    end
end)
