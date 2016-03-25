#LUA

###基础：

 - lua区分大小写，
 - 不等于 ~=
 - 尽量不要用_大写字母，LUA有特殊用途。
 - 行注释- -, 块注释：
```
--[[   
.....    
--]]
```
取消注释的话，加多一横：
```
---[[
......
--]]
```
中间还可以加入等号，如：--[==[ ...... ]==]


 - 直接执行lua的头一行加入：
```
#!/usr/local/bin/lua
或者
#!/usr/bin/env lua
```
 - 真假判断：if (0) == true   if (1) == true  if (nil) == false

```
if (0) then
    print("abc")
end
[result]: abc
```
 - 变量互换：
 
```
   a, b = b, a
   a, b = fun()
```
如此设计主要用于函数的多个返回值接收

 - 【尽可能的使用局部变量，内存回收更快。】


###字符串
 - 字符串连接符号：.. 
 - 获取字符串长度,直接在变量前面加#；
 - 长字符串定义可以使用[==[...]==]，但是不转义。
 
```
    str = “abcd" .. "efg"  ==> "abcdefg
    print(#str)    ==>  7

    str = [==[
    abcd\n
    \123\"kasldfasdf";'';;'
    ]==]
```
 - 字符串不可变，每次赋值都会创建新的空间，在读取文本文件的时候会出现大量的空间创建操作，效率非常低下：
 
```
local buff = ""
for line in io.lines() do
    buff = buff .. line .. "\n"
end
```
解决办法：赋值到table, table.concat来处理。

应尽量避免io.write(aa..bb..cc)，应该使用io.write(aa, bb, cc)。可以避免连接操作，效率跟高。

- 格式化：string.format()：
- 
```
string.format("%d, %x, %s", int, hex, string)

> a='a "proasfasdf"\\string'
> print(string.format("%q",a))
"a \"proasfasdf\"\\string"
> print(a)
a "proasfasdf"\string
```

- sub()。 负数索引，从字符串的尾数开始计数，-1最后一个字符，以此类推。
```
> s = "[in buckets]"
> print(string.sub(s, 2, -2))

in buckets
```
- 字符串模式匹配，未使用perl, posix的模式匹配，自己精简重写。
```
> s="hello world"
> print(string.find(s, "world"))
7	11
> n, m = string.find(s, "world")
> print(string.sub(s, n, m))
world
> print(string.gsub(s, "world", "apple"))
hello apple	1
```
```
> date = "Today is 18/3/1992"
> d = string.match(date, "%d+/%d+/%d+")
> print(d)
18/3/1992
> print(string.find(date, "%d+/%d+/%d+"))
10      19
```

- and or
a and b -- 如果a 为false，则返回a，否则返回b
a or b -- 如果a 为true，则返回a，否则返回b

一个很实用的技巧：如果x 为false 或者nil 则给x 赋初始值v
x = x or v
等价于
if not x then x = v  end

C 语言中的三元运算符
a ? b : c
在Lua 中可以这样实现：
(a and b) or c


###TABLE
 - table通常以1作为数组起始值，arr[1] = "first";
    #table名称表示table的长度。 table的尾元素：arr[#arr]，不建议用#做table的空判断，#table通常以碰到第一个nil的元素，计算table的大小。

```
    a = {}
    a[100] = 1
    a.abc = 'abc'
    print(#a)  ===> 0
```

- 对table的使用，要预先分配大小，减少rehash。当我们把新的见解给table额的时候，若数组合哈希表已经满了，更会触发一个再哈希，再哈希的代价是高昂的，首先会在内存中分配一个新的长度的数组，然后将所有记录再全部哈希一遍，将原来的记录转移到新数组中。

 - table空判断：
