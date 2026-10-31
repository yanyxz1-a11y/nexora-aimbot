-- LocalScript em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurações de FOV
local FOV_MIN = 60
local FOV_MAX = 300
local FOV_INICIAL = 160
local ESPESSURA = 2

-- Configurações de Smooth (Velocidade/Força do Lock)
local SMOOTH_MIN = 1       -- Mais suave / lento
local SMOOTH_MAX = 30      -- Mais forte / rápido
local SMOOTH_INICIAL = 8

local CORES = {
	Color3.fromRGB(255, 0, 0),     -- Vermelho
	Color3.fromRGB(0, 255, 0),     -- Verde
	Color3.fromRGB(0, 150, 255),   -- Azul
	Color3.fromRGB(255, 255, 0),   -- Amarelo
	Color3.fromRGB(255, 255, 255), -- Branco
}

local LARGURA_PAINEL = 240
local ALTURA_PAINEL = 385
local ALTURA_TITULO = 35

local ativo = false
local mostrarFOV = true
local wallcheck = false
local teamCheck = true       -- ON por padrão: não mira em aliados
local alvo = nil
local mirando = false
local arrastandoSliderFOV = false
local arrastandoSliderSmooth = false
local arrastandoPainel = false
local minimizado = false
local corSelecionada = 1

local tamanhoAtual = FOV_INICIAL
local smoothAtual = SMOOTH_INICIAL

-- ===== GUI principal =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SimpleAimbot"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(LARGURA_PAINEL, ALTURA_PAINEL)
frame.Position = UDim2.fromOffset(20, 200)
frame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
frame.BorderSizePixel = 0
frame.ClipsDescendants = false
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local creditos = Instance.new("TextLabel")
creditos.Size = UDim2.new(1, 0, 0, 18)
creditos.Position = UDim2.new(0, 0, 1, 4)
creditos.BackgroundTransparency = 1
creditos.Text = "Made By yanyxz"
creditos.TextColor3 = Color3.fromRGB(150, 150, 150)
creditos.Font = Enum.Font.Gotham
creditos.TextSize = 13
creditos.Parent = frame

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, -40, 0, ALTURA_TITULO)
titulo.BackgroundTransparency = 1
titulo.Text = "Simple Aimbot"
titulo.TextColor3 = Color3.new(1, 1, 1)
titulo.Font = Enum.Font.GothamBold
titulo.TextSize = 18
titulo.Parent = frame

local btnMin = Instance.new("TextButton")
btnMin.Size = UDim2.fromOffset(28, 24)
btnMin.Position = UDim2.new(1, -34, 0, 5)
btnMin.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
btnMin.BorderSizePixel = 0
btnMin.TextColor3 = Color3.new(1, 1, 1)
btnMin.Font = Enum.Font.GothamBold
btnMin.TextSize = 18
btnMin.Text = "-"
btnMin.Parent = frame
Instance.new("UICorner", btnMin).CornerRadius = UDim.new(0, 6)

local conteudo = Instance.new("Frame")
conteudo.Size = UDim2.new(1, 0, 0, ALTURA_PAINEL - ALTURA_TITULO)
conteudo.Position = UDim2.fromOffset(0, ALTURA_TITULO)
conteudo.BackgroundTransparency = 1
conteudo.Parent = frame

-- Botão Liga/Desliga
local botaoAimbot = Instance.new("TextButton")
botaoAimbot.Size = UDim2.new(0, 200, 0, 32)
botaoAimbot.Position = UDim2.new(0.5, -100, 0, 4)
botaoAimbot.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoAimbot.BorderSizePixel = 0
botaoAimbot.TextColor3 = Color3.new(1, 1, 1)
botaoAimbot.Font = Enum.Font.GothamSemibold
botaoAimbot.TextSize = 15
botaoAimbot.Text = "Aimbot: OFF"
botaoAimbot.Parent = conteudo
Instance.new("UICorner", botaoAimbot).CornerRadius = UDim.new(0, 8)

-- Botão Show FOV
local botaoMostrar = Instance.new("TextButton")
botaoMostrar.Size = UDim2.new(0, 200, 0, 26)
botaoMostrar.Position = UDim2.new(0.5, -100, 0, 42)
botaoMostrar.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoMostrar.BorderSizePixel = 0
botaoMostrar.TextColor3 = Color3.new(1, 1, 1)
botaoMostrar.Font = Enum.Font.GothamSemibold
botaoMostrar.TextSize = 13
botaoMostrar.Text = "Show FOV: ON"
botaoMostrar.Parent = conteudo
Instance.new("UICorner", botaoMostrar).CornerRadius = UDim.new(0, 8)

