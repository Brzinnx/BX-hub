-- // CONFIGURAÇÕES INICIAIS
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local runService = game:GetService("RunService")
local replicatedStorage = game:GetService("ReplicatedStorage")

-- Supondo que esses objetos existam no seu jogo:
local baseDoor = workspace:WaitForChild("BaseDoor_" .. player.Name) -- Nome da porta da sua base
local moneyCollector = workspace:WaitForChild("MoneyCollector_" .. player.Name)
local brainrots = workspace:WaitForChild("Brainrots")

local teleportBasePosition = workspace:WaitForChild("BaseTeleport_" .. player.Name).Position

-- // 1. AUTO BLOQUEAR BASE
local function autoBlockBase()
	if baseDoor then
		baseDoor.CanCollide = true
		baseDoor.Transparency = 0
	end
end

-- // 2. AUTO COLETAR DINHEIRO
local function autoCollectMoney()
	if moneyCollector then
		firetouchinterest(humanoidRootPart, moneyCollector, 0)
		wait(0.2)
		firetouchinterest(humanoidRootPart, moneyCollector, 1)
	end
end

-- // 3. ROUBAR OU COMPRAR BRAINROTS
local function handleBrainrotClick(brainrot)
	if not brainrot:IsA("Model") then return end

	local clickDetector = brainrot:FindFirstChildOfClass("ClickDetector")
	if not clickDetector then return end

	clickDetector.MouseClick:Connect(function()
		print("Você clicou no Brainrot!")
		
		-- Simular que você pegou o brainrot na mão
		brainrot.Parent = character
		brainrot:SetPrimaryPartCFrame(humanoidRootPart.CFrame * CFrame.new(0, 2, 2))
		
		-- Esperar 3 segundos
		wait(3)

		-- Teleportar para a base
		character:MoveTo(teleportBasePosition)

		-- Mover o brainrot pra base
		wait(0.5)
		brainrot.Parent = workspace:WaitForChild("MinhaBase_" .. player.Name)
		brainrot:SetPrimaryPartCFrame(CFrame.new(teleportBasePosition + Vector3.new(2, 1, 0)))

		print("Brainrot roubado com sucesso!")
	end)
end

-- // 4. CONECTAR EVENTOS AOS BRAINROTS
for _, brainrot in pairs(brainrots:GetChildren()) do
	handleBrainrotClick(brainrot)
end

-- Atualizar caso novos brainrots apareçam
brainrots.ChildAdded:Connect(handleBrainrotClick)

-- // 5. LOOP PARA AUTO FUNÇÕES
runService.RenderStepped:Connect(function()
	autoBlockBase()
	autoCollectMoney()
end)
