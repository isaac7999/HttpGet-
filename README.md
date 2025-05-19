local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")
local destino = root.Position
local yOffset = 0 -- Para empilhar os carros

-- Função para ancorar/desancorar partes do veículo
local function setAnchored(model, state)
	for _, part in ipairs(model:GetDescendants()) do
		if part:IsA("BasePart") then
			part.Anchored = state
		end
	end
end

-- Função para pegar todos os assentos livres
local function getAllFreeSeats()
	local seats = {}
	for _, v in ipairs(workspace:GetDescendants()) do
		if v:IsA("VehicleSeat") and not v.Occupant then
			table.insert(seats, v)
		end
	end
	return seats
end

-- Função para forçar o jogador a sentar no assento e dirigir
local function forceSit(seat)
	humanoid.Sit = false
	task.wait(0.2)
	root.CFrame = seat.CFrame * CFrame.new(0, 2.5, 0)

	local success = false
	for i = 1, 10 do
		humanoid:ChangeState(Enum.HumanoidStateType.Seated)
		seat:Sit(humanoid)
		task.wait(0.3)
		if seat.Occupant == humanoid then
			success = true
			break
		end
	end
	return success
end

-- Função principal para puxar e empilhar todos os carros
local function puxarEmpilharTodosCarros()
	local usados = {}
	local seats = getAllFreeSeats()

	for _, seat in ipairs(seats) do
		if usados[seat] then continue end
		local vehicle = seat:FindFirstAncestorOfClass("Model")
		if vehicle and vehicle.PrimaryPart and not usados[vehicle] then
			usados[seat] = true
			usados[vehicle] = true

			-- Tenta sentar corretamente
			if forceSit(seat) then
				-- Ancorar e teleportar o veículo para a posição original + empilhamento
				setAnchored(vehicle, true)
				task.wait(0.1)

				local targetPos = destino + Vector3.new(0, yOffset, 0)
				vehicle:SetPrimaryPartCFrame(CFrame.new(targetPos))
				task.wait(0.2)

				setAnchored(vehicle, false)
				humanoid.Sit = false -- Sai do assento
				task.wait(0.3)

				yOffset += 10 -- Aumenta o empilhamento vertical
			else
				warn("Falha ao sentar no assento.")
			end
		end
	end
end

-- Executa
puxarEmpilharTodosCarros()