-- ===== Slider FOV =====
local textoFOV = Instance.new("TextLabel")
textoFOV.Size = UDim2.new(1, 0, 0, 18)
textoFOV.Position = UDim2.fromOffset(0, 74)
textoFOV.BackgroundTransparency = 1
textoFOV.TextColor3 = Color3.new(1, 1, 1)
textoFOV.Font = Enum.Font.Gotham
textoFOV.TextSize = 13
textoFOV.Parent = conteudo

local barraFOV = Instance.new("Frame")
barraFOV.Size = UDim2.new(0, 200, 0, 8)
barraFOV.Position = UDim2.new(0.5, -100, 0, 96)
barraFOV.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
barraFOV.BorderSizePixel = 0
barraFOV.Parent = conteudo
Instance.new("UICorner", barraFOV).CornerRadius = UDim.new(1, 0)

local bolinhaFOV = Instance.new("Frame")
bolinhaFOV.Size = UDim2.fromOffset(16, 16)
bolinhaFOV.AnchorPoint = Vector2.new(0.5, 0.5)
bolinhaFOV.Position = UDim2.new(0, 0, 0.5, 0)
bolinhaFOV.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
bolinhaFOV.BorderSizePixel = 0
bolinhaFOV.Parent = barraFOV
Instance.new("UICorner", bolinhaFOV).CornerRadius = UDim.new(1, 0)

-- ===== Slider Smooth =====
local textoSmooth = Instance.new("TextLabel")
textoSmooth.Size = UDim2.new(1, 0, 0, 18)
textoSmooth.Position = UDim2.fromOffset(0, 112)
textoSmooth.BackgroundTransparency = 1
textoSmooth.TextColor3 = Color3.new(1, 1, 1)
textoSmooth.Font = Enum.Font.Gotham
textoSmooth.TextSize = 13
textoSmooth.Parent = conteudo

local barraSmooth = Instance.new("Frame")
barraSmooth.Size = UDim2.new(0, 200, 0, 8)
barraSmooth.Position = UDim2.new(0.5, -100, 0, 134)
barraSmooth.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
barraSmooth.BorderSizePixel = 0
barraSmooth.Parent = conteudo
Instance.new("UICorner", barraSmooth).CornerRadius = UDim.new(1, 0)

local bolinhaSmooth = Instance.new("Frame")
bolinhaSmooth.Size = UDim2.fromOffset(16, 16)
bolinhaSmooth.AnchorPoint = Vector2.new(0.5, 0.5)
bolinhaSmooth.Position = UDim2.new(0, 0, 0.5, 0)
bolinhaSmooth.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
bolinhaSmooth.BorderSizePixel = 0
bolinhaSmooth.Parent = barraSmooth
Instance.new("UICorner", bolinhaSmooth).CornerRadius = UDim.new(1, 0)

-- Seletor de Cores
local textoCor = Instance.new("TextLabel")
textoCor.Size = UDim2.new(1, 0, 0, 18)
textoCor.Position = UDim2.fromOffset(0, 150)
textoCor.BackgroundTransparency = 1
textoCor.Text = "FOV colour"
textoCor.TextColor3 = Color3.new(1, 1, 1)
textoCor.Font = Enum.Font.Gotham
textoCor.TextSize = 13
textoCor.Parent = conteudo

local swatches = {}
local TAM_SWATCH = 24
local ESPACO = 10
local larguraTotal = #CORES * TAM_SWATCH + (#CORES - 1) * ESPACO
local inicioX = (LARGURA_PAINEL - larguraTotal) / 2

for i, cor in ipairs(CORES) do
	local sw = Instance.new("TextButton")
	sw.Size = UDim2.fromOffset(TAM_SWATCH, TAM_SWATCH)
	sw.Position = UDim2.fromOffset(inicioX + (i - 1) * (TAM_SWATCH + ESPACO), 174)
	sw.BackgroundColor3 = cor
	sw.BorderColor3 = Color3.fromRGB(255, 255, 255)
	sw.BorderSizePixel = 0
	sw.Text = ""
	sw.AutoButtonColor = true
	sw.Parent = conteudo
	Instance.new("UICorner", sw).CornerRadius = UDim.new(1, 0)
	swatches[i] = sw

	sw.MouseButton1Click:Connect(function()
		corSelecionada = i
		contorno.Color = CORES[i]
		for idx, btn in ipairs(swatches) do
			btn.BorderSizePixel = (idx == i) and 2 or 0
		end
	end)
end

