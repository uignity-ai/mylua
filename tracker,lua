-- db_report_v3h_web — Equip Best + kirim Google Sheet
-- Loader device:
--   getgenv().UNIQ_IDENT = "device1"
--   loadstring(game:HttpGet("https://RAW_URL_KAMU.lua"))()

local UNIQ_IDENT = (getgenv and getgenv().UNIQ_IDENT)
	or (getgenv and getgenv().deviceidentifer)
	or _G.UNIQ_IDENT
	or _G.deviceidentifer
	or "device1"

local WEBHOOK_URL = (getgenv and getgenv().WEBHOOK_URL)
	or "https://script.google.com/macros/s/AKfycbxtItIfdhfctNkCfCvYDHfzhT_iTB9sNPSAELqqKQilFZJdFoX4eDvTuLaOWQOokhlD/exec"

local LOOP_MINUTES = tonumber((getgenv and getgenv().LOOP_MINUTES) or _G.LOOP_MINUTES) or 30

local genv = (getgenv and getgenv()) or _G
genv._DB_REPORT_V3G = false
genv._DB_REPORT_V3F = false
task.wait(0.15)
if genv._DB_REPORT_V3H then
	warn("[DB] already")
	return
end
genv._DB_REPORT_V3H = true

local function L(s)
	s = tostring(s)
	pcall(function()
		local p = ""
		pcall(function()
			p = readfile("Workspace/db_report_log.txt") or ""
		end)
		if #p > 40000 then
			p = p:sub(-20000)
		end
		writefile("Workspace/db_report_log.txt", p .. os.date("%H:%M:%S ") .. s .. "\n")
	end)
	warn("[DB] " .. s)
end

local function fmtRate(n)
	n = tonumber(n) or 0
	local a = math.abs(n)
	local num, suf
	if a >= 1e12 then
		num, suf = n / 1e12, "t"
	elseif a >= 1e9 then
		num, suf = n / 1e9, "b"
	elseif a >= 1e6 then
		num, suf = n / 1e6, "m"
	elseif a >= 1e3 then
		num, suf = n / 1e3, "k"
	else
		local s = string.format("%.1f", n)
		s = s:gsub("%.0$", "")
		return s .. "/s"
	end
	local s = string.format("%.1f", num)
	s = s:gsub("%.0$", "")
	return s .. suf .. "/s"
end

local FALLBACK_RANK = {
	Common = 1, Uncommon = 2, SuperRare = 2, Rare = 3, Epic = 4,
	Legendary = 5, Mythic = 6, BrainrotGod = 6, Rainbow = 6, Prismatic = 6,
	Cosmic = 7, Secret = 8, Limited = 9, Eternal = 9,
	Transcendent = 10, Divine = 10, Titan = 11,
	LightDark = 12, ["Light & Dark"] = 12, ["Squishy God"] = 6,
}

local AssetDir = nil
pcall(function()
	local Assets = require(game:GetService("ReplicatedStorage").Data.Assets)
	AssetDir = Assets and Assets.Directory
end)

local function rarityInfo(cat)
	local name, rank = "-", 0
	if AssetDir and cat then
		local e = AssetDir[tostring(cat)]
		if type(e) == "table" and type(e.Rarity) == "table" then
			local r = e.Rarity
			name = tostring(r.DisplayName or r._id or "-")
			rank = tonumber(r.Rank) or 0
		end
	end
	if rank == 0 and name ~= "-" then
		rank = FALLBACK_RANK[name] or 0
	end
	return name, rank
end

local function findRF(sub)
	for _, x in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
		if x:IsA("RemoteFunction") and x:GetFullName():find(sub, 1, true) then
			return x
		end
	end
end

local lp = game:GetService("Players").LocalPlayer

local function equipBest()
	local rf = findRF("Haul/WearBest")
	if not rf then
		L("WearBest RF missing")
		return false
	end
	local ok, ret = pcall(function()
		return rf:InvokeServer()
	end)
	L("EquipBest ok=" .. tostring(ok) .. " ret=" .. tostring(ret))
	return ok and ret == true
end