```
a = {}; b = {}
a['abc'] = 100
print(next(a))              ==> abc   100
--print(_G.next(a))         ==> abc   100
if (a == b)                 ==> 永远都是false
```  
 - 添加删除
 尾部添加删除
 table.insert(t, value)
 table.remove(t)
 头部添加删除
 table.insert(t, 1, value)
 table.remove(t, 1)

 - table字段判断
 没有什么办法，通常做法是建立另外一个tabel做索引判断：
 
```
local items = { apple=true, orange=true, pear=true, banana=true }
if (items.apple) .... 
```
 - table遍历
```
a = {}
for k,v in pairs(a) do
    print(k .. v)
end
```
- 常用table函数：table.insert(t, value), table.remove(t, index)
如此可以使用#table，计算table的大小。

- raw方法
lua_rawget:
用法同lua_gettable,但更快(因为当key不存在时不用访问元方法__index)
lua_rawset:
用法同lua_settable,但更快(因为当key不存在时不用访问元方法__newindex)

- 序列化双向转换:参考：
```
function tostring(tt)
    local l={}
    for e in pairs(tt) do
        l[#l+1] = e
    end
    return "{".. table.concat(l, ", ").."}"
end

function tt.print(s)
    print(tt.tostring(s))
end

tt={....略...}
> tt.print(tt)
{1, 2, 12, aasd, key}
```

```
function print_r(root)
    local cache = {  [root] = "." }
    local function _dump(t,space,name)
        local temp = {}
        for k,v in pairs(t) do
            local key = tostring(k)
            if cache[v] then
                table.insert(temp,"+" .. key .. " {" .. cache[v].."}")
            elseif type(v) == "table" then
                local new_key = name .. "." .. key
                cache[v] = new_key
                table.insert(temp,"+" .. key .. _dump(v,space .. (next(t,k) and "|" or " " ).. string.rep(" ",#key),new_key))
            else
                table.insert(temp,"+" .. key .. " [" .. tostring(v).."]")
            end
        end
        return table.concat(temp,"\n"..space)
    end
    print(_dump(root, "",""))
end
```
http://blog.csdn.net/icyday/article/details/8099197

skynet/lualib/logger.lua



###函数
 - 参数默认值, 多返回值，...参数
 
```
local function tt(n)
    n = n or 1
    count = count + n
    return n, count
end

print(tt())         ==> n, count
print(tt().."abc")  ===> nabc
```
 - 匿名函数
 
```
    fun = function (a, b) return (a.name > b.name) end
```

 - 把函数存入table，通过如此来实现面向对象的开发。大部分的lib库都是采用这种机制，如io.read,math.sin
 
 ```
 lib = {}
 lib.foo = function (x, y)
    print("hello");
 end
 
 lib.foo()      ===> hello
 ```
 
 - 沙盒封转，将系统原有的函数进行再次封装
 
```
oldsin = math.sin
math.sin = function (x)
    return oldsin(x*math.pi/180)
end
```
- 尾调用消除，因为是最后的函数调用就不需要再保留之前的栈空间。递归时候很管用，内存不溢出。

```
function f(x)
    return g(x)
end
```

- 闭合函数，变量i为非局部变量

```
function tt()
    local i=0;
    return function()
    i=i+1;
    return i;
    end;
end

c1 = {f1=tt()};
print("f1"..c1.f1()); 
print("f1"..c1.f1());

f11
f12

```

###编译和执行
 - loadfile会从一个文件加载lua代码块,loadstring()类似，不执行，只是编译代码，并返回结果，失败返回nil及错误描述，如：

```
> print(loadfile("hello.lua"))
nil	hello.lua:3: syntax error near 'printmsg'

```
 - packet.loadlib()加载C函数库，失败返回nil及错误描述。该返回值也是lua错误处理的基本方法。
 
 - pcall错误返回false及错误信息，正确返回true。

```
> print(pcall(function() a="a"+1 end))
false	stdin:1: attempt to perform arithmetic on a string value
```

