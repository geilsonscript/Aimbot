GetLocalPlayer(), etc.
]]

local aimbotEnabled = true
local aimFOV = 45
local aimSmooth = 0.2
local target = nil

-- Interface Simples
function drawMenu()
    print("===== AIMBOT MENU =====")
    print("Status: " .. (aimbotEnabled and "Ativado" or "Desativado"))
    print("FOV: " .. aimFOV)
    print("Smooth: " .. aimSmooth)
    print("========================")
end

-- Verifica se é inimigo
function isEnemy(player)
    local localPlayer = GetLocalPlayer()
    return not IsTeammate(localPlayer, player)
end

-- Calcula ângulo entre o player e o inimigo
function calculateAngle(targetPos)
    local localPos = GetPosition(GetLocalPlayer())
    local dx = targetPos.x - localPos.x
    local dy = targetPos.y - localPos.y
    local dz = targetPos.z - localPos.z
    return math.atan2(dy, dx), math.atan2(dz, math.sqrt(dx*dx + dy*dy))
end

-- Encontra o inimigo mais próximo dentro do FOV
function getClosestEnemy()
    local players = GetPlayers()
    local localPos = GetPosition(GetLocalPlayer())
    local bestTarget = nil
    local bestDist = math.huge

    for _, player in ipairs(players) do
        if player ~= GetLocalPlayer() and isEnemy(player) and IsAlive(player) then
            local pos = GetPosition(player)
            local dist = GetDistance(localPos, pos)
            local angleToTarget = GetAngleTo(pos)

            if dist < bestDist and angleToTarget < aimFOV then
                bestDist = dist
                bestTarget = player
            end
        end
    end
    return bestTarget
end

-- Aponta para o alvo suavemente
function aimAtTarget(target)
    local targetPos = GetBonePosition(target, "head") -- exemplo com 'cabeça'
    local yaw, pitch = calculateAngle(targetPos)
    local currentYaw, currentPitch = GetViewAngles()

    local newYaw = currentYaw + (yaw - currentYaw) * aimSmooth
    local newPitch = currentPitch + (pitch - currentPitch) * aimSmooth

    SetViewAngles(newYaw, newPitch)
end

-- Loop principal do AIMBOT
function onTick()
    if not aimbotEnabled then return end

    local target = getClosestEnemy()
    if target then
        aimAtTarget(target)
    end
end

-- Simula menu e ciclo
drawMenu()
RegisterTickFunction(onTick)# Aimbot
