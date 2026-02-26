local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // MUTAÇÕES DISPONÍVEIS
local MutacaoCores = {
    Gold    = Color3.fromRGB(255, 215, 0),
    Diamond = Color3.fromRGB(0, 220, 255),
    Rainbow = Color3.fromRGB(255, 100, 200),
    Galaxy  = Color3.fromRGB(160, 80, 255),
}

-- // Lê os brainrots do PlayerGui dinamicamente
-- Estrutura: PlayerGui.UI.Frames.Index.Frame.[Mutacao].[BrainrotName]
local function GetBrainrotsDoInventario()
    local resultado = {} -- { nome = "TimCheese", mutacao = "Gold" }
    local gui = LocalPlayer:FindFirstChild("PlayerGui")
    if not gui then return resultado end

    local ui = gui:FindFirstChild("UI")
    if not ui then return resultado end

    -- caminho: UI.Frames.Index.Frame
    local frames = ui:FindFirstChild("Frames")
    if not frames then return resultado end
    local index = frames:FindFirstChild("Index")
    if not index then return resultado end
    local frame = index:FindFirstChild("Frame")
    if not frame then return resultado end

    -- itera pelas pastas de mutação (Gold, Diamond, Rainbow, Galaxy...)
    for _, mutacaoFolder in ipairs(frame:GetChildren()) do
        local mutacaoNome = mutacaoFolder.Name -- "Gold", "Diamond", etc.
        if MutacaoCores[mutacaoNome] then      -- só processa mutações conhecidas
            for _, brainrotFrame in ipairs(mutacaoFolder:GetChildren()) do
                if brainrotFrame:IsA("Frame") then
                    -- pega o TextLabel BrainrotName
                    local lblNome = brainrotFrame:FindFirstChild("BrainrotName")
                    if lblNome and lblNome:IsA("TextLabel") then
                        table.insert(resultado, {
                            nome    = lblNome.Text,
                            mutacao = mutacaoNome,
                            frameNome = brainrotFrame.Name -- ex: "RhinoToasterino"
                        })
                    end
                end
            end
        end
    end

    return resultado
end

-- // ESTADO
local AlvosSelecionados = {} -- chave: "nomebrainrot|mutacao" -> true
local FarmAtivo = false
local FiltroMutacao = nil -- nil = mostrar todos

