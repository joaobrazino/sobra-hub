# sobra-hub
print("SCRIPT INICIADO: sobralZX Hub está tentando carregar.") -- Mensagem de depuração no início

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local localPlayer = Players.LocalPlayer

-- Senha para acesso ao hub
local PASSWORD = "SZX" -- <--- A senha definida para o hub é "SZX"!

-- Variáveis principais (aba principal)
local AIMBOT_ON = false
local ESP_ON = false
local HITBOX = 5
local TRANSP = 5
local HITBOX_ENABLED = false

-- Variáveis pessoais (aba pessoal)
local SPEED = 20
local JUMP = 50
local FOV = 70

-- Funções originais (mantidas iguais)
local function applyHitboxExpander(enable)
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local part = player.Character.HumanoidRootPart
            if enable then
                part.Size = Vector3.new(HITBOX, HITBOX, HITBOX)
                part.Transparency = TRANSP / 30
                part.Color = Color3.fromRGB(255,255,255)
                part.Material = Enum.Material.ForceField
                part.CanCollide = false
            else
                part.Size = Vector3.new(2,2,1)
                part.Transparency = 0
                part.Material = Enum.Material.Plastic
                part.CanCollide = true
            end
        end
    end
end

local function getClosestEnemyToCrosshair()
    local camera = workspace.CurrentCamera
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local headPos = camera:WorldToViewportPoint(player.Character.Head.Position)
            if headPos.Z > 0 then
                local dist = (Vector2.new(headPos.X, headPos.Y) - Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

local function aimCameraToTarget(target)
    local camera = workspace.CurrentCamera
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local headPos = target.Character.Head.Position
        camera.CFrame = CFrame.new(camera.CFrame.Position, headPos)
    end
end

-- ESP
local ESP_BARS = {}
local function createESPBar(character)
    local bar = Instance.new("BoxHandleAdornment")
    bar.Adornee = character:FindFirstChild("HumanoidRootPart")
    bar.Size = Vector3.new(4, 7, 1)
    bar.Color3 = Color3.fromRGB(255,255,255)
    bar.Transparency = 0.3
    bar.AlwaysOnTop = true
    bar.ZIndex = 10
    bar.Parent = workspace
    return bar
end

local function enableESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not ESP_BARS[player] then
                local bar = createESPBar(player.Character)
                ESP_BARS[player] = bar
            end
        end
    end
end

local function disableESP()
    for _, bar in pairs(ESP_BARS) do
        if bar then bar:Destroy() end
    end
    ESP_BARS = {}
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        if ESP_ON then
            local bar = createESPBar(char)
            ESP_BARS[player] = bar
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if ESP_BARS[player] then
        ESP_BARS[player]:Destroy()
        ESP_BARS[player] = nil
    end
end)

-- Aplicar Speed, Jump e FOV
local function applyPersonal()
    if localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid") then
        localPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = SPEED
        localPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = JUMP
    end
    if workspace.CurrentCamera then
        workspace.CurrentCamera.FieldOfView = FOV
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    if AIMBOT_ON then
        local target = getClosestEnemyToCrosshair()
        if target then
            aimCameraToTarget(target)
        end
    end
    if ESP_ON then
        enableESP()
    else
        disableESP()
    end
    if HITBOX_ENABLED then
        applyHitboxExpander(true)
    else
        applyHitboxExpander(false)
    end
    applyPersonal()
end)

-- Função para criar cantos arredondados
local function createCorner(obj, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 6)
    corner.Parent = obj
    return corner
end

-- Função para animação de hover
local function addHoverEffect(button)
    local originalColor = button.BackgroundColor3
    
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(70, 0, 70) -- Roxo mais escuro para hover
        }):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            BackgroundColor3 = originalColor
        }):Play()
    end)
end

-- Função para atualizar as cores dos botões ON/OFF
local function updateToggleButtons(onButton, offButton, isOn)
    if isOn then
        onButton.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo para ON
        offButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30) -- Cinza escuro para OFF
    else
        onButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30) -- Cinza escuro para ON
        offButton.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo para OFF
    end
end

-- ====================================================================
-- INÍCIO DA INTERFACE DE SENHA
-- ====================================================================

local passwordGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
passwordGui.Name = "PasswordScreen"
passwordGui.Enabled = true -- Inicialmente visível

