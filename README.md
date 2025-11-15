--[ Services ]--
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local DataStoreService = game:GetService("DataStoreService")
local GroupService = game:GetService("GroupService")
local DynamicIconsStore = DataStoreService:GetDataStore("DynamicIcons")

--[ Configuration ]--

local AuthorizedUserIds = {2640273189} -- Add your UserId here
local DynamicIcons = {}

local VIPGamepassId = 1529551170
local RainbowUsernameGamepass = 1529369174

--[ Overhead GUI ]--
local OverheadGUIs = ReplicatedStorage:WaitForChild("OverheadGUIs")
local OverheadGui = OverheadGUIs:WaitForChild("OverheadGui")
local Icons = OverheadGUIs:WaitForChild("OverheadIcons")
local IconPlaceHolder = OverheadGui:WaitForChild("IconFrame"):WaitForChild("Icon")
local TopTags = OverheadGUIs:WaitForChild("OverheadTopTag")
local TopTagsPlaceHolder = OverheadGui:WaitForChild("TopTagFrame"):WaitForChild("TopTag")

--[ DataStores for Leaderboards ]--
local topDonatedStore = DataStoreService:GetOrderedDataStore("DonationDonated")
local topRaisedStore = DataStoreService:GetOrderedDataStore("DonationRaised")
local topLevelStore = DataStoreService:GetOrderedDataStore("PlayerLevelDataLevel")

--[ Colors for Top 1,2,3 <3 ]
local COLORS = {
	Gold = Color3.fromRGB(255, 215, 0),
	Silver = Color3.fromRGB(192, 192, 192),
	Bronze = Color3.fromRGB(205, 127, 50)
}

--[ Roles ]--

local GroupId = 14743466

-- please be careful when changing anything to this table!!!
local roles = {
	[255] = {Color = COLORS.White, Rainbow = true},
	[254] = {Color = COLORS.White, Rainbow = true},
	[240] = {Color = COLORS.White, Rainbow = true},
	[230] = {Color = Color3.fromRGB(170, 96, 255),  Rainbow = false},
	[150] = {Color = Color3.fromRGB(170, 96, 255),  Rainbow = false},
	[140] = {Color = Color3.fromRGB(108, 191, 255), Rainbow = false},
	[130] = {Color = Color3.fromRGB(125, 255, 125), Rainbow = false},
	[100] = {Color = Color3.fromRGB(245, 111, 255), Rainbow = false},
	[90]  = {Color = Color3.fromRGB(255, 42, 92),   Rainbow = false},
	[1]   = {Color = Color3.fromRGB(255, 255, 255), Rainbow = false},
	[0]   = {Color = Color3.fromRGB(255, 255, 255), Rainbow = false}
}

--[ Icon Lookup Table ]--
local iconLookup = {
	Owner = {userIds = {76508795}},
	CoOwner = {rank = 254},
	Developer = {minRank = 240, userIds = {76508795}},
	Scripter = {userIds = {76508795}},
	HeadAdmin = {rank = 230},
	Admin = {rank = 150},
	Patrol = {rank = 140, userIds = {}},
	TourGuide = {rank = 130, userIds = {}},
	ServerBooster = {rank = 100, userIds ={9424606450}},
	ContentCreator = {rank = 90, userIds = {76508795}},
	VIP = {gamePassId = VIPGamepassId},
	Premium = {membershipType = Enum.MembershipType.Premium},
	Member = {minRank = 1}
}

local RestrictedIcons = {
	"Owner", "CoOwner", "Developer", "Scripter",
	"HeadAdmin", "Member", "Premium"
}

--[ Icon Order ]--
local iconOrder = {
	"Owner", "CoOwner", "Developer", "Scripter", "HeadAdmin", "Admin", "Patrol", "TourGuide",
	"ServerBooster", "ContentCreator", "VIP", "Premium", "Member"
}

--[ Clan Configuration ]--
local clanGroupIds = {
	622184568,

}

--[ Custom Ranks ]--
local customRanks = {
	["x"] = "FOR TESTING REASONS",
	["x2"] = "ANOTHER RANK",
	["x1"] = "ANOTEHRSEHRS"
	-- ... add more custom ranks here
}

--[ Helper Functions ]--
local function addCommas(number)
	local formatted = tostring(number)
	local k = 3
	while true do
		formatted, k = string.gsub(formatted, "^(-?%d+)(%d%d%d)", '%1,%2')
		if k == 0 then
			break
		end
	end
	return formatted
end

