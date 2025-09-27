-- GUI Top AutoFarm Florianópolis RP
-- Combina Auto Sacola, Auto Lixeiro, Rejoin e Anti-AFK

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local vu = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")

-- Flags
local ativoSacola = false
local ativoLixeiro = false
local ativoAFK = true -- por padrão ativo
local coletarThreadLixeiro = nil

-- Configurações Lixeiro
local velocidadeCorrendo = 24
local tempoCorrida = 6
local tempoSegurarF = 2

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoFarmGUI"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 340, 0, 260)
mainFrame.Position = UDim2.new(0.5, -170, 0.5, -130)
mainFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.Draggable = true

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundTransparency = 1
title.Text = "FRP|by bores/pablinho"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.Parent = mainFrame

-- Função para criar botões
local function createButton(text, pos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 150, 0, 45)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Text = text
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 16
    btn.Parent = mainFrame
    return btn
end

-- Botões
local btnSacola = createButton("Auto Sacola [OFF]", UDim2.new(0, 20, 0, 70))
local btnLixeiro = createButton("Auto Lixeiro [OFF]", UDim2.new(0, 170, 0, 70))
local btnRejoin = createButton("Rejoin", UDim2.new(0, 95, 0, 130))
local btnAFK = createButton("Anti-AFK [ON]", UDim2.new(0, 20, 0, 190))

-- Label de status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 30)
statusLabel.Position = UDim2.new(0, 0, 1, -30)
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.fromRGB(0,255,0)
statusLabel.Text = "Status: Tudo parado"
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 14
statusLabel.Parent = mainFrame

-- Função para atualizar status
local function atualizarStatus()
    local sacola = ativoSacola and "ON" or "OFF"
    local lixeiro = ativoLixeiro and "ON" or "OFF"
    local afk = ativoAFK and "ON" or "OFF"
    statusLabel.Text = "Sacola: "..sacola.." | Lixeiro: "..lixeiro.." | Anti-AFK: "..afk
end

-- Função Auto Sacola
local vim = game:GetService("VirtualInputManager")
task.spawn(function()
    while true do
        task.wait(0.5)
        if ativoSacola then
            vim:SendKeyEvent(true, Enum.KeyCode.F, false, game)
            task.wait(2)
            vim:SendKeyEvent(false, Enum.KeyCode.F, false, game)
        end
    end
end)

-- Funções Lixeiro
local function getHumanoid()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    return character, humanoid
end

local function andarAte(posicao, tempo)
    local _, humanoid = getHumanoid()
    if not humanoid then return end
    humanoid.WalkSpeed = velocidadeCorrendo
    humanoid:MoveTo(posicao)
    task.wait(tempo)
end

local function segurarF(prompt, tempo)
    if not prompt then return end
    pcall(function() fireproximityprompt(prompt, 1) end)
    task.wait(tempo)
    pcall(function() fireproximityprompt(prompt, 0) end)
end

local function loopColetaLixeiro()
    local trabaio = workspace:FindFirstChild("Floripa_workspace") and workspace["Floripa_workspace"].Empregos and workspace["Floripa_workspace"].Empregos:FindFirstChild("Lixeiro")
    if not trabaio then return end
    local entrega = trabaio:FindFirstChild("Entregar")
    local pegar = trabaio:FindFirstChild("Pegar")
    if not entrega or not pegar or not entrega:FindFirstChild("Bloco") or not pegar:FindFirstChild("Bloco") then return end

    while ativoLixeiro do
        andarAte(pegar.Bloco.Position, tempoCorrida)
        segurarF(pegar.Bloco:FindFirstChild("ProximityPrompt"), tempoSegurarF)
        andarAte(entrega.Bloco.Position, tempoCorrida)
        segurarF(entrega.Bloco:FindFirstChild("ProximityPrompt"), tempoSegurarF)
        task.wait(0.2)
    end
end

local function setAtivoLixeiro(valor)
    ativoLixeiro = valor
    if ativoLixeiro then
        if not coletarThreadLixeiro or coletarThreadLixeiro.Status == "dead" then
            coletarThreadLixeiro = task.spawn(loopColetaLixeiro)
        end
    end
end

-- Botão Sacola
btnSacola.MouseButton1Click:Connect(function()
    ativoSacola = not ativoSacola
    btnSacola.Text = ativoSacola and "Auto Sacola [ON]" or "Auto Sacola [OFF]"
    atualizarStatus()
end)

-- Botão Lixeiro
btnLixeiro.MouseButton1Click:Connect(function()
    setAtivoLixeiro(not ativoLixeiro)
    btnLixeiro.Text = ativoLixeiro and "Auto Lixeiro [ON]" or "Auto Lixeiro [OFF]"
    atualizarStatus()
end)

-- Botão Anti-AFK
btnAFK.MouseButton1Click:Connect(function()
    ativoAFK = not ativoAFK
    btnAFK.Text = ativoAFK and "Anti-AFK [ON]" or "Anti-AFK [OFF]"
    atualizarStatus()
end)

-- Botão Rejoin
btnRejoin.MouseButton1Click:Connect(function()
    local placeId = game.PlaceId
    TeleportService:Teleport(placeId, player)
end)

-- Anti-AFK geral
player.Idled:Connect(function()
    if not ativoAFK then return end
    pcall(function()
        vu:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(0.4)
        vu:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
    pcall(function()
        vu:CaptureController()
        vu:ClickButton2(Vector2.new(0,0))
    end)
end)

-- Mensagem inicial
print("✅ AutoFarm GUI carregada! Use os botões para ativar/desativar funções.")
