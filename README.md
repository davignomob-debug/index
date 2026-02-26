local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local MutacaoCores = {
    Gold    = Color3.fromRGB(255, 215, 0),
    Diamond = Color3.fromRGB(0, 220, 255),
    Rainbow = Color3.fromRGB(255, 100, 200),
    Galaxy  = Color3.fromRGB(160, 80, 255),
}

-- // NORMALIZAÇÃO
local function NormalizarTexto(t)
    t = tostring(t or "")
    t = t:gsub("%s+", " ")
    t = t:match("^%s*(.-)%s*$") or t
    return t
end

-- // INVENTÁRIO
local function GetBrainrotsDoInventario()
    local resultado = {}
    local gui = LocalPlayer:FindFirstChild("PlayerGui")
    if not gui then return resultado end
    local ui = gui:FindFirstChild("UI")
    if not ui then return resultado end
    local frames = ui:FindFirstChild("Frames")
    if not frames then return resultado end
    local index = frames:FindFirstChild("Index")
    if not index then return resultado end
    local frame = index:FindFirstChild("Frame")
    if not frame then return resultado end

    for _, mutFolder in ipairs(frame:GetChildren()) do
        if MutacaoCores[mutFolder.Name] then
            for _, brFrame in ipairs(mutFolder:GetChildren()) do
                if brFrame:IsA("Frame") then
                    local lbl = brFrame:FindFirstChild("BrainrotName")
                    if lbl and lbl:IsA("TextLabel") and lbl.Text ~= "" then
                        table.insert(resultado, {
                            nome    = NormalizarTexto(lbl.Text),
                            mutacao = mutFolder.Name,
                        })
                    end
                end
            end
        end
    end
    return resultado
end

local AlvosSelecionados = {}
local FarmAtivo         = false
local FiltroMutacao     = nil

-- // UI
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "AutoFarm_Mutacao_V2"

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(330, 0)
Main.AutomaticSize = Enum.AutomaticSize.Y
Main.Position = UDim2.new(0.5, -165, 0.25, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 255, 127)
stroke.Thickness = 2

local pad = Instance.new("UIPadding", Main)
pad.PaddingTop    = UDim.new(0, 44)
pad.PaddingBottom = UDim.new(0, 10)
pad.PaddingLeft   = UDim.new(0, 10)
pad.PaddingRight  = UDim.new(0, 10)

local mainLayout = Instance.new("UIListLayout", Main)
mainLayout.FillDirection       = Enum.FillDirection.Vertical
mainLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
mainLayout.SortOrder           = Enum.SortOrder.LayoutOrder
mainLayout.Padding             = UDim.new(0, 6)

-- // TOPBAR
local TopBar = Instance.new("Frame", Main)
TopBar.Size                = UDim2.new(1, 0, 0, 40)
TopBar.Position            = UDim2.new(0, 0, 0, 0)
TopBar.BackgroundTransparency = 1
TopBar.ZIndex              = 10

local Title = Instance.new("TextLabel", TopBar)
Title.Size                 = UDim2.new(1, -44, 1, 0)
Title.Position             = UDim2.new(0, 10, 0, 0)
Title.Text                 = "AUTO FARM - MUTAÇÕES"
Title.TextColor3           = Color3.new(1,1,1)
Title.BackgroundTransparency = 1
Title.Font                 = Enum.Font.GothamBold
Title.TextSize             = 15
Title.TextXAlignment       = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size              = UDim2.fromOffset(28, 28)
CloseBtn.Position          = UDim2.new(1, -34, 0, 6)
CloseBtn.Text              = "X"
CloseBtn.TextColor3        = Color3.new(1,1,1)
CloseBtn.BackgroundColor3  = Color3.fromRGB(40,40,40)
CloseBtn.Font              = Enum.Font.GothamBold
CloseBtn.TextSize          = 13
CloseBtn.BorderSizePixel   = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 5)
CloseBtn.MouseEnter:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(180,30,30) end)
CloseBtn.MouseLeave:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40) end)
CloseBtn.MouseButton1Click:Connect(function() FarmAtivo = false; sg:Destroy() end)

local function NewItem(h)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, h)
    f.BackgroundTransparency = 1
    f.BorderSizePixel = 0
    f.Parent = Main
    return f