local passwordFrame = Instance.new("Frame", passwordGui)
passwordFrame.Name = "PasswordFrame"
passwordFrame.Size = UDim2.new(0, 300, 0, 180)
passwordFrame.Position = UDim2.new(0.5, -150, 0.5, -90)
passwordFrame.BackgroundColor3 = Color3.fromRGB(50, 0, 50) -- Fundo roxo escuro
passwordFrame.BorderColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
passwordFrame.BorderSizePixel = 3
createCorner(passwordFrame, 10)

local passwordTopBar = Instance.new("Frame", passwordFrame)
passwordTopBar.Size = UDim2.new(1, 0, 0, 40)
passwordTopBar.Position = UDim2.new(0, 0, 0, 0)
passwordTopBar.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
passwordTopBar.BorderSizePixel = 0
createCorner(passwordTopBar, 8)

local passwordTitle = Instance.new("TextLabel", passwordTopBar)
passwordTitle.Size = UDim2.new(1, 0, 1, 0)
passwordTitle.BackgroundTransparency = 1
passwordTitle.Text = "Acesso Requerido"
passwordTitle.Font = Enum.Font.GothamBold
passwordTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
passwordTitle.TextSize = 22

local passwordLabel = Instance.new("TextLabel", passwordFrame)
passwordLabel.Size = UDim2.new(0.8, 0, 0, 30)
passwordLabel.Position = UDim2.new(0.5, -120, 0, 60)
passwordLabel.BackgroundTransparency = 1
passwordLabel.Text = "Digite a senha para continuar:"
passwordLabel.Font = Enum.Font.Gotham
passwordLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
passwordLabel.TextSize = 16
passwordLabel.TextXAlignment = Enum.TextXAlignment.Center

local passwordTextBox = Instance.new("TextBox", passwordFrame)
passwordTextBox.Size = UDim2.new(0.7, 0, 0, 35)
passwordTextBox.Position = UDim2.new(0.5, -105, 0, 95)
passwordTextBox.PlaceholderText = "Senha"
passwordTextBox.Text = ""
passwordTextBox.Font = Enum.Font.GothamBold
passwordTextBox.TextSize = 18
passwordTextBox.TextColor3 = Color3.fromRGB(0, 0, 0)
passwordTextBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
passwordTextBox.ClearTextOnFocus = true
passwordTextBox.TextXAlignment = Enum.TextXAlignment.Center
passwordTextBox.BorderSizePixel = 0
passwordTextBox.TextScaled = false -- Garante que o texto não se estique
createCorner(passwordTextBox, 5)

local submitButton = Instance.new("TextButton", passwordFrame)
submitButton.Size = UDim2.new(0, 100, 0, 35)
submitButton.Position = UDim2.new(0.5, -50, 0, 140)
submitButton.Text = "Entrar"
submitButton.Font = Enum.Font.GothamBold
submitButton.TextSize = 18
submitButton.TextColor3 = Color3.fromRGB(255, 255, 255)
submitButton.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
submitButton.BorderSizePixel = 0
createCorner(submitButton, 5)
addHoverEffect(submitButton)

local messageLabel = Instance.new("TextLabel", passwordFrame)
messageLabel.Size = UDim2.new(0.9, 0, 0, 20)
messageLabel.Position = UDim2.new(0.5, -135, 0, 120)
messageLabel.BackgroundTransparency = 1
messageLabel.Text = ""
messageLabel.Font = Enum.Font.Gotham
messageLabel.TextColor3 = Color3.fromRGB(255, 50, 50) -- Mantido vermelho para mensagens de erro
messageLabel.TextSize = 14
messageLabel.TextXAlignment = Enum.TextXAlignment.Center

submitButton.MouseButton1Click:Connect(function()
    local enteredPassword = passwordTextBox.Text
    if enteredPassword == PASSWORD then
        messageLabel.Text = "Senha correta! Carregando Hub..."
        passwordGui.Enabled = false -- Desabilita a tela de senha
        game:GetService("CoreGui").sobralZX_HUB.Enabled = true -- Habilita o hub principal
        print("🔥 sobralZX Hub - Versão Estética (Preto/Roxo) carregada! 🔥") -- Mensagem de carregamento atualizada
    else
        messageLabel.Text = "Senha incorreta. Tente novamente."
        passwordTextBox.Text = "" -- Limpa o campo de senha
    end
end)

-- Permite pressionar Enter para submeter a senha
passwordTextBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        submitButton.MouseButton1Click:Fire()
    end
end)