###协程
- 小例子, coroutine的table完成协程的基本操作，coroutine.create(func()), coroutine.resume(co), coroutine.yield(), coroutine.status(co)，执行错误返回false及错误信息。
4个状态：suspended,running,dead,normal(当协程调用另外一个协程时就是normal状态)

```
co = coroutine.create(function()
    for i=0,10 do
        print("co", i)
        coroutine.yield()
    end
end)

> print(coroutine.status(co))
suspended
> coroutine.resume(co)
co	0
....
> coroutine.resume(co)
co	10
> print(coroutine.resume(co))
false	cannot resume dead coroutine
> print(coroutine.status(co))
dead

> print(coroutine.resume(co))
false	stdin:2: attempt to compare nil with number
```
- 协程数据交换，实际应用中非常常见

```
co = coroutine.create(function(a,b)
    coroutine.yield(a+b, a-b)
    return 6,7
end)

> print(coroutine.resume(co, 20, 10)) 
true	30	10
> print(coroutine.resume(co, 20, 10))
true	6	7
> print(coroutine.resume(co, 20, 10))
false	cannot resume dead coroutine
```
 - filter()对数据进行预处理，filter和producer都是返回一个coroutine对象，详细参考：9.2章节
 - coroutine.wrap()
 
 - c语言的yield()需要交出程序的控制权，需要强制调用return返回。
 
##疑问：
为什么一个用户一个协程？
线程的管理是不是放在lua中去？
lualib-src/skynet.c   

lualib/skynet.lua
local c = require "skynet.c"


###文件处理
 - Entry 与 dofile()
csv文件, 12.1章节

 - 简单io操作, 预定义句柄：io.stdio, io.stdout, io.stderr

```
> f = io.open("tt.txt", r)
> s = f:read("*all")
> io.stderr:write(s)
> f:write(s)
> f:close()
```

- 按块和行来读取文件

```
local lines, rest = f:read(BUFSIZE, "*line")
```



###元表与元方法

```
_a1 = {20, 1, key1 = "hello", key2 = "world", lang = "lua"}  
_a2 = {  
     key1 = "hello new",  
     key2 = "world new"  
 }  
print("\na2的metatable:",getmetatable(_a2)) 
print("\nlanguage:", _a2["lang"])
setmetatable(_a2, {__index = _a1}) 
print("\na2的metatable:",getmetatable(_a2)) 
print("\nlanguage:", _a2["lang"])

function tostring(tt)
    local l={}
    for e in pairs(tt) do
        l[#l+1] = e
    end
    return "{".. table.concat(l, ", ").."}"
end

setmetatable(_a2, {__tostring = totring}) 
print(_a2)

```

元方法：
__add: 所挂接table的加法操作
__mul: 乘法操作
__div: 除法操作
__sub: 减法操作
__unm: 负操作, 即: -table的含义
__tostring: 当table作为tostring()函式之参数被呼叫时的行为(例如: print(table)时将呼叫tostring(table)作为输出结果)
__concat: 连接操作(".."运算符)
__index: 当table中不存在的key值被试图获取时的行为
__newindex: 在table中产生新key值时的行为
__gc：垃圾回收元函数，当Lua回收表格时就会调用它。


通过元方法可以实现只读表，跟踪表的访问。

###环境
- Lua将所有的全局变量保存在一个常规的table：_G{}
print_table(_G)打印全局变量

### 模块
 - require:使用模块， module:创建模块，强制再次加载：

```
packet.loaded['module1'] = nil
require 'module1'
```
 - require为指定模块查找Lua文件，通过loadfile加载文件；如果未找到lua文件，就会去查找C程序库，通过loadlib来加载代码。查找的路径：

```
package.cpath [/usr/local/lib/lua/5.2/?.so;/usr/local/lib/lua/5.2/loadall.so;./?.so]

package.path [/usr/local/share/lua/5.2/?.lua;/usr/local/share/lua/5.2/?/init.lua;/usr/local/lib/lua/5.2/?.lua;/usr/local/lib/lua/5.2/?/init.lua;./?.lua]
```
- lua模块定义：参数：package.seeall 访问外部函数

