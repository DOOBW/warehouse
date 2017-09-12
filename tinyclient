--[[
  protocol:
    Player@getpass                                       -->  key to chat/clipboard
    Player@[sync]                                        -->  items from Player to db of Player
    Player@[list]                                        -->  list of items in db of Player
    Player@[give modname:itemname:meta size]             -->  item from db to Player
    Player@[transfer target modname:itemname:meta size]  -->  item from db of Player to db of Target
    Player@[make newCellName]                            -->  create new cell with name
    Player@[cells]                                       -->  user list
    [] = crypt(deflate(command))
]]

local component = require('component')
local ser = require('serialization')
local event = require('event')
local term = require('term')
local modem = component.modem
local data = component.data
local port = 1

local function crypt(pswd, text, o)
  local key = data.md5(pswd)
  local check = data.sha256(key)
  if o == 'e' then
    local iv = data.random(16)
    local d = data.encrypt(text, key, iv)
    return iv..check..d
  else
    local iv, cv, d = text:sub(1,16), text:sub(17,48), text:sub(49)
    if cv == check then
      return data.decrypt(d, key, iv)
    end
  end
end

local CURRENT_USER = ({event.pull('key_down')})[5]
local CURRENT_KEY = ''
print(CURRENT_USER)
modem.open(port)
while true do
  term.write('> ')
  local i = io.read()
  if i == 'getpass' then
    modem.broadcast(port, CURRENT_USER..'@'..i)
    print(CURRENT_USER..'@'..i)
  elseif i == 'key' then
    term.write('key> ')
    CURRENT_KEY = term.read(_,_,_,'*'):sub(1,-2)
  elseif i == 'cell' then
    term.write('cell> ')
    CURRENT_USER = io.read()
  else
    modem.broadcast(port, CURRENT_USER..'@'..crypt(CURRENT_KEY, i, 'e'))
    if i == 'list' or i == 'cells' then
      local e = {event.pull(1, 'modem_message')}
      local msg, cell = e[6]
      if msg then
        for i = 1, #msg do
          if msg:sub(i, i) == '@' then
            cell, msg = msg:sub(1, i-1), msg:sub(i+1)
            break
          end
        end
        local tbl = ser.unserialize(data.inflate(crypt(CURRENT_KEY, msg)))
        for i, j in pairs(tbl) do
          print('['..i..'] '..j)
        end
      end
    end
  end
end