end

-- // TOGGLE
local toggleRow = NewItem(38)
local Toggle = Instance.new("TextButton", toggleRow)
Toggle.Size             = UDim2.new(1, 0, 1, 0)
Toggle.Text             = "AUTO FARM: OFF"
Toggle.BackgroundColor3 = Color3.fromRGB(35,35,35)
Toggle.TextColor3       = Color3.new(1,1,1)
Toggle.Font             = Enum.Font.GothamBold
Toggle.TextSize         = 15
Toggle.BorderSizePixel  = 0
Instance.new("UICorner", Toggle)
Toggle.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    Toggle.Text             = FarmAtivo and "FARMANDO... (ON)" or "AUTO FARM: OFF"
    Toggle.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0,100,50) or Color3.fromRGB(35,35,35)
end)

-- // LABEL FILTRO
local lblFiltro = Instance.new("TextLabel", NewItem(16))
lblFiltro.Size               = UDim2.new(1, 0, 1, 0)
lblFiltro.Text               = "FILTRAR POR MUTAÇÃO:"
lblFiltro.TextColor3         = Color3.fromRGB(160,160,160)
lblFiltro.BackgroundTransparency = 1
lblFiltro.Font               = Enum.Font.GothamBold
lblFiltro.TextSize           = 11
lblFiltro.TextXAlignment     = Enum.TextXAlignment.Left

-- // BOTÕES MUTAÇÃO
local mutRow = NewItem(32)
local MutFrame = Instance.new("ScrollingFrame", mutRow)
MutFrame.Size               = UDim2.new(1, 0, 1, 0)
MutFrame.BackgroundTransparency = 1
MutFrame.ScrollBarThickness = 3
MutFrame.ScrollingDirection = Enum.ScrollingDirection.X
MutFrame.CanvasSize         = UDim2.new(0, 380, 0, 0)
MutFrame.BorderSizePixel    = 0
local mutLayout = Instance.new("UIListLayout", MutFrame)
mutLayout.FillDirection = Enum.FillDirection.Horizontal
mutLayout.Padding       = UDim.new(0, 5)

local listFrame
local botoesMut = {}

local function AtualizarVisLista()
    if not listFrame then return end
    for _, c in ipairs(listFrame:GetChildren()) do
        if c:IsA("Frame") then
            c.Visible = FiltroMutacao == nil or c:GetAttribute("Mutacao") == FiltroMutacao
        end
    end
end

local function AtualizarBotoesMut()
    for key, btn in pairs(botoesMut) do
        if key == "todos" then
            btn.BackgroundColor3 = FiltroMutacao == nil and Color3.fromRGB(0,180,90) or Color3.fromRGB(40,40,40)
        else
            local cor = MutacaoCores[key] or Color3.fromRGB(80,80,80)
            btn.BackgroundColor3 = FiltroMutacao == key and cor or Color3.fromRGB(40,40,40)
        end
    end
end

local function MakeMutBtn(parent, label, w, onClick)
    local btn = Instance.new("TextButton", parent)
    btn.Size             = UDim2.fromOffset(w, 26)
    btn.Text             = label
    btn.Font             = Enum.Font.GothamBold
    btn.TextSize         = 10
    btn.TextColor3       = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    btn.BorderSizePixel  = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
    btn.MouseButton1Click:Connect(onClick)
    return btn
end

local b = MakeMutBtn(MutFrame, "TODOS", 55, function()
    FiltroMutacao = nil; AtualizarBotoesMut(); AtualizarVisLista()
end)
b.BackgroundColor3 = Color3.fromRGB(0,180,90)
botoesMut["todos"] = b

for mutNome, _ in pairs(MutacaoCores) do
    botoesMut[mutNome] = MakeMutBtn(MutFrame, mutNome:upper(), 70, function()
        FiltroMutacao = mutNome; AtualizarBotoesMut(); AtualizarVisLista()
    end)
end

