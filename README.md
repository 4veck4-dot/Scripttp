-- Script de Teleporte com UI Limpa (Sem Título e Sem Fechar)
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local selectedPlayer = nil
local cameraLocked = false
local cameraConnection = nil

-- ============================================
-- CONFIGURAÇÕES
-- ============================================
local settings = {
    cameraEnabled = true,
    teleportEnabled = true,
}

local gui = Instance.new("ScreenGui")
gui.Name = "TP_System"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer.PlayerGui

-- ============================================
-- CORES
-- ============================================
local colors = {
    pink = Color3.fromRGB(255, 150, 200),
    purple = Color3.fromRGB(180, 100, 255),
    blue = Color3.fromRGB(100, 200, 255),
    mint = Color3.fromRGB(150, 255, 200),
    yellow = Color3.fromRGB(255, 230, 100),
    dark = Color3.fromRGB(20, 15, 25),
    card = Color3.fromRGB(35, 25, 45),
    red = Color3.fromRGB(255, 60, 60),
    redDark = Color3.fromRGB(180, 30, 30),
    orange = Color3.fromRGB(255, 180, 50),
    green = Color3.fromRGB(0, 200, 100),
    gray = Color3.fromRGB(80, 80, 80),
    white = Color3.fromRGB(255, 255, 255),
}

-- ============================================
-- IDs DAS IMAGENS
-- ============================================
local imageIDs = {
    list = "rbxassetid://6034818370",
    teleport = "rbxassetid://6031091732",
    camera = "rbxassetid://6023423488",
    settings = "rbxassetid://6034628390",
}

-- ============================================
-- FUNÇÃO PARA CRIAR BOTÃO COM IMAGEM
-- ============================================
local function createImageButton(imageId, position, bgColor, size)
    size = size or 50
    local btn = Instance.new("ImageButton")
    btn.Size = UDim2.new(0, size, 0, size)
    btn.Position = position
    btn.BackgroundColor3 = bgColor or colors.pink
    btn.BackgroundTransparency = 0.15
    btn.Image = imageId
    btn.ImageColor3 = Color3.fromRGB(255, 255, 255)
    btn.ImageTransparency = 0
    btn.Parent = gui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn
    
    local shadow = Instance.new("UIShadow")
    shadow.Parent = btn
    
    return btn
end

-- ============================================
-- CRIAR BOTÕES PRINCIPAIS
-- ============================================
local listBtn = createImageButton(imageIDs.list, UDim2.new(0, 10, 0.5, -115), colors.pink)
local tpBtn = createImageButton(imageIDs.teleport, UDim2.new(0, 10, 0.5, -55), colors.purple)
local camBtn = createImageButton(imageIDs.camera, UDim2.new(0, 10, 0.5, 5), colors.blue)
local settingsBtn = createImageButton(imageIDs.settings, UDim2.new(0, 10, 0.5, 65), colors.red, 50)

-- ============================================
-- JANELA DE SELEÇÃO (JOGADORES - SEM TÍTULO)
-- ============================================
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 230)
frame.Position = UDim2.new(0.5, -110, 0.5, -115)
frame.BackgroundColor3 = colors.dark
frame.BackgroundTransparency = 0.05
frame.BorderSizePixel = 2
frame.BorderColor3 = colors.pink
frame.Visible = false
frame.Parent = gui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 12)
frameCorner.Parent = frame

-- ============================================
-- LISTA DE JOGADORES (SEM TÍTULO)
-- ============================================
local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -12, 1, -12)
list.Position = UDim2.new(0, 6, 0, 6)
list.BackgroundColor3 = colors.card
list.BackgroundTransparency = 0.3
list.BorderSizePixel = 1
list.BorderColor3 = colors.purple
list.ScrollBarThickness = 6
list.CanvasSize = UDim2.new(0, 0, 0, 0)
list.AutomaticCanvasSize = Enum.AutomaticSize.Y
list.Parent = frame

local listCorner2 = Instance.new("UICorner")
listCorner2.CornerRadius = UDim.new(0, 8)
listCorner2.Parent = list

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 4)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = list

-- ============================================
-- STATUS DO JOGADOR SELECIONADO
-- ============================================
local selectedText = Instance.new("TextLabel")
selectedText.Size = UDim2.new(1, -12, 0, 22)
selectedText.Position = UDim2.new(0, 6, 1, -28)
selectedText.BackgroundColor3 = colors.card
selectedText.BackgroundTransparency = 0.4
selectedText.BorderSizePixel = 1
selectedText.BorderColor3 = colors.mint
selectedText.Text = "💫 Nenhum"
selectedText.TextColor3 = Color3.fromRGB(200, 200, 200)
selectedText.TextScaled = true
selectedText.Font = Enum.Font.Gotham
selectedText.Parent = frame

