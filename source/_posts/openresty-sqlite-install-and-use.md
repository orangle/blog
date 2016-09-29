title: OpenResty中安装和使用LJSQlite3
comments: true
toc: true
date: 2016-09-27 18:34:15
tags: [openresty, sqlite3]
categories: [openresty]
---

<!-- more -->
> 尽管Openresty和sqlite结合并不是好的选择，sqlite的阻塞会导致nginx整体性能的极速下降，但是总有些小应用，也不需要高性能，就是为了展示点东西，nginx+lua 一并做了。 性能有要求场景下请不要那么玩，然后在网上找到了[LJSQlite3](http://scilua.org/ljsqlite3.html)这么一个库，正好是基于cffi的，不错。

* [ljsqlite3 on github](https://github.com/stepelu/lua-ljsqlite3)
* [ljsqlite3 doc](http://scilua.org/ljsqlite3.html)


具体的操作和测试代码都放在下面了

nginx conf

```
location /sqlite {
    content_by_lua_file conf/lua/sqlitetest.lua;
    # curl 'http://127.0.0.1:8008/sqlite'
}
```

sqlitetest.lua
```
-- ljsqlite3 测试使用

-- 安装的步骤
-- ljsqlite3 基于luajit cffi写的，所以首先要安装sqlite在机器上, lua-xsys是它的依赖库
-- cd usr/local/openresty/lualib
-- git clone https://github.com/stepelu/lua-xsys.git && mv lua-xsys xsys
-- git clone https://github.com/stepelu/lua-ljsqlite3.git && mv lua-ljsqlite3 ljsqlite3

local sql = require 'ljsqlite3'
local string = string
local unpack = unpack

-- Open a temporary in-memory database, when conn is closed, database is not exist
-- when the request end, then database is also not exit, so think twice
-- local conn = sql.open("")

-- use a db file, rwc mean if the file not exits, create it
-- please notice permission of dir and file
local conn = sql.open("/tmp/test.db", 'rwc')

local function create_table(conn)
    -- create a table, insert into a record
    --
    conn:exec([[
    DROP TABLE IF EXISTS t;
    CREATE TABLE t(id TEXT, num REAL);
    ]])
    ngx.say("create table t ok")
end

local function insert_record(conn, id, num)
    local sqlstr = "INSERT INTO t VALUES('"..id.."', "..num..");"
    local res, nrow = conn:exec(sqlstr)
    -- if success res is nil and nrow is 0
    if nrow == 0 then
        ngx.say("insert ok")
    else
        ngx.say("insert failure")
    end
end

local function show_record(conn, id)
    local sqlstr = "SELECT * FROM t WHERE id=='"..id.."'"
    local id, num = conn:rowexec(sqlstr)
    -- if multiple records returned, error
    ngx.say("id:"..tostring(id))
    ngx.say("num:"..tostring(num))
end

local function all_record(conn)
    local stmt = conn:prepare("SELECT * FROM t")
    local row, names = stmt:step({}, {})
    while stmt:step(row) do
        local id, num = unpack(row)
        ngx.say("id:"..id.." num:"..num)
    end
end

local function update_record(conn, id, num)
    local sqlstr = "UPDATE t set num="..num.." where id='"..id.."';"
    local res, nrow = conn:exec(sqlstr)
    ngx.say(res)
    ngx.say(nrow)
    ngx.say("update ok")
end

local function remove_record(conn, id)
    -- 这种写法比较适合多条语句
    local stmt = conn:prepare("delete from t where id=?")
    stmt:reset():bind(id):step()
    ngx.say("remove ok")
end

-- 解析 op 和 参数，然后操作，主要熟悉api，并没有rest的风格。。。
-- sql 也就直接拼接了
-- curl 'http://127.0.0.1:8008/sqlite?op=xxx&id=xx&num=xx'
local op = ngx.var.arg_op
local id = ngx.var.arg_id
local num = ngx.var.arg_num

if op == nil then
    ngx.exit(404)
end

if op == 'create' then
    create_table(conn)
elseif op == "insert" then
    insert_record(conn, id, num)
elseif op == "showone" then
    show_record(conn, id)
elseif op == "showall" then
    all_record(conn)
elseif op == "update" then
    update_record(conn, id, num)
elseif op == "remove" then
    remove_record(conn, id)
end

conn:close()
ngx.exit(200)
```

sqlitetest.lua的api不是针对openresty，如果想要在openresty中真正的使用，还要做一些封装等工作。
