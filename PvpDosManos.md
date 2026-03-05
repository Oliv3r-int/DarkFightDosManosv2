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
    Preto = {
        Background = Color3.fromRGB(0, 0, 0),
        Surface = Color3.fromRGB(20, 20, 20),
        Primary = Color3.fromRGB(80, 80, 80),
        PrimaryLight = Color3.fromRGB(120, 120, 120),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(150, 150, 150),
    },
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

local currentTheme = "Preto"  -- Mudei para Preto como padrão (você pode alterar)
local COLORS = THEMES[currentTheme]

-- ==================== CONFIGURAÇÕES ====================
local CONFIG = {
    ToggleKey = Enum.KeyCode.P,
    
    ExtraHit = {
        Event = ReplicatedStorage:WaitForChild("Events"):WaitForChild("Punch"),
        Args = {0, 0.1, 1},
        Cooldown = 0,
    },
    
    MetalSkin = {
        Enabled = false,
        Key = Enum.KeyCode.R,
        Event = ReplicatedStorage:WaitForChild("Events"):WaitForChild("Transform"),
        Cooldown = 2,
        LastUse = 0,
    },
    
    RealJump = {
        Enabled = false,
        Key = Enum.KeyCode.LeftAlt,
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

-- ==================== VARIÁVEIS ====================
local uiOpen = true
local selectedKey = nil
local keyChangeMode = false
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
    if not selectedKey then return end
    local now = tick()
    if now - lastExtraHitTime < CONFIG.ExtraHit.Cooldown then return end
    lastExtraHitTime = now
    if not player.Character or not player.Character:FindFirstChild("Humanoid") then return end
    pcall(function()
        CONFIG.ExtraHit.Event:FireServer(unpack(CONFIG.ExtraHit.Args))
    end)
end

local function toggleMetalSkin()
    local now = tick()
    if now - CONFIG.MetalSkin.LastUse < CONFIG.MetalSkin.Cooldown then return end
    CONFIG.MetalSkin.LastUse = now
    CONFIG.MetalSkin.Enabled = not CONFIG.MetalSkin.Enabled
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

-- ==================== FUNÇÃO DE TROCA DE TECLA ====================
local function startKeyChange()
    if keyChangeMode then return end
    keyChangeMode = true
    uiElements.KeyValue.Text = "Pressione uma tecla..."
    uiElements.KeyValue.TextColor3 = COLORS.Accent

    local connection
    connection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == CONFIG.ToggleKey then return end
        if not CONFIG.AllowMouseButtons and input.UserInputType ~= Enum.UserInputType.Keyboard then return end

        if input.UserInputType == Enum.UserInputType.Keyboard then
            selectedKey = input.KeyCode
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 or
               input.UserInputType == Enum.UserInputType.MouseButton2 or
               input.UserInputType == Enum.UserInputType.MouseButton3 or
               input.UserInputType == Enum.UserInputType.MouseButton4 or
               input.UserInputType == Enum.UserInputType.MouseButton5 then
            selectedKey = input.UserInputType
        else
            return
        end

        updateKeyDisplay()
        keyChangeMode = false
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
    
    for i, tabBtn in ipairs(uiElements.TabButtons) do
        tabBtn.BackgroundColor3 = COLORS.Surface
        tabBtn.TextColor3 = COLORS.TextDim
    end
    uiElements.TabButtons[uiElements.CurrentTab].BackgroundColor3 = COLORS.Primary
    uiElements.TabButtons[uiElements.CurrentTab].TextColor3 = COLORS.Text
    
    for _, page in ipairs({uiElements.Page1, uiElements.Page2, uiElements.Page3}) do
        page.BackgroundColor3 = COLORS.Background
        for _, child in ipairs(page:GetChildren()) do
            if child:IsA("Frame") and child.Name == "Line" then
                child.BackgroundColor3 = COLORS.Primary
            end
        end
    end
    
    -- Página 1 (Temas)
    if uiElements.ThemeDropdown then
        uiElements.ThemeDropdown.BackgroundColor3 = COLORS.Primary
        uiElements.ThemeDropdown.TextColor3 = COLORS.Text
        uiElements.ThemeDropdown.Text = currentTheme
    end
    if uiElements.ThemeOptions then
        for _, btn in ipairs(uiElements.ThemeOptions) do
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
        end
    end
    
    -- Página 2 (Extra Hit + ESP)
    if uiElements.ExtraCard then
        uiElements.ExtraCard.BackgroundColor3 = COLORS.Surface
        if uiElements.ExtraTitle then uiElements.ExtraTitle.TextColor3 = COLORS.Primary end
    end
    if uiElements.ESPCard then
        uiElements.ESPCard.BackgroundColor3 = COLORS.Surface
        if uiElements.ESPTitle then uiElements.ESPTitle.TextColor3 = COLORS.Primary end
    end
    if uiElements.KeyValue then uiElements.KeyValue.TextColor3 = COLORS.Text end
    if uiElements.ChangeBtn then uiElements.ChangeBtn.BackgroundColor3 = COLORS.Primary end
    if uiElements.ResetBtn then
        uiElements.ResetBtn.BorderColor3 = COLORS.Primary
        uiElements.ResetBtn.BackgroundColor3 = COLORS.Surface
        uiElements.ResetBtn.TextColor3 = COLORS.Text
    end
    if uiElements.ActivateBtn then uiElements.ActivateBtn.BackgroundColor3 = COLORS.Primary end
    if uiElements.CooldownLabel then uiElements.CooldownLabel.TextColor3 = COLORS.TextDim end
    if uiElements.CooldownValue then uiElements.CooldownValue.TextColor3 = COLORS.Text end
    if uiElements.CooldownBtns then
        for _, btn in ipairs(uiElements.CooldownBtns) do
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
        end
    end
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
    if uiElements.BoxColorFrame then uiElements.BoxColorFrame.BackgroundColor3 = CONFIG.ESP.Box.Color end
    if uiElements.TextColorFrame then uiElements.TextColorFrame.BackgroundColor3 = CONFIG.ESP.Text.Color end
    
    -- Página 3 (Jump + Metal)
    if uiElements.MetalCard then
        uiElements.MetalCard.BackgroundColor3 = COLORS.Surface
        if uiElements.MetalTitle then uiElements.MetalTitle.TextColor3 = COLORS.Primary end
        if uiElements.MetalInfo then uiElements.MetalInfo.TextColor3 = COLORS.TextDim end
    end
    if uiElements.RealJumpCard then
        uiElements.RealJumpCard.BackgroundColor3 = COLORS.Surface
        if uiElements.RealJumpTitle then uiElements.RealJumpTitle.TextColor3 = COLORS.Primary end
        if uiElements.RealJumpInfo then uiElements.RealJumpInfo.TextColor3 = COLORS.TextDim end
    end
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
    if uiElements.RealJumpCooldownLabel then uiElements.RealJumpCooldownLabel.TextColor3 = COLORS.TextDim end
    if uiElements.RealJumpCooldownValue then uiElements.RealJumpCooldownValue.TextColor3 = COLORS.Text end
    if uiElements.RealJumpCooldownBtns then
        for _, btn in ipairs(uiElements.RealJumpCooldownBtns) do
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
        end
    end
    
    if uiElements.CloseBtn then uiElements.CloseBtn.BackgroundColor3 = COLORS.Primary end
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
    main.Size = UDim2.new(0, 500, 0, 560)  -- altura ajustada para acomodar tudo
    main.Position = UDim2.new(0.5, -250, 0.5, -280)
    main.BackgroundColor3 = COLORS.Background
    main.BorderSizePixel = 0
    main.Visible = uiOpen
    main.Parent = gui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = main

    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 20, 1, 20)
    shadow.Position = UDim2.new(0, -10, 0, -10)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6015897843"
    shadow.ImageColor3 = Color3.new(0,0,0)
    shadow.ImageTransparency = 0.5
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(10,10,10,10)
    shadow.Parent = main

    local title = Instance.new("TextLabel")
    title.Name = "TitleLabel"
    title.Size = UDim2.new(0.5, -20, 0, 30)
    title.Position = UDim2.new(0, 12, 0, 8)
    title.BackgroundTransparency = 1
    title.Text = "# SCRIPT"
    title.TextColor3 = COLORS.Primary
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.TextSize = 20
    title.Font = Enum.Font.GothamBold
    title.Parent = main

    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseBtn"
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 8)
    closeBtn.BackgroundColor3 = COLORS.Primary
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = COLORS.Text
    closeBtn.TextSize = 20
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 6)
    closeCorner.Parent = closeBtn
    closeBtn.Parent = main

    closeBtn.MouseEnter:Connect(function()
        TweenService:Create(closeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    closeBtn.MouseLeave:Connect(function()
        TweenService:Create(closeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    -- Barra de abas
    local tabBar = Instance.new("Frame")
    tabBar.Name = "TabBar"
    tabBar.Size = UDim2.new(1, -24, 0, 38)
    tabBar.Position = UDim2.new(0, 12, 0, 42)
    tabBar.BackgroundTransparency = 1
    tabBar.Parent = main

    local tabButtons = {}
    local tabNames = {"✦ Temas", "⛇ Extra Hit + ESP", "♚ INF + Metal"}
    for i, name in ipairs(tabNames) do
        local btn = Instance.new("TextButton")
        btn.Name = "TabBtn"..i
        btn.Size = UDim2.new(0.33, -4, 1, -4)
        btn.Position = UDim2.new((i-1)*0.33, i*2, 0, 0)
        btn.BackgroundColor3 = COLORS.Surface
        btn.Text = name
        btn.TextColor3 = COLORS.TextDim
        btn.TextSize = 15
        btn.Font = Enum.Font.GothamBold
        btn.BorderSizePixel = 0
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        btn.Parent = tabBar
        table.insert(tabButtons, btn)
    end

    -- Container das páginas
    local pageContainer = Instance.new("Frame")
    pageContainer.Name = "PageContainer"
    pageContainer.Size = UDim2.new(1, -24, 1, -120)
    pageContainer.Position = UDim2.new(0, 12, 0, 90)
    pageContainer.BackgroundTransparency = 1
    pageContainer.Parent = main

    local page1 = Instance.new("Frame")
    page1.Name = "Page1"
    page1.Size = UDim2.new(1, 0, 1, 0)
    page1.BackgroundColor3 = COLORS.Background
    page1.BorderSizePixel = 0
    page1.Visible = true
    page1.Parent = pageContainer

    local page2 = Instance.new("Frame")
    page2.Name = "Page2"
    page2.Size = UDim2.new(1, 0, 1, 0)
    page2.BackgroundColor3 = COLORS.Background
    page2.BorderSizePixel = 0
    page2.Visible = false
    page2.Parent = pageContainer

    local page3 = Instance.new("Frame")
    page3.Name = "Page3"
    page3.Size = UDim2.new(1, 0, 1, 0)
    page3.BackgroundColor3 = COLORS.Background
    page3.BorderSizePixel = 0
    page3.Visible = false
    page3.Parent = pageContainer

    -- ===== PÁGINA 1: TEMA =====
    local line1 = Instance.new("Frame")
    line1.Name = "Line"
    line1.Size = UDim2.new(0.9, 0, 0, 1)
    line1.Position = UDim2.new(0.05, 0, 0, 5)
    line1.BackgroundColor3 = COLORS.Primary
    line1.BorderSizePixel = 0
    line1.Parent = page1

    local themeLabel = Instance.new("TextLabel")
    themeLabel.Size = UDim2.new(1, -20, 0, 28)
    themeLabel.Position = UDim2.new(0, 10, 0, 15)
    themeLabel.BackgroundTransparency = 1
    themeLabel.Text = "Temas"
    themeLabel.TextColor3 = COLORS.Text
    themeLabel.TextXAlignment = Enum.TextXAlignment.Left
    themeLabel.TextSize = 18
    themeLabel.Font = Enum.Font.GothamBold
    themeLabel.Parent = page1

    local themeDropdown = Instance.new("TextButton")
    themeDropdown.Name = "ThemeDropdown"
    themeDropdown.Size = UDim2.new(0, 160, 0, 35)
    themeDropdown.Position = UDim2.new(0, 10, 0, 50)
    themeDropdown.BackgroundColor3 = COLORS.Primary
    themeDropdown.Text = currentTheme
    themeDropdown.TextColor3 = COLORS.Text
    themeDropdown.TextSize = 15
    themeDropdown.Font = Enum.Font.GothamBold
    themeDropdown.BorderSizePixel = 0
    local dropdownCorner = Instance.new("UICorner")
    dropdownCorner.CornerRadius = UDim.new(0, 6)
    dropdownCorner.Parent = themeDropdown
    themeDropdown.Parent = page1

    local themeOptions = {}
    local options = {"Vinho", "Azul", "Verde", "Roxo"}
    for i, opt in ipairs(options) do
        local btn = Instance.new("TextButton")
        btn.Name = "ThemeBtn_"..opt
        btn.Size = UDim2.new(0, 160, 0, 30)
        btn.Position = UDim2.new(0, 10, 0, 85 + (i-1)*32)
        btn.BackgroundColor3 = COLORS.Primary
        btn.Text = opt
        btn.TextColor3 = COLORS.Text
        btn.TextSize = 14
        btn.Font = Enum.Font.Gotham
        btn.BorderSizePixel = 0
        btn.Visible = false
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = btn
        btn.Parent = page1
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

    -- ===== PÁGINA 2: EXTRA HIT + ESP =====
    -- Card Extra Hit
    local extraCard = Instance.new("Frame")
    extraCard.Name = "ExtraCard"
    extraCard.Size = UDim2.new(1, 0, 0, 160)
    extraCard.Position = UDim2.new(0, 0, 0, 0)
    extraCard.BackgroundColor3 = COLORS.Surface
    extraCard.BorderSizePixel = 0
    local extraCorner = Instance.new("UICorner")
    extraCorner.CornerRadius = UDim.new(0, 8)
    extraCorner.Parent = extraCard
    extraCard.Parent = page2

    -- Sombra do card
    local extraShadow = Instance.new("ImageLabel")
    extraShadow.Size = UDim2.new(1, 6, 1, 6)
    extraShadow.Position = UDim2.new(0, -3, 0, -3)
    extraShadow.BackgroundTransparency = 1
    extraShadow.Image = "rbxassetid://6015897843"
    extraShadow.ImageColor3 = Color3.new(0,0,0)
    extraShadow.ImageTransparency = 0.7
    extraShadow.ScaleType = Enum.ScaleType.Slice
    extraShadow.SliceCenter = Rect.new(10,10,10,10)
    extraShadow.Parent = extraCard

    local extraTitle = Instance.new("TextLabel")
    extraTitle.Name = "ExtraTitle"
    extraTitle.Size = UDim2.new(1, -12, 0, 24)
    extraTitle.Position = UDim2.new(0, 6, 0, 4)
    extraTitle.BackgroundTransparency = 1
    extraTitle.Text = "EXTRA HIT"
    extraTitle.TextColor3 = COLORS.Primary
    extraTitle.TextXAlignment = Enum.TextXAlignment.Left
    extraTitle.TextSize = 17
    extraTitle.Font = Enum.Font.GothamBold
    extraTitle.Parent = extraCard

    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(0.5, -5, 0, 20)
    keyLabel.Position = UDim2.new(0, 8, 0, 32)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA CONFIGURADA"
    keyLabel.TextColor3 = COLORS.TextDim
    keyLabel.TextXAlignment = Enum.TextXAlignment.Left
    keyLabel.TextSize = 13
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = extraCard

    local keyValue = Instance.new("TextLabel")
    keyValue.Name = "KeyValue"
    keyValue.Size = UDim2.new(0.5, -5, 0, 22)
    keyValue.Position = UDim2.new(0, 8, 0, 52)
    keyValue.BackgroundTransparency = 1
    keyValue.Text = "NENHUMA"
    keyValue.TextColor3 = COLORS.Text
    keyValue.TextXAlignment = Enum.TextXAlignment.Left
    keyValue.TextSize = 14
    keyValue.Font = Enum.Font.GothamBold
    keyValue.Parent = extraCard

    local btnRow = Instance.new("Frame")
    btnRow.Name = "BtnRow"
    btnRow.Size = UDim2.new(1, -16, 0, 32)
    btnRow.Position = UDim2.new(0, 8, 0, 80)
    btnRow.BackgroundTransparency = 1
    btnRow.Parent = extraCard

    local changeBtn = Instance.new("TextButton")
    changeBtn.Name = "ChangeBtn"
    changeBtn.Size = UDim2.new(0, 75, 0, 28)
    changeBtn.Position = UDim2.new(0, 0, 0, 0)
    changeBtn.BackgroundColor3 = COLORS.Primary
    changeBtn.Text = "TROCAR"
    changeBtn.TextColor3 = COLORS.Text
    changeBtn.TextSize = 12
    changeBtn.Font = Enum.Font.GothamBold
    changeBtn.BorderSizePixel = 0
    local changeCorner = Instance.new("UICorner")
    changeCorner.CornerRadius = UDim.new(0, 4)
    changeCorner.Parent = changeBtn
    changeBtn.Parent = btnRow

    changeBtn.MouseEnter:Connect(function()
        TweenService:Create(changeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    changeBtn.MouseLeave:Connect(function()
        TweenService:Create(changeBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    local resetBtn = Instance.new("TextButton")
    resetBtn.Name = "ResetBtn"
    resetBtn.Size = UDim2.new(0, 75, 0, 28)
    resetBtn.Position = UDim2.new(0, 83, 0, 0)
    resetBtn.BackgroundColor3 = COLORS.Surface
    resetBtn.Text = "RESETAR"
    resetBtn.TextColor3 = COLORS.Text
    resetBtn.TextSize = 12
    resetBtn.Font = Enum.Font.GothamBold
    resetBtn.BorderSizePixel = 1
    resetBtn.BorderColor3 = COLORS.Primary
    local resetCorner = Instance.new("UICorner")
    resetCorner.CornerRadius = UDim.new(0, 4)
    resetCorner.Parent = resetBtn
    resetBtn.Parent = btnRow

    resetBtn.MouseEnter:Connect(function()
        TweenService:Create(resetBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight, TextColor3 = COLORS.Text}):Play()
    end)
    resetBtn.MouseLeave:Connect(function()
        TweenService:Create(resetBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Surface, TextColor3 = COLORS.Text}):Play()
    end)

    local activateBtn = Instance.new("TextButton")
    activateBtn.Name = "ActivateBtn"
    activateBtn.Size = UDim2.new(0, 75, 0, 28)
    activateBtn.Position = UDim2.new(0, 166, 0, 0)
    activateBtn.BackgroundColor3 = COLORS.Primary
    activateBtn.Text = "⚡ ATIVAR"
    activateBtn.TextColor3 = COLORS.Text
    activateBtn.TextSize = 12
    activateBtn.Font = Enum.Font.GothamBold
    activateBtn.BorderSizePixel = 0
    local activateCorner = Instance.new("UICorner")
    activateCorner.CornerRadius = UDim.new(0, 4)
    activateCorner.Parent = activateBtn
    activateBtn.Parent = btnRow

    activateBtn.MouseEnter:Connect(function()
        TweenService:Create(activateBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.PrimaryLight}):Play()
    end)
    activateBtn.MouseLeave:Connect(function()
        TweenService:Create(activateBtn, TweenInfo.new(0.2), {BackgroundColor3 = COLORS.Primary}):Play()
    end)

    -- Cooldown Extra Hit
    local cooldownArea = Instance.new("Frame")
    cooldownArea.Name = "CooldownArea"
    cooldownArea.Size = UDim2.new(1, -16, 0, 32)
    cooldownArea.Position = UDim2.new(0, 8, 0, 118)
    cooldownArea.BackgroundTransparency = 1
    cooldownArea.Parent = extraCard

    local cooldownLabel = Instance.new("TextLabel")
    cooldownLabel.Name = "CooldownLabel"
    cooldownLabel.Size = UDim2.new(0, 75, 0, 28)
    cooldownLabel.Position = UDim2.new(0, 0, 0, 0)
    cooldownLabel.BackgroundTransparency = 1
    cooldownLabel.Text = "Cooldown:"
    cooldownLabel.TextColor3 = COLORS.TextDim
    cooldownLabel.TextSize = 13
    cooldownLabel.Font = Enum.Font.Gotham
    cooldownLabel.Parent = cooldownArea

    local cooldownValue = Instance.new("TextLabel")
    cooldownValue.Name = "CooldownValue"
    cooldownValue.Size = UDim2.new(0, 45, 0, 28)
    cooldownValue.Position = UDim2.new(0, 210, 0, 0)
    cooldownValue.BackgroundTransparency = 1
    cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    cooldownValue.TextColor3 = COLORS.Text
    cooldownValue.TextSize = 14
    cooldownValue.Font = Enum.Font.GothamBold
    cooldownValue.Parent = cooldownArea

    local btnMinus = Instance.new("TextButton")
    btnMinus.Size = UDim2.new(0, 28, 0, 28)
    btnMinus.Position = UDim2.new(0, 180, 0, 0)
    btnMinus.BackgroundColor3 = COLORS.Primary
    btnMinus.Text = "-"
    btnMinus.TextColor3 = COLORS.Text
    btnMinus.TextSize = 16
    btnMinus.Font = Enum.Font.GothamBold
    btnMinus.BorderSizePixel = 0
    local minusCorner = Instance.new("UICorner")
    minusCorner.CornerRadius = UDim.new(0, 4)
    minusCorner.Parent = btnMinus
    btnMinus.Parent = cooldownArea

    btnMinus.MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.ExtraHit.Cooldown - 0.1)
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)

    local btnPlus = Instance.new("TextButton")
    btnPlus.Size = UDim2.new(0, 28, 0, 28)
    btnPlus.Position = UDim2.new(0, 255, 0, 0)
    btnPlus.BackgroundColor3 = COLORS.Primary
    btnPlus.Text = "+"
    btnPlus.TextColor3 = COLORS.Text
    btnPlus.TextSize = 16
    btnPlus.Font = Enum.Font.GothamBold
    btnPlus.BorderSizePixel = 0
    local plusCorner = Instance.new("UICorner")
    plusCorner.CornerRadius = UDim.new(0, 4)
    plusCorner.Parent = btnPlus
    btnPlus.Parent = cooldownArea

    btnPlus.MouseButton1Click:Connect(function()
        local new = CONFIG.ExtraHit.Cooldown + 0.1
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        cooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)

    -- Card ESP
    local espCard = Instance.new("Frame")
    espCard.Name = "ESPCard"
    espCard.Size = UDim2.new(1, 0, 0, 170)
    espCard.Position = UDim2.new(0, 0, 0, 170)
    espCard.BackgroundColor3 = COLORS.Surface
    espCard.BorderSizePixel = 0
    local espCorner = Instance.new("UICorner")
    espCorner.CornerRadius = UDim.new(0, 8)
    espCorner.Parent = espCard
    espCard.Parent = page2

    local espShadow = Instance.new("ImageLabel")
    espShadow.Size = UDim2.new(1, 6, 1, 6)
    espShadow.Position = UDim2.new(0, -3, 0, -3)
    espShadow.BackgroundTransparency = 1
    espShadow.Image = "rbxassetid://6015897843"
    espShadow.ImageColor3 = Color3.new(0,0,0)
    espShadow.ImageTransparency = 0.7
    espShadow.ScaleType = Enum.ScaleType.Slice
    espShadow.SliceCenter = Rect.new(10,10,10,10)
    espShadow.Parent = espCard

    local espTitle = Instance.new("TextLabel")
    espTitle.Name = "ESPTitle"
    espTitle.Size = UDim2.new(1, -12, 0, 24)
    espTitle.Position = UDim2.new(0, 6, 0, 4)
    espTitle.BackgroundTransparency = 1
    espTitle.Text = "ESP"
    espTitle.TextColor3 = COLORS.Primary
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.TextSize = 17
    espTitle.Font = Enum.Font.GothamBold
    espTitle.Parent = espCard

    -- Linha Caixa
    local boxLabel = Instance.new("TextLabel")
    boxLabel.Size = UDim2.new(0.5, -5, 0, 22)
    boxLabel.Position = UDim2.new(0, 8, 0, 32)
    boxLabel.BackgroundTransparency = 1
    boxLabel.Text = "Caixa colorida"
    boxLabel.TextColor3 = COLORS.TextDim
    boxLabel.TextXAlignment = Enum.TextXAlignment.Left
    boxLabel.TextSize = 13
    boxLabel.Font = Enum.Font.Gotham
    boxLabel.Parent = espCard

    local boxToggleBtn = Instance.new("TextButton")
    boxToggleBtn.Name = "BoxToggleBtn"
    boxToggleBtn.Size = UDim2.new(0, 45, 0, 24)
    boxToggleBtn.Position = UDim2.new(1, -95, 0, 30)
    boxToggleBtn.BackgroundColor3 = CONFIG.ESP.Box.Enabled and COLORS.Accent or COLORS.Surface
    boxToggleBtn.Text = CONFIG.ESP.Box.Enabled and "ON" or "OFF"
    boxToggleBtn.TextColor3 = COLORS.Text
    boxToggleBtn.TextSize = 12
    boxToggleBtn.Font = Enum.Font.GothamBold
    boxToggleBtn.BorderSizePixel = 1
    boxToggleBtn.BorderColor3 = COLORS.Primary
    local boxToggleCorner = Instance.new("UICorner")
    boxToggleCorner.CornerRadius = UDim.new(0, 4)
    boxToggleCorner.Parent = boxToggleBtn
    boxToggleBtn.Parent = espCard

    local boxColorFrame = Instance.new("Frame")
    boxColorFrame.Name = "BoxColorFrame"
    boxColorFrame.Size = UDim2.new(0, 22, 0, 22)
    boxColorFrame.Position = UDim2.new(1, -45, 0, 31)
    boxColorFrame.BackgroundColor3 = CONFIG.ESP.Box.Color
    boxColorFrame.BorderSizePixel = 0
    local boxColorCorner = Instance.new("UICorner")
    boxColorCorner.CornerRadius = UDim.new(0, 4)
    boxColorCorner.Parent = boxColorFrame
    boxColorFrame.Parent = espCard

    local boxColorBtn = Instance.new("TextButton")
    boxColorBtn.Name = "BoxColorBtn"
    boxColorBtn.Size = UDim2.new(0, 22, 0, 22)
    boxColorBtn.Position = UDim2.new(1, -45, 0, 31)
    boxColorBtn.BackgroundTransparency = 1
    boxColorBtn.Text = "✎"
    boxColorBtn.TextColor3 = COLORS.Text
    boxColorBtn.TextSize = 16
    boxColorBtn.Font = Enum.Font.GothamBold
    boxColorBtn.BorderSizePixel = 0
    boxColorBtn.Parent = espCard

    -- Linha Texto
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0.5, -5, 0, 22)
    textLabel.Position = UDim2.new(0, 8, 0, 62)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = "Texto (nome)"
    textLabel.TextColor3 = COLORS.TextDim
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextSize = 13
    textLabel.Font = Enum.Font.Gotham
    textLabel.Parent = espCard

    local textToggleBtn = Instance.new("TextButton")
    textToggleBtn.Name = "TextToggleBtn"
    textToggleBtn.Size = UDim2.new(0, 45, 0, 24)
    textToggleBtn.Position = UDim2.new(1, -95, 0, 60)
    textToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.Enabled and COLORS.Accent or COLORS.Surface
    textToggleBtn.Text = CONFIG.ESP.Text.Enabled and "ON" or "OFF"
    textToggleBtn.TextColor3 = COLORS.Text
    textToggleBtn.TextSize = 12
    textToggleBtn.Font = Enum.Font.GothamBold
    textToggleBtn.BorderSizePixel = 1
    textToggleBtn.BorderColor3 = COLORS.Primary
    local textToggleCorner = Instance.new("UICorner")
    textToggleCorner.CornerRadius = UDim.new(0, 4)
    textToggleCorner.Parent = textToggleBtn
    textToggleBtn.Parent = espCard

    local textColorFrame = Instance.new("Frame")
    textColorFrame.Name = "TextColorFrame"
    textColorFrame.Size = UDim2.new(0, 22, 0, 22)
    textColorFrame.Position = UDim2.new(1, -45, 0, 61)
    textColorFrame.BackgroundColor3 = CONFIG.ESP.Text.Color
    textColorFrame.BorderSizePixel = 0
    local textColorCorner = Instance.new("UICorner")
    textColorCorner.CornerRadius = UDim.new(0, 4)
    textColorCorner.Parent = textColorFrame
    textColorFrame.Parent = espCard

    local textColorBtn = Instance.new("TextButton")
    textColorBtn.Name = "TextColorBtn"
    textColorBtn.Size = UDim2.new(0, 22, 0, 22)
    textColorBtn.Position = UDim2.new(1, -45, 0, 61)
    textColorBtn.BackgroundTransparency = 1
    textColorBtn.Text = "✎"
    textColorBtn.TextColor3 = COLORS.Text
    textColorBtn.TextSize = 16
    textColorBtn.Font = Enum.Font.GothamBold
    textColorBtn.BorderSizePixel = 0
    textColorBtn.Parent = espCard

    -- Linha Distância
    local distLabel = Instance.new("TextLabel")
    distLabel.Size = UDim2.new(0.5, -5, 0, 22)
    distLabel.Position = UDim2.new(0, 8, 0, 92)
    distLabel.BackgroundTransparency = 1
    distLabel.Text = "Mostrar distância"
    distLabel.TextColor3 = COLORS.TextDim
    distLabel.TextXAlignment = Enum.TextXAlignment.Left
    distLabel.TextSize = 13
    distLabel.Font = Enum.Font.Gotham
    distLabel.Parent = espCard

    local distToggleBtn = Instance.new("TextButton")
    distToggleBtn.Name = "DistToggleBtn"
    distToggleBtn.Size = UDim2.new(0, 45, 0, 24)
    distToggleBtn.Position = UDim2.new(1, -55, 0, 90)
    distToggleBtn.BackgroundColor3 = CONFIG.ESP.Text.ShowDistance and COLORS.Accent or COLORS.Surface
    distToggleBtn.Text = CONFIG.ESP.Text.ShowDistance and "ON" or "OFF"
    distToggleBtn.TextColor3 = COLORS.Text
    distToggleBtn.TextSize = 12
    distToggleBtn.Font = Enum.Font.GothamBold
    distToggleBtn.BorderSizePixel = 1
    distToggleBtn.BorderColor3 = COLORS.Primary
    local distToggleCorner = Instance.new("UICorner")
    distToggleCorner.CornerRadius = UDim.new(0, 4)
    distToggleCorner.Parent = distToggleBtn
    distToggleBtn.Parent = espCard

    -- ===== PÁGINA 3: JUMP + METAL =====
    -- Card Metal Skin
    local metalCard = Instance.new("Frame")
    metalCard.Name = "MetalCard"
    metalCard.Size = UDim2.new(1, 0, 0, 80)
    metalCard.Position = UDim2.new(0, 0, 0, 0)
    metalCard.BackgroundColor3 = COLORS.Surface
    metalCard.BorderSizePixel = 0
    local metalCorner = Instance.new("UICorner")
    metalCorner.CornerRadius = UDim.new(0, 8)
    metalCorner.Parent = metalCard
    metalCard.Parent = page3

    local metalShadow = Instance.new("ImageLabel")
    metalShadow.Size = UDim2.new(1, 6, 1, 6)
    metalShadow.Position = UDim2.new(0, -3, 0, -3)
    metalShadow.BackgroundTransparency = 1
    metalShadow.Image = "rbxassetid://6015897843"
    metalShadow.ImageColor3 = Color3.new(0,0,0)
    metalShadow.ImageTransparency = 0.7
    metalShadow.ScaleType = Enum.ScaleType.Slice
    metalShadow.SliceCenter = Rect.new(10,10,10,10)
    metalShadow.Parent = metalCard

    local metalTitle = Instance.new("TextLabel")
    metalTitle.Name = "MetalTitle"
    metalTitle.Size = UDim2.new(1, -12, 0, 24)
    metalTitle.Position = UDim2.new(0, 6, 0, 4)
    metalTitle.BackgroundTransparency = 1
    metalTitle.Text = "⌨ METAL SKIN"
    metalTitle.TextColor3 = COLORS.Primary
    metalTitle.TextXAlignment = Enum.TextXAlignment.Left
    metalTitle.TextSize = 17
    metalTitle.Font = Enum.Font.GothamBold
    metalTitle.Parent = metalCard

    local metalInfo = Instance.new("TextLabel")
    metalInfo.Name = "MetalInfo"
    metalInfo.Size = UDim2.new(0.5, -5, 0, 20)
    metalInfo.Position = UDim2.new(0, 8, 0, 32)
    metalInfo.BackgroundTransparency = 1
    metalInfo.Text = "⌨ Tecla R (cooldown 2s)"
    metalInfo.TextColor3 = COLORS.TextDim
    metalInfo.TextXAlignment = Enum.TextXAlignment.Left
    metalInfo.TextSize = 12
    metalInfo.Font = Enum.Font.Gotham
    metalInfo.Parent = metalCard

    local metalToggleBtn = Instance.new("TextButton")
    metalToggleBtn.Name = "MetalToggleBtn"
    metalToggleBtn.Size = UDim2.new(0, 50, 0, 26)
    metalToggleBtn.Position = UDim2.new(1, -60, 0, 28)
    metalToggleBtn.BackgroundColor3 = CONFIG.MetalSkin.Enabled and COLORS.Accent or COLORS.Surface
    metalToggleBtn.Text = CONFIG.MetalSkin.Enabled and "ON" or "OFF"
    metalToggleBtn.TextColor3 = COLORS.Text
    metalToggleBtn.TextSize = 12
    metalToggleBtn.Font = Enum.Font.GothamBold
    metalToggleBtn.BorderSizePixel = 1
    metalToggleBtn.BorderColor3 = COLORS.Primary
    local metalToggleCorner = Instance.new("UICorner")
    metalToggleCorner.CornerRadius = UDim.new(0, 4)
    metalToggleCorner.Parent = metalToggleBtn
    metalToggleBtn.Parent = metalCard

    -- Card Real Jump
    local realJumpCard = Instance.new("Frame")
    realJumpCard.Name = "RealJumpCard"
    realJumpCard.Size = UDim2.new(1, 0, 0, 110)
    realJumpCard.Position = UDim2.new(0, 0, 0, 90)
    realJumpCard.BackgroundColor3 = COLORS.Surface
    realJumpCard.BorderSizePixel = 0
    local realJumpCorner = Instance.new("UICorner")
    realJumpCorner.CornerRadius = UDim.new(0, 8)
    realJumpCorner.Parent = realJumpCard
    realJumpCard.Parent = page3

    local realJumpShadow = Instance.new("ImageLabel")
    realJumpShadow.Size = UDim2.new(1, 6, 1, 6)
    realJumpShadow.Position = UDim2.new(0, -3, 0, -3)
    realJumpShadow.BackgroundTransparency = 1
    realJumpShadow.Image = "rbxassetid://6015897843"
    realJumpShadow.ImageColor3 = Color3.new(0,0,0)
    realJumpShadow.ImageTransparency = 0.7
    realJumpShadow.ScaleType = Enum.ScaleType.Slice
    realJumpShadow.SliceCenter = Rect.new(10,10,10,10)
    realJumpShadow.Parent = realJumpCard

    local realJumpTitle = Instance.new("TextLabel")
    realJumpTitle.Name = "RealJumpTitle"
    realJumpTitle.Size = UDim2.new(1, -12, 0, 24)
    realJumpTitle.Position = UDim2.new(0, 6, 0, 4)
    realJumpTitle.BackgroundTransparency = 1
    realJumpTitle.Text = "⌨ INF "
    realJumpTitle.TextColor3 = COLORS.Primary
    realJumpTitle.TextXAlignment = Enum.TextXAlignment.Left
    realJumpTitle.TextSize = 17
    realJumpTitle.Font = Enum.Font.GothamBold
    realJumpTitle.Parent = realJumpCard

    local realJumpInfo = Instance.new("TextLabel")
    realJumpInfo.Name = "RealJumpInfo"
    realJumpInfo.Size = UDim2.new(0.5, -5, 0, 20)
    realJumpInfo.Position = UDim2.new(0, 8, 0, 32)
    realJumpInfo.BackgroundTransparency = 1
    realJumpInfo.Text = "⌨ Tecla: ALT"
    realJumpInfo.TextColor3 = COLORS.TextDim
    realJumpInfo.TextXAlignment = Enum.TextXAlignment.Left
    realJumpInfo.TextSize = 12
    realJumpInfo.Font = Enum.Font.Gotham
    realJumpInfo.Parent = realJumpCard

    local realJumpToggleBtn = Instance.new("TextButton")
    realJumpToggleBtn.Name = "RealJumpToggleBtn"
    realJumpToggleBtn.Size = UDim2.new(0, 50, 0, 26)
    realJumpToggleBtn.Position = UDim2.new(1, -60, 0, 28)
    realJumpToggleBtn.BackgroundColor3 = CONFIG.RealJump.Enabled and COLORS.Accent or COLORS.Surface
    realJumpToggleBtn.Text = CONFIG.RealJump.Enabled and "ON" or "OFF"
    realJumpToggleBtn.TextColor3 = COLORS.Text
    realJumpToggleBtn.TextSize = 12
    realJumpToggleBtn.Font = Enum.Font.GothamBold
    realJumpToggleBtn.BorderSizePixel = 1
    realJumpToggleBtn.BorderColor3 = COLORS.Primary
    local realJumpToggleCorner = Instance.new("UICorner")
    realJumpToggleCorner.CornerRadius = UDim.new(0, 4)
    realJumpToggleCorner.Parent = realJumpToggleBtn
    realJumpToggleBtn.Parent = realJumpCard

    -- Cooldown Real Jump
    local realJumpCooldownArea = Instance.new("Frame")
    realJumpCooldownArea.Name = "RealJumpCooldownArea"
    realJumpCooldownArea.Size = UDim2.new(1, -16, 0, 32)
    realJumpCooldownArea.Position = UDim2.new(0, 8, 0, 65)
    realJumpCooldownArea.BackgroundTransparency = 1
    realJumpCooldownArea.Parent = realJumpCard

    local realJumpCooldownLabel = Instance.new("TextLabel")
    realJumpCooldownLabel.Name = "RealJumpCooldownLabel"
    realJumpCooldownLabel.Size = UDim2.new(0, 75, 0, 28)
    realJumpCooldownLabel.Position = UDim2.new(0, 0, 0, 0)
    realJumpCooldownLabel.BackgroundTransparency = 1
    realJumpCooldownLabel.Text = "Cooldown:"
    realJumpCooldownLabel.TextColor3 = COLORS.TextDim
    realJumpCooldownLabel.TextSize = 13
    realJumpCooldownLabel.Font = Enum.Font.Gotham
    realJumpCooldownLabel.Parent = realJumpCooldownArea

    local realJumpCooldownValue = Instance.new("TextLabel")
    realJumpCooldownValue.Name = "RealJumpCooldownValue"
    realJumpCooldownValue.Size = UDim2.new(0, 45, 0, 28)
    realJumpCooldownValue.Position = UDim2.new(0, 210, 0, 0)
    realJumpCooldownValue.BackgroundTransparency = 1
    realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    realJumpCooldownValue.TextColor3 = COLORS.Text
    realJumpCooldownValue.TextSize = 14
    realJumpCooldownValue.Font = Enum.Font.GothamBold
    realJumpCooldownValue.Parent = realJumpCooldownArea

    local realJumpBtnMinus = Instance.new("TextButton")
    realJumpBtnMinus.Size = UDim2.new(0, 28, 0, 28)
    realJumpBtnMinus.Position = UDim2.new(0, 180, 0, 0)
    realJumpBtnMinus.BackgroundColor3 = COLORS.Primary
    realJumpBtnMinus.Text = "-"
    realJumpBtnMinus.TextColor3 = COLORS.Text
    realJumpBtnMinus.TextSize = 16
    realJumpBtnMinus.Font = Enum.Font.GothamBold
    realJumpBtnMinus.BorderSizePixel = 0
    local realJumpMinusCorner = Instance.new("UICorner")
    realJumpMinusCorner.CornerRadius = UDim.new(0, 4)
    realJumpMinusCorner.Parent = realJumpBtnMinus
    realJumpBtnMinus.Parent = realJumpCooldownArea

    realJumpBtnMinus.MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.RealJump.Cooldown - 0.1)
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)

    local realJumpBtnPlus = Instance.new("TextButton")
    realJumpBtnPlus.Size = UDim2.new(0, 28, 0, 28)
    realJumpBtnPlus.Position = UDim2.new(0, 255, 0, 0)
    realJumpBtnPlus.BackgroundColor3 = COLORS.Primary
    realJumpBtnPlus.Text = "+"
    realJumpBtnPlus.TextColor3 = COLORS.Text
    realJumpBtnPlus.TextSize = 16
    realJumpBtnPlus.Font = Enum.Font.GothamBold
    realJumpBtnPlus.BorderSizePixel = 0
    local realJumpPlusCorner = Instance.new("UICorner")
    realJumpPlusCorner.CornerRadius = UDim.new(0, 4)
    realJumpPlusCorner.Parent = realJumpBtnPlus
    realJumpBtnPlus.Parent = realJumpCooldownArea

    realJumpBtnPlus.MouseButton1Click:Connect(function()
        local new = CONFIG.RealJump.Cooldown + 0.1
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        realJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)

    return {
        MainFrame = main,
        Page1 = page1,
        Page2 = page2,
        Page3 = page3,
        TabButtons = tabButtons,
        CurrentTab = 1,
        ThemeDropdown = themeDropdown,
        ThemeOptions = themeOptions,
        -- Página 2
        ExtraCard = extraCard,
        ExtraTitle = extraTitle,
        ESPCard = espCard,
        ESPTitle = espTitle,
        KeyValue = keyValue,
        ChangeBtn = changeBtn,
        ResetBtn = resetBtn,
        ActivateBtn = activateBtn,
        CooldownLabel = cooldownLabel,
        CooldownValue = cooldownValue,
        CooldownBtns = {btnMinus, btnPlus},
        BoxToggleBtn = boxToggleBtn,
        BoxColorBtn = boxColorBtn,
        BoxColorFrame = boxColorFrame,
        TextToggleBtn = textToggleBtn,
        TextColorBtn = textColorBtn,
        TextColorFrame = textColorFrame,
        DistToggleBtn = distToggleBtn,
        -- Página 3
        MetalCard = metalCard,
        MetalTitle = metalTitle,
        MetalInfo = metalInfo,
        MetalToggleBtn = metalToggleBtn,
        RealJumpCard = realJumpCard,
        RealJumpTitle = realJumpTitle,
        RealJumpInfo = realJumpInfo,
        RealJumpToggleBtn = realJumpToggleBtn,
        RealJumpCooldownLabel = realJumpCooldownLabel,
        RealJumpCooldownValue = realJumpCooldownValue,
        RealJumpCooldownBtns = {realJumpBtnMinus, realJumpBtnPlus},
        CloseBtn = closeBtn,
    }
