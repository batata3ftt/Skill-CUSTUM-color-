-- ===== CUSTOM COLOR SKILLS =====
-- Troque o nome da cor abaixo:
-- Opcoes: BLACK | BLUE | ROXO | LARANJA | GREEN (RIN) | RED (NEON)
--         GREEN | PINK (NEON) | WHITE (NEON) | GREEN (AIKU) | YELLOW | RED (BLOOD)
local CHOSEN_COLOR_NAME = "BLUE"

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer

local COLORS = {
    ["BLACK"]        = Color3.fromRGB(0,   0,   0),
    ["BLUE"]         = Color3.fromRGB(65,  133, 172),
    ["ROXO"]         = Color3.fromRGB(118, 9,   141),
    ["LARANJA"]      = Color3.fromRGB(198, 116, 57),
    ["GREEN (RIN)"]  = Color3.fromRGB(33,  182, 127),
    ["RED (NEON)"]   = Color3.fromRGB(218, 102, 102),
    ["GREEN"]        = Color3.fromRGB(4,   107, 75),
    ["PINK (NEON)"]  = Color3.fromRGB(222, 118, 140),
    ["WHITE (NEON)"] = Color3.fromRGB(172, 172, 182),
    ["GREEN (AIKU)"] = Color3.fromRGB(129, 185, 91),
    ["YELLOW"]       = Color3.fromRGB(168, 154, 96),
    ["RED (BLOOD)"]  = Color3.fromRGB(81,  14,  37),
}

local selectedColor = COLORS[CHOSEN_COLOR_NAME] or Color3.fromRGB(0, 0, 0)

local function getColor()
    return selectedColor
end

local function getSeq()
    return ColorSequence.new(selectedColor)
end

local function isBallPart(v)
    local p = v.Parent
    while p and p ~= Workspace do
        if p.Name:lower() == "ball" then return true end
        p = p.Parent
    end
    if v.Name == "Ball" and v:IsA("BasePart") then return true end
    return false
end

local function isGKDescendant(v)
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character and v:IsDescendantOf(p.Character) then
            if p.Character:GetAttribute("IsGK")
            or p.Character:GetAttribute("GK")
            or p.Name:lower():find("gk")
            or p.Name:lower():find("keeper") then
                return true
            end
        end
    end
    return false
end

local function isStaticMap(v)
    if v:IsA("Terrain") or v:IsA("Sky") then return true end
    local top = v
    while top.Parent and top.Parent ~= Workspace do
        top = top.Parent
    end
    if top.Parent ~= Workspace then return false end
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character and top == p.Character then return false end
    end
    local tname = top.Name:lower()
    local isVFX = tname:find("vfx") or tname:find("effect") or tname:find("skill")
        or tname:find("fx") or tname:find("goal") or tname:find("explosion")
        or tname:find("ball") or tname:find("tackle") or tname:find("shot")
        or tname:find("trail") or tname:find("aura") or tname:find("particle")
        or tname:find("emit") or tname:find("tmp") or tname:find("cutscene")
    if isVFX then return false end
    return true
end

local function applyColor(v)
    if not v or not v.Parent then return end
    if isBallPart(v) then return end
    if isGKDescendant(v) then return end
    if isStaticMap(v) then return end
    pcall(function()
        if v:IsA("ParticleEmitter") then
            v.Color = getSeq()
            v.LightEmission = 0
            v.LightInfluence = 1
            v.Transparency = NumberSequence.new(0)
        elseif v:IsA("Trail") then
            v.Color = getSeq()
            v.LightEmission = 0
            v.Transparency = NumberSequence.new(0)
        elseif v:IsA("Beam") then
            v.Color = getSeq()
            v.LightEmission = 0
            v.Transparency = NumberSequence.new(0)
        elseif v:IsA("Highlight") then
            v.FillColor = getColor()
            v.OutlineColor = getColor()
            v.FillTransparency = 0.3
            v.OutlineTransparency = 0
        end
    end)
end

local function isEffect(v)
    return v:IsA("ParticleEmitter") or v:IsA("Trail")
        or v:IsA("Beam") or v:IsA("Highlight")
end

-- Ego e Flow bar
pcall(function()
    local r = math.floor(selectedColor.R * 255)
    local g = math.floor(selectedColor.G * 255)
    local b = math.floor(selectedColor.B * 255)
    player.Ego.Value = r .. "," .. g .. "," .. b
end)

pcall(function()
    local gui = player.PlayerGui:WaitForChild("MainmenuGui")
    local f2 = gui:WaitForChild("Frame2")
    local fb = f2:WaitForChild("Flow-bar-inside")
    fb.BackgroundColor3 = getColor()
    fb.BorderColor3 = getColor()
end)

-- Loop para manter ego e flow bar atualizados
task.spawn(function()
    while true do
        pcall(function()
            local r = math.floor(selectedColor.R * 255)
            local g = math.floor(selectedColor.G * 255)
            local b = math.floor(selectedColor.B * 255)
            local val = r .. "," .. g .. "," .. b
            if player.Ego.Value ~= val then player.Ego.Value = val end
        end)
        pcall(function()
            local gui = player.PlayerGui:FindFirstChild("MainmenuGui")
            if gui then
                local f2 = gui:FindFirstChild("Frame2")
                if f2 then
                    local fb = f2:FindFirstChild("Flow-bar-inside")
                    if fb then
                        fb.BackgroundColor3 = getColor()
                        fb.BorderColor3 = getColor()
                    end
                end
            end
        end)
        task.wait(0.3)
    end
end)

-- Scan inicial workspace
task.spawn(function()
    local all = Workspace:GetDescendants()
    for i = 1, #all, 150 do
        for j = i, math.min(i + 149, #all) do
            if isEffect(all[j]) then applyColor(all[j]) end
        end
        task.wait()
    end
end)

-- Scan inicial ReplicatedStorage
task.spawn(function()
    local all = ReplicatedStorage:GetDescendants()
    for i = 1, #all, 150 do
        for j = i, math.min(i + 149, #all) do
            if isEffect(all[j]) then applyColor(all[j]) end
        end
        task.wait()
    end
end)

-- Tempo real workspace
Workspace.DescendantAdded:Connect(function(v)
    if isEffect(v) then
        task.defer(function() applyColor(v) end)
    end
end)

-- Tempo real ReplicatedStorage
ReplicatedStorage.DescendantAdded:Connect(function(v)
    if isEffect(v) then
        task.defer(function() applyColor(v) end)
    end
end)

-- Chars de todos os players
for _, p in pairs(Players:GetPlayers()) do
    if p.Character then
        p.Character.DescendantAdded:Connect(function(v)
            if isEffect(v) then
                task.defer(function() applyColor(v) end)
            end
        end)
    end
end

Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        char.DescendantAdded:Connect(function(v)
            if isEffect(v) then
                task.defer(function() applyColor(v) end)
            end
        end)
    end)
end)

print("Color Skills carregado! Cor: " .. CHOSEN_COLOR_NAME)