-- ====================================================================
-- FIM DA INTERFACE DE SENHA
-- ====================================================================


-- GUI do Hub principal
local gui = Instance.new("ScreenGui", game:GetService("CoreGui"))
gui.Name = "sobralZX_HUB"
gui.Enabled = false -- Inicialmente desabilitado, só será habilitado após a senha

-- Ícone SZ (melhorado)
local iconFrame = Instance.new("Frame", gui)
iconFrame.Name = "SZIconFrame"
iconFrame.Size = UDim2.new(0, 65, 0, 65)
iconFrame.Position = UDim2.new(0, 10, 0, 10)
iconFrame.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Fundo roxo para o frame do ícone
iconFrame.BorderColor3 = Color3.fromRGB(128, 0, 128) -- Borda roxa
iconFrame.BorderSizePixel = 2
iconFrame.Active = true
iconFrame.Draggable = true
createCorner(iconFrame, 8)

-- O ícone de texto "SZ" foi removido e substituído por uma imagem
local iconImage = Instance.new("ImageLabel", iconFrame)
iconImage.Name = "SZImage"
iconImage.Size = UDim2.new(1, 0, 1, 0)
iconImage.Position = UDim2.new(0, 0, 0, 0)
iconImage.BackgroundTransparency = 0 -- Garante que o fundo do ImageLabel seja visível
iconImage.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Fundo roxo para o ImageLabel
-- Usando um placeholder com cores roxas e texto para simular a imagem do Gengar.
-- **IMPORTANTE:** Para usar sua própria imagem, você precisará carregá-la no Roblox
-- e substituir este URL pelo Asset ID (ex: "rbxassetid://SEU_ID_DO_ASSET").
iconImage.Image = "https://placehold.co/65x65/500050/FFFFFF?text=SZ" -- Placeholder temporário com fundo roxo escuro e texto BRANCO
iconImage.ScaleType = Enum.ScaleType.Fit -- Garante que a imagem se ajuste ao frame
iconImage.ZIndex = 2 -- Garante que a imagem fique acima do fundo do frame

-- Hub principal (melhorado)
local frame = Instance.new("Frame", gui)
frame.Name = "sobralZX_Hub"
frame.BackgroundColor3 = Color3.fromRGB(50, 0, 50) -- Fundo roxo escuro para o hub
frame.BorderColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
frame.BorderSizePixel = 3
frame.Size = UDim2.new(0, 370, 0, 260)
frame.Position = UDim2.new(0.5, -185, 0.5, -130)
frame.Visible = false -- O hub começa invisível, o ícone o ativa
frame.Active = true
frame.Draggable = true
createCorner(frame, 10)

-- Barra superior decorativa
local topBar = Instance.new("Frame", frame)
topBar.Size = UDim2.new(1, 0, 0, 45)
topBar.Position = UDim2.new(0, 0, 0, 0)
topBar.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
topBar.BorderSizePixel = 0
createCorner(topBar, 8)

-- Ajustar barra para não sobrepor os cantos
local topBarMask = Instance.new("Frame", topBar)
topBarMask.Size = UDim2.new(1, 0, 0, 20)
topBarMask.Position = UDim2.new(0, 0, 1, -20)
topBarMask.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo

-- Título
local title = Instance.new("TextLabel", topBar)
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "sobralZX Hub"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 24
title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão para alternar abas (melhorado)
local abaBtn = Instance.new("TextButton", topBar)
abaBtn.Size = UDim2.new(0, 90, 0, 25)
abaBtn.Position = UDim2.new(1, -100, 0, 10)
abaBtn.Text = "Pessoal"
abaBtn.Font = Enum.Font.GothamBold
abaBtn.TextSize = 14
abaBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
abaBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
abaBtn.BorderSizePixel = 0
createCorner(abaBtn, 5)
addHoverEffect(abaBtn)

-- Frame Principal
local principalFrame = Instance.new("Frame", frame)
principalFrame.Size = UDim2.new(1, 0, 1, -55)
principalFrame.Position = UDim2.new(0, 0, 0, 55)
principalFrame.BackgroundTransparency = 0 -- Tornar visível
principalFrame.BackgroundColor3 = Color3.fromRGB(50, 0, 50) -- Fundo roxo escuro

