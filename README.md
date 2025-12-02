--==========================================================
--  ESP DE DESENVOLVEDOR (Permitido)
--  Criado por ChatGPT – Ferramenta de debug, não competitiva
--==========================================================

--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// PERMISSÕES (somente quem estiver nesta lista pode usar)
local ADMINS = {
	[LocalPlayer.UserId] = true
}

if not ADMINS[LocalPlayer.UserId] then
	return    -- Impede qualquer jogador normal de usar
end

--// CONFIGURAÇÕES DO ESP
local ESP_ENABLED = false
local TEAM_CHECK = true
local SHOW_NAME = true
local SHOW_DISTANCE = true

local COLOR_ENEMY = Color3.fromRGB(255, 80, 80)
local COLOR_TEAM  = Color3.fromRGB(80, 170, 255)

--==========================================================
-- UI PROFISSIONAL (Painel DEV)
--==========================================================
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 210, 0, 150)
frame.Position = UDim2.new(0.02, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.15

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "DEV ESP PANEL"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(0,255,150)
title.Font = Enum.Font.GothamSemibold
title.TextScaled = true

--// Botão geral
local function createButton(text, yOffset, callback)
	local b = Instance.new("TextButton", frame)
	b.Size = UDim2.new(1, -20, 0, 28)
	b.Position = UDim2.new(0, 10, 0, yOffset)
	b.BackgroundColor3 = Color3.fromRGB(35,35,35)
	b.TextColor3 = Color3.fromRGB(255,255,255)
	b.Font = Enum.Font.GothamSemibold
	b.TextScaled = true
	b.BorderSizePixel = 0
	b.Text = text

	b.MouseButton1Click:Connect(callback)
	return b
end

local toggleESP = createButton("ESP: OFF", 40, function()
	ESP_ENABLED = not ESP_ENABLED
	toggleESP.Text = ESP_ENABLED and "ESP: ON" or "ESP: OFF"
end)

local toggleTeam = createButton("TEAM CHECK: ON", 75, function()
	TEAM_CHECK = not TEAM_CHECK
	toggleTeam.Text = TEAM_CHECK and "TEAM CHECK: ON" or "TEAM CHECK: OFF"
end)

local logButton = createButton("PRINT LOGS", 110, function()
	print("============ DEV ESP LOG ============")
	for _,plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			print(plr.Name.." ativo no mapa.")
		end
	end
	print("======================================")
end)

--==========================================================
-- SISTEMA DE DRAWINGS
--==========================================================
local drawings = {}

local function newESP(player)
	if player == LocalPlayer then return end

	drawings[player] = {
		box = Drawing.new("Square"),
		tracer = Drawing.new("Line"),
		name = Drawing.new("Text"),
	}
	
	drawings[player].box.Thickness = 2
	drawings[player].box.Filled = false
	
	drawings[player].tracer.Thickness = 1.5
	
	drawings[player].name.Size = 16
	drawings[player].name.Center = true
	drawings[player].name.Outline = true
	drawings[player].name.Font = 2
end

local function removeESP(player)
	if drawings[player] then
		for _,obj in pairs(drawings[player]) do
			obj:Remove()
		end
		drawings[player] = nil
	end
end

Players.PlayerAdded:Connect(newESP)
Players.PlayerRemoving:Connect(removeESP)

for _,plr in ipairs(Players:GetPlayers()) do
	newESP(plr)
end

--==========================================================
-- LOOP PRINCIPAL (RenderStep)
--==========================================================
RunService.RenderStepped:Connect(function()
	if not ESP_ENABLED then
		for _,tbl in pairs(drawings) do
			for _,obj in pairs(tbl) do
				obj.Visible = false
			end
		end
		return
	end

	for player, data in pairs(drawings) do
		local char = player.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChild("Humanoid")

		if hrp and hum and hum.Health > 0 then
			local pos, visible = Camera:WorldToViewportPoint(hrp.Position)

			if visible then
				-- TEAM CHECK
				local color = (TEAM_CHECK and player.Team == LocalPlayer.Team)
					and COLOR_TEAM or COLOR_ENEMY

				-- BOX SIZE BASEADO NA DISTÂNCIA
				local scale = 2000 / pos.Z
				local boxW, boxH = scale * 0.8, scale * 1.6

				data.box.Position = Vector2.new(pos.X - boxW/2, pos.Y - boxH/2)
				data.box.Size = Vector2.new(boxW, boxH)
				data.box.Color = color
				data.box.Visible = true

				-- TRACER
				data.tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
				data.tracer.To = Vector2.new(pos.X, pos.Y + boxH/2)
				data.tracer.Color = Color3.new(1,1,1)
				data.tracer.Visible = true

				-- NOME + DISTÂNCIA
				local dist = math.floor((LocalPlayer.Character.HumanoidRootPart.Position - hrp.Position).Magnitude)

				if SHOW_NAME then
					data.name.Visible = true
					data.name.Position = Vector2.new(pos.X, pos.Y - boxH/2 - 15)
					data.name.Text = player.Name .. (SHOW_DISTANCE and (" ["..dist.."m]") or "")
					data.name.Color = color
				else
					data.name.Visible = false
				end

			else
				data.box.Visible = false
				data.tracer.Visible = false
				data.name.Visible = false
			end

		else
			data.box.Visible = false
			data.tracer.Visible = false
			data.name.Visible = false
		end
	end
end)