-- // BOTÃO ATUALIZAR
local refRow = NewItem(28)
local RefreshBtn = Instance.new("TextButton", refRow)
RefreshBtn.Size             = UDim2.new(1, 0, 1, 0)
RefreshBtn.Text             = "↺ ATUALIZAR INVENTÁRIO"
RefreshBtn.Font             = Enum.Font.GothamBold
RefreshBtn.TextSize         = 12
RefreshBtn.TextColor3       = Color3.new(1,1,1)
RefreshBtn.BackgroundColor3 = Color3.fromRGB(30,60,90)
RefreshBtn.BorderSizePixel  = 0
Instance.new("UICorner", RefreshBtn).CornerRadius = UDim.new(0, 5)

-- // LISTA SCROLL
local listRow = NewItem(210)
listFrame = Instance.new("ScrollingFrame", listRow)
listFrame.Size               = UDim2.new(1, 0, 1, 0)
listFrame.BackgroundColor3   = Color3.fromRGB(22,22,22)
listFrame.ScrollBarThickness = 4
listFrame.BorderSizePixel    = 0
listFrame.CanvasSize         = UDim2.new(0, 0, 0, 0)
Instance.new("UICorner", listFrame)
Instance.new("UIListLayout", listFrame)

local function UpdateTitle()
    local n = 0
    for _ in pairs(AlvosSelecionados) do n += 1 end
    Title.Text = n == 0 and "AUTO FARM - MUTAÇÕES"
        or n == 1 and "1 SELECIONADO"
        or n .. " SELECIONADOS"
end