-- // UI
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "AutoFarm_Mutacao_V1"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(340, 440)
    Main.Position = UDim2.new(0.5, -170, 0.3, 0)
    Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(0, 255, 127)
    stroke.Thickness = 2

    -- // TÍTULO
    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, -40, 0, 40)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.Text = "AUTO FARM - MUTAÇÕES"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16

    -- // FECHAR
    local CloseBtn = Instance.new("TextButton", Main)
    CloseBtn.Size = UDim2.fromOffset(30, 30)
    CloseBtn.Position = UDim2.new(1, -35, 0, 7)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.new(1,1,1)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 14
    CloseBtn.BorderSizePixel = 0
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
    CloseBtn.MouseEnter:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(180,30,30) end)
    CloseBtn.MouseLeave:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40) end)
    CloseBtn.MouseButton1Click:Connect(function() sg:Destroy(); FarmAtivo = false end)

    -- // TOGGLE FARM
    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 38)
    Toggle.Position = UDim2.new(0.05, 0, 0, 44)
    Toggle.Text = "AUTO FARM: OFF"
    Toggle.BackgroundColor3 = Color3.fromRGB(35,35,35)
    Toggle.TextColor3 = Color3.new(1,1,1)
    Toggle.Font = Enum.Font.GothamBold
    Toggle.TextSize = 15
    Toggle.BorderSizePixel = 0
    Instance.new("UICorner", Toggle)

    -- // LABEL FILTRO
    local FiltroLabel = Instance.new("TextLabel", Main)
    FiltroLabel.Size = UDim2.new(0.9, 0, 0, 16)
    FiltroLabel.Position = UDim2.new(0.05, 0, 0, 88)
    FiltroLabel.Text = "FILTRAR POR MUTAÇÃO:"
    FiltroLabel.TextColor3 = Color3.fromRGB(160,160,160)
    FiltroLabel.BackgroundTransparency = 1
    FiltroLabel.Font = Enum.Font.GothamBold
    FiltroLabel.TextSize = 11
    FiltroLabel.TextXAlignment = Enum.TextXAlignment.Left

    -- // BOTÕES DE MUTAÇÃO (scroll horizontal)
    local MutFrame = Instance.new("ScrollingFrame", Main)
    MutFrame.Size = UDim2.new(0.9, 0, 0, 38)
    MutFrame.Position = UDim2.new(0.05, 0, 0, 107)
    MutFrame.BackgroundTransparency = 1
    MutFrame.ScrollBarThickness = 3
    MutFrame.ScrollingDirection = Enum.ScrollingDirection.X
    MutFrame.CanvasSize = UDim2.new(0, 400, 0, 0)
    local mutLayout = Instance.new("UIListLayout", MutFrame)
    mutLayout.FillDirection = Enum.FillDirection.Horizontal
    mutLayout.Padding = UDim.new(0, 5)

    -- variável para controlar botões e lista (declaradas antes do uso abaixo)
    local listFrame
    local botoesMut = {}

    local function AtualizarVisibilidadeLista()
        if not listFrame then return end
        for _, child in ipairs(listFrame:GetChildren()) do
            if child:IsA("Frame") then
                if FiltroMutacao == nil then
                    child.Visible = true
                else
                    child.Visible = (child:GetAttribute("Mutacao") == FiltroMutacao)
                end
            end
        end
    end

    local function AtualizarBotoesMut()
        for key, btn in pairs(botoesMut) do
            if key == "__todos__" then
                btn.BackgroundColor3 = (FiltroMutacao == nil)
                    and Color3.fromRGB(0,180,90)
                    or Color3.fromRGB(40,40,40)
            else
                local cor = MutacaoCores[key] or Color3.fromRGB(80,80,80)
                btn.BackgroundColor3 = (FiltroMutacao == key)
                    and cor
                    or Color3.fromRGB(40,40,40)
            end
        end
    end

    -- botão TODOS
    local btnTodos = Instance.new("TextButton", MutFrame)
    btnTodos.Size = UDim2.fromOffset(58, 28)
    btnTodos.Text = "TODOS"
    btnTodos.Font = Enum.Font.GothamBold
    btnTodos.TextSize = 10
    btnTodos.TextColor3 = Color3.new(1,1,1)
    btnTodos.BackgroundColor3 = Color3.fromRGB(0,180,90)
    btnTodos.BorderSizePixel = 0
    Instance.new("UICorner", btnTodos).CornerRadius = UDim.new(0, 5)
    botoesMut["__todos__"] = btnTodos
    btnTodos.MouseButton1Click:Connect(function()
        FiltroMutacao = nil
        AtualizarBotoesMut()
        AtualizarVisibilidadeLista()
    end)

    -- botões por mutação
    for mutNome, mutCor in pairs(MutacaoCores) do
        local btn = Instance.new("TextButton", MutFrame)
        btn.Size = UDim2.fromOffset(72, 28)
        btn.Text = mutNome:upper()
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 10
        btn.TextColor3 = Color3.new(1,1,1)
        btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
        btn.BorderSizePixel = 0
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
        botoesMut[mutNome] = btn
        btn.MouseButton1Click:Connect(function()
            FiltroMutacao = mutNome
            AtualizarBotoesMut()
            AtualizarVisibilidadeLista()
        end)
    end

    -- // BOTÃO ATUALIZAR LISTA
    local RefreshBtn = Instance.new("TextButton", Main)
    RefreshBtn.Size = UDim2.new(0.9, 0, 0, 26)
    RefreshBtn.Position = UDim2.new(0.05, 0, 0, 150)
    RefreshBtn.Text = "↺ ATUALIZAR INVENTÁRIO"
    RefreshBtn.Font = Enum.Font.GothamBold
    RefreshBtn.TextSize = 12
    RefreshBtn.TextColor3 = Color3.new(1,1,1)
    RefreshBtn.BackgroundColor3 = Color3.fromRGB(30,60,90)
    RefreshBtn.BorderSizePixel = 0
    Instance.new("UICorner", RefreshBtn).CornerRadius = UDim.new(0, 6)

    -- // LISTA DE BRAINROTS
    listFrame = Instance.new("ScrollingFrame", Main)
    listFrame.Size = UDim2.new(0.9, 0, 0, 200)
    listFrame.Position = UDim2.new(0.05, 0, 0, 182)
    listFrame.BackgroundColor3 = Color3.fromRGB(22,22,22)
    listFrame.ScrollBarThickness = 4
    listFrame.BorderSizePixel = 0
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    Instance.new("UICorner", listFrame)
    local listLayout = Instance.new("UIListLayout", listFrame)

    local function UpdateTitle()
        local count = 0
        for _ in pairs(AlvosSelecionados) do count += 1 end
        if count == 0 then Title.Text = "AUTO FARM - MUTAÇÕES"
        elseif count == 1 then Title.Text = "1 SELECIONADO"
        else Title.Text = count .. " SELECIONADOS" end
    end

    local function PopularLista()
        -- limpa lista atual
        for _, c in ipairs(listFrame:GetChildren()) do
            if c:IsA("Frame") then c:Destroy() end
        end
        AlvosSelecionados = {}
        UpdateTitle()

        local brainrots = GetBrainrotsDoInventario()

        if #brainrots == 0 then
            local aviso = Instance.new("TextLabel", listFrame)
            aviso.Size = UDim2.new(1, 0, 0, 40)
            aviso.Text = "Nenhum brainrot encontrado!\nVerifique o caminho da UI."
            aviso.TextColor3 = Color3.fromRGB(200, 80, 80)
            aviso.BackgroundTransparency = 1
            aviso.Font = Enum.Font.Gotham
            aviso.TextSize = 12
            aviso.TextWrapped = true
            listFrame.CanvasSize = UDim2.new(0, 0, 0, 40)
            return
        end

        listFrame.CanvasSize = UDim2.new(0, 0, 0, #brainrots * 38)

        for _, dado in ipairs(brainrots) do
            local chave = dado.nome:lower() .. "|" .. dado.mutacao

            local row = Instance.new("Frame", listFrame)
            row.Size = UDim2.new(1, 0, 0, 38)
            row.BackgroundColor3 = Color3.fromRGB(28,28,28)
            row.BorderSizePixel = 0
            row:SetAttribute("Mutacao", dado.mutacao)

            -- badge de mutação (cor)
            local badge = Instance.new("Frame", row)
            badge.Size = UDim2.fromOffset(6, 38)
            badge.Position = UDim2.new(0, 0, 0, 0)
            badge.BackgroundColor3 = MutacaoCores[dado.mutacao] or Color3.fromRGB(100,100,100)
            badge.BorderSizePixel = 0

            -- nome do brainrot
            local lblNome = Instance.new("TextLabel", row)
            lblNome.Size = UDim2.new(0.65, 0, 1, 0)
            lblNome.Position = UDim2.new(0, 10, 0, 0)
            lblNome.Text = dado.nome
            lblNome.Font = Enum.Font.Gotham
            lblNome.TextSize = 13
            lblNome.TextColor3 = Color3.fromRGB(200,200,200)
            lblNome.BackgroundTransparency = 1
            lblNome.TextXAlignment = Enum.TextXAlignment.Left
            lblNome.TextTruncate = Enum.TextTruncate.AtEnd

            -- tag de mutação
            local lblMut = Instance.new("TextLabel", row)
            lblMut.Size = UDim2.new(0.3, 0, 1, 0)
            lblMut.Position = UDim2.new(0.68, 0, 0, 0)
            lblMut.Text = dado.mutacao
            lblMut.Font = Enum.Font.GothamBold
            lblMut.TextSize = 12
            lblMut.TextColor3 = MutacaoCores[dado.mutacao] or Color3.fromRGB(150,150,150)
            lblMut.BackgroundTransparency = 1
            lblMut.TextXAlignment = Enum.TextXAlignment.Left

            -- botão invisível por cima
            local btn = Instance.new("TextButton", row)
            btn.Size = UDim2.new(1, 0, 1, 0)
            btn.BackgroundTransparency = 1
            btn.Text = ""
            btn.ZIndex = 5

            btn.MouseButton1Click:Connect(function()
                if AlvosSelecionados[chave] then
                    AlvosSelecionados[chave] = nil
                    row.BackgroundColor3 = Color3.fromRGB(28,28,28)
                    lblNome.TextColor3 = Color3.fromRGB(200,200,200)
                else
                    AlvosSelecionados[chave] = { nome = dado.nome, mutacao = dado.mutacao }
                    row.BackgroundColor3 = Color3.fromRGB(20,45,30)
                    lblNome.TextColor3 = Color3.fromRGB(0,255,127)
                end
                UpdateTitle()
            end)

            -- aplica filtro atual
            if FiltroMutacao ~= nil and dado.mutacao ~= FiltroMutacao then
                row.Visible = false
            end
        end
    end

    RefreshBtn.MouseButton1Click:Connect(PopularLista)
    PopularLista() -- carrega na inicialização

    -- // ARRASTAR
    local dragging, dragInput, dragStart, startPos
    Main.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true; dragStart = input.Position; startPos = Main.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    Main.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    return Toggle
end

local ToggleBtn = CreateUI()

ToggleBtn.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    ToggleBtn.Text = FarmAtivo and "FARMANDO... (ON)" or "AUTO FARM: OFF"
    ToggleBtn.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0,100,50) or Color3.fromRGB(35,35,35)
