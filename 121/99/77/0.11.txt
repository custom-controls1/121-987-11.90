local AS = game:GetService("AssetService")
local queue = {}
local busy = false

local function make(id)
    local ok, res
    if Content and Content.fromAssetId then
        ok, res = pcall(function()
            return AS:CreateMeshPartAsync(Content.fromAssetId(id))
        end)
    else
        ok, res = pcall(function()
            return AS:CreateMeshPartAsync("rbxassetid://" .. tostring(id))
        end)
    end
    if ok then return res end
    return nil
end

local pump
pump = function()
    if busy then return end
    local job = table.remove(queue, 1)
    if not job then return end
    busy = true
    task.spawn(function()
        local mesh = make(job.id)
        busy = false
        if job.cb then pcall(job.cb, job.id, mesh) end
        task.defer(pump)
    end)
end

return function(id, cb)
    queue[#queue + 1] = { id = id, cb = cb }
    pump()
end