```
module('sapi', package.seeall) 

或者：
local module_name = ...
local M  = {}
_G[module_name] = M
packet.loaded[module_name] = M
```

- 良好行为的C程序库应该导出一个名为"luaopen_模块名"的函数，require会在链接完程序库后，尝试调用这个函数。


###对象
- table就是对象

```
Account = {balance=0,
        withdraw = function(self, v)
                    self.balance = self.balance - v
                    end
        }

function Account:deposit(v)
    self.balance = self.balance + v
end


> Account:withdraw(1000)
> print_r(Account);
+withdraw [function: 0x7f7fe2502c30]
+balance [-1000]
+deposit [function: 0x7f7fe25033e0]
> Account:deposit(10000)
> print_r(Account);
+withdraw [function: 0x7f7fe2502c30]
+balance [9000]
+deposit [function: 0x7f7fe25033e0]
```
引入self规避Account = nil的错误。


##弱引用table
代码与垃圾回收器之间的交互

- 小例子：动态编译

```
local results = {}
setmetatable(results, {__mode = "v"})  --使value成为弱引用。"k"=key "v"=value
function mem_loadstring(s)
    local res = results[s]
    if res == nil then
        res = assert(loadstring(s))
        result[s] = res
    end
    return res
end
```
- table默认值，如果defautls没有弱引用key, 就会持久保存下去。

```
local defaults = {}
setmetatable(defaults, {__mode = "k"})
local mt = {__index = function(t) 
            return defaults[t] 
            end}

function setDefault(t, d)
    defaults[t] = d
    setmetatable(t, mt)
end
```

###调试
- 自省机制
debug.getlocal()
debug.getinfo()

- 钩子

### CAPI

- 栈
几乎所有的API函数都会用到栈，luaL_loadbuffer将编译好的程序块或者错误消息留在栈中，lua_pcall会调用栈中的函数。
Lua严格的按LIFO(Last in First out), C代码则不会，可任意检索，插入，删除。
最简单的C和API数据交换方式：lua_getglobal(), lua_set_global()


- 栈索引
参考物：栈底， 第一个压入栈底的元素为1，之后为2，3，类推。
参考物：栈顶， 如果从栈顶来获取元素，如果栈内三个元素，栈顶为-1（最后压入元素），越向栈底靠近，则为-2, -3。

- 压入元素
void lua_pushnil(lua_State *L);
void lua_pushboolean(lua_State *L, int bool);
void lua_pushnumber(lua_State *L, lua_Number n);
void lua_pushinteger(lua_State *L, lua_Integer n);
void lua_pushlstring(lua_State *L, const char *s, size_t len);
void lua_pushstring(lua_State *L, const char *s);

void lua_setfield (lua_State *L, int index, const char *k);
做一个等价于 t[k] = v 的操作， 这里 t 是给出的有效索引 index 处的值， 而 v 是栈顶的那个值。
lua_pushvalue(L, -4) 并不是往栈顶插入元素-4， 而是把在栈中位置为-4的元素copy之后插入于栈顶中

- 查询元素
int lua_is*(lua_State *L, int index);
lua_isnumber不会检查值是否为数字类型，而是检查值能否转换为数字类型。
lua_isstring也类似，因此，对于任意数字,lua_isstring都会返回真。

- lua_type(lua_State *L, int index);会返回元素的类型。

- lua_to*从栈中获取一个值，如果指定的元素不正确的类型，则会返回0或者NULL.


```
static void stackDump(lua_State* L)
{
    int i;
    int top = lua_gettop(L);   // 获取栈中元素数量
    for ( i=1; i<=top; i++)    // 遍历
    {
        int t = lua_type(L,i); // 获取元素类型
        switch(t)
        {
            case LUA_TSTRING:      // 字符串
                printf("'%s'",lua_tostring(L,i));
                break;
            case LUA_TBOOLEAN:
                printf(lua_toboolean(L,i)?"true":"false");
                break;
            case LUA_TNUMBER:      // 数字
                printf("%g",lua_tonumber(L,i));
                break;
            default:               // 其它值
                printf("%s",lua_typename(L,t));
                break;
        }
        printf("\t");
    }
    printf("\n");
}
```