-- Aimbot (melhorado)
local aimbotLabel = Instance.new("TextLabel", principalFrame)
aimbotLabel.Size = UDim2.new(0, 120, 0, 35)
aimbotLabel.Position = UDim2.new(0.05, 0, 0, 15)
aimbotLabel.Text = "Aimbot"
aimbotLabel.Font = Enum.Font.GothamBold
aimbotLabel.TextSize = 18
aimbotLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotLabel.BackgroundTransparency = 1
aimbotLabel.TextXAlignment = Enum.TextXAlignment.Left

local aimbotOn = Instance.new("TextButton", principalFrame)
aimbotOn.Size = UDim2.new(0, 40, 0, 25)
aimbotOn.Position = UDim2.new(0.65, 0, 0, 20)
aimbotOn.Text = "ON"
aimbotOn.Font = Enum.Font.GothamBold
aimbotOn.TextSize = 14
aimbotOn.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotOn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
aimbotOn.BorderSizePixel = 0
createCorner(aimbotOn, 4)
addHoverEffect(aimbotOn)

local aimbotOff = Instance.new("TextButton", principalFrame)
aimbotOff.Size = UDim2.new(0, 40, 0, 25)
aimbotOff.Position = UDim2.new(0.78, 0, 0, 20)
aimbotOff.Text = "OFF"
aimbotOff.Font = Enum.Font.GothamBold
aimbotOff.TextSize = 14
aimbotOff.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotOff.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
aimbotOff.BorderSizePixel = 0
createCorner(aimbotOff, 4)
addHoverEffect(aimbotOff)

-- ESP (melhorado) - Posição ajustada
local espLabel = Instance.new("TextLabel", principalFrame)
espLabel.Size = UDim2.new(0, 120, 0, 35)
espLabel.Position = UDim2.new(0.05, 0, 0, 55) -- Posição ajustada
espLabel.Text = "ESP"
espLabel.Font = Enum.Font.GothamBold
espLabel.TextSize = 18
espLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
espLabel.BackgroundTransparency = 1
espLabel.TextXAlignment = Enum.TextXAlignment.Left

local espOn = Instance.new("TextButton", principalFrame)
espOn.Size = UDim2.new(0, 40, 0, 25)
espOn.Position = UDim2.new(0.65, 0, 0, 60) -- Posição ajustada
espOn.Text = "ON"
espOn.Font = Enum.Font.GothamBold
espOn.TextSize = 14
espOn.TextColor3 = Color3.fromRGB(255, 255, 255)
espOn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
espOn.BorderSizePixel = 0
createCorner(espOn, 4)
addHoverEffect(espOn)

local espOff = Instance.new("TextButton", principalFrame)
espOff.Size = UDim2.new(0, 40, 0, 25)
espOff.Position = UDim2.new(0.78, 0, 0, 60) -- Posição ajustada
espOff.Text = "OFF"
espOff.Font = Enum.Font.GothamBold
espOff.TextSize = 14
espOff.TextColor3 = Color3.fromRGB(255, 255, 255)
espOff.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
espOff.BorderSizePixel = 0
createCorner(espOff, 4)
addHoverEffect(espOff)

-- Hitbox (melhorado) - Posição ajustada
local hitboxLabel = Instance.new("TextLabel", principalFrame)
hitboxLabel.Size = UDim2.new(0, 120, 0, 35)
hitboxLabel.Position = UDim2.new(0.05, 0, 0, 95) -- Posição ajustada
hitboxLabel.Text = "Hitbox"
hitboxLabel.Font = Enum.Font.GothamBold
hitboxLabel.TextSize = 18
hitboxLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxLabel.BackgroundTransparency = 1
hitboxLabel.TextXAlignment = Enum.TextXAlignment.Left

local hitboxBox = Instance.new("TextBox", principalFrame)
hitboxBox.Size = UDim2.new(0, 45, 0, 25)
hitboxBox.Position = UDim2.new(0.6, 0, 0, 100) -- Posição ajustada
hitboxBox.Text = tostring(HITBOX)
hitboxBox.Font = Enum.Font.GothamBold
hitboxBox.TextSize = 14
hitboxBox.TextColor3 = Color3.fromRGB(0, 0, 0)
hitboxBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
hitboxBox.ClearTextOnFocus = false
hitboxBox.TextXAlignment = Enum.TextXAlignment.Center
hitboxBox.BorderSizePixel = 0
createCorner(hitboxBox, 4)

