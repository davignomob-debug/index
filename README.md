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

    for _, mutacaoFolder in ipairs(frame:GetChildren()) do
        local mutacaoNome = mutacaoFolder.Name
        if MutacaoCores[mutacaoNome] then
            for _, brainrotFrame in ipairs(mutacaoFolder:GetChildren()) do
                if brainrotFrame:IsA("Frame") then
                    local lblNome = brainrotFrame:FindFirstChild("BrainrotName")
                    if lblNome and lblNome:IsA("TextLabel") then
                        table.insert(resultado, {
                            nome    = lblNome.Text,
                            mutacao = mutacaoNome,
                            frameNome = brainrotFrame.Name
                        })
                    end
                end
            end
        end
    end
    return resultado
end

-- // ESTADO
local AlvosSelecionados = {}
local FarmAtivo = false
local FiltroMutacao = nil

-- // UI
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "AutoFarm_Mutacao_V1"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(340, 400)
    Main.Position = UDim2.new(0.5, -170, 0.3, 0)
    Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(0, 255, 127)
    stroke.Thickness = 2

    -- // LAYOUT INTERNO
    local layout = Instance.new("UIListLayout", Main)
    layout.Padding = UDim.new(0, 6)
    layout.FillDirection = Enum.FillDirection.Vertical
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    local pad = Instance.new("UIPadding", Main)
    pad.PaddingTop = UDim.new(0, 40)
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

    -- // TÍTULO
    local Title = Instance.new("TextLabel", MakeRow(30))
    Title.Size = UDim2.new(1, -30, 1, 0)
    Title.Text = "AUTO FARM - MUTAÇÕES"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16
    Title.TextXAlignment = Enum.TextXAlignment.Left

    -- // FECHAR
    local CloseBtn = Instance.new("TextButton", Title.Parent)
    CloseBtn.Size = UDim2.fromOffset(28, 28)
    CloseBtn.Position = UDim2.new(1, -28, 0, 0)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.new(1,1,1)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 14
    CloseBtn.BorderSizePixel = 0
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
    CloseBtn.MouseEnter:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(180,30,30) end)
    CloseBtn.MouseLeave:Connect(function() CloseBtn.BackgroundColor3 =极简版代码，已移除所有不必要的空格和注释，专注于核心功能。