local function getIncomePerSec()
	local sum = 0
	local live = findRF("PenRoster/AskLiveSnapshot")
	if live then
		local ok, data = pcall(function()
			return live:InvokeServer()
		end)
		if ok and type(data) == "table" then
			for _, plot in pairs(data) do
				if type(plot) == "table" and plot.OwnerUserId == lp.UserId then
					for _, rec in pairs(plot.Records or {}) do
						if type(rec) == "table" and type(rec.MoneyPerSecond) == "number" then
							sum = sum + rec.MoneyPerSecond
						end
					end
				end
			end
		end
	end
	return sum
end

local function getSpeed()
	local ls = lp:FindFirstChild("leaderstats")
	if ls then
		for _, name in ipairs({ "Speed", "WalkSpeed", "SPD" }) do
			local v = ls:FindFirstChild(name)
			if v and tonumber(v.Value) then
				return tonumber(v.Value)
			end
		end
	end
	return 0
end

local function getTopPets()
	local pets = {}
	local live = findRF("PenRoster/AskLiveSnapshot")
	if not live then
		return pets
	end
	local ok, data = pcall(function()
		return live:InvokeServer()
	end)
	if not ok or type(data) ~= "table" then
		return pets
	end
	for _, plot in pairs(data) do
		if type(plot) == "table" and plot.OwnerUserId == lp.UserId then
			for uid, rec in pairs(plot.Records or {}) do
				if type(rec) == "table" then
					local id = rec.ItemData
					local cat = type(id) == "table" and (id.Category or id.DisplayName) or nil
					local name = cat or tostring(uid)
					if type(id) == "table" and id.DisplayName then
						name = id.DisplayName
					end
					local rarName, rarRank = rarityInfo(cat or name)
					pets[#pets + 1] = {
						name = tostring(name),
						rarity = rarName,
						rank = rarRank,
						income = type(rec.MoneyPerSecond) == "number" and rec.MoneyPerSecond or 0,
					}
				end
			end
		end
	end
	table.sort(pets, function(a, b)
		if (a.rank or 0) ~= (b.rank or 0) then
			return (a.rank or 0) > (b.rank or 0)
		end
		return (a.income or 0) > (b.income or 0)
	end)
	return pets
end

local function formatTop3(pets)
	local parts = {}
	for i = 1, math.min(3, #pets) do
		local p = pets[i]
		parts[#parts + 1] = p.name .. " " .. p.rarity .. " " .. fmtRate(p.income)
	end
	if #parts == 0 then
		return "-"
	end
	return table.concat(parts, " | ")
end

local function urlenc(s)
	s = tostring(s or "")
	return (s:gsub("([^%w%-_%.~])", function(c)
		return string.format("%%%02X", string.byte(c))
	end))
end

local function httpGet(url)
	local req = (syn and syn.request) or http_request or request or (http and http.request)
	if req then
		local ok, res = pcall(function()
			return req({ Url = url, Method = "GET" })
		end)
		if ok and res then
			return res.Body or res.body
		end
	end
	if game and game.HttpGet then
		local ok, body = pcall(function()
			return game:HttpGet(url)
		end)
		if ok then
			return body
		end
	end
	return nil
end

local function send()
	equipBest()
	task.wait(1.5)

	local pets = getTopPets()
	local params = {
		username = lp.Name,
		["income/s"] = fmtRate(getIncomePerSec()),
		["speed/s"] = fmtRate(getSpeed()),
		["top 3 pet"] = formatTop3(pets),
		["uniq-ident"] = UNIQ_IDENT,
	}
	local q = {}
	for k, v in pairs(params) do
		q[#q + 1] = urlenc(k) .. "=" .. urlenc(v)
	end
	local url = WEBHOOK_URL .. "?" .. table.concat(q, "&")
	L("send id=" .. UNIQ_IDENT .. " user=" .. params.username
		.. " income=" .. params["income/s"]
		.. " speed=" .. params["speed/s"]
		.. " top=[" .. params["top 3 pet"] .. "]")

	local body = httpGet(url)
	if body then
		L("http ok " .. tostring(body):sub(1, 120))
	else
		L("http fail")
	end
end

task.spawn(function()
	task.wait(3)
	L("armed v3h_web id=" .. UNIQ_IDENT .. " loop=" .. LOOP_MINUTES .. "m")
	local ok, err = pcall(send)
	if not ok then
		L("ERR " .. tostring(err))
	end
	while genv._DB_REPORT_V3H do
		task.wait(LOOP_MINUTES * 60)
		ok, err = pcall(send)
		if not ok then
			L("ERR " .. tostring(err))
		end
	end
end)