local hitboxToggle = Instance.new("TextButton", principalFrame)
hitboxToggle.Size = UDim2.new(0, 50, 0, 25)
hitboxToggle.Position = UDim2.new(0.75, 0, 0, 100) -- Posição ajustada
hitboxToggle.Text = (HITBOX_ENABLED and "ON" or "OFF")
hitboxToggle.Font = Enum.Font.GothamBold
hitboxToggle.TextSize = 14
hitboxToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxToggle.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
hitboxToggle.BorderSizePixel = 0
createCorner(hitboxToggle, 4)
addHoverEffect(hitboxToggle)

-- Transparência (melhorado) - Posição ajustada
local transLabel = Instance.new("TextLabel", principalFrame)
transLabel.Size = UDim2.new(0, 120, 0, 35)
transLabel.Position = UDim2.new(0.05, 0, 0, 135) -- Posição ajustada
transLabel.Text = "Transparência"
transLabel.Font = Enum.Font.GothamBold
transLabel.TextSize = 16
transLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
transLabel.BackgroundTransparency = 1
transLabel.TextXAlignment = Enum.TextXAlignment.Left

local transBox = Instance.new("TextBox", principalFrame)
transBox.Size = UDim2.new(0, 45, 0, 25)
transBox.Position = UDim2.new(0.6, 0, 0, 140) -- Posição ajustada
transBox.Text = tostring(TRANSP)
transBox.Font = Enum.Font.GothamBold
transBox.TextSize = 14
transBox.TextColor3 = Color3.fromRGB(0, 0, 0)
transBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
transBox.ClearTextOnFocus = false
transBox.TextXAlignment = Enum.TextXAlignment.Center
transBox.BorderSizePixel = 0
createCorner(transBox, 4)

-- Frame Pessoal (mantido igual)
local pessoalFrame = Instance.new("Frame", frame)
pessoalFrame.Size = UDim2.new(1, 0, 1, -55)
pessoalFrame.Position = UDim2.new(0, 0, 0, 55)
pessoalFrame.BackgroundTransparency = 0 -- Tornar visível
pessoalFrame.BackgroundColor3 = Color3.fromRGB(50, 0, 50) -- Fundo roxo escuro
pessoalFrame.Visible = false

-- SPEED (mantido igual)
local speedLbl = Instance.new("TextLabel", pessoalFrame)
speedLbl.Size = UDim2.new(0, 120, 0, 35)
speedLbl.Position = UDim2.new(0.05, 0, 0, 20)
speedLbl.Text = "Speed"
speedLbl.Font = Enum.Font.GothamBold
speedLbl.TextSize = 18
speedLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLbl.BackgroundTransparency = 1
speedLbl.TextXAlignment = Enum.TextXAlignment.Left

local speedBox = Instance.new("TextBox", pessoalFrame)
speedBox.Size = UDim2.new(0, 60, 0, 25)
speedBox.Position = UDim2.new(0.65, 0, 0, 25)
speedBox.Text = tostring(SPEED)
speedBox.Font = Enum.Font.GothamBold
speedBox.TextSize = 14
speedBox.TextColor3 = Color3.fromRGB(0, 0, 0)
speedBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
speedBox.ClearTextOnFocus = false
speedBox.TextXAlignment = Enum.TextXAlignment.Center
speedBox.BorderSizePixel = 0
createCorner(speedBox, 4)

-- JUMP (mantido igual)
local jumpLbl = Instance.new("TextLabel", pessoalFrame)
jumpLbl.Size = UDim2.new(0, 120, 0, 35)
jumpLbl.Position = UDim2.new(0.05, 0, 0, 70)
jumpLbl.Text = "Jump Power"
jumpLbl.Font = Enum.Font.GothamBold
jumpLbl.TextSize = 18
jumpLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpLbl.BackgroundTransparency = 1
jumpLbl.TextXAlignment = Enum.TextXAlignment.Left

local jumpBox = Instance.new("TextBox", pessoalFrame)
jumpBox.Size = UDim2.new(0, 60, 0, 25)
jumpBox.Position = UDim2.new(0.65, 0, 0, 75)
jumpBox.Text = tostring(JUMP)
jumpBox.Font = Enum.Font.GothamBold
jumpBox.TextSize = 14
jumpBox.TextColor3 = Color3.fromRGB(0, 0, 0)
jumpBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
jumpBox.ClearTextOnFocus = false
jumpBox.TextXAlignment = Enum.TextXAlignment.Center
jumpBox.BorderSizePixel = 0
createCorner(jumpBox, 4)

