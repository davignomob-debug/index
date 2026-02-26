local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "AutoFarm_V2"

local Brainrots = {
    "Tim Cheese", "LiriliLarila", "Fluri Flura", "Cacto Hipopotamo",
    "Pipi Potato", "Tric Trac Barabum", "Burbaloni Loliloli", "Boneca Ambalabu",
    "Trippi Troppi", "Svinina Bombardino", "Bambini Crostini", "Avocadini Guffo",
    "Bandito Bobrito", "Tatatata Sahur", "Gangster Footera", "Tung Tung Sahur",
    "Cappuccino Assassino", "Pipi Kiwi", "Orangutini Ananassini", "Talpa Di Fero",
    "Espresso Signora", "Brr Brr Patapim", "Rhino Toasterino", "Brr Bicus Dicus",
    "Strawberrelli Flamingelli", "Bananito Delfinito", "Balerina Capucina",
    "Glorbo Fruttodrillo", "Blueberrinni Octopusini", "Chimpanzini Bananini",
    "Bombardiro Crocodilo", "Elefanto Cocofanto", "Bombombini Gusini",
    "Pandaccini Bananini", "Chef Crabracadabra", "Gorillo Watermelondrillo",
    "Frigo Camelo", "Girafa Celestre", "Ganganzelli Trulala", "Tigroligre Frutonni",
    "Tralalerodon", "Esok", "La Vaca", "Strawberry",
    "Capitano Clash Warnini", "Meowl", "Garama & Madundung"
}

local AlvosSelecionados = {}
local FarmAtivo = false
local ValorMinimo = 0
local dragging, dragInput, dragStart, startPos = false, nil, nil, nil

local function ParseValor(texto)
    if not texto or texto == "" then return 0 end
    local s = texto:upper():gsub("%s",""):gsub("%$",""):gsub("/S","")
    local num, suf = s:match("([%d%.]+)([KMB]?)")
    if not num then return 0 end
    num = tonumber(num) or 0
    if suf == "K" then num = num * 1e3
    elseif suf == "M" then num = num * 1e6
    elseif suf == "B" then num = num * 1e9 end
    return num
end

local function GetValorPS(obj)
    for _, c in pairs(obj:GetDescendants()) do
        if c:IsA("TextLabel") and c.Text:find("/s") then
            return ParseValor(c.Text)
        end
    end
    return 0
end

-- // UI PRINCIPAL
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(380, 400)
Main.Position = UDim2.new(0.5, -190, 0.3, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 255, 127)
stroke.Thickness = 2

local TopBar = Instance.new("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -45, 1, 0)
Title.Position = UDim2.new(0, 8, 0, 0)
Title.Text = "AUTO FARM INFINITO"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 17
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.fromOffset(28, 28)
CloseBtn.Position = UDim2.new(1, -34, 0, 6)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseEnter:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(180,30,30) end)
CloseBtn.MouseLeave:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40) end)
CloseBtn.MouseButton1Click:Connect(function() FarmAtivo = false; sg:Destroy() end)

-- // LAYOUT INTERNO (evita espaços manuais)
local layout = Instance.new("UIListLayout", Main)
layout.Padding = UDim.new(0, 6)
layout.FillDirection = Enum.FillDirection.Vertical
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
local pad = Instance.new("UIPadding", Main)
pad.PaddingTop = UDim.new(0, 44)
pad.PaddingBottom = UDim.new(0, 8)
pad.PaddingLeft = UDim.new(0, 12)
pad.PaddingRight = UDim.new(0, 12)

local function MakeRow(h)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, h)
    f.BackgroundTransparency = 1
    f.Parent = Main
    return f
end

-- // TOGGLE
local Toggle = Instance.new("TextButton", MakeRow(40))
Toggle.Size = UDim2.new(1, 0, 1, 0)
Toggle.Text = "AUTO FARM: OFF"
Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.Font = Enum.Font.GothamBold
Toggle.TextSize = 16
Toggle.BorderSizePixel = 0
Instance.new("UICorner", Toggle)
Toggle.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    Toggle.Text = FarmAtivo and "FARMANDO... (ON)" or "AUTO FARM: OFF"
    Toggle.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0,100,50) or Color3.fromRGB(35,35,35)
end)

-- // LABEL + INPUT VALOR
local lblVal = Instance.new("TextLabel", MakeRow(18))
lblVal.Size = UDim2.new(1, 0, 1, 0)
lblVal.Text = "VALOR MÍNIMO /s  (ex: 500K, 1M, 1B):"
lblVal.TextColor3 = Color3.fromRGB(0, 255, 127)
lblVal.BackgroundTransparency = 1
lblVal.Font = Enum.Font.GothamBold
lblVal.TextSize = 12
lblVal.TextXAlignment = Enum.TextXAlignment.Left