-- Botão Wallcheck
local botaoWall = Instance.new("TextButton")
botaoWall.Size = UDim2.new(0, 200, 0, 28)
botaoWall.Position = UDim2.new(0.5, -100, 0, 210)
botaoWall.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoWall.BorderSizePixel = 0
botaoWall.TextColor3 = Color3.new(1, 1, 1)
botaoWall.Font = Enum.Font.GothamSemibold
botaoWall.TextSize = 13
botaoWall.Text = "Wallcheck: OFF"
botaoWall.Parent = conteudo
Instance.new("UICorner", botaoWall).CornerRadius = UDim.new(0, 8)

-- Botão Team Check
local botaoTeam = Instance.new("TextButton")
botaoTeam.Size = UDim2.new(0, 200, 0, 28)
botaoTeam.Position = UDim2.new(0.5, -100, 0, 246)
botaoTeam.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoTeam.BorderSizePixel = 0
botaoTeam.TextColor3 = Color3.new(1, 1, 1)
botaoTeam.Font = Enum.Font.GothamSemibold
botaoTeam.TextSize = 13
botaoTeam.Text = "Team Check: ON"
botaoTeam.Parent = conteudo
Instance.new("UICorner", botaoTeam).CornerRadius = UDim.new(0, 8)

-- ===== Círculo FOV =====
local bola = Instance.new("Frame")
bola.Name = "FOV"
bola.Size = UDim2.fromOffset(tamanhoAtual, tamanhoAtual)
bola.AnchorPoint = Vector2.new(0.5, 0.5)
bola.Position = UDim2.new(0.5, 0, 0.5, 0)
bola.BackgroundTransparency = 1
bola.BorderSizePixel = 0
bola.ZIndex = 5
bola.Visible = false
bola.Parent = screenGui
Instance.new("UICorner", bola).CornerRadius = UDim.new(0.5, 0)

local contorno = Instance.new("UIStroke")
contorno.Color = CORES[corSelecionada]
contorno.Thickness = ESPESSURA
contorno.Parent = bola

local function atualizarVisibilidadeFOV()
	bola.Visible = ativo and mostrarFOV
end

-- ===== Atualização de Sliders =====
local function pctDoMouse(barraTarget: Frame, x: number): number
	return (x - barraTarget.AbsolutePosition.X) / barraTarget.AbsoluteSize.X
end

local function atualizarTamanhoFOV(pct: number)
	pct = math.clamp(pct, 0, 1)
	tamanhoAtual = math.floor(FOV_MIN + (FOV_MAX - FOV_MIN) * pct)
	bola.Size = UDim2.fromOffset(tamanhoAtual, tamanhoAtual)
	bolinhaFOV.Position = UDim2.new(pct, 0, 0.5, 0)
	textoFOV.Text = `FOV: {tamanhoAtual}`
end

local function atualizarSmooth(pct: number)
	pct = math.clamp(pct, 0, 1)
	smoothAtual = math.floor((SMOOTH_MIN + (SMOOTH_MAX - SMOOTH_MIN) * pct) * 10) / 10
	bolinhaSmooth.Position = UDim2.new(pct, 0, 0.5, 0)
	textoSmooth.Text = `Smooth (Força): {smoothAtual}`
end

-- Inicializa valores visuais
atualizarTamanhoFOV((FOV_INICIAL - FOV_MIN) / (FOV_MAX - FOV_MIN))
atualizarSmooth((SMOOTH_INICIAL - SMOOTH_MIN) / (SMOOTH_MAX - SMOOTH_MIN))

-- Eventos Slider FOV
barraFOV.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSliderFOV = true
		atualizarTamanhoFOV(pctDoMouse(barraFOV, input.Position.X))
	end
end)

bolinhaFOV.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSliderFOV = true
	end
end)

-- Eventos Slider Smooth
barraSmooth.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSliderSmooth = true
		atualizarSmooth(pctDoMouse(barraSmooth, input.Position.X))
	end
end)

bolinhaSmooth.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSliderSmooth = true
	end
end)

-- ===== Minimizar / Restaurar =====
btnMin.MouseButton1Click:Connect(function()
	minimizado = not minimizado
	conteudo.Visible = not minimizado
	btnMin.Text = minimizado and "+" or "-"
	frame.Size = minimizado
		and UDim2.fromOffset(LARGURA_PAINEL, ALTURA_TITULO)
		or UDim2.fromOffset(LARGURA_PAINEL, ALTURA_PAINEL)
end)

-- ===== Arrastar Janela =====
local offsetArraste = Vector2.zero