local selectedCorner = Instance.new("UICorner")
selectedCorner.CornerRadius = UDim.new(0, 6)
selectedCorner.Parent = selectedText

-- ============================================
-- MENU DE CONFIGURAÇÕES (SEM TÍTULO)
-- ============================================
local settingsFrame = Instance.new("Frame")
settingsFrame.Size = UDim2.new(0, 220, 0, 120)
settingsFrame.Position = UDim2.new(0.5, -110, 0.5, -60)
settingsFrame.BackgroundColor3 = colors.dark
settingsFrame.BackgroundTransparency = 0.05
settingsFrame.BorderSizePixel = 2
settingsFrame.BorderColor3 = colors.red
settingsFrame.Visible = false
settingsFrame.Parent = gui

local settingsCorner = Instance.new("UICorner")
settingsCorner.CornerRadius = UDim.new(0, 12)
settingsCorner.Parent = settingsFrame

-- ============================================
-- TOGGLES (SEM TÍTULO)
-- ============================================

local function createToggle(parent, text, yPos, defaultState, onChange)
    -- Container do toggle
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, -12, 0, 40)
    container.Position = UDim2.new(0, 6, 0, yPos)
    container.BackgroundTransparency = 1
    container.Parent = parent
    
    -- Botão toggle
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundColor3 = defaultState and colors.green or colors.gray
    btn.BackgroundTransparency = 0.2
    btn.BorderSizePixel = 1
    btn.BorderColor3 = defaultState and colors.green or colors.gray
    btn.Text = text .. "  " .. (defaultState and "✅ LIGADO" or "❌ DESLIGADO")
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextScaled = true
    btn.Font = Enum.Font.Gotham
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Parent = container
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn
    
    local state = defaultState
    
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and colors.green or colors.gray
        btn.BorderColor3 = state and colors.green or colors.gray
        btn.Text = text .. "  " .. (state and "✅ LIGADO" or "❌ DESLIGADO")
        
        if onChange then
            onChange(state)
        end
    end)
    
    return btn
end

-- ============================================
-- CRIAR TOGGLES
-- ============================================

-- Toggle 1: Câmera (Y = 10)
local cameraToggle = createToggle(settingsFrame, "🎥 Câmera", 10, settings.cameraEnabled, function(state)
    settings.cameraEnabled = state
    camBtn.Visible = state
    if not state and cameraLocked then
        cameraLocked = false
        if cameraConnection then
            cameraConnection:Disconnect()
            cameraConnection = nil
        end
        camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        camBtn.BackgroundColor3 = colors.blue
    end
end)

-- Toggle 2: Teleporte (Y = 55)
local teleportToggle = createToggle(settingsFrame, "⚡ Teleporte", 55, settings.teleportEnabled, function(state)
    settings.teleportEnabled = state
    tpBtn.Visible = state
end)

-- ============================================
-- FUNÇÃO ATUALIZAR LISTA
-- ============================================
local function updateList()
    for _, child in ipairs(list:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -12, 0, 35)
            btn.BackgroundColor3 = colors.card
            btn.BackgroundTransparency = 0.2
            btn.BorderSizePixel = 1
            btn.BorderColor3 = colors.purple
            btn.Text = player.Name
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextScaled = true
            btn.Font = Enum.Font.Gotham
            btn.Parent = list
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 6)
            btnCorner.Parent = btn
            
            btn.MouseButton1Click:Connect(function()
                selectedPlayer = player
                for _, b in ipairs(list:GetChildren()) do
                    if b:IsA("TextButton") then
                        b.BackgroundColor3 = colors.card
                        b.BackgroundTransparency = 0.2
                        b.BorderColor3 = colors.purple
                        b.Text = string.gsub(b.Text, "⭐ ", "")
                    end
                end
                btn.BackgroundColor3 = colors.mint
                btn.BackgroundTransparency = 0.1
                btn.BorderColor3 = colors.pink
                btn.Text = "⭐ " .. player.Name
                selectedText.Text = "💖 " .. player.Name
                selectedText.TextColor3 = colors.mint
                frame.Visible = false
            end)
        end
    end
    
    local count = #list:GetChildren()
    list.CanvasSize = UDim2.new(0, 0, 0, math.max(count * 42, 100))
end

-- ============================================
-- FUNÇÕES DOS BOTÕES
-- ============================================
listBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
    if frame.Visible then 
        updateList()
        if selectedPlayer then
            selectedText.Text = "💖 " .. selectedPlayer.Name
            selectedText.TextColor3 = colors.mint
        end
    end
end)

-- ============================================
-- BOTÃO CONFIGURAÇÕES
-- ============================================
settingsBtn.MouseButton1Click:Connect(function()
    settingsFrame.Visible = not settingsFrame.Visible
    TweenService:Create(settingsBtn, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 55, 0, 55)
    }):Play()
    task.wait(0.2)
    TweenService:Create(settingsBtn, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 50, 0, 50)
    }):Play()