local InputValor = Instance.new("TextBox", MakeRow(34))
InputValor.Size = UDim2.new(1, 0, 1, 0)
InputValor.PlaceholderText = "vazio = qualquer valor"
InputValor.Text = ""
InputValor.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
InputValor.TextColor3 = Color3.new(1, 1, 1)
InputValor.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
InputValor.Font = Enum.Font.Gotham
InputValor.TextSize = 13
InputValor.BorderSizePixel = 0
InputValor.ClearTextOnFocus = false
Instance.new("UICorner", InputValor)
InputValor.FocusLost:Connect(function()
    ValorMinimo = ParseValor(InputValor.Text)
    InputValor.TextColor3 = ValorMinimo > 0 and Color3.fromRGB(0,255,127) or Color3.new(1,1,1)
end)

-- // LABEL LISTA
local lblList = Instance.new("TextLabel", MakeRow(18))
lblList.Size = UDim2.new(1, 0, 1, 0)
lblList.Text = "BRAINROTS (vazio = todos):"
lblList.TextColor3 = Color3.fromRGB(0, 255, 127)
lblList.BackgroundTransparency = 1
lblList.Font = Enum.Font.GothamBold
lblList.TextSize = 12
lblList.TextXAlignment = Enum.TextXAlignment.Left

-- // LISTA SCROLL
local rowList = MakeRow(210)
local ListBrainrot = Instance.new("ScrollingFrame", rowList)
ListBrainrot.Size = UDim2.new(1, 0, 1, 0)
ListBrainrot.CanvasSize = UDim2.new(0, 0, 0, #Brainrots * 30)
ListBrainrot.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ListBrainrot.ScrollBarThickness = 4
ListBrainrot.BorderSizePixel = 0
Instance.new("UICorner", ListBrainrot)
Instance.new("UIListLayout", ListBrainrot)

for _, name in ipairs(Brainrots) do
    local b = Instance.new("TextButton", ListBrainrot)
    b.Size = UDim2.new(1, 0, 0, 30)
    b.Text = name
    b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
    b.Font = Enum.Font.Gotham
    b.TextSize = 13
    b.BorderSizePixel = 0
    b.MouseButton1Click:Connect(function()
        local lower = name:lower()
        if AlvosSelecionados[lower] then
            AlvosSelecionados[lower] = nil
            b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
            b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        else
            AlvosSelecionados[lower] = true
            b.TextColor3 = Color3.fromRGB(0, 255, 127)
            b.BackgroundColor3 = Color3.fromRGB(20, 50, 35)
        end
        local count = 0
        for _ in pairs(AlvosSelecionados) do count += 1 end
        Title.Text = count == 0 and "AUTO FARM INFINITO" or count .. " SELECIONADO(S)"
    end)
end

-- // ARRASTAR
TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = Main.Position
    end
end)
TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- // BUSCA PROMPT
local function FindPrompt()
    local folder = workspace:FindFirstChild("Client")
        and workspace.Client:FindFirstChild("Path")
        and workspace.Client.Path:FindFirstChild("Brainrots")
    if not folder then return nil end

    local melhor, melhorValor = nil, -1

    for _, obj in pairs(folder:GetChildren()) do
        -- prompt fica em HumanoidRootPart
        local hrpObj = obj:FindFirstChild("HumanoidRootPart")
        if not hrpObj then continue end
        local prompt = hrpObj:FindFirstChildOfClass("ProximityPrompt")
        if not prompt then continue end

        local texto = (prompt.ObjectText or ""):lower()
        if texto == "" then continue end

        -- checa brainrot selecionado
        local ok = next(AlvosSelecionados) == nil
        if not ok then
            for alvo in pairs(AlvosSelecionados) do
                if texto:find(alvo, 1, true) then ok = true; break end
            end
        end
        if not ok then continue end

        -- checa valor mínimo
        local val = GetValorPS(obj)
        if val < ValorMinimo then continue end

        if val > melhorValor then
            melhorValor = val
            melhor = { prompt = prompt, hrp = hrpObj }
        end
    end

    return melhor
end

-- // LOOP
task.spawn(function()
    while true do
        task.wait(0.1)
        if not FarmAtivo then continue end

        local resultado = FindPrompt()
        if not resultado then continue end

        local char = LocalPlayer.Character
        local hrpPlayer = char and char:FindFirstChild("HumanoidRootPart")
        if not hrpPlayer then continue end

        pcall(function()
            -- teleporta pro HumanoidRootPart do brainrot
            hrpPlayer.CFrame = resultado.hrp.CFrame * CFrame.new(0, 0, 3)
            task.wait(0.1)
            resultado.prompt.HoldDuration = 0
            fireproximityprompt(resultado.prompt)
            task.wait(0.2)
        end)
    end
end)
