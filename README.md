```lua
local HttpService = game:GetService("HttpService")

local req =
	request or
	http_request


local owner = "heisenburgah"
local repo = "potential-octo-invention"
local branch = "main"

local function encodePath(path)
	return path:gsub("([^%w%-%._~/])", function(c)
		return string.format("%%%02X", string.byte(c))
	end)
end

local treeUrl = string.format(
	"https://api.github.com/repos/%s/%s/git/trees/%s?recursive=1",
	owner,
	repo,
	branch
)

local response = req({
	Url = treeUrl,
	Method = "GET"
})

assert(response and response.Body, "Failed to fetch repo tree")

local data = HttpService:JSONDecode(response.Body)
local firstLuaFile

for _, item in ipairs(data.tree) do
	if item.type == "blob" then
		local path = item.path:lower()

		if path:match("%.lua$") or path:match("%.luau$") then
			firstLuaFile = item.path
			break
		end
	end
end

assert(firstLuaFile, "No .lua or .luau file found in repo")

local rawUrl = string.format(
	"https://raw.githubusercontent.com/%s/%s/%s/%s",
	owner,
	repo,
	branch,
	encodePath(firstLuaFile)
)

local rawResponse = req({
	Url = rawUrl,
	Method = "GET"
})

assert(rawResponse and rawResponse.Body, "Failed to fetch Lua file")
loadstring(rawResponse.Body)()
```
