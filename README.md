local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local p = Players.LocalPlayer
local ativado = false
local animTrack = nil

-- === CONFIGURAÇÃO INICIAL ===
local ID_ANIMACAO = "rbxassetid://180436148" 
local NOME_DO_ACESSORIO = "Carro"           
local velocidade = 50 
-- ==============================================

-- INTERFACE PRINCIPAL
local gui = Instance.new("ScreenGui")
gui.Name = "MenuCarroGui"
gui.ResetOnSpawn = false
gui.Parent = p:WaitForChild("PlayerGui")

-- BOTÃO PARA ABRIR/FECHAR O MENU
local botaoMenu = Instance.new("TextButton", gui)
botaoMenu.Size = UDim2.new(0, 85, 0, 30)
botaoMenu.Position = UDim2.new(0.12, 0, 0.02, 0)
botaoMenu.Text = "Menu 🚗"
botaoMenu.TextSize = 12
botaoMenu.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
botaoMenu.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", botaoMenu).CornerRadius = UDim.new(0, 6)

-- JANELA DO MENU
local frameMenu = Instance.new("Frame", gui)
frameMenu.Size = UDim2.new(0, 180, 0, 140)
frameMenu.Position = UDim2.new(0.12, 0, 0.08, 0)
frameMenu.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frameMenu.Visible = false
Instance.new("UICorner", frameMenu)

-- 1. BOTÃO LIGAR / DESLIGAR
local botaoAtivar = Instance.new("TextButton", frameMenu)
botaoAtivar.Size = UDim2.new(0, 150, 0, 35)
botaoAtivar.Position = UDim2.new(0.08, 0, 0.12, 0)
botaoAtivar.Text = "MODO CARRO: OFF"
botaoAtivar.TextSize = 12
botaoAtivar.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
botaoAtivar.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", botaoAtivar)

-- 2. CONTROLE DE VELOCIDADE
local textoVel = Instance.new("TextLabel", frameMenu)
textoVel.Size = UDim2.new(0, 160, 0, 25)
textoVel.Position = UDim2.new(0.05, 0, 0.45, 0)
textoVel.Text = "Velocidade: " .. velocidade
textoVel.TextColor3 = Color3.fromRGB(255, 255, 255)
textoVel.BackgroundTransparency = 1

local bMaisVel = Instance.new("TextButton", frameMenu)
bMaisVel.Size = UDim2.new(0, 40, 0, 28)
bMaisVel.Position = UDim2.new(0.55, 0, 0.65, 0)
bMaisVel.Text = "+10"
bMaisVel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
bMaisVel.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", bMaisVel)

local bMinusVel = Instance.new("TextButton", frameMenu)
bMinusVel.Size = UDim2.new(0, 40, 0, 28)
bMinusVel.Position = UDim2.new(0.15, 0, 0.65, 0)
bMinusVel.Text = "-10"
bMinusVel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
bMinusVel.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", bMinusVel)

-- Alternar Menu
botaoMenu.MouseButton1Click:Connect(function()
    frameMenu.Visible = not frameMenu.Visible
    botaoMenu.Text = frameMenu.Visible and "Fechar ❌" or "Menu 🚗"
end)

local function gerenciarAnimasPadrao(char, pausar)
    local animateScript = char:FindFirstChild("Animate")
    if animateScript then
        animateScript.Disabled = pausar
        if pausar then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                for _, track in pairs(hum:GetPlayingAnimationTracks()) do track:Stop() end
            end
        end
    end
end

-- EVENTOS DE VELOCIDADE
bMaisVel.MouseButton1Click:Connect(function()
    velocidade = math.min(velocidade + 10, 250)
    textoVel.Text = "Velocidade: " .. velocidade
end)

bMinusVel.MouseButton1Click:Connect(function()
    velocidade = math.max(velocidade - 10, 16)
    textoVel.Text = "Velocidade: " .. velocidade
end)

-- LIGAR / DESLIGAR
botaoAtivar.MouseButton1Click:Connect(function()
    ativado = not ativado
    botaoAtivar.Text = ativado and "MODO CARRO: ON" or "MODO CARRO: OFF"
    botaoAtivar.BackgroundColor3 = ativado and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    
    local char = p.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    
    if ativado then
        gerenciarAnimasPadrao(char, true)
        
        local anim = Instance.new("Animation")
        anim.AnimationId = ID_ANIMACAO
        animTrack = hum:LoadAnimation(anim)
        animTrack.Priority = Enum.AnimationPriority.Action4
        animTrack.Looped = true
        animTrack:Play()
    else
        if animTrack then animTrack:Stop() animTrack = nil end
        gerenciarAnimasPadrao(char, false)
    end
end)

p.CharacterAdded:Connect(function(novoChar)
    if ativado then
        task.wait(0.5)
        local hum = novoChar:WaitForChild("Humanoid")
        gerenciarAnimasPadrao(novoChar, true)
        
        local anim = Instance.new("Animation")
        anim.AnimationId = ID_ANIMACAO
        animTrack = hum:LoadAnimation(anim)
        animTrack.Priority = Enum.AnimationPriority.Action4
        animTrack.Looped = true
        animTrack:Play()
    end
end)

-- LOOP PRINCIPAL (MÉTODO TPWALK ORIGINAL)
RunService.Heartbeat:Connect(function(deltaTime)
    if not ativado then return end
    
    local char = p.Character
    if not char then return end
    
    local hum = char:FindFirstChildOfClass("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local acessorio = char:FindFirstChild(NOME_DO_ACESSORIO)
    local handle = acessorio and acessorio:FindFirstChild("Handle")
    
    if not hrp or not hum then return end

    -- Movimentação TpWalk Original
    if hum.MoveDirection.Magnitude > 0 then
        local direcaoMovimento = hum.MoveDirection
        local novaPosicao = hrp.Position + (direcaoMovimento * velocidade * deltaTime)
        local direcaoOlhar = Vector3.new(direcaoMovimento.X, 0, direcaoMovimento.Z).Unit
        
        hrp.CFrame = CFrame.lookAt(novaPosicao, novaPosicao + direcaoOlhar)
    else
        local lookVector = hrp.CFrame.LookVector
        local direcaoReta = Vector3.new(lookVector.X, 0, lookVector.Z).Unit
        hrp.CFrame = CFrame.lookAt(hrp.Position, hrp.Position + direcaoReta)
    end

    -- Ajusta o carro na cintura
    if handle then
        local posPersonagem = hrp.Position
        local lookVector = hrp.CFrame.LookVector
        local direcaoCarro = Vector3.new(lookVector.X, 0, lookVector.Z).Unit
        
        handle.CFrame = CFrame.lookAt(posPersonagem, posPersonagem + direcaoCarro)
        handle.CanCollide = false
    end
end)