- 通用的lua函数调用：call_va("f", "dd>d", x, y, &z);
call_va(const char* func, const char *sig);

- 返回table
```
int func_return_table(lua_State *L)
{
 lua_newtable(L);//创建一个表格，放在栈顶
 lua_pushstring(L, "mydata");//压入key
 lua_pushnumber(L,66);//压入value
 lua_settable(L,-3);//弹出key,value，并设置到table里面去

 lua_pushstring(L, "subdata");//压入key
 lua_newtable(L);//压入value,也是一个table
 lua_pushstring(L, "mydata");//压入subtable的key
 lua_pushnumber(L,53);//value
 lua_settable(L,-3);//弹出key,value,并设置到subtable

 lua_settable(L,-3);//这时候父table的位置还是-3,弹出key,value(subtable),并设置到table里去
 lua_pushstring(L, "mydata2");//同上
 lua_pushnumber(L,77);
 lua_settable(L,-3);
 return 1;//堆栈里现在就一个table.其他都被弹掉了。
}

返回的表结构是:
{
 "mydata" = 66,
 "mydate2" = 77,
 "subdata" = 
 {
  "mydata" = 53
 }
}
```

```
C通过lua文件获取table
        lua_getglobal(L, "tt");

        lua_pushstring(L, "android");
        lua_gettable(L, -2);
或者：
        lua_getfield(L, -1, "iOS");
```

- 第三方cmodule，如使用luaL_register需要改为 luaL_newlib。如lfs库luaL_register (L, "lfs", fslib) 改为luaL_newlib(L,fslib);。
这里本来第二个参数是表明，非nil是把所有接口放到一个全局变量table中，nil就是所有接口都是全局函数。现在是强制取消全局接口了。



###编译lua
yum install readline-dev
vim lua Make add gcc option -fPIC(for .so)


###状态保持

- C API提供3个地方保存数据：注册表，环境变量，upvalue， Lua函数的话就是全局变量，环境变量和非局部的变量(closure中)。

- 注册表是全局的table只能通过CAPI访问，
lua_getfield(L,  LUA_REGISTRYINDEX,  "key")
int key  = lua_ref(L, LUA_REGISTRYINDEX)：
从栈顶弹出一个数据，同时生成一个整数Key写入注册表内。对应的释放函数：
lua_unref(L, REGSITRYINDEX, key);
```
static const void *key = 0;

lua_pushlightuserdata(L, (void *)&key);
lua_pushnumber(L, myNumber);
lua_settable(L, LUA_REGISTRYINDEX);

lua_pushlightuserdata(L, (void *)&key);
lua_gettable(L, LUA_REGISTRYINDEX);
myNumber = lua_tonumber(L, -1);
```

- 环境: 模块变量的存储

```
lua_newtable(L);
lua_replace(L, LUA_ENVIRONINDEX);
luaL_register(L, <库名>, <函数名>);

```

-upvalue类似与C语言的静态变量机制



###userdata
创建内存块：
void * lua_newuserdata(Lua_State *L, size_t size);

char *p  = (char *) lua_newuserdata(L, 1024);

从栈获取内存块指针：
char *p = (char *)lua_touserdata(L, 1);


###元表
可以做传入的数据指针判断，可以处理__gc垃圾回收

luaL_newmetatable(L, tname)
luaL_getmetatable(L, tname)
luaL_checkudata(L, index, tname)




###其他
- 日期格式化：

