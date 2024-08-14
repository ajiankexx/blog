NOTE: 文章内容按照时间`pr`合入的时间线进行排列。

仓库地址：[gcs-back-end](https://github.com/CMIPT/gcs-back-end)

# Add docker creator and format action
`pr`的链接：[gcs-pull-1](https://github.com/CMIPT/gcs-back-end/pull/1)

## Google Java Style Format
本次`pr`添加了`docker`的创建和格式化`action`。该`action`能够在创建合入`master`和`develop`的
`pr`时，自动使用`Google Java Style Format`对`Java`代码进行格式化，并且自动创建一个新的`commit`提交
格式化后的代码。

创建这个`git action`的主要目的是为了保证整个团队编码风格的一致性。

Issues: 
- [ ] 这个`action`目前还有一些问题，比如新创建的提交依然会触发一次该`action`。

NOTE: `git action`的代码中出现了`secrets.PAT`，该字段需要在仓库的设置部分自行进行设置，值的内容是一
个`token`该`token`具有仓库的写权限，这样自动提交的时候才能保证能够成功提交。而`token`的创建方法是在
个人的`github`账号中。

在`github`中创建`token`：

![](https://raw.githubusercontent.com/Kaiser-Yang/image-hosting-site/main/20240421-20250421/20240801102637.png)

如何在仓库的`secrets`中添加`token`：

![](https://raw.githubusercontent.com/Kaiser-Yang/image-hosting-site/main/20240421-20250421/20240801102200.png)

## Docker Creator
本次`pr`中还添加了一个第三方依赖，用于创建`docker`镜像。这个镜像能够自动创建一个`docker`镜像并安装
一些基础的工具，例如`ssh`等。

第三方依赖的地址：[docker-creator](https://github.com/CMIPT/docker-script.git)

# Initialize the project
`pr`的链接：[gcs-pull-2](https://github.com/CMIPT/gcs-back-end/pull/2)

本次`pr`是从[spring-io](https://start.spring.io/)上创建了一个`Spring Boot`的项目，然后将其解压到了
仓库中。

TODO:
- [ ] 添加`spring`的基本使用介绍

# Build the initial database script
`pr`的链接：[gcs-pull-3](https://github.com/CMIPT/gcs-back-end/pull/3)

# Provide Python scripts to process Json files
`pr`的链接：[gcs-pull-6](https://github.com/CMIPT/gcs-back-end/pull/6)

本次的`pr`提供了处理`Json`文件的`Python`脚本，脚本接受一个参数，表示文件所在目录。默认值为
`../config.json`。脚本内置函数将`Json`文件的内容值存入对象`a`, 并可以通过点操作符`.`访问对象属性来
获取对应内容值。

```python
def loadJsonAsObject(file_path: str):
    with open(file_path, 'r', encoding='utf-8') as file:
        data = json.load(file)
    # transmit dictronary into object
    return json.loads(json.dumps(data), object_hook=lambda d: SimpleNamespace(**d))


parser = argparse.ArgumentParser(description="Process JSON file.")
parser.add_argument('file_path', nargs='?', default='../config.json', help="Path to the JSON file")
args = parser.parse_args()
```

下面介绍一下上述代码使用的`argparse`功能：
* `argparse.ArgumentParser`：该类用于创建一个参数解析器对象。`description`参数可以用于设置命令行工具的简短描述。
* `parser.add_argument`：定义程序可以接受的命令行参数。
    - `file_path`：这是定义的命令行参数名。它将接受用户输入的文件路径。
    - `nargs='?'`：表示该参数是可选的。如果用户没有提供该参数，则使用默认值。
    - `default='../config.json'`：如果用户没有提供`file_path`参数，则默认使用`../config.json`作为文件路径。
    - `help="Path to the JSON file"`：提供该参数的帮助信息，当用户使用`--help`选项时会显示这些信息。
* `args = parser.parse_args()`：解析命令行参数，并将结果存储在`args`对象中。

函数`loadJsonAsObject()`使用`json.loads()`时默认将内容解析成字典对象`dict`, 使用`SimpleNamespace`参数
后，允许像访问对象属性一样访问字典键。

# Add spring-doc for restful api
`pr`的链接：[gcs-pull-7](https://github.com/CMIPT/gcs-back-end/pull/7)

本次的`pr`主要是添加了`spring-doc`的依赖，用于生成`restful api`的文档。

集成文档的时候开始在尝试使用`spring-fox`和`spring-swagger`。结果发现和`spring3.x`的版本不兼容，然后
搜索资料发现了`spring-doc`这个依赖，然后就使用了这个依赖。

TODO:
- [ ] 添加`spring-doc`的使用介绍

# Finish part of the deploy script
`pr`的链接：[gcs-pull-9](https://github.com/CMIPT/gcs-back-end/pull/9)

创建这个脚本的目的是希望能够仅仅通过编写一个`json`的配置文件，就能够完成整个项目的部署。这个`pr`完成
了部分的功能，还有一些功能没有完成。

主脚本是一个`bash`脚本，在`bash`脚本中会自动安装`python`等依赖，然后调用`python`脚本来完成部署。部署
的主要逻辑是通过读取`json`文件，然后根据配置文件完成一系列处理，最后将`jar`通过`Sys V init`注册成
一个服务。

`Sys V init`创建服务的方式非常简单，只需要在`/etc/init.d/`目录下放置一个可执行文件，然后即可通过
`service xxx start`来启动服务。在本次的`pr`中，直接创建了一个软连接连接到了打包出来的`jar`文件，但是
在后续的使用过程中发现这样做存在问题。

在本次的提交中，我发现`junit`可以使用`spring-boot-test`进行替代，于是将之前的`junit`的依赖替换成了
`spring-boot-test`，并且删除了之前添加的`log4j`依赖（后续可以自己设置`spring-boot`的日志系统）。

下面介绍一下几个脚本有什么作用：
* `deploy_ubuntu.sh`：这个脚本是主脚本，用于在`ubuntu`系统上部署项目，这个脚本再安装一些依赖后便将控制权交给`script/deploy_helper.py`。
* `script/deploy_helper.py`：这个脚本读取`json`配置文件，然后根据配置文件完成一系列操作，最后将`jar`注册成一个服务。
* `script/get_jar_position.sh`：获取`jar`的位置，这个脚本会在`deploy_helper.py`中被调用，用于获取`mvn package`输出位置。

Issues: 
- [x] 这个脚本在某些环境使用的时候出现了问题，详见[gcs-issues-14](https://github.com/CMIPT/gcs-back-end/issues/14)
- [x] 错误的添加使用了`spring-boot-test`导致添加了`spring-boot-starter-webflux`依赖，这个依赖目前应该是不用的。

# Add MIT license and developers info
`pr`的链接：[gcs-pull-13](https://github.com/CMIPT/gcs-back-end/pull/13)

本次`pr`主要是添加了`MIT`的开源协议，以及开发者的信息。

# Substitute the Sys V init with systemd
`pr`的链接：[gcs-pull-15](https://github.com/CMIPT/gcs-back-end/pull/15)

本次`pr`主要是将`Sys V init`部署服务的方式替换成了`systemd`。这样做是因为发现现代的`Linux`系统更多地
使用`systemd`来管理服务，且`systemd`的启动速度等更加优秀，配置更加简单。这次的提交修复了
[gcs-issues-14](https://github.com/CMIPT/gcs-back-end/issues/14)。

这次的提交中还增加了`config_default.json`文件，`config_default.json`文件存储默认的配置，当一个配置项
没有被用户重写的时候会使用默认的配置。

同时新增了一个`clean_ubuntu.sh`脚本用于清空创建的服务等。

下面简单介绍一下`systemd`的使用方法。

要使用`systemd`来管理服务，首先需要创建一个`service`文件，然后将这个文件放置在`/etc/systemd/system/`
目录下。`service`文件的内容大致如下：

```shell
[Unit]
Description=Git server center back-end service
After=network.target

[Service]
PIDFile=/var/run/gcs.pid
User=gcs
WorkingDirectory=/opt/gcs
Restart=always
RestartSec=5
ExecStart=/usr/bin/java -jar /opt/gcs/gcs.jar

[Install]
WantedBy=multi-user.target
```

上面的大部分字段是不用解释的，这里给出一些注意事项：
* 路径全部使用绝对路径。
* `systemctl enable gcs`表示将服务设置为开机启动，但是如果当前的服务并没有启动，那么这个命令并不会启动服务。
* `systemctl start gcs`表示启动当前的服务，这与`gcs`是否被设置为开机启动无关。
* `systemctl disable gcs`和`systemctl stop gcs`的关系与上述类似。

NOTE: 简单解释一下`systemctl enable gcs`和`systemctl disable gcs`的原理。两者只做了一件事情：
根据`[Install]`部分的内容创建或者删除软连接，例如上面的例子中，`systemctl enable gcs`会在
`/etc/systemd/system/multi-user.target.wants/`目录下创建一个`gcs.service`的软连接，而
`systemctl disable gcs`会删除这个软连接。这个软连接的作用是在`multi-user.target`启动的时候启动`gcs`
而`Linux`系统启动的时候会启动`multi-user.target`，所以`gcs`也能在开机的时候自动启动。

# Add command_checker and setup_logger to script/deploy_helper.py
`pr`的链接：[gcs-pull-17](https://github.com/CMIPT/gcs-back-end/pull/17)

本次`pr`修改了`script/deploy_helper.py`，增加了日志功能。在执行`deploy_helper.py`时，如果命令执行失败，
会输出执行失败的命令的日志信息，包括时间，日志等级，行号，执行失败的命令。

```python
import logging
import inspect
```
首先，实现这个`pr`的功能，需要导入`logging`和`inspect`模块。`logging`模块是`Python`内置的日志模块，
`inspect`模块是`Python`内置模块，在这里用于获取命令的行号。

```python
def setup_logger(log_level=logging.INFO):
    """
    Configure the global logging system.

    :param log_level: Set the logging level, defaulting to INFO.
    """
    logging.basicConfig(level=log_level,
                        format='%(asctime)s -%(levelname)s- in %(pathname)s:%(caller_lineno)d: %(message)s', 
                        datefmt='%Y-%m-%d %H:%M:%S')
```
`setup_logger`函数用于配置全局的日志系统。`log_level`参数用于设置日志级别，默认为`INFO`。`logging.basicConfig`
定义输出日志的格式，包括时间，日志等级，行号，执行失败的命令。

```python
def command_checker(status_code: int, message: str, expected_code: int = 0):
    """
    Check if the command execution status code meets the expected value.

    :param status_code: The actual status code of the command execution.
    :param message: The log message to be recorded.
    :param expected_code: The expected status code, defaulting to 0.
    """
    if status_code != expected_code:
        caller_frame = inspect.currentframe().f_back
        logging.error(message, extra={'caller_lineno': caller_frame.f_lineno})
        exit(status_code)
```
`command_checker`用于比较命令执行返回的状态码与期望的状态码是否一致，如果不一致，则说明命令执行失败，
则打印出相应的日志，且返回状态码，如果一致则说明命令执行成功，不做任何操作。
- `status_code`: 命令执行的实际状态码
- `message`: 要打印的日志信息
- `expected_code`: 期望的状态码，默认为0

```python
message_tmp = '''\
The command below failed:
    {0}
Expected status code 0, got status code {1}
'''
```
`message_tmp`是一个模板字符串，用于格式化输出日志信息，在这里会将执行失败的命令和状态码输出到日志中。

# Remove unsed dependency and add doc for configuration
`pr`的链接：[gcs-pull-22](https://github.com/CMIPT/gcs-back-end/pull/22)

本次`pr`主要是将`spring-boot-starter-webflux`依赖删除，因为这个依赖是多余的。同时添加了一个`README-zh.md`
文件，用于存储配置文件的说明。

# Finish the script for deploying in docker
`pr`的链接：[gcs-pull-24](https://github.com/CMIPT/gcs-back-end/pull/24)

在本次的提交中，增加了自动在 `docker` 中部署的功能。在编写这部分功能的时候，发现 `docker` 在默认情况下
是不能够使用 `systemd` 的，只有当指明 `--privileged=true` 的时候才能够使用 `systemd`。这个参数的作用
是让 `docker` 在容器中运行的时候拥有直接操作宿主机的权限。如果通过样的方式创建 `docker` 失去了 `docker`
的部分安全性，因此我将在 `docker` 中的部署改用成了 `Sys Init V` 的方式，而在物理机上的部署继续保持
`systemd` 的方式。

`Sys Init V` 的脚本模板来自于 [_service.md](https://gist.github.com/naholyr/4275302) 。我对其中进行了
部分的修改，得到了如下的文件：

```bash
PIDDIR=$(dirname "$PIDFILE")
LOGDIR=$(dirname "$LOGFILE")
start() {
  if [ -f "$PIDDIR/$PIDNAME" ] && kill -0 "$(cat "$PIDDIR/$PIDNAME")"; then
    echo 'Service already running' >&2
    return 1
  fi
  echo 'Starting service…' >&2
  local CMD="$SCRIPT &> \"$LOGFILE\" & echo \$!"
  su -c "mkdir -p ""$PIDDIR" "$RUNAS"
  su -c "mkdir -p ""$LOGDIR" "$RUNAS"
  su -c "$CMD" "$RUNAS" > "$PIDFILE"
  echo 'Service started' >&2
}

stop() {
  if [ ! -f "$PIDFILE" ] || ! kill -0 "$(cat "$PIDFILE")"; then
    echo 'Service not running' >&2
    return 1
  fi
  echo 'Stopping service…' >&2
  kill -15 "$(cat "$PIDFILE")" && rm -f "$PIDFILE"
  echo 'Service stopped' >&2
}

uninstall() {
  echo -n "Are you really sure you want to uninstall this service? That cannot be undone. [yes|No] "
  local SURE
  read SURE
  if [ "$SURE" = "yes" ]; then
    stop
    rm -f "$PIDFILE"
    echo "Notice: log file is not be removed: '$LOGFILE'" >&2
    update-rc.d -f "$NAME" remove
    rm -fv "$0"
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  uninstall)
    uninstall
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|uninstall}"
esac
```

上面的脚本需要在最开始添加以下内容才能运行 (等号后面需要添加值)：

```bash
#!/bin/env bash
NAME=
SCRIPT=
RUNAS=
PIDFILE=
LOGFILE=
```

我通过 `Python` 脚本读取 `json` 配置文件，然后将配置文件的内容写入到 `Sys Init V` 的脚本中，最后将
后将这个脚本拷贝到指定目录：

```python
def create_sys_v_init_service(config):
    try:
        with open('script/service_tmp.sh', 'r') as f:
            service_content = f.read()
    except Exception as e:
        command_checker(1, f"Error: {e}")
        return

    header = f'''#!/bin/env bash
NAME={config.serviceName}
SCRIPT="{parse_iterable_into_str([config.serviceStartJavaCommand] +
config.serviceStartJavaArgs + [config.serviceStartJarFile])}"
RUNAS={config.serviceUser}
PIDFILE={config.servicePIDFile}
LOGFILE={config.serviceLogFile}
'''
    service_content = header + service_content
    log_debug(f"service_content:\n {service_content}")

    res = os.system(
        f"echo '{service_content}' | {sudo_cmd} tee {config.serviceSysVInitDirectory}/{config.serviceName}")
    command_checker(res, f"Failed to create {config.serviceSysVInitDirectory}/{config.serviceName}")
    res = os.system(f'{sudo_cmd} chmod +x {config.serviceSysVInitDirectory}/{config.serviceName}')
    command_checker(
        res, f"Failed to chmod +x {config.serviceSysVInitDirectory}/{config.serviceName}")

    if logging.getLogger().level == logging.DEBUG:
        try:
            with open(f'{config.serviceSysVInitDirectory}/{config.serviceName}', 'r') as f:
                log_debug(f"Service content:\n {f.read()}")
        except Exception as e:
            command_checker(1, f"Error: {e}")
            return
```

除了这些更改以外，将依赖的安装交给了 `Python` 脚本管理，`bash` 脚本仅仅负责安装 `python` 依赖。
