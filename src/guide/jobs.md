# 延时任务和定时任务

在游戏中，会存在大量需要延时或定时执行的操作。例如出发的部队需要2个小时才能到达目的地。

对于这类操作，GBC 提供了 `Jobs` 接口来处理。下面先看看基本用法：

~~~lua
-- 军营模块
local gbc = cc.import("#gbc")
local BarrackAction = cc.class("BarrackAction", gbc.ActionBase)

-- 出兵的接口
function BarrackAction:sentAction(arg)
    -- 这些数据将包含在任务中，并在任务时间到达时传递给指定的接口
    local data = {
        troops   = "knight",
        quantity = 30,
        level    = 2,
    }

    -- 取得 jobs 接口，并添加任务
    local jobs = self:getInstance():getJobs()
    jobs:add({
        action = 'battle.arrival', -- 任务传递给 battle.arrival
        data   = data, -- 要传递给任务接口的数据
        delay  = 10, -- 任务延迟 10 秒执行
    })
end

return BarrackAction
~~~

当 `barrack.sent` 接口被调用后，一个延时任务就添加到了系统中。

系统会在指定时间到达后，调用任务指定的 `battle.arrival` 接口。

~~~lua
local gbc = cc.import("#gbc")
local BattleAction = cc.class("BattleAction", gbc.ActionBase)

BattleAction.ACCEPTED_REQUEST_TYPE = "worker" -- 指定该接口只用于 Job 后台任务

function BattleAction:arrivalAction(job)
    -- 整个任务会作为参数传入接口

    print(job.delay) -- 任务设定的等待时间
    print(job.pri)   -- 任务设定的优先级
    print(job.ttr)   -- 任务的执行时间限制

    -- barrack.sent 中提供的数据，会作为 job.data 参数
    local troops   = job.data.troops
    local quantity = job.data.quantity
    local level    = job.data.level

    ...
end

return BattleAction
~~~

**注意**：`BattleAction.lua` 文件必须放在应用的 `jobs` 目录中。

1.  重要的事情说三遍：Job 的接口文件必须放在应用的 `jobs` 目录中。
2.  重要的事情说三遍：Job 的接口文件必须放在应用的 `jobs` 目录中。
3.  重要的事情说三遍：Job 的接口文件必须放在应用的 `jobs` 目录中。


## 任务执行结果

由于任务接口执行后，并不能直接返回结果给添加任务的接口。所以任务接口应该将执行结果写入数据库，或者通过消息接口通知客户端。

如果任务执行出错，建议采用以下两种处理方式：

1.  返回 `false` 表示任务执行失败，任务将再次启动。通常在出现一些数据写入冲突时可以采用这种方式，让系统自动重新执行任务。

2.  添加日志，并删除任务，然后返回 `false`。由于任务已被删除，所以不会重新启动。但 GBC 会记录返回 `false` 的任务情况到日志中，以供查询。

在任务执行期间如果调用了 `error()`、`throw()` 等中断执行的函数，都会导致任务重新执行。


## Jobs:提供的接口

### `Jobs:add()` - 添加一个延时执行的任务

说明：

-   `Jobs:add(args)`: `arg` 是一个 table，包含下列字段:
    -   `action`: 任务到期时调用哪一个接口
    -   `data`: 要传递给任务接口的数据，必须是 `table`
    -   `delay`: 任务的等待时间
    -   `pri`: （可选）任务的优先级，数字越小优先级越高，默认为 2048，表示普通优先级。低于 1024 的优先级表示紧急任务。同等延迟时间的任务，优先级高的会先执行。
    -   `ttr`: （可选）任务接口可以用多少时间来处理任务。默认为 10 秒。如果在指定时间内任务没有处理完成，该任务会重新回到队列中。因此对于可能耗时较长的任务，应该指定较大的 `ttr` 值。

`add()` 如果成功，将返回一个整数，作为 `JobId`。后续可以用 `JobId` 移除任务或暂停任务。

`add()` 如果失败，将返回 `nil` 和错误信息，可以用以下的代码判断：

~~~lua
local jobid, err = jobs:add(....)
if not jobid then
    -- 进行错误处理
    print(err)
end
~~~


### `Jobs:at()` - 添加一个定时任务

说明：

-   `Jobs:at(arg)`: `arg` 是一个 `table`，与 `add()` 接口相比仅仅是 `delay` 字段改为 `time` 字段：
    -   `time`: 自 1970 年以来的秒数，指定任务执行的时间。
    -   其他参数和返回值同 `Jobs:add()` 接口。

由于存在时区问题，因此可以用以下代码获得指定时间的秒数（UTC）：

~~~lua
local time = os.gettime({2015, 12, 24, 22, 30})
~~~

PS: `os.gettime()` 函数是 GBC 提供的自定义函数，并非标准库函数。


### `Jobs:delete()` - 删除任务

说明：

-   `Jobs:delete(jobid)`
    -   `jobid`: 要删除任务的 `JobId`，由 `Jobs:add()` 和 `Jobs:at()` 接口返回。

`delete()` 如果成功，返回 `true`，否则返回 `nil` 和错误信息。


### `Jobs:get()` - 查询一个任务

说明：

-   `Jobs:get(jobid)`
    -   `jobid`: 如果指定的任务还未删除，则返回包含所有任务信息的 `table`：

~~~lua
local job, err = jobs:get(jobid)
if not job then
    -- 进行错误处理
    print(err)
else
    -- 打印 job 内容
    print(job.id)
    print(job.action)
    print(job.delay)
    print(job.pri)
    print(job.ttr)
    cc.dump(job.data)
end
~~~

如果指定的任务不存在，则返回 `nil` 和错误信息。


### `Jobs:getready()` - 查询一个到达指定时间的任务，如果没有任务则一直等待直到超时

说明：

-   `Jobs:getready()`

`getready()` 如果成功，返回一个 `table`，结果同 `get()`。如果失败，则返回 `nil` 和错误信息。