local function PopularLista()
    for _, c in ipairs(listFrame:GetChildren()) do
        if c:IsA("Frame") then c:Destroy() end
    end
    AlvosSelecionados = {}
    UpdateTitle()

    local lista = GetBrainrotsDoInventario()

    if #lista == 0 then
        local av = Instance.new("TextLabel", listFrame)
        av.Size               = UDim2.new(1, 0, 0, 40)
        av.Text               = "Nenhum brainrot encontrado!"
        av.TextColor3         = Color3.fromRGB(200,80,80)
        av.BackgroundTransparency = 1
        av.Font               = Enum.Font.Gotham
        av.TextSize           = 12
        av.TextWrapped        = true
        listFrame.CanvasSize  = UDim2.new(0,0,0,40)
        return
    end

    listFrame.CanvasSize = UDim2.new(0, 0, 0, #lista * 36)

    for _, dado in ipairs(lista) do
        local chave = dado.nome:lower() .. "|" .. dado.mutacao

        local row = Instance.new("Frame", listFrame)
        row.Size             = UDim2.new(1, 0, 0, 36)
        row.BackgroundColor3 = Color3.fromRGB(28,28,28)
        row.BorderSizePixel  = 0
        row:SetAttribute("Mutacao", dado.mutacao)

        local badge = Instance.new("Frame", row)
        badge.Size             = UDim2.fromOffset(5, 36)
        badge.BackgroundColor3 = MutacaoCores[dado.mutacao] or Color3.fromRGB(100,100,100)
        badge.BorderSizePixel  = 0

        local lblN = Instance.new("TextLabel", row)
        lblN.Size             = UDim2.new(0.63, 0, 1, 0)
        lblN.Position         = UDim2.new(0, 9, 0, 0)
        lblN.Text             = dado.nome
        lblN.Font             = Enum.Font.Gotham
        lblN.TextSize         = 13
        lblN.TextColor3       = Color3.fromRGB(200,200,200)
        lblN.BackgroundTransparency = 1
        lblN.TextXAlignment   = Enum.TextXAlignment.Left
        lblN.TextTruncate     = Enum.TextTruncate.AtEnd

        local lblM = Instance.new("TextLabel", row)
        lblM.Size             = UDim2.new(0.33, 0, 1, 0)
        lblM.Position         = UDim2.new(0.66, 0, 0, 0)
        lblM.Text             = dado.mutacao
        lblM.Font             = Enum.Font.GothamBold
        lblM.TextSize         = 11
        lblM.TextColor3       = MutacaoCores[dado.mutacao] or Color3.fromRGB(150,150,150)
        lblM.BackgroundTransparency = 1
        lblM.TextXAlignment   = Enum.TextXAlignment.Left

        local btn = Instance.new("TextButton", row)
        btn.Size              = UDim2.new(1, 0, 1, 0)
        btn.BackgroundTransparency = 1
        btn.Text              = ""
        btn.ZIndex            = 5
        btn.MouseButton1Click:Connect(function()
            if AlvosSelecionados[chave] then
                AlvosSelecionados[chave] = nil
                row.BackgroundColor3 = Color3.fromRGB(28,28,28)
                lblN.TextColor3      = Color3.fromRGB(200,200,200)
            else
                AlvosSelecionados[chave] = { nome = dado.nome, mutacao = dado.mutacao }
                row.BackgroundColor3 = Color3.fromRGB(20,45,30)
                lblN.TextColor3      = Color3.fromRGB(0,255,127)
            end
            UpdateTitle()
        end)

        row.Visible = FiltroMutacao == nil or dado.mutacao == FiltroMutacao
    end
end

RefreshBtn.MouseButton1Click:Connect(PopularLista)
PopularLista()

-- // ARRASTAR
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = Main.Position
    end
end)
Main.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement then dragInput = i end
end)
UserInputService.InputChanged:Connect(function(i)
    if i == dragInput and dragging then
        local d = i.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- // VERIFICAÇÃO DE PROMPT (nome + mutação)
local function VerificaPrompt(prompt, paiObj)
    local nomePai    = paiObj.Name
    local nomeAvo    = paiObj.Parent and paiObj.Parent.Name or ""
    local textoPrompt = prompt.ObjectText or ""
    local acaoPrompt  = prompt.ActionText or ""
    local caminhoFull = prompt:GetFullName() or ""

    local campos = {
        NormalizarTexto(nomePai):lower(),
        NormalizarTexto(nomeAvo):lower(),
        NormalizarTexto(textoPrompt):lower(),
        NormalizarTexto(acaoPrompt):lower(),
        NormalizarTexto(caminhoFull):lower(),
    }

    local camposSemEspaco = {}
    for _, texto in ipairs(campos) do
        camposSemEspaco[#camposSemEspaco + 1] = texto:gsub("%s+", "")
    end

    for _, dado in pairs(AlvosSelecionados) do
        local nomeBusca       = NormalizarTexto(dado.nome):lower()
        local nomeBuscaSemEsp = nomeBusca:gsub("%s+", "")
        local mutBusca        = dado.mutacao:lower()
        local mutBuscaSemEsp  = mutBusca:gsub("%s+", "")

        local temNome = false
        local temMut  = false

        for i, texto in ipairs(campos) do
            local textoSemEsp = camposSemEspaco[i]

            if not temNome then
                if texto:find(nomeBusca, 1, true) or textoSemEsp:find(nomeBuscaSemEsp, 1, true) then
                    temNome = true
                end
            end

            if not temMut then
                if texto:find(mutBusca, 1, true) or textoSemEsp:find(mutBuscaSemEsp, 1, true) then
                    temMut = true
                end
            end

            if temNome and temMut then return true end
        end
    end

    return false
end

-- // BUSCA DO PROMPT
local function FindPromptDoAlvo()
    local folder = workspace:FindFirstChild("Client")
        and workspace.Client:FindFirstChild("Path")
        and workspace.Client.Path:FindFirstChild("Brainrots")

    if folder then
        for _, obj in ipairs(folder:GetChildren()) do
            local hrpObj = obj:FindFirstChild("HumanoidRootPart")
            if not hrpObj then continue end
            local prompt = hrpObj:FindFirstChildOfClass("ProximityPrompt")
            if not prompt then continue end
            if VerificaPrompt(prompt, obj) then
                return prompt, hrpObj
            end
        end
    end

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local part = obj.Parent
            if VerificaPrompt(obj, part) then
                return obj, part
            end
        end
    end

    return nil, nil
end

-- // LOOP PRINCIPAL
task.spawn(function()
    while true do
        task.wait(0.1)
        if not FarmAtivo or next(AlvosSelecionados) == nil then continue end

        local prompt, hrpObj = FindPromptDoAlvo()
        if not prompt then continue end

        local char      = LocalPlayer.Character
        local hrpPlayer = char and char:FindFirstChild("HumanoidRootPart")
        if not hrpPlayer then continue end

        pcall(function()
            hrpPlayer.CFrame = hrpObj.CFrame * CFrame.new(0, 0, 3)
            task.wait(0.1)
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
            task.wait(0.2)
        end)
    end
end)
