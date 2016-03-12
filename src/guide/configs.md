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

关于 `app_entry.conf` 和 `app_config.lua` 的示例，可以看看 GBC 自带的 `welcome` 和 `tests` 两个应用。
