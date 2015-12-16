# 编码规范

这份规范包括了命名规则、惯例用法等内容。包含下列几个方面：

-   应用程序目录结构
-   包结构
-   命名
    -   包命名
    -   类命名
    -   函数与函数命名
    -   变量命名
    -   常量命名
    -   事件命名
-   参数
-   返回值
-   错误处理
-   定义模块
-   定义类
-   HTTP/WebSocket 外部接口
-   延迟任务接口

<br />

## 应用程序目录结构

每一个 app 推荐采用如下目录结构：

~~~
+-- <APP_ROOT>/
  +-- HttpInstance.lua
  +-- WebSocketInstance.lua
  +-- WorkerInstance.lua
  +-- CommandLineInstance.lua
  |
  +-- conf/
  | +-- app_entry.conf
  | \-- app_config.lua
  |
  +-- actions/
  | \-- HelloAction.lua
  |
  +-- jobs/
  | \-- JobsHandler.lua
  |
  +-- packages/
    +-- <PACKAGE_NAME>/
      \-- <PACKAGE_NAME>.lua
~~~

说明：

-   `<APP_ROOT>` 是 app 根目录，必须放在没有空格和中文字符的目录中。
-   四个 `????Instance.lua` 文件是可选的，分别针对 HTTP 请求、WebSocket 连接、延迟任务和命令行的启动代码。
-   `conf/app_entry.conf` 文件是必选的，包含了 Nginx 初始化 app 需要的设置。
-   `conf/app_config.lua` 包含 app 的特定设置。
-   `actions` 目录放置所有供 HTTP 请求和 WebSocket 连接调用的接口。
-   `jobs` 目录放置所有延迟任务的处理接口。
-   `packages` 目录放置所有扩展包。

参考：

-   关于四个 `????Instance.lua` 启动文件和 `conf` 目录中的配置文件，请参考[启动应用程序](startup-app.md)文档。
-   `actions` 目录中的接口定义请参考[定义外部访问的接口](exports-api.md)文档。
-   `jobs` 目录中的接口定义请参考[处理延迟任务](jobs.md)文档。
-   `packages` 目录请参考本文档后续的“包结构”部分。

<br />


## 包结构

为了方便组织应用内容，GBC 支持一种简单的包结构。每一个包是一个目录，其中包含一个与目录同名的 `.lua` 文件。

例如一个名为 `hello` 的包，保存在 `packages/hello/hello.lua` 中：

~~~
local _M = {}

function _M.say()
    print("hello")
end

return _M
~~~

要使用上面名为 `hello` 的包，只需要调用：

~~~lua
local hello = cc.import("#hello")
hello.say()
~~~

在一个包中，可以有任意数量的文件，例如 GBC 自带的 `packages/gbc` 中就包含了整个 GBC 框架的所有模块文件。

说明：

-   包目录和文件必须是全小写名
-   包目录中必须有一个和目录同名的 `.lua` 文件，供 `cc.import()` 载入
-   包目录和文件必须放到应用程序的 `packages` 目录中

<br />


## 命名

命名分为多个部分。


### 包命名

虽然 Lua 支持更多字符，但考虑到不同运行环境的兼容性问题，包名字必须是全小写字母或者数字、下划线组成。此外包名字应该能够反应扩展包的实际用途。

如果一个扩展包中包含多个模块，应该采用如下的命名方式：

~~~
+-- <PACKAGE_NAME>/
  +-- <PACKAGE_NAME>.lua
  +-- <MODULE_NAME>.lua
  \-- <MODULE_NAME>.lua
~~~

例如 GBC 核心的扩展包目录结构如下：

~~~
+-- gbc
  +-- gbc.lua
  +-- ActionBase.lua
  +-- InstanceBase.lua
  \-- 更多文件
~~~

在 `gbc.lua` 中，分别载入不同模块：

~~~lua
local _M = {
    ActionBase   = cc.import(".ActionBase"),
    InstanceBase = cc.import(".InstanceBase"),

    ...
}

return _M
~~~

在需要使用 `gbc` 扩展包的地方，可以用如下代码：

~~~lua
local gbc = cc.import("#gbc")
local HelloAction = cc.class("HelloAction", gbc.ActionBase)
~~~


### 类命名

类名应该以单个或多个单词命名，每个单词首字母大写。例如：

~~~
HelloAction
JobsHandler
~~~

如果一个类是基础类，建议使用 `Base` 做最后一个单词。例如：

~~~
ActionBase
InstanceBase
~~~


### 函数与函数命名

函数与函数的命名基本规则是：

-   第一个词为“动词”或者“介词”，例如 `set`, `get`, `on`, `after` 等。
-   完整的函数名类似 `setPosition`, `getOpacity`, `afterUserSignIn`。

根据函数的不同用途，采用了不同的命名规则：

-   如果是作为基础函数来使用，那么函数名应该参考 Lua 标准库，使用全小写命名。例如：

    ~~~lua
    cc.class()
    cc.printf()
    string.ucfirst()
    ~~~