titulo.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoPainel = true
		offsetArraste = Vector2.new(input.Position.X, input.Position.Y) - frame.AbsolutePosition
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input.UserInputType ~= Enum.UserInputType.MouseMovement then return end

	if arrastandoSliderFOV then
		atualizarTamanhoFOV(pctDoMouse(barraFOV, input.Position.X))
	end

	if arrastandoSliderSmooth then
		atualizarSmooth(pctDoMouse(barraSmooth, input.Position.X))
	end

	if arrastandoPainel then
		local vp = camera.ViewportSize
		local x = math.clamp(input.Position.X - offsetArraste.X, 0, vp.X - frame.AbsoluteSize.X)
		local y = math.clamp(input.Position.Y - offsetArraste.Y, 0, vp.Y - frame.AbsoluteSize.Y - 22)
		frame.Position = UDim2.fromOffset(x, y)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSliderFOV = false
		arrastandoSliderSmooth = false
		arrastandoPainel = false
	end
end)

-- ===== Verificação de Time (Team Check) =====
local function ehAliado(outroPlayer: Player): boolean
	if not teamCheck then return false end

	-- Verificação por objeto Team (Roblox padrão)
	if player.Team ~= nil and outroPlayer.Team ~= nil then
		return player.Team == outroPlayer.Team
	end

	-- Verificação por cor do time (usado em muitos jogos de FPS)
	if player.TeamColor ~= nil and outroPlayer.TeamColor ~= nil then
		return player.TeamColor == outroPlayer.TeamColor
	end

	return false
end

-- ===== Wallcheck =====
local function temLinhaDeVisao(p: Player, origem: Vector3, destino: Vector3): boolean
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = { player.Character, p.Character }

	local direcao = destino - origem
	local resultado = workspace:Raycast(origem, direcao, params)
	return resultado == nil
end

local function alvoVisivel(p: Player, cabeca: BasePart): boolean
	if not wallcheck then return true end
	return temLinhaDeVisao(p, camera.CFrame.Position, cabeca.Position)
end

-- ===== Detecção no FOV =====
local function jogadorNoFOV(p: Player): boolean
	-- Ignora aliados se o Team Check estiver ligado
	if ehAliado(p) then return false end

	local char = p.Character
	if not char then return false end

	local hum = char:FindFirstChildOfClass("Humanoid")
	local cabeca = char:FindFirstChild("Head")
	if not hum or hum.Health <= 0 or not cabeca then return false end

	local pos, naFrente = camera:WorldToViewportPoint(cabeca.Position)
	if not naFrente then return false end

	local centro = bola.AbsolutePosition + (bola.AbsoluteSize / 2)
	local raio = bola.AbsoluteSize.X / 2
	if (Vector2.new(pos.X, pos.Y) - centro).Magnitude > raio then return false end

	return alvoVisivel(p, cabeca)
end

local function atualizarAlvo()
	if alvo and jogadorNoFOV(alvo) then return end

	alvo = nil
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= player and jogadorNoFOV(p) then
			alvo = p
			return
		end
	end
end

-- ===== Botões =====
botaoAimbot.MouseButton1Click:Connect(function()
	ativo = not ativo
	botaoAimbot.Text = ativo and "Aimbot: ON" or "Aimbot: OFF"
	atualizarVisibilidadeFOV()

	if not ativo then
		alvo = nil
		mirando = false
	end
end)

botaoMostrar.MouseButton1Click:Connect(function()
	mostrarFOV = not mostrarFOV
	botaoMostrar.Text = mostrarFOV and "Show FOV: ON" or "Show FOV: OFF"
	atualizarVisibilidadeFOV()
end)

botaoWall.MouseButton1Click:Connect(function()
	wallcheck = not wallcheck
	botaoWall.Text = wallcheck and "Wallcheck: ON" or "Wallcheck: OFF"
	alvo = nil
end)

botaoTeam.MouseButton1Click:Connect(function()
	teamCheck = not teamCheck
	botaoTeam.Text = teamCheck and "Team Check: ON" or "Team Check: OFF"
	alvo = nil
end)

-- ===== Botão Direito (Mira) =====
UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = false
	end
end)

-- ===== Loop de Renderização e Suavização =====
RunService.RenderStepped:Connect(function(dt: number)
	if not ativo then return end

	atualizarAlvo()

	if mirando and alvo and alvo.Character then
		local cabeca = alvo.Character:FindFirstChild("Head")
		if cabeca then
			local posAtual = camera.CFrame.Position
			local cframeAlvo = CFrame.lookAt(posAtual, cabeca.Position)

			local alpha = 1 - math.exp(-smoothAtual * dt)
			camera.CFrame = camera.CFrame:Lerp(cframeAlvo, alpha)
		end
	end
end)