end)

-- // BUSCA PROMPT — compara nome do prompt com nome do brainrot E mutação
local function FindPromptDoAlvo()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local nomeObj = (obj.Parent and obj.Parent.Name or ""):lower()
            local textoPrompt = (obj.ObjectText or ""):lower()
            for chave, dado in pairs(AlvosSelecionados) do
                local nomeBusca = dado.nome:lower()
                local mutBusca  = dado.mutacao:lower()
                -- tenta achar o brainrot E a mutação no nome/texto do prompt
                if (nomeObj:find(nomeBusca) or textoPrompt:find(nomeBusca)) then
                    -- verifica mutação se o nome do objeto pai ou avô contiver
                    local paiNome = (obj.Parent and obj.Parent.Parent and obj.Parent.Parent.Name or ""):lower()
                    if paiNome:find(mutBusca) or nomeObj:find(mutBusca) or textoPrompt:find(mutBusca) then
                        return obj
                    end
                end
            end
        end
    end
    return nil
end

-- // LOOP PRINCIPAL
task.spawn(function()
    while true do
        task.wait(0.1)
        if not FarmAtivo or next(AlvosSelecionados) == nil then continue end

        local prompt = FindPromptDoAlvo()
        if not prompt then continue end

        local item = prompt.Parent
        local targetPart = item:IsA("BasePart") and item or item:FindFirstChildWhichIsA("BasePart")
        if not targetPart then continue end

        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then continue end

        pcall(function()
            hrp.CFrame = targetPart.CFrame * CFrame.new(0, 2, 0)
            task.wait(0.05)
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
            task.wait(0.2)
        end)
    end
end)