-   其他情况，函数应该由单个或多个单词命名，除第一个单词外的其他单词首字母大写。例如：

    ~~~lua
    countAction()
    echoAction()
    ~~~

-   如果是供模块或类内部使用的函数或函数，应该在命名前添加 `_` 字符。例如：

    ~~~lua
    _openSession()
    _newRedis()
    ~~~

-   如果是事件处理函数，建议使用 `on` 开头。例如：

    ~~~lua
    onConnected()
    onDisconnected()
    ~~~

-   对外部函数的本地引用，应该使用 `模块名_函数名` 的形式。例如：

    ~~~lua
    local string_format = string.format
    local os_time = os.time
    ~~~

除了以上规则，还建议采用以下的惯例：


-   立即改变对象状态的函数

    命名规范：

    -   **动词** + [名词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。

    示例：

    ~~~lua
    node:move(...)   -- 立即移动到指定位置
    node:rotate(...) -- 立即旋转到指定角度
    node:show()      -- 显示对象
    node:align(...)  -- 对齐对象
    ~~~

-   持续改变对象状态的函数

    命名规范：

    -   **动词** + [名词] + [介词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。
    -   介词通常选择 `to`, `by` 等：
        -   `to` 表示忽略对象的当前状态，最终达到指定状态
        -   `by` 表示以当前状态为基础，改变一定程度，最终状态由当前状态和改变程度决定

    示例：

    ~~~lua
    node:moveTo(...) -- 移动到指定位置
    node:moveBy(...) -- 在当前位置基础上移动一定距离
    ~~~

-   在对象上执行操作

    命名规范：

    -   **动词** + [名词] + [副词 | 介词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。

    示例：

    ~~~lua
    node:add(...)                  -- 添加子对象
    node:addTo(...)                -- 将对象添加到父对象
    node:playAnimationOnce(...)    -- 在对象上播放一次动画
    node:playAnimationForever(...) -- 在对象上持续播放动画
    ~~~


### 变量命名

变量命名应该简洁明了，除第一个单词外的其他单词首字母大写。例如：

~~~lua
local username
local sessionId
~~~

如果是类的内部成员变量，应该在命名前添加 `_` 字符。例如：

~~~lua
self._session
self._count
~~~

如果是用于 `for` 等语法中的占位符变量，可以直接使用 `_` 作为变量名。例如：

~~~lua
for _, v in ipairs(arr) do
    print(v)
end
~~~


### 常量命名

使用全大写，单词之间以 "_" 下划线分隔。例如：

~~~lua
local DEFAULT_DELAY = 1
Constants.DEFAULT_ACTION = "index"
~~~


### 事件命名

事件命名规则与常量相同，采用全大写，单词之间以 "_" 下划线分隔。例如：

~~~lua
local Bear = cc.class("Bear")

Bear.EVENT = table.readonly({
    WALK = "WALK",
    RUN  = "RUN",
})
~~~

使用函数：

~~~lua
local bear = Bear:new()
bear:addEventListener(Bear.EVENT.WALK, cc.handler(self, self.onWalk))
~~~

在定义类的 `EVENT` 事件列表时，使用了 `table.readonly()` 函数：

-   `table.readonly()` 可以将一个 `table` 设置为只读，确保事件列表不会在运行时被修改。
-   其次在运行时如果访问了列表中没有定义的值，也会抛出错误，方便排除 bug。

<br />


## 参数

如果输入参数有重要性优先级，则按照优先级排列。例如：

~~~lua
display.newSprite(filename, x, y)
~~~

如果有要操作的目标，则目标应该作为第一个参数。例如：

~~~lua
transition.move(target, ...)
~~~

<br />


## 返回值

返回值按照函数和函数的功能设计，采用以下规则：

-   对于不涉及具体逻辑的功能性函数或函数，应该只返回一个值。
-   如果有多个值需要返回，应该包装为 `table`。
-   如果函数执行中发生错误，而这个错误是可以恢复或者需要返回给调用者的，应该返回 `nil` 和错误信息字符串。
-   如果是类的函数，并且调用者不需要返回值，可以返回 `self`，以实现链式调用：

    ~~~lua
    obj:doSomething():again()
    ~~~

<br />


## 延迟任务接口

<br />


## 错误处理

对于错误处理，不同上下文需要不同的机制。基本上按照错误的严重程度来划分：

-   必须打断执行的，使用 `cc.throw()` 函数直接抛出错误信息。
-   允许继续执行的，使用 `cc.printerror()` 函数输出错误信息，然后继续执行。由于 `cc.printerror()` 会打印调用栈，所以在日志里可以看到这个问题发生的具体位置。
-   如果只是需要提醒开发者注意的意外情况，而非错误，应该使用 `cc.printwarn()` 输出警告信息。
-   如果只是单纯的调试信息，使用 `cc.printinfo()` 和 `cc.printdebug()` 输出。两者区别在于会被不同的 `cc.DEBUG` 设定所过滤。
-   如果一个函数或函数在内部发生错误时需要把具体的错误信息返回给调用者，那么应该使用如下形式：

    ~~~lua
    function test(arg)
        if type(arg) == "string" then
            return arg .. " hello"
        else
            return nil, "invalid parameter"
        end
    end

    local result, err = test(arg)
    if not result then
        -- 如果第一个返回值为 nil，则表示发生了错误
        cc.printerror(err)
    else
        ...
    end
    ~~~

<br />


## 定义模块

模块是指一个单独的 `.lua` 文件，并且可以用 `require()` 或者 `cc.import()` 函数来载入。

模块必须定义为一个 `local` `table`，并且在 `.lua` 文件最后 `return` 这个 `table`。例如：

~~~lua
-- external references
local string_format = string.format

-- declare a module
local _M = {}

-- private function references
local _concat

-- exports API
function _M.say(name)
    print(_concat("hello", name))
end

-- private

_concat = function(str1, str2)
    return string_format("%s, %s", str1, str2)
end

-- return the module
return _M
~~~

规范：

-   在源代码最前面添加外部引用
-   以 `_M` 作为定义模块的 `table`
-   所有需要导出的接口，定义为 `_M` 的 `function`
-   如果是仅用于模块内部的变量和函数，全部定义为 `local`
-   内部函数以前引用的形式定义，然后在 `--private` 后实现函数

<br />


## 定义类

类都是定义在模块中。例如：

~~~lua
local gbc = cc.import("#gbc")
local HelloAction = cc.class("HelloAction", gbc.ActionBase)

function HelloAction:sayAction(args)
    ...
end

return HelloAction
~~~

模块中的类与模块定义遵循同样的规范。

<br />


## HTTP/WebSocket 外部接口

在 GBC 中，接口的命名规则如下：

-   接口（文档里其他地方称为 `action`）的命名一律全小写。
-   `action` 由 `.` 符号分割，至少有两部分。例如 `hello.say`、`user.signin`
-   `action` 可以由多于两个部分组成。例如 `game.battle.attack`
-   `action` 中，以 `.` 分割的最后一部分，是 `action function`。
-   `action` 中，以 `.` 分割的倒数第二部分，是 `action module`。
-   `action` 中，以 `.` 分割的其他部分，是 `directory`。

`action module` 对应一个接口类：

-   接口类的命名是首字母大写的 `action module` 加上 `Action`。例如 `hello.say` 接口对应的接口类名字是 `HelloAction`。

`action function` 对应一个接口函数：

-   接口函数的命名是 `action function` 加上 `Action`。例如 `hello.say` 接口对应的接口函数名是 `sayAction()`。

其他规则：

-   接口类需要放在同名的 `.lua` 文件中。
-   接口类必须是 `gbc.ActionBase` 的继承类。
-   接口类必须通过 `ACCEPTED_REQUEST_TYPE` 字段明确定义接口可以接受的请求类型。
-   接口类必须放在应用程序的 `actions/` 目录中。

因此 `hello.say` 接口就是从客户端调用应用程序 `<APP_ROOT>/actions/HelloAction.lua` 的 `sayAction()` 函数。

假如接口名中包含 `directory` 部分，则接口文件也需要放在对应目录下。例如：`game.battle.attack` 对应 `<APP_ROOT>/actions/game/BattleAction.lua` 文件。

接口类示例：

~~~lua
local gbc = cc.import("#gbc")
local HelloAction = cc.class("HelloAction", gbc.ActionBase)

HelloAction.ACCEPTED_REQUEST_TYPE = {"http", "websocket"}

function HelloAction:init()
    self._number = math.random()
end

function HelloAction:sayAction(args)
    local username = args.username
    return {text = string.format("%s say %s", username, tostring(self._number))}
end

return HelloAction
~~~

说明：

-   当客户端请求 `hello.????` 的任何接口时，都会构造 `HelloAction` 类的实例。
-   `ACCEPTED_REQUEST_TYPE` 可以是单一请求类型的字符串，或者是包含多种请求类型的 `table`。
-   `init()` 函数会在接口类构造后调用，可以在这里做一些初始化工作。
-   `sayAction()` 函数会在客户端请求 `hello.say` 时调用。
-   接口函数只有一个参数，类型为 `table`，保存了请求包含的所有数据。

总结：

-   请求 `hello.say` 时，会首先载入 `<APP_ROOT>/actions/HelloAction.lua` 文件。然后创建其中定义的 `HelloAction` 对象的一个实例，最后调用 `sayAction()` 函数。
-   请求 `game.battle.attack` 时，会首先载入 `<APP_ROOT>/actions/game/BattleAction.lua` 文件。然后创建其中定义的 `BattleAction` 对象的一个实例，最后调用 `attackAction()` 函数。

<br />


## 延迟任务接口

延迟任务接口和 HTTP/WebSocket 接口的定义规则只有以下区别：

-   `action module` 对应的类名是添加 `Handler`。例如 `map.addres` 对应 `MapHandler` 类。
-   接口类放在 `<APP_ROOT>/jobs` 目录中。

\-EOF\-