end)

-- ============================================
-- FUNÇÃO TELEPORTE
-- ============================================
tpBtn.MouseButton1Click:Connect(function()
    if not settings.teleportEnabled then
        tpBtn.ImageColor3 = Color3.fromRGB(255, 100, 0)
        task.wait(0.3)
        tpBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    if not selectedPlayer then
        tpBtn.ImageColor3 = Color3.fromRGB(255, 0, 0)
        task.wait(0.4)
        tpBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    local target = selectedPlayer.Character
    if not target or not target:FindFirstChild("HumanoidRootPart") then
        tpBtn.ImageColor3 = Color3.fromRGB(255, 0, 0)
        task.wait(0.4)
        tpBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        selectedPlayer = nil
        selectedText.Text = "💫 Nenhum"
        selectedText.TextColor3 = Color3.fromRGB(200, 200, 200)
        return
    end
    
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        tpBtn.ImageColor3 = Color3.fromRGB(255, 230, 100)
        tpBtn.BackgroundColor3 = colors.yellow
        
        local att = Instance.new("Attachment")
        att.Parent = hrp
        
        local particles = Instance.new("ParticleEmitter")
        particles.Texture = "rbxassetid://2782978380"
        particles.Rate = 150
        particles.Lifetime = NumberRange.new(0.4)
        particles.SpreadAngle = Vector2.new(360, 360)
        particles.VelocityInheritance = 0
        particles.Speed = NumberRange.new(8)
        particles.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, colors.pink),
            ColorSequenceKeypoint.new(0.5, colors.purple),
            ColorSequenceKeypoint.new(1, colors.blue)
        })
        particles.Size = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 1.5),
            NumberSequenceKeypoint.new(1, 0)
        })
        particles.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(1, 1)
        })
        particles.Parent = att
        
        hrp.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        
        task.wait(0.6)
        particles:Destroy()
        att:Destroy()
        
        tpBtn.ImageColor3 = Color3.fromRGB(0, 255, 0)
        tpBtn.BackgroundColor3 = colors.mint
        task.wait(0.4)
        tpBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        tpBtn.BackgroundColor3 = colors.purple
    end
end)

-- ============================================
-- FUNÇÃO CÂMERA PERSEGUIDORA
-- ============================================
local function toggleCamera()
    if not settings.cameraEnabled then
        camBtn.ImageColor3 = Color3.fromRGB(255, 100, 0)
        task.wait(0.3)
        camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    if cameraLocked then
        cameraLocked = false
        if cameraConnection then
            cameraConnection:Disconnect()
            cameraConnection = nil
        end
        camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        camBtn.BackgroundColor3 = colors.blue
        return
    end
    
    if not selectedPlayer then
        camBtn.ImageColor3 = Color3.fromRGB(255, 0, 0)
        task.wait(0.5)
        camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    local target = selectedPlayer.Character
    if not target or not target:FindFirstChild("HumanoidRootPart") then
        camBtn.ImageColor3 = Color3.fromRGB(255, 0, 0)
        task.wait(0.5)
        camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
        return
    end
    
    cameraLocked = true
    camBtn.ImageColor3 = Color3.fromRGB(255, 180, 50)
    camBtn.BackgroundColor3 = colors.orange
    
    if cameraConnection then
        cameraConnection:Disconnect()
    end
    
    cameraConnection = RunService.RenderStepped:Connect(function()
        if not cameraLocked then return end
        
        if not selectedPlayer then
            cameraLocked = false
            camBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
            camBtn.BackgroundColor3 = colors.blue
            if cameraConnection then
                cameraConnection:Disconnect()
                cameraConnection = nil
            end
            return
        end
        
        local targetChar = selectedPlayer.Character
        if targetChar and targetChar:FindFirstChild("HumanoidRootPart") then
            local targetPos = targetChar.HumanoidRootPart.Position
            local currentCamPos = Camera.CFrame.Position
            local offset = currentCamPos - (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position or Vector3.new(0, 0, 0))
            local newCamPos = targetPos + offset
            local smoothPos = currentCamPos:Lerp(newCamPos, 0.3)
            Camera.CFrame = CFrame.lookAt(smoothPos, targetPos)
        end
    end)
end

camBtn.MouseButton1Click:Connect(toggleCamera)

-- ============================================
-- TECLA DE ATALHO (F5 = Câmera)
-- ============================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F5 then
        toggleCamera()
    end
end)

-- ============================================
-- ATUALIZAR LISTA
-- ============================================
Players.PlayerAdded:Connect(updateList)
Players.PlayerRemoving:Connect(updateList)

updateList()

print("🔴 Sistema Teleporte com UI Limpa carregado!")
print("👥 = Lista | ⚡ = Teleporte | 🎥 = Câmera | 🔴 = Configurações")
print("📌 Pressione F5 para ativar/desativar a câmera")