--[ Predefined Rainbow Color Sequence ]--
local rainbowColorSequence = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
	ColorSequenceKeypoint.new(0.1666, Color3.fromRGB(255, 165, 0)),
	ColorSequenceKeypoint.new(0.3333, Color3.fromRGB(255, 255, 0)),
	ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 255, 0)),
	ColorSequenceKeypoint.new(0.6666, Color3.fromRGB(0, 0, 255)),
	ColorSequenceKeypoint.new(0.8333, Color3.fromRGB(75, 0, 130)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(148, 0, 211)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0))
})

local function updateLevelTextColor(levelLabel, levelValue, gradient, gradientScript)
	levelLabel.TextColor3 = Color3.new(1, 1, 1) -- plain white
	if gradient then
		gradient.Enabled = false
	end
	if gradientScript then
		gradientScript.Disabled = true
	end
end

local function updateIcons(plr, levelValue, iconFrame, clanIcon)
	local iconsToAdd = {}
	local rankInGroup = plr:GetRankInGroup(GroupId)

	for iconName, data in pairs(iconLookup) do
		if (data.userIds and table.find(data.userIds, plr.UserId)) or
			(data.levelThreshold and levelValue >= data.levelThreshold) or
			(data.rank and rankInGroup == data.rank) or
			(data.minRank and rankInGroup and rankInGroup >= data.minRank) or
			(data.gamePassId and MarketplaceService:UserOwnsGamePassAsync(plr.UserId, data.gamePassId)) or
			(data.membershipType and plr.MembershipType == data.membershipType) then
			table.insert(iconsToAdd, iconName)
		end
	end

	if DynamicIcons[plr.UserId] then
		for _, iconName in ipairs(DynamicIcons[plr.UserId]) do
			table.insert(iconsToAdd, iconName)
		end
	end

	if clanIcon then
		for _, clanGroupId in ipairs(clanGroupIds) do
			if plr:IsInGroup(clanGroupId) then
				local groupInfo = GroupService:GetGroupInfoAsync(clanGroupId)
				clanIcon.Image = groupInfo.EmblemUrl
				clanIcon.Visible = true
				break
			end
		end

		if not clanIcon.Image then
			clanIcon.Visible = false
		end
	end

	table.sort(iconsToAdd, function(a, b)
		return table.find(iconOrder, a) < table.find(iconOrder, b)
	end)

	for _, icon in iconFrame:GetChildren() do
		if icon:IsA("ImageLabel") and not table.find(iconsToAdd, icon.Name) then
			icon:Destroy()
		end
	end

	for _, iconName in ipairs(iconsToAdd) do
		if not iconFrame:FindFirstChild(iconName) and Icons:FindFirstChild(iconName) then
			local iconClone = IconPlaceHolder:Clone()
			iconClone.Image = Icons:FindFirstChild(iconName).Image
			iconClone.Name = iconName
			iconClone.Parent = iconFrame
		end
	end
end

--[ Load Icons on Player Join ]--
Players.PlayerAdded:Connect(function(player)
	local success, savedIcons = pcall(function()
		return DynamicIconsStore:GetAsync(player.UserId)
	end)

	if success and savedIcons then
		DynamicIcons[player.UserId] = savedIcons
	else
		DynamicIcons[player.UserId] = {}
	end

	player.CharacterAdded:Connect(function(char)
		local level = player:WaitForChild("leaderstats").Level
		local overheadGuiClone = char:WaitForChild("HumanoidRootPart"):WaitForChild("OverheadGui") or char:FindFirstChild("HumanoidRootPart"):FindFirstChild("OverheadGui")

		if overheadGuiClone then
			local iconFrame = overheadGuiClone:WaitForChild("IconFrame")
			local clanIcon = overheadGuiClone.UsernameFrame.Clan
			updateIcons(player, level.Value, iconFrame, clanIcon)
		else
			warn("OverheadGui clone not found in HumanoidRootPart!")
		end
	end)
end)

Players.PlayerRemoving:Connect(function(player)
	if DynamicIcons[player.UserId] then
		pcall(function()
			DynamicIconsStore:SetAsync(player.UserId, DynamicIcons[player.UserId])
		end)
	end
end)

--[ Discord Webhook Configuration ]--
local webhookUrl = "https://discord.com/api/webhooks/1339087200003817512/kbTuG7vhJyaWGseCTAgMVCWnASKFHMcxbzFUwZkikqTSgQPOIFgF1ds-X3koqHYGUtQu" -- Replace with your Discord webhook URL

--[ Enable HttpService ]--
local httpService = game:GetService("HttpService")

