# 安装 GameBox Cloud

推荐从源代码安装 GBC（GameBox Cloud 的简称），这样可以适应不同操作系统和运行环境。

目前 GBC 支持的生产运行环境有：

-   CentOS 6+
-   Ubuntu 14+

支持的开发环境有：

-   Mac OS X
-   各种 Linux 发行版


## 下载 GBC

由于 GBC 由 OpenResty、Redis 等软件组成，因此需要下载安装包，并从源代码安装。

推荐从 [https://github.com/dualface/gbc-core](https://github.com/dualface/gbc-core) 下载最新版本 GBC：

-   打开 "releases" 页面，从中下载最新版本的 `.zip` 或者 `.tar.gz` 压缩包
-   或者使用 `git` 命令 `clone` GBC 仓库：

```bash
git clone https://github.com/dualface/gbc-core.git
```


## 搭建开发环境

有两种方式搭建开发环境：

-   使用 `make.sh` 脚本
-   使用 [Vagrant](https://www.vagrantup.com/) 安装


### 使用 `make.sh` 安装脚本

下载的 `gbc-core` 源代码解压缩到需要的目录后，进入目录执行 `make.sh` 脚本即可完成安装。

注意：`gbc-core` 源代码应该放在没有空格和中文字符的目录中

安装完成后即可使用 `start_server --debug` 启动 GBC 进行测试。


### 使用 Vagrant 安装

如果使用 Windows 环境，则必须借助 Vagrant。在使用之前，需要安装 VirtualBox 最新版本和 Vagrant 最新版本。

-   VirtualBox: [https://www.virtualbox.org/](https://www.virtualbox.org/)
-   Vagrant: [https://www.vagrantup.com/](https://www.vagrantup.com/)

安装完成后，进入 `gbc-core` 源代码目录，执行：

```bash
cd gbc-core
vagrant up
```

执行 `vagrant up` 后，会下载虚拟机映像，然后启动虚拟机进行安装。

安装完成后，打开浏览器访问 `http://localhost:8088/` 即可访问 GBC 欢迎页面。


## 生产环境安装

在生产环境，首先要考虑安全问题，所以建议按照以下步骤进行：

1.  为 GBC 建立独立的用户，例如 `gbc`：

    ```bash
    useradd -s /bin/false -m gbc
    ```

2.  将 GBC 安装到 `gbc` 用户所在目录，并修改文件所有者：

    ```bash
    cd gbc-core
    sudo ./make.sh --prefix=/home/gbc/gbc-core
    sudo chown -R gbc:gbc /home/gbc/gbc-core
    ```

经过上述三步，我们就准备好了 GBC 的运行环境。

在部分操作系统，例如 CentOS 上，由于安全限定，还需要为 `gbc` 用户授权网络端口等权限，具体请参考相关文档。


## 启动与停止 GBC

进入安装目录，执行：

```bash
./start_server
```

即可启动 GBC（ROOT_DIR 输出根据安装目录不同有所区别）：

```markdown
ROOT_DIR=/opt/gbc-core

Start GameBox Cloud Core

[CMD] supervisord -c /opt/gbc-core/tmp/supervisord.conf

Start supervisord DONE

beanstalkd                       STARTING
nginx                            STARTING
redis                            STARTING
worker-tests:00                  STARTING
worker-welcome:00                STARTING
worker-welcome:01                STARTING
```

要检查 GBC 是否正常启动，执行：

```bash
./check_server
```

如果输出：

```markdown
beanstalkd                       RUNNING   pid 12352, uptime 0:01:11
nginx                            RUNNING   pid 12347, uptime 0:01:11
redis                            RUNNING   pid 12350, uptime 0:01:11
worker-tests:00                  RUNNING   pid 12351, uptime 0:01:11
worker-welcome:00                RUNNING   pid 12348, uptime 0:01:11
worker-welcome:01                RUNNING   pid 12349, uptime 0:01:11
```

说明 GBC 工作正常。


### 停止 GBC

执行：

```bash
./stop_server
```


### 以调试模式启动 GBC

正常模式启动的 GBC，在开发者修改脚本后，必须重启才能生效。

为了提高开发效率，GBC 提供了调试模式，只需要执行：

```bash
./start_server --debug
```

在调试模式下，修改脚本后无需重新启动 GBC，下一次请求或操作就会让新脚本立即生效。


### 在 Vagrant 中控制 GBC 启动和停止

每次执行 `vagrant up` 都会自动以调试模式启动 GBC，我们也可以登录 Vagrant 创建的虚拟机来控制 GBC：

```bash
vagrant ssh

cd /opt/gbc-core
./check_server
./stop_server
```


## 用调试模式做开发

使用 `make.sh` 或者 Vagrant 方式安装运行 GBC 时，只要修改了源代码目录中的相应文件，就可以理解生效。