end

-- ==================== CONEXÕES DA UI ====================
local function setupUIEvents()
    for i, btn in ipairs(uiElements.TabButtons) do
        btn.MouseButton1Click:Connect(function()
            for j, otherBtn in ipairs(uiElements.TabButtons) do
                otherBtn.BackgroundColor3 = COLORS.Surface
                otherBtn.TextColor3 = COLORS.TextDim
            end
            btn.BackgroundColor3 = COLORS.Primary
            btn.TextColor3 = COLORS.Text
            uiElements.CurrentTab = i
            uiElements.Page1.Visible = (i == 1)
            uiElements.Page2.Visible = (i == 2)
            uiElements.Page3.Visible = (i == 3)
        end)
    end

    uiElements.ChangeBtn.MouseButton1Click:Connect(startKeyChange)

    uiElements.ResetBtn.MouseButton1Click:Connect(function()
        selectedKey = nil
        updateKeyDisplay()
    end)

    uiElements.ActivateBtn.MouseButton1Click:Connect(sendExtraHit)

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

    uiElements.CooldownBtns[1].MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.ExtraHit.Cooldown - 0.1)
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        uiElements.CooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)
    uiElements.CooldownBtns[2].MouseButton1Click:Connect(function()
        local new = CONFIG.ExtraHit.Cooldown + 0.1
        CONFIG.ExtraHit.Cooldown = tonumber(string.format("%.1f", new))
        uiElements.CooldownValue.Text = tostring(CONFIG.ExtraHit.Cooldown)
    end)

    uiElements.MetalToggleBtn.MouseButton1Click:Connect(toggleMetalSkin)

    uiElements.RealJumpToggleBtn.MouseButton1Click:Connect(function()
        CONFIG.RealJump.Enabled = not CONFIG.RealJump.Enabled
        uiElements.RealJumpToggleBtn.Text = CONFIG.RealJump.Enabled and "ON" or "OFF"
        uiElements.RealJumpToggleBtn.BackgroundColor3 = CONFIG.RealJump.Enabled and COLORS.Accent or COLORS.Surface
    end)

    uiElements.RealJumpCooldownBtns[1].MouseButton1Click:Connect(function()
        local new = math.max(0, CONFIG.RealJump.Cooldown - 0.1)
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        uiElements.RealJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)
    uiElements.RealJumpCooldownBtns[2].MouseButton1Click:Connect(function()
        local new = CONFIG.RealJump.Cooldown + 0.1
        CONFIG.RealJump.Cooldown = tonumber(string.format("%.1f", new))
        uiElements.RealJumpCooldownValue.Text = tostring(CONFIG.RealJump.Cooldown)
    end)

    uiElements.CloseBtn.MouseButton1Click:Connect(function()
        uiOpen = false
        uiElements.MainFrame.Visible = false
    end)

    -- Arrastar interface
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
    
    if input.KeyCode == CONFIG.ToggleKey then
        uiOpen = not uiOpen
        if uiElements.MainFrame then uiElements.MainFrame.Visible = uiOpen end
        return
    end
    
    if keyChangeMode then return end
    
    if input.KeyCode == CONFIG.MetalSkin.Key then
        toggleMetalSkin()
        return
    end
    
    if CONFIG.RealJump.Enabled and (input.KeyCode == Enum.KeyCode.LeftAlt or input.KeyCode == Enum.KeyCode.RightAlt) then
        doRealJump()
        return
    end
    
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