--[ Helper Function to Send Discord Webhook ]--
local function sendDiscordWebhook(player, targetPlayer, iconName, action)


	local embed = {
		["title"] = "Icon " .. action .. " Logs",
		["description"] = "Staff " .. "**" .. player.Name .. "** " .. action .. " **" .. iconName .. "** Icon to **" .. targetPlayer.Name .. "**.",
		["color"] = tonumber(6517205),
		["fields"] = {
			{
				["name"] = "Staff Profile",
				["value"] = "[View Profile](https://www.roblox.com/users/" .. player.UserId .. "/profile)",
				["inline"] = true
			},
			{
				["name"] = "Target Player Profile",
				["value"] = "[View Profile](https://www.roblox.com/users/" .. targetPlayer.UserId .. "/profile)",
				["inline"] = true
			},
		},
		["thumbnail"] = {
			["url"] = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. player.UserId .. "&width=420&height=420&format=png"
		},
		["image"] = {
			["url"] = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. targetPlayer.UserId .. "&width=420&height=420&format=png"
		},
		["footer"] = {
			["text"] = "Luminara Technologies",
			["icon_url"] = "https://i.imghippo.com/files/DcF8089B.png"
		}
	}

	local payload = {
		["content"] = "",
		["embeds"] = {embed}
	}

	local jsonPayload = httpService:JSONEncode(payload)

	local success, response = pcall(function()
		return httpService:PostAsync(webhookUrl, jsonPayload, Enum.HttpContentType.ApplicationJson, false)
	end)

	if not success then
		warn("Failed to send Discord webhook:", response)
	end
end

--[ Command Handler ]--
local function handleCommand(player, message)
	if not table.find(AuthorizedUserIds, player.UserId) then
		return
	end

	local args = string.split(message, " ")
	if args[1] == "/addicon" and #args >= 3 then
		local targetPlayerName = args[2]
		local iconName = args[3]

		if table.find(RestrictedIcons, iconName) then
			warn("Cannot add restricted icon:", iconName)
			return
		end

		local targetPlayer = Players:FindFirstChild(targetPlayerName)
		if not targetPlayer then
			warn("Player not found:", targetPlayerName)
			return
		end

		if not iconLookup[iconName] then
			warn("Icon not found:", iconName)
			return
		end

		if not DynamicIcons[targetPlayer.UserId] then
			DynamicIcons[targetPlayer.UserId] = {}
		end
		table.insert(DynamicIcons[targetPlayer.UserId], iconName)

		pcall(function()
			DynamicIconsStore:SetAsync(targetPlayer.UserId, DynamicIcons[targetPlayer.UserId])
		end)

		if targetPlayer.Character then
			local overheadGuiClone = targetPlayer.Character:FindFirstChild("HumanoidRootPart"):FindFirstChild("OverheadGui")
			if overheadGuiClone then
				local iconFrame = overheadGuiClone:WaitForChild("IconFrame")
				updateIcons(targetPlayer, targetPlayer:WaitForChild("leaderstats").Level.Value, iconFrame)
			else
				warn("OverheadGui clone not found for target player!")
			end
		else
			warn("Target player's character not found.")
		end

		sendDiscordWebhook(player, targetPlayer, iconName, "added")

		print("Added icon", iconName, "to player", targetPlayerName)

	elseif args[1] == "/removeicon" and #args >= 3 then
		local targetPlayerName = args[2]
		local iconName = args[3]

		local targetPlayer = Players:FindFirstChild(targetPlayerName)
		if not targetPlayer then
			warn("Player not found:", targetPlayerName)
			return
		end

		if not DynamicIcons[targetPlayer.UserId] then
			warn("No dynamic icons found for player:", targetPlayerName)
			return
		end

		local index = table.find(DynamicIcons[targetPlayer.UserId], iconName)
		if index then
			table.remove(DynamicIcons[targetPlayer.UserId], index)
			print("Removed icon", iconName, "from player", targetPlayerName)
		else
			warn("Icon not found for player:", iconName)
			return
		end

		pcall(function()
			DynamicIconsStore:SetAsync(targetPlayer.UserId, DynamicIcons[targetPlayer.UserId])
		end)

		if targetPlayer.Character then
			local overheadGuiClone = targetPlayer.Character:FindFirstChild("HumanoidRootPart"):FindFirstChild("OverheadGui")
			if overheadGuiClone then
				local iconFrame = overheadGuiClone:WaitForChild("IconFrame")
				updateIcons(targetPlayer, targetPlayer:WaitForChild("leaderstats").Level.Value, iconFrame)
			else
				warn("OverheadGui clone not found for target player!")
			end
		else
			warn("Target player's character not found.")
		end

		sendDiscordWebhook(player, targetPlayer, iconName, "removed")
	end
end

--[ Connect Command Handler ]--
Players.PlayerAdded:Connect(function(player)
	player.Chatted:Connect(function(message)
		handleCommand(player, message)
	end)
end)


