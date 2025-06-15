-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Tela de carregamento "SCRIPT BRENO HUB"
local loadingGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
loadingGui.Name = "BrenoHubLoading"

local loadingText = Instance.new("TextLabel", loadingGui)
loadingText.Size = UDim2.new(0.6, 0, 0.1, 0)
loadingText.Position = UDim2.new(0.2, 0, 0.45, 0)
loadingText.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
loadingText.BackgroundTransparency = 0.3
loadingText.Text = "🔄 SCRIPT BRENO HUB EXECUTANDO..."
loadingText.TextColor3 = Color3.fromRGB(0, 255, 0)
loadingText.Font = Enum.Font.SourceSansBold
loadingText.TextScaled = true
Instance.new("UICorner", loadingText).CornerRadius = UDim.new(0, 12)

-- Fade out após 3 segundos
task.delay(3, function()
	local fade = TweenService:Create(loadingText, TweenInfo.new(1), {TextTransparency = 1, BackgroundTransparency = 1})
	fade:Play()
	fade.Completed:Wait()
	loadingGui:Destroy()
end)

-- GUI principal
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "Shooter_GUI"

-- Função botão animado
local function createButton(name, posY, callback)
	local button = Instance.new("TextButton")
	button.Parent = gui
	button.Size = UDim2.new(0, 160, 0, 35)
	button.Position = UDim2.new(0, 20, 0, posY)
	button.Text = name
	button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.SourceSansBold
	button.TextScaled = true
	Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)

	-- Hover animação
	button.MouseEnter:Connect(function()
		TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(70, 70, 70)}):Play()
	end)
	button.MouseLeave:Connect(function()
		TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 50, 50)}):Play()
	end)

	button.MouseButton1Click:Connect(function()
		TweenService:Create(button, TweenInfo.new(0.1), {TextColor3 = Color3.fromRGB(0, 255, 0)}):Play()
		callback()
		wait(0.1)
		TweenService:Create(button, TweenInfo.new(0.2), {TextColor3 = Color3.fromRGB(255, 255, 255)}):Play()
	end)

	return button
end

-- Variáveis
local espEnabled = false
local aimbotEnabled = false
local infiniteJumpEnabled = false
local holdingAimbotKey = false
local fovRadius = 100
local espTable = {}

-- ESP
function createESP(player)
	if player == LocalPlayer then return end
	if player.Character and player.Character:FindFirstChild("Head") then
		local box = Drawing.new("Square")
		box.Color = Color3.fromRGB(0, 255, 0)
		box.Thickness = 1
		box.Transparency = 1
		box.Filled = false

		local nameTag = Drawing.new("Text")
		nameTag.Size = 14
		nameTag.Color = Color3.fromRGB(255, 255, 255)
		nameTag.Center = true
		nameTag.Outline = true

		espTable[player] = {box = box, nameTag = nameTag}
	end
end

function removeESP(player)
	if espTable[player] then
		espTable[player].box:Remove()
		espTable[player].nameTag:Remove()
		espTable[player] = nil
	end
end

RunService.RenderStepped:Connect(function()
	if espEnabled then
		for _, player in ipairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild("Head") then
				if not espTable[player] then createESP(player) end
				local headPos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
				local box = espTable[player].box
				local nameTag = espTable[player].nameTag
				if onScreen then
					local root = player.Character:FindFirstChild("HumanoidRootPart")
					if root then
						local size = Vector3.new(2, 3, 1.5)
						local tl = Camera:WorldToViewportPoint((root.CFrame * CFrame.new(-size.X/2, size.Y/2, 0)).Position)
						local br = Camera:WorldToViewportPoint((root.CFrame * CFrame.new(size.X/2, -size.Y/2, 0)).Position)

						box.Size = Vector2.new(math.abs(tl.X - br.X), math.abs(tl.Y - br.Y))
						box.Position = Vector2.new(math.min(tl.X, br.X), math.min(tl.Y, br.Y))
						box.Visible = true

						nameTag.Text = player.Name
						nameTag.Position = Vector2.new(headPos.X, headPos.Y - 20)
						nameTag.Visible = true
					end
				else
					box.Visible = false
					nameTag.Visible = false
				end
			end
		end
	else
		for _, esp in pairs(espTable) do
			esp.box.Visible = false
			esp.nameTag.Visible = false
		end
	end
end)

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Thickness = 1
fovCircle.Radius = fovRadius
fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
fovCircle.Visible = true
fovCircle.Filled = false

-- Aimbot
RunService.RenderStepped:Connect(function()
	fovCircle.Visible = aimbotEnabled
	fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

	if not aimbotEnabled or not holdingAimbotKey then return end

	local closest = nil
	local shortest = fovRadius

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild("Head") then
			local head = player.Character.Head
			local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - fovCircle.Position).Magnitude
				if dist < shortest then
					shortest = dist
					closest = head
				end
			end
		end
	end

	if closest then
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Position)
	end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
	if infiniteJumpEnabled then
		local char = LocalPlayer.Character
		if char and char:FindFirstChildOfClass("Humanoid") then
			char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- Tecla Aimbot
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.Q then
		holdingAimbotKey = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.Q then
		holdingAimbotKey = false
	end
end)

-- Botões
createButton("ESP: Toggle", 100, function()
	espEnabled = not espEnabled
end)

createButton("Aimbot: Toggle", 140, function()
	aimbotEnabled = not aimbotEnabled
end)

createButton("Infinite Jump", 180, function()
	infiniteJumpEnabled = not infiniteJumpEnabled
end)

createButton("FOV +", 220, function()
	fovRadius = math.min(fovRadius + 10, 300)
	fovCircle.Radius = fovRadius
end)

createButton("FOV -", 260, function()
	fovRadius = math.max(fovRadius - 10, 30)
	fovCircle.Radius = fovRadius
end)

-- Atualizar ESP para novos jogadores
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		task.wait(1)
		createESP(player)
	end)
end)

Players.PlayerRemoving:Connect(removeESP)
