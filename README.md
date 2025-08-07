-- SERVIÃ‡OS E VARIÃVEIS
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

local pos1, pos2, pos3, pos4 = nil, nil, nil, nil
local tweenEmExecucao = false
local VELOCIDADE_TELEGUIADO = 60

-- CORES VERDE NEON
local corFundo = Color3.fromRGB(10, 30, 10)
local corVerdeNeon = Color3.fromRGB(0, 255, 0)
local corVerdeEscuro = Color3.fromRGB(0, 180, 0)

local function criarNeon(frame)
	local neon = Instance.new("UIStroke")
	neon.Color = corVerdeNeon
	neon.Thickness = 2
	neon.Transparency = 0.3
	neon.Parent = frame
end

-- GUI PRINCIPAL
local mainGui = Instance.new("ScreenGui")
mainGui.Name = "LaGrandeCombinaison"
mainGui.ResetOnSpawn = false
mainGui.Enabled = true -- Habilitado desde el inicio
mainGui.Parent = PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 250)
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -125)
mainFrame.BackgroundColor3 = corFundo
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = mainGui

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 15)
criarNeon(mainFrame)

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, 0, 0, 45)
titulo.BackgroundTransparency = 1
titulo.Text = "la_grande hub"
titulo.Font = Enum.Font.GothamBlack
titulo.TextScaled = true
titulo.TextColor3 = corVerdeNeon
titulo.Parent = mainFrame

local function criarBotao(texto, ordem)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0, 260, 0, 45)
	b.Position = UDim2.new(0.5, -130, 0, 55 + (ordem - 1) * 50)
	b.Text = texto
	b.TextColor3 = corFundo
	b.Font = Enum.Font.GothamBlack
	b.TextSize = 20
	b.AutoButtonColor = false
	b.BackgroundColor3 = corVerdeNeon
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
	criarNeon(b)
	b.Parent = mainFrame
	return b
end

local botao1 = criarBotao("ðŸ“ Marcar posiÃ§Ã£o 1", 1)
local botao2 = criarBotao("ðŸ“ Marcar posiÃ§Ã£o 2", 2)
local botao3 = criarBotao("ðŸ“ Marcar posiÃ§Ã£o 3", 3)
local botao4 = criarBotao("ðŸ“ Marcar posiÃ§Ã£o 4", 4)
local botaoGo = criarBotao("ðŸš€ Teleguiado", 5)

-- Marcar posiÃ§Ãµes
for i, botao in ipairs({botao1, botao2, botao3, botao4}) do
	botao.MouseButton1Click:Connect(function()
		local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			if i == 1 then pos1 = hrp.Position
			elseif i == 2 then pos2 = hrp.Position
			elseif i == 3 then pos3 = hrp.Position
			elseif i == 4 then pos4 = hrp.Position end

			botao.Text = "âœ… PosiÃ§Ã£o " .. i .. " marcada!"
			task.delay(2, function()
				botao.Text = "ðŸ“ Marcar posiÃ§Ã£o " .. i
			end)
		end
	end)
end

-- FunÃ§Ã£o mover com invisibilidade
local function moverPara(destino, velocidade)
	if tweenEmExecucao or not destino then return end
	tweenEmExecucao = true

	local char = player.Character or player.CharacterAdded:Wait()
	local hrp = char:WaitForChild("HumanoidRootPart")
	local humanoid = char:WaitForChild("Humanoid")

	-- Invisibilidade e anti-colisÃ£o
	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") then
			part.Transparency = 1
			part.CanCollide = false
			part.Anchored = false
			if part:IsA("MeshPart") then
				part.LocalTransparencyModifier = 1
			end
		elseif part:IsA("Decal") then
			part.Transparency = 1
		end
	end

	humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
	humanoid.Health = humanoid.MaxHealth
	humanoid.PlatformStand = false
	humanoid.Sit = false

	local dist = (hrp.Position - destino).Magnitude
	local tempo = dist / velocidade

	local tween = TweenService:Create(hrp, TweenInfo.new(tempo, Enum.EasingStyle.Linear), {Position = destino})
	tween:Play()
	tween.Completed:Wait()

	hrp.Velocity = Vector3.zero
	hrp.RotVelocity = Vector3.zero
	humanoid.WalkSpeed = 16
	tweenEmExecucao = false
end

botaoGo.MouseButton1Click:Connect(function()
	for _, destino in ipairs({pos1, pos2, pos3, pos4}) do
		if destino then
			moverPara(destino, VELOCIDADE_TELEGUIADO)
			task.wait(0.1)
		end
	end
end)

-- Ãcone para abrir/fechar GUI principal
local iconGui = Instance.new("ScreenGui")
iconGui.Name = "IconeScript"
iconGui.ResetOnSpawn = false
iconGui.Enabled = true -- Habilitado por padrÃ£o
iconGui.Parent = PlayerGui

local icon = Instance.new("TextButton")
icon.Size = UDim2.new(0, 60, 0, 60)
icon.Position = UDim2.new(0, 10, 0, 10)
icon.Text = "LÃ¡"
icon.TextColor3 = corVerdeNeon
icon.TextSize = 28
icon.Font = Enum.Font.GothamBlack
icon.BackgroundColor3 = corFundo
icon.Parent = iconGui

Instance.new("UICorner", icon).CornerRadius = UDim.new(1, 0)
criarNeon(icon)

icon.MouseButton1Click:Connect(function()
	mainGui.Enabled = not mainGui.Enabled
end)

-- Anti-reset e invisibilidade no respawn
player.CharacterAdded:Connect(function(char)
	local humanoid = char:WaitForChild("Humanoid")
	local hrp = char:WaitForChild("HumanoidRootPart")

	for _, p in ipairs(char:GetChildren()) do
		if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" and (p.Position - hrp.Position).Magnitude < 2 then
			p:Destroy()
		end
	end

	task.spawn(function()
		while hrp and hrp.Parent do
			for _, part in pairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.Transparency = 0
					part.CanCollide = true
					part.Anchored = false
					if part:IsA("MeshPart") then
						part.LocalTransparencyModifier = 0
					end
					if (part.Position - hrp.Position).Magnitude > 5 and part.Name ~= "HumanoidRootPart" then
						pcall(function() part:Destroy() end)
					end
				elseif part:IsA("Decal") then
					part.Transparency = 0
				end
			end
			task.wait(1)
		end
	end)

	humanoid.Died:Connect(function()
		task.wait(0.5)
		local newChar = player.Character or player.CharacterAdded:Wait()
		local newHrp = newChar:WaitForChild("HumanoidRootPart")
		if pos1 then
			newHrp.CFrame = CFrame.new(pos1)
		end
	end)
end)
