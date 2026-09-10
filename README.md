-- **Sistema Anti-Lag Paralelo para Delta Executor**
local TaskManager = {
    registry = {},
}

-- Crear una tarea asíncrona no bloqueante
function TaskManager:CreateTask(id, func)
    local routine = coroutine.create(func)
    self.registry[id] = {
        co = routine,
        status = "ready"
    }
end

-- Procesar tareas de forma distribuida usando Heartbeat de Roblox
function TaskManager:Init()
    local RunService = game:GetService("RunService")
    
    RunService.Heartbeat:Connect(function(deltaTime)
        for id, task in pairs(self.registry) do
            if task.status ~= "dead" then
                -- Reanudar la corrutina en cada frame de forma segura
                local success, err = coroutine.resume(task.co)
                
                if not success then
                    warn("Delta Anti-Lag Error [" .. tostring(id) .. "]: " .. tostring(err))
                    task.status = "dead"
                elseif coroutine.status(task.co) == "dead" then
                    task.status = "dead"
                end
            end
        end
    end)
end

-- Ejemplo de uso: Tarea pesada (ej. escaneo masivo o carga de instancias)
TaskManager:CreateTask("CargaMasiva", function()
    print("Iniciando tarea pesada en segundo plano...")
    
    -- Simulamos un bucle pesado que causaría un tirón (lag spike)
    for i = 1, 3000 do
        -- Coloca aquí tu lógica pesada (ej: buscar partes, procesar tablas grandes)
        
        -- Ceder el control cada 50 iteraciones para mantener estables los FPS
        if i % 50 == 0 then
            coroutine.yield()
        end
    end
    
    print("¡Tarea finalizada fluidamente sin congelar el juego!")
end)

-- Iniciar el administrador de tareas
TaskManager:Init()