-- FOV (mantido igual)
local fovLbl = Instance.new("TextLabel", pessoalFrame)
fovLbl.Size = UDim2.new(0, 120, 0, 35)
fovLbl.Position = UDim2.new(0.05, 0, 0, 120)
fovLbl.Text = "Field of View"
fovLbl.Font = Enum.Font.GothamBold
fovLbl.TextSize = 18
fovLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLbl.BackgroundTransparency = 1
fovLbl.TextXAlignment = Enum.TextXAlignment.Left

local fovBox = Instance.new("TextBox", pessoalFrame)
fovBox.Size = UDim2.new(0, 60, 0, 25)
fovBox.Position = UDim2.new(0.65, 0, 0, 125)
fovBox.Text = tostring(FOV)
fovBox.Font = Enum.Font.GothamBold
fovBox.TextSize = 14
fovBox.TextColor3 = Color3.fromRGB(0, 0, 0)
fovBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fovBox.ClearTextOnFocus = false
fovBox.TextXAlignment = Enum.TextXAlignment.Center
fovBox.BorderSizePixel = 0
createCorner(fovBox, 4)

-- Conectar eventos originais
aimbotOn.MouseButton1Click:Connect(function()
    AIMBOT_ON = true
    updateToggleButtons(aimbotOn, aimbotOff, AIMBOT_ON)
end)

aimbotOff.MouseButton1Click:Connect(function()
    AIMBOT_ON = false
    updateToggleButtons(aimbotOn, aimbotOff, AIMBOT_ON)
end)

espOn.MouseButton1Click:Connect(function()
    ESP_ON = true
    updateToggleButtons(espOn, espOff, ESP_ON)
end)

espOff.MouseButton1Click:Connect(function()
    ESP_ON = false
    updateToggleButtons(espOn, espOff, ESP_ON)
end)

hitboxBox.FocusLost:Connect(function()
    local val = tonumber(hitboxBox.Text)
    if val and val >= 1 and val <= 100 then
        HITBOX = val
    else
        hitboxBox.Text = tostring(HITBOX)
    end
end)

hitboxToggle.MouseButton1Click:Connect(function()
    HITBOX_ENABLED = not HITBOX_ENABLED
    hitboxToggle.Text = (HITBOX_ENABLED and "ON" or "OFF")
end)

transBox.FocusLost:Connect(function()
    local val = tonumber(transBox.Text)
    if val and val >= 1 and val <= 30 then
        TRANSP = val
    else
        transBox.Text = tostring(TRANSP)
    end
end)

speedBox.FocusLost:Connect(function()
    local val = tonumber(speedBox.Text)
    if val and val >= 1 and val <= 100 then
        SPEED = val
    else
        speedBox.Text = tostring(SPEED)
    end
end)

jumpBox.FocusLost:Connect(function()
    local val = tonumber(jumpBox.Text)
    if val and val >= 1 and val <= 100 then
        JUMP = val
    else
        jumpBox.Text = tostring(JUMP)
    end
end)

fovBox.FocusLost:Connect(function()
    local val = tonumber(fovBox.Text)
    if val and val >= 1 and val <= 340 then
        FOV = val
    else
        fovBox.Text = tostring(FOV)
    end
end)

-- Alternar abas
local emPrincipal = true
abaBtn.MouseButton1Click:Connect(function()
    emPrincipal = not emPrincipal
    principalFrame.Visible = emPrincipal
    pessoalFrame.Visible = not emPrincipal
    abaBtn.Text = emPrincipal and "Pessoal" or "Principal"
end)

-- Ícone SZ abre/fecha o Hub
iconFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        frame.Visible = not frame.Visible
        if frame.Visible then
            frame.Position = UDim2.new(0, iconFrame.Position.X.Offset + iconFrame.Size.X.Offset + 10, 0, iconFrame.Position.Y.Offset)
        end
    end
end)

iconFrame:GetPropertyChangedSignal("Position"):Connect(function()
    if frame.Visible then
        frame.Position = UDim2.new(0, iconFrame.Position.X.Offset + iconFrame.Size.X.Offset + 10, 0, iconFrame.Position.Y.Offset)
    end
end)

-- Atualiza o estado inicial dos botões ON/OFF
updateToggleButtons(aimbotOn, aimbotOff, AIMBOT_ON)
updateToggleButtons(espOn, espOff, ESP_ON)
