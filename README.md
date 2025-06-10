local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local rs = game:GetService("RunService")
local replicatedStorage = game:GetService("ReplicatedStorage")

local chosenBrainrot = "Brainrot Lendário" -- Nome padrão, pode ser alterado via GUI
local stealing = false

-- GUI simples
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "BrainrotStealerGui"

local drop = Instance.new("TextBox", gui)
drop.Size = UDim2.new(0, 200, 0, 30)
drop.Position = UDim2.new(0, 10, 0, 10)
drop.PlaceholderText = "Digite o nome do Brainrot"
drop.Text = ""

local stealButton = Instance.new("TextButton", gui)
stealButton.Size = UDim2.new(0, 200, 0, 30)
stealButton.Position = UDim2.new(0, 10, 0, 50)
stealButton.Text = "Ativar Roubo"

-- PULO INFINITO
UIS.JumpRequest:Connect(function()
	player.Character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
end)

-- COLETA AUTOMÁTICA DE DINHEIRO
local function autoCollectMoney()
	local moneyPart = workspace:FindFirstChild("MoneyCollector") -- nome genérico
	if moneyPart then
		firetouchinterest(player.Character.HumanoidRootPart, moneyPart, 0)
		wait(0.1)
		firetouchinterest(player.Character.HumanoidRootPart, moneyPart, 1)
	end
end

-- BLOQUEAR BASE
local function autoBlockBase()
	local button = workspace:FindFirstChild("BlockButton_" .. player.Name) -- exemplo de nome
	if button and button:IsA("Part") then
		local cd = button:FindFirstChildOfClass("ClickDetector")
		if cd then
			fireclickdetector(cd)
		end
	end
end

-- ROUBAR BRAINROT
local function stealBrainrot(brainrot)
	if not brainrot:IsA("Model") or not brainrot.PrimaryPart then return end
	print("Roubando:", brainrot.Name)
	brainrot.Parent = player.Character
	brainrot:SetPrimaryPartCFrame(player.Character.HumanoidRootPart.CFrame * CFrame.new(0, 2, 2))
	wait(3)
	local teleportTo = workspace:FindFirstChild("BaseTeleport_" .. player.Name)
	if teleportTo then
		player.Character:MoveTo(teleportTo.Position)
		wait(0.5)
		local base = workspace:FindFirstChild("Base_" .. player.Name)
		if base then
			brainrot.Parent = base
			brainrot:SetPrimaryPartCFrame(base:GetModelCFrame() + Vector3.new(2, 1, 0))
		end
	end
end

-- VERIFICAR JOGADORES PARA ROUBAR
local function tryToSteal()
	for _, otherPlayer in pairs(game.Players:GetPlayers()) do
		if otherPlayer ~= player then
			local theirBase = workspace:FindFirstChild("Base_" .. otherPlayer.Name)
			if theirBase then
				for _, obj in pairs(theirBase:GetChildren()) do
					if obj:IsA("Model") and obj.Name == chosenBrainrot then
						stealBrainrot(obj)
						return
					end
				end
			end
		end
	end
end

-- CONECTA GUI
stealButton.MouseButton1Click:Connect(function()
	if drop.Text ~= "" then
		chosenBrainrot = drop.Text
	end
	stealing = not stealing
	stealButton.Text = stealing and "Desativar Roubo" or "Ativar Roubo"
end)

-- LOOP
rs.RenderStepped:Connect(function()
	autoCollectMoney()
	autoBlockBase()
	if stealing then
		tryToSteal()
	end
end)
