# 配置文件

GBC 中的配置文件有两类：

-   GBC 全局配置文件
-   应用配置文件

全局配置文件用于调整 GBC 的运行设定，而应用配置文件则用于调整应用的运行设定。


## 全局配置

GBC 最核心的全局配置文件就是 `conf/config.lua`，分为三个部分：

```lua

local config = {
    DEBUG = cc.DEBUG_VERBOSE,

    -- all apps
    apps = {
        -- 应用名 = "应用绝对路径",
        welcome = "_GBC_CORE_ROOT_/apps/welcome",
        ...
    },

    -- default app config
    app = {
        ...
    },

    -- server config
    server = {
        nginx = {
            numOfWorkers = 4, -- nginx 的进程数量
            port = 8088 -- 对外服务端口
        },

        -- internal memory database
        redis = {
            ...
        },

        -- background job server
        beanstalkd = {
            ...
        },
    }
}

return config
```

-   `apps` 定义了 GBC 启动时要载入的应用

    每一个应用一个唯一的名字，对应该应用所在目录的绝对路径。名字不能包含特殊字符，最好只使用字母和数字的组合。

-   `app` 定义了所有应用共享的默认配置

    修改这里的设置会影响所有应用，所以强烈建议不要改动这些设置。我们可以在每个应用的 `app_config.lua` 里覆盖这些默认设置。

-   `server` 定义了 GBC 中各项服务的配置

    绝大多数情况下，只需要修改 `nginx` 有关的两项：

    -   `nginx.numOfWorkers` 是 nginx 的进程数量，应该设置为和服务器拥有的 CPU 个数相同
    -   `nginx.port` 是对外服务的端口号，根据需要自行设置

**特别说明：**

GBC 安装时自动安装的 Redis 是仅用于 GBC 内部运行所需的内存数据库，不应该在应用中使用这个内部 Redis。关于 Redis 的问题，参考[在应用中使用 Redis](use-redis.md)。


## 应用配置文件

每个应用有两个配置文件：

-   `应用目录/conf/app_entry.conf` 定义该应用的访问入口
-   `应用目录/conf/app_config.lua` 定义该应用的自定义设置

### `app_entry.conf`

这个文件的内容如下：

```bash
location = /ENTRY {
    content_by_lua 'nginxBootstrap:runapp("_APP_ROOT_")';
}
```

文件中的 `ENTRY` 表示用什么地址访问应用定义的接口。

例如 GBC 服务器的地址是 `localhost:8088`，而 `ENTRY` 是 `hello`，则应用的访问地址就是 `localhost:8088/hello`。

### `app_config.lua`

这个文件中的内容会直接覆盖掉全局配置文件 `conf/config.lua` 中的 `app` 部分。所以如果需要修改应用运行设定，应该放在 `app_config.lua` 中。

可以指定的运行时设定：

选项 | 说明
-----|-----
`messageFormat` | 指定客户端和服务端交换数据的消息格式，默认为 `json`。
`defaultAcceptedRequestType` | 指定服务端接口默认支持哪种请求方式，默认为 `http`，可以设定为 `websocket` 或者 `{"http", "websocket"}` 以支持多种请求方式。但为了安全起见，应该设置为 `http`，然后在需要支持其他请求类型的接口里再明确指定。
`sessionExpiredTime` | 指定 Session 的失效时间。Session 建立后，如果没有调用 `Session:setKeepAlive()` 方法，则超过指定时间这个 Session 就会失效。
`httpEnabled` | 指定该应用是否接受 HTTP 请求。
`httpMessageFormat` | 指定客户端以 HTTP 请求和服务端交互时使用的消息格式，默认为 `json`。
`websocketEnabled` | 指定该应用是否接受 WebSocket 请求。
`websocketMessageFormat` | 指定客户端以 WebSocket 请求和服务端交互时使用的消息格式，默认为 `json`。
`websocketsTimeout` | 指定客户端和服务端使用 WebSocket 交互时，服务端等待客户端消息的超时时间。这个时间设置为较短时，可以更快检查到客户端已经断开连接，但会些微增加服务器负担。默认为 60 秒。
`websocketsMaxPayloadLen` | 指定客户端和服务端使用 WebSocket 交互时，每个消息的最大长度（字节），默认为 `16KB`。
`jobMessageFormat` | 指定后台任务的消息格式，默认为 `json`。
`numOfJobWorkers` | 指定该应用启动多少个后台进程。推荐按照服务器的 CPU 个数来设置。
`jobWorkerRequests` | 指定每一个后台任务进程处理多少个任务就重启一次（避免因为代码问题造成的内存泄露）。

关于 `app_entry.conf` 和 `app_config.lua` 的示例，可以看看 GBC 自带的 `welcome` 和 `tests` 两个应用。