--[ PlayerAdded Event ]--
Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function(char)
		local humanoid = char:WaitForChild("Humanoid")

		humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None


		local overheadGuiClone = OverheadGui:Clone()
		overheadGuiClone.UsernameFrame.Username.Text = plr.DisplayName
		overheadGuiClone.Parent = char:WaitForChild("HumanoidRootPart")


		local iconFrame = overheadGuiClone:WaitForChild("IconFrame")
		local clanIcon = overheadGuiClone.UsernameFrame.Clan
		local levelGradient = overheadGuiClone.Level.UIGradient
		local levelGradientScript = overheadGuiClone.Level.UIScript
		local roleGradient = overheadGuiClone.Rank.UIGradient
		local roleScript = overheadGuiClone.Rank.Script
		local usernameGradient = overheadGuiClone.UsernameFrame.Username.UIGradient
		local usernameGradientScript = overheadGuiClone.UsernameFrame.Username.UIScript
		local topTagFrame = overheadGuiClone:WaitForChild("TopTagFrame")

		local rank = 0
		local role = "JOIN OFFFICIAL TAMBAYAN GROUP"
		pcall(function()
			rank = plr:GetRankInGroup(GroupId)
			if rank > 0 then
				role = plr:GetRoleInGroup(GroupId)
			end
		end)

		if customRanks[plr.Name] then
			overheadGuiClone.Rank.Text = customRanks[plr.Name]
		else
			overheadGuiClone.Rank.Text = role
		end

		task.wait(5)

		local level = plr:WaitForChild("leaderstats"):WaitForChild("Level") -- Moved this line here
		if level then
			overheadGuiClone.Level.Text = "Level: "..addCommas(level.Value)

			updateLevelTextColor(overheadGuiClone.Level, level.Value, levelGradient, levelGradientScript)
			updateIcons(plr, level.Value, iconFrame, clanIcon) -- Pass clanIcon to the function

			level:GetPropertyChangedSignal("Value"):Connect(function()
				overheadGuiClone.Level.Text = "Level: " .. addCommas(level.Value)
				updateLevelTextColor(overheadGuiClone.Level, level.Value, levelGradient, levelGradientScript)
				updateIcons(plr, level.Value, iconFrame, clanIcon) -- Pass clanIcon to the function
			end)
		end

		if MarketplaceService:UserOwnsGamePassAsync(plr.UserId, RainbowUsernameGamepass) then
			usernameGradient.Enabled = true
			usernameGradient.Color = rainbowColorSequence
			usernameGradientScript.Disabled = false
		else
			usernameGradient.Enabled = false
		end

		roleGradient.Enabled = roles[rank].Rainbow
		if roles[rank].Rainbow then
			roleGradient.Color = rainbowColorSequence
		end
		roleScript.Disabled = not roles[rank].Rainbow
		roleScript.Parent.TextColor3 = roles[rank].Color

		local function updateTopTags()
			while true do
				local tagsToShow = {}

				local function checkLeaderboard(store, statName)
					local success, pages = pcall(function()
						return store:GetSortedAsync(false, 3)
					end)

					if not success then
						warn("Error getting leaderboard data:", pages)
						return
					end

					local topPlayers = pages:GetCurrentPage()

					for i = 1, 3 do
						if topPlayers[i] then
							if string.lower(topPlayers[i].key) == string.lower(plr.UserId) then
								table.insert(tagsToShow, {name = "Top" .. i .. statName, rank = i})
							end
						end
					end
				end

				checkLeaderboard(topRaisedStore, "Raised")
				checkLeaderboard(topDonatedStore, "Donated")
				checkLeaderboard(topLevelStore, "Level")

				for _, child in ipairs(topTagFrame:GetChildren()) do
					if child:IsA("TextLabel") then
						child:Destroy()
					end
				end

				for _, tagData in ipairs(tagsToShow) do
					local tag = TopTags:FindFirstChild(tagData.name)
					if tag then
						local tagClone = TopTagsPlaceHolder:Clone()
						tagClone.Text = tag.Text
						tagClone.Name = tagData.name
						tagClone.Visible = true
						tagClone.Parent = topTagFrame

						if tagData.rank == 1 then
							tagClone.TextColor3 = COLORS.Gold
						elseif tagData.rank == 2 then
							tagClone.TextColor3 = COLORS.Silver
						elseif tagData.rank == 3 then
							tagClone.TextColor3 = COLORS.Bronze
						end
					else
						warn("Tag not found:", tagData.name)
					end
				end

				task.wait(600)
			end
		end

		updateTopTags()

	end)
end)
