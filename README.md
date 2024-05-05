local UILib = loadstring(game:HttpGet('https://raw.githubusercontent.com/StepBroFurious/Script/main/HydraHubUi.lua'))()

local Player = game.Players.LocalPlayer
local Characters = workspace.Characters

local RepStor = game:GetService("ReplicatedStorage")

local Window = UILib.new("Fight for Survival v0.0.1", Player.UserId, "Free")
local Category = Window:Category("Main", "http://www.roblox.com/asset/?id=8395621517")

local MainButton = Category:Button("Main", "http://www.roblox.com/asset/?id=8395621517")

local Tabs = {  
	Main = MainButton:Section("Main", "Left"),
	Teleportations = MainButton:Section("Teleportation", "Right")
}

-- // CONFIG (script) \\ --

local CheatConfings = {
	EnableDisable = {
		["BallSocketConstraint"] = false,
		["Motor6D"] = true
	},
}

-- // CHEAT UI \\ --

local RagTog = Tabs.Main:Toggle({ Title = "Anti Ragdoll", Description = "", Default = false }, function(a) end)
local Farm = Tabs.Main:Toggle(
    { Title = "Auto Farm", Description = "", Default = false },
    function(a)
        if a and Player.Character then
            Window:Notification({Title = "!!!!", Desc = "Started"})
            Player.Character:PivotTo(CFrame.new(62, 1.3, -63))
            Player.Character.Humanoid.Died:Connect(function() -- This disconnects when human is gone
                --RepStor.Remotes.LoadCharacter:FireServer()
                task.delay(5.5, function() -- Avarege spawntime maybe??
                    Player.Character:PivotTo(CFrame.new(62, 1.3, -63))
                end)
            end)
        end
    end
)

local But1 = Tabs.Teleportations:Button(
    { Title = "Portal", ButtonName = "Teleport" , Description = "Teleports to portal" },
    function()
        if Player.Character then
            Window:Notification({Title = "Done", Desc = "Yayy"})
            Player.Character:PivotTo(CFrame.new(62, 1.3, -63))
        end
    end
)

-- // CHEAT FUNCTIONS \\ --

local function ReBuild(Char : Model)
	for i, v in pairs(Char:GetDescendants()) do 
		if CheatConfings.EnableDisable[v.ClassName] then
			v.Enabled = CheatConfings.EnableDisable[v.ClassName]
		end
	end
end

local function M1Fire()
    local args = {
        [1] = "M1",
        [2] = Vector3.new(12, 12, 12)
    }
    
    RepStor.Remotes.Skill:FireServer(unpack(args))
end

local function LoopTargetYield(Char, TimeToTarget)
    local m1t = tick()
    local m1cd = 0.6
    local TimeC = tick() + TimeToTarget

    repeat
        if Player.Character and Char then
            if Char:FindFirstChild("Torso") then
                local BackPos = Char.Torso.Position + (Char.Torso.CFrame.LookVector * 0.8)
                Player.Character:PivotTo(CFrame.new(BackPos, Char.Torso.Position))
            end
        end

        if tick() > (m1t + m1cd) then
            m1t = (tick())
            M1Fire()
        end

        task.wait()
    until (tick() > TimeC)
end

local function CharAdded(Character : Model)
	local Human = Character:WaitForChild("Humanoid")

    local function OnStateChange(oldstate, newstate)
        local ife = RagTog:getValue()
        if not ife then return end
		if (newstate == Enum.HumanoidStateType.Ragdoll) or (newstate == Enum.HumanoidStateType.Physics) then
			ReBuild(Character)
			Human:SetStateEnabled(Enum.HumanoidStateType.Physics, false) -- just incase
			Human:ChangeState(Enum.HumanoidStateType.GettingUp)

			Human.PlatformStand = false
		end
    end

	Human.StateChanged:Connect(OnStateChange)
end

task.spawn(function()
    while task.wait() do
        local GetVal = Farm:getValue()
        if Player.Character and GetVal then
            for i, Character in pairs(Characters:GetChildren()) do
                local GetVal2 = Farm:getValue()
                if not (Character == Player.Character) and Character:FindFirstChild("Torso") and GetVal2 then
                    local CFT = Character.Torso.CFrame
                    if CFT.Position.Y > 30 then -- Check if theyre playing
                        -- TPING to target
                        LoopTargetYield(Character, 1.5) -- yields
                        -- Tp to sky
                        Player.Character:PivotTo(CFrame.new(1, 15555, 1))
                        task.wait(0.9)
                    end
                end
            end
        end
    end
end)

-- // Script Boot

if Player.Character then
	CharAdded(Player.Character)
end

Player.CharacterAdded:Connect(CharAdded)    