```
> print(os.time())
1408409867
> print(os.date("%y-%m-%d %H:%M:%S", os.time()))
14-08-19 08:59:31
> print(os.date("%Y-%m-%d %H:%M:%S", os.time()))
2014-08-19 08:59:34
```
lua.h中声明或定义的函数名都以lua_作为前缀。
头文件lauxlib.h定义的函数名都以luaL_作为前缀。



_G, _ENV


将 lua compiler 和 lua runtime 拆开为两个 dll，客户端发布出去的时候，只提供 lua runtime，而开发版本提供 lua compiler，防止外挂破解了 vm 之后，直接写 lua脚本的外挂。

LUA非常方便嵌入到 C 语言中协同工作。并且可以获得不错的执行性能。如果有必要，还可以引入 LuaJIT 来进一步提升运行速度。

`todo`:确认lua的table可不可以直接存redis，还是要序列化？

***

lua_State \*lua_newstate (lua_Alloc f, void *ud);

创建新的、独立的状态机。如果不能创建状态机则返回NULL（由于内存不足）。参数f是分配器函数；Lua通过该函数为状态机执行全部的内存分配操作。第二参数ud是个不透明的指针，在每次调用中Lua将其传入分配器。
***

lua_State \*lua_newthread (lua_State *L);

创建新线程，将其压栈，并返回指向lua_State的指针，它表示该新线程。本函数返回的新状态机与初始状态机共享所有全局对象（例如表），但具有独立的执行栈。
没有关闭或销毁线程的显式函数。像任何Lua对象一样，线程受垃圾收集的支配。
***
***


根据配置文件可以自动打包和解析数据，比手动一个个地写要省事多了，也提高了准确性：

```
packet = {
    "时间报文",    
    {"报文标识",        "char", 1},    
    {"报文长度",        "uint", 1},
    {"回应时间：年",    "uint", 1},
    {"回应时间：月",    "uint", 1, {1, 12}},
    {"回应时间：日",    "uint", 1, {1, 31}},
    {"回应时间：时",    "uint", 1, {0, 23}},
    {"回应时间：分",    "uint", 1, {0, 60}},
    {"回应时间：秒",    "uint", 1, {0, 60}},
}

```

```
function skynet.send(addr, typename, ...)
	local p = proto[typename]
	if watching_service[addr] == false then
		error("Service is dead")
	end
	return c.send(addr, p.id, 0 , p.pack(...))
end
```



lua 是这样做的。它把一个 table 分成数组段和 hash 段两个部分。数字 key 一般放在数组段中，没有初始化过的 key 值全部设置为 nil 。当数字 key 过于离散的时候，部分较大的数字 key 会被移到 hash段中去。



###源码阅读

Recommended reading order:
lmathlib.c, lstrlib.c: get familiar with the external C API. Don't bother with the pattern matcher though. Just the easy functions.

lapi.c: Check how the API is implemented internally. Only skim this to get a feeling for the code. Cross-reference to lua.h and luaconf.h as needed.

lobject.h: tagged values and object representation. skim through this first. you'll want to keep a window with this file open all the time.

lstate.h: state objects. ditto.

lopcodes.h: bytecode instruction format and opcode definitions. easy.

lvm.c: scroll down to luaV_execute, the main interpreter loop. see how all of the instructions are implemented. skip the details for now. reread later.

ldo.c: calls, stacks, exceptions, coroutines. tough read.

lstring.c: string interning. cute, huh?

ltable.c: hash tables and arrays. tricky code.

ltm.c: metamethod handling, reread all of lvm.c now.
You may want to reread lapi.c now.

ldebug.c: surprise waiting for you. abstract interpretation is used to find object names for tracebacks. does bytecode verification, too.

lparser.c, lcode.c: recursive descent parser, targetting a register-based VM. start from chunk() and work your way through. read the expression parser and the code 
generator parts last.

lgc.c: incremental garbage collector. take your time.
Read all the other files as you see references to them. Don't let your stack get too deep though.



在线交叉阅读链接
http://www.lua.org/source/5.1/





