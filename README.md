# Docker 上的弹性堆栈 (ELK)

[![Elastic Stack version](https://img.shields.io/badge/Elastic%20Stack-9.2.1-00bfb3?style=flat&logo=elastic-stack)](https://www.elastic.co/blog/category/releases)
[![Build Status](https://github.com/deviantony/docker-elk/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/deviantony/docker-elk/actions/workflows/ci.yml?query=branch%3Amain)

使用 Docker 和 Docker Compose 运行最新版本的 [Elastic stack][elk-stack]。

它使您能够使用 Elasticsearch 的搜索/聚合功能来分析任何数据集，
Kibana 的可视化能力。

基于 Elastic 的[官方 Docker 镜像][elastic-docker]：

* [Elasticsearch](https://github.com/elastic/elasticsearch/tree/main/distribution/docker)
* [Logstash](https://github.com/elastic/logstash/tree/main/docker)
* [Kibana](https://github.com/elastic/kibana/tree/main/src/dev/build/tasks/os_packages/docker_generator)

其他可用的堆栈变体：

* [<<<CODE_117>>>](https://github.com/deviantony/docker-elk/tree/tls)：在 Elasticsearch、Kibana 中启用 TLS 加密（选择加入），
和舰队

> [！重要的]
> 默认情况下启用[白金][订阅]功能，[试用][许可证管理]持续时间为**30天**。后
> 在此评估期内，您将保留无缝访问 Open Basic 许可证中包含的所有免费功能，
> 无需人工干预，且不会丢失任何数据。请参阅【如何禁用付费
> features](#how-to-disable-paid-features) 部分选择退出此行为。

---



<图片>
<source media="(prefers-color-scheme: dark)" srcset="https://github.com/user-attachments/assets/6f67cbc0-ddee-44bf-8f4d-7fd2d70f5217">
<img alt="动画演示" src="https://github.com/user-attachments/assets/501a340a-e6df-4934-90a2-6152b462c14a">
</图片>

---

## 哲学

docker-elk 的主要目标是使 Elastic 堆栈尽可能容易进入。它不是**的蓝图
生产就绪部署**，而是一个促进调整和探索的_模板_。

作者相信良好的文档胜过复杂的自动化。项目的默认配置是故意的
最低限度且不持己见。初始设置不依赖于任何外部依赖项，并且使用尽可能少的脚本
启动并运行所必需的。

---

## 内容

1. [Requirements](#requirements)
   * [Host setup](#host-setup)
   * [Docker Desktop](#docker-desktop)
     * [Windows](#windows)
     * [macOS](#macos)
1. [Usage](#usage)
   * [Bringing up the stack](#bringing-up-the-stack)
   * [Initial setup](#initial-setup)
     * [Setting up user authentication](#setting-up-user-authentication)
     * [Injecting data](#injecting-data)
   * [Cleanup](#cleanup)
   * [Version selection](#version-selection)
1. [Configuration](#configuration)
   * [How to configure Elasticsearch](#how-to-configure-elasticsearch)
   * [How to configure Kibana](#how-to-configure-kibana)
   * [How to configure Logstash](#how-to-configure-logstash)
   * [How to disable paid features](#how-to-disable-paid-features)
   * [How to scale out the Elasticsearch cluster](#how-to-scale-out-the-elasticsearch-cluster)
   * [How to re-execute the setup](#how-to-re-execute-the-setup)
   * [How to reset a password programmatically](#how-to-reset-a-password-programmatically)
1. [Extensibility](#extensibility)
   * [How to add plugins](#how-to-add-plugins)
   * [How to enable the provided extensions](#how-to-enable-the-provided-extensions)
1. [JVM tuning](#jvm-tuning)
   * [How to specify the amount of memory used by a service](#how-to-specify-the-amount-of-memory-used-by-a-service)
   * [How to enable a remote JMX connection to a service](#how-to-enable-a-remote-jmx-connection-to-a-service)
1. [Going further](#going-further)
   * [Plugins and integrations](#plugins-and-integrations)

## 要求

### 主机设置

* [Docker 引擎][docker-install] 版本 **18.06.0** 或更高版本
* [Docker Compose][compose-install] 版本 **2.0.0** 或更高版本
* 1.5 GB 内存

> [！笔记]
> 特别是在 Linux 上，请确保您的用户具有与 Docker 交互的[所需权限][linux-postinstall]
> 守护进程。

默认情况下，堆栈公开以下端口：

* 5044：Logstash Beats 输入
* 50000：Logstash TCP 输入
* 9600：Logstash监控API
* 9200：Elasticsearch HTTP
* 9300：Elasticsearch TCP 传输
* 5601：基巴纳

> [！警告]
> Elasticsearch 的 [bootstrap 检查][bootstrap-checks] 被故意禁用，以方便 Elastic 的设置
> 开发环境中的堆栈。对于生产设置，我们建议用户根据以下内容设置其主机
> Elasticsearch 文档中的说明：[重要系统配置][es-sys-config]。

### Docker 桌面

#### 视窗

如果您使用的是 _Docker Desktop for Windows_ 的旧版 Hyper-V 模式，请确保 [文件
共享][桌面文件共享]已启用`C:`驾驶。

#### macOS

_Docker Desktop for Mac_ 的默认配置允许从以下位置挂载文件`/Users/`, `/Volume/`, `/private/`,
`/tmp`和`/var/folders`只。确保存储库已克隆到这些位置之一或遵循
[文档][桌面文件共享]中的说明添加更多位置。

## 用法

> [！警告]
> 您必须使用以下命令重建堆栈映像`docker compose build`每当你切换分支或更新
> [version](#version-selection)已经存在的堆栈。

### 调出堆栈

使用以下命令将此存储库克隆到将运行堆栈的 Docker 主机上：

```sh
git clone https://github.com/deviantony/docker-elk.git
```

然后，通过执行以下命令初始化 docker-elk 所需的 Elasticsearch 用户和组：

```sh
docker compose up setup
```

或者（但强烈推荐），使用以下命令生成 Kibana 的加密密钥并复制其输出
到 Kibana 配置文件（`kibana/config/kibana.yml`):

```sh
docker compose up kibana-genkeys
```

如果一切顺利并且设置完成且没有错误，请启动其他堆栈组件：

```sh
docker compose up
```

> [！笔记]
> 您还可以通过附加以下内容在后台运行所有服务（分离模式）`-d`标记上述命令。

给 Kibana 大约一分钟的时间来初始化，然后通过在 Web 中打开 <http://localhost:5601> 来访问 Kibana Web UI
浏览器并使用以下（默认）凭据登录：

* 用户：*弹性*
* 密码：*更改我*

> [！笔记]
> 初次启动时，`elastic`, `logstash_internal`和`kibana_system`Elasticsearch用户已初始化
> 与中定义的密码的值[<<<CODE_130>>>](.env)文件（_“changeme”_默认情况下）。第一个是
> [builtin superuser][builtin-users]，另外两个分别用于Kibana和Logstash进行通信
> 弹性搜索。此任务仅在堆栈的_初始_启动期间执行。更改用户密码
> 它们初始化后，请参阅下一节中的说明。

### 初始设置

#### 设置用户身份验证

> [！笔记]
> 请参阅[Elasticsearch 中的安全设置][es-security] 禁用身份验证。

> [！警告]
> 从 Elastic v8.0.0 开始，不再可能使用引导特权运行 Kibana`elastic`用户。

默认情况下为所有上述用户设置的_“changeme”_密码是**不安全**。为了提高安全性，我们将
将所有上述 Elasticsearch 用户的密码重置为随机密码。

1. 重置默认用户的密码

以下命令重置密码`elastic`, `logstash_internal`和`kibana_system`用户。请注意
其中。

    ```sh
    docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic
    ```

    ```sh
    docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal
    ```

    ```sh
    docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
    ```

如果有需要（例如，如果您想通过 Beats 和 [收集监控信息][ls-monitoring]）
其他组件），请随时重复此操作以完成其余的[内置
用户][内置用户]。

1. 替换配置文件中的用户名和密码

更换密码`elastic`用户里面的`.env`文件，其中包含上一步中生成的密码。
它的值不被任何核心组件使用，但是[extensions](#how-to-enable-the-provided-extensions)用它来
连接到 Elasticsearch。

> [!注意]
> 如果您不打算使用所提供的任何内容[extensions](#how-to-enable-the-provided-extensions)， 或者
> 更喜欢创建自己的角色和用户来验证这些服务，删除
    > `ELASTIC_PASSWORD`条目来自`.env`堆栈初始化后完全文件。

更换密码`logstash_internal`用户里面的`.env`文件中生成的密码
上一步。它的值在 Logstash 管道文件中引用（`logstash/pipeline/logstash.conf`).

更换密码`kibana_system`用户里面的`.env`文件包含之前生成的密码
步。它的值在 Kibana 配置文件中引用（`kibana/config/kibana.yml`).

请参阅[Configuration](#configuration)有关这些配置文件的更多信息，请参阅下面的部分。

1. 重新启动 Logstash 和 Kibana 以使用新密码重新连接到 Elasticsearch

    ```sh
    docker compose up -d logstash kibana
    ```

> [！笔记]
> 要了解有关 Elastic Stack 安全性的更多信息，请访问 [Secure the Elastic Stack][sec-cluster]。

#### 注入数据

通过在 Web 浏览器中打开 <http://localhost:5601> 启动 Kibana Web UI，并使用以下凭据登录
在：

* 用户：*弹性*
* 密码：*\<您生成的弹性密码>*

现在堆栈已完全配置，您可以继续注入一些日志条目。

附带的 Logstash 配置允许您通过 TCP 端口 50000 发送数据。例如，您可以使用其中之一
以下命令 - 取决于您安装的版本`nc`(Netcat) — 提取日志文件的内容
`/path/to/logfile.log`在 Elasticsearch 中，通过 Logstash：

```sh
# Execute `nc -h` to determine your `nc` version

cat /path/to/logfile.log | nc -q0 localhost 50000          # BSD
cat /path/to/logfile.log | nc -c localhost 50000           # GNU
cat /path/to/logfile.log | nc --send-only localhost 50000  # nmap
```

您还可以加载 Kibana 安装提供的示例数据。

### 清理

默认情况下，Elasticsearch 数据保留在卷内。

为了完全关闭堆栈并删除所有持久数据，请使用以下 Docker Compose 命令：

```sh
docker compose --profile=setup down -v
```

### 版本选择

该存储库与 Elastic stack 的最新版本保持一致。这`main`分支跟踪当前专业
版本（9.x）。

要使用核心 Elastic 组件的不同版本，只需更改内部的版本号[<<<CODE_148>>>](.env)
文件。如果您要升级现有堆栈，请记住使用以下命令重建所有容器映像`docker compose build`
命令。

> [！重要的]
> 在执行操作之前，请务必注意每个组件的[官方升级说明][升级]
> 堆栈升级。

单独的分支也支持旧的主要版本：

* [<<<CODE_150>>>](https://github.com/deviantony/docker-elk/tree/release-8.x)：8.x系列
* [<<<CODE_151>>>](https://github.com/deviantony/docker-elk/tree/release-7.x)：7.x 系列（停产）
* [<<<CODE_152>>>](https://github.com/deviantony/docker-elk/tree/release-6.x)：6.x 系列（停产）
* [<<<CODE_153>>>](https://github.com/deviantony/docker-elk/tree/release-5.x)：5.x 系列（停产）

## 配置

> [！重要的]
> 配置不会动态重新加载，任何配置后您都需要重新启动各个组件
> 改变。

### 如何配置 Elasticsearch

Elasticsearch 配置存储在 [`elasticsearch/config/elasticsearch.yml`][配置-es]。

您还可以通过在 Compose 文件中设置环境变量来指定要覆盖的选项：

```yml
elasticsearch:

  environment:
    network.host: _non_loopback_
    cluster.name: my-cluster
```

有关如何在 Docker 中配置 Elasticsearch 的更多详细信息，请参阅以下文档页面
容器：[使用 Docker 安装 Elasticsearch][es-docker]。

### 如何配置 Kibana

Kibana 默认配置存储在 [`kibana/config/kibana.yml`][配置-kbn]。

您还可以通过在 Compose 文件中设置环境变量来指定要覆盖的选项：

```yml
kibana:

  environment:
    SERVER_NAME: kibana.example.org
```

有关如何在 Docker 中配置 Kibana 的更多详细信息，请参阅以下文档页面
容器：[使用 Docker 安装 Kibana][kbn-docker]。

### 如何配置 Logstash

Logstash配置存储在[`logstash/config/logstash.yml`][配置-ls]。

您还可以通过在 Compose 文件中设置环境变量来指定要覆盖的选项：

```yml
logstash:

  environment:
    LOG_LEVEL: debug
```

有关如何在 Docker 中配置 Logstash 的更多详细信息，请参阅以下文档页面
容器：[为 Docker 配置 Logstash][ls-docker]。

### 如何禁用付费功能

您可以在到期日期之前取消正在进行的试用 - 从而恢复到基本许可证 - 可以通过 [许可证
Kibana的管理][license-mngmt]面板，或者使用Elasticsearch的`start_basic`[许可 API][许可证 API]。请
请注意，如果许可证未切换到，第二个选项是恢复对 Kibana 访问的唯一方法`basic`
或在试用期满之前升级。

通过切换 Elasticsearch 的值来更改许可证类型`xpack.license.self_generated.type`设置从
`trial`到`basic`（请参阅[许可证设置][许可证设置]）仅在**在初始设置之前完成后才有效。**
试用开始后，功能丢失`trial`到`basic`_必须_使用两者之一进行确认
第一段中描述的方法。

### 如何横向扩展Elasticsearch集群

按照 Wiki 中的说明进行操作：[Scaling out Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

### 如何重新执行设置

再次运行设置容器并重新初始化在其中定义密码的所有用户`.env`文件，
简单地“向上”`setup`再次编写服务：

```console
$ docker compose up setup
 ⠿ Container docker-elk-elasticsearch-1  Running
 ⠿ Container docker-elk-setup-1          Created
Attaching to docker-elk-setup-1
...
docker-elk-setup-1  | [+] User 'monitoring_internal'
docker-elk-setup-1  |    ⠿ User does not exist, creating
docker-elk-setup-1  | [+] User 'beats_system'
docker-elk-setup-1  |    ⠿ User exists, setting password
docker-elk-setup-1 exited with code 0
```

### 如何以编程方式重置密码

如果出于任何原因您无法使用 Kibana 更改用户的密码（包括 [内置
users][builtin-users]），您可以改用 Elasticsearch API 并获得相同的结果。

在下面的例子中，我们重置了密码`elastic`用户（注意 URL 中的“/user/elastic”）：

```sh
curl -XPOST -D- 'http://localhost:9200/_security/user/elastic/_password' \
    -H 'Content-Type: application/json' \
    -u elastic:<your current elastic password> \
    -d '{"password" : "<your new password>"}'
```

## 可扩展性

### 如何添加插件

要将插件添加到任何 ELK 组件，您必须：

1. 添加一个`RUN`对应的声明`Dockerfile`（例如。`RUN logstash-plugin install logstash-filter-json`)
1. 将关联的插件代码配置添加到服务配置中（例如Logstash输入/输出）
1. 使用以下命令重建图像`docker compose build`命令

### 如何启用提供的扩展

里面有一些扩展可用[<<<CODE_171>>>](extensions)目录。这些扩展提供了以下功能
不是标准 Elastic 堆栈的一部分，但可用于通过额外的集成来丰富它。

这些扩展的文档在每个单独的子目录中以每个扩展为基础提供。一些
其中需要手动更改默认的 ELK 配置。

## JVM调优

### 如何指定服务使用的内存量

Elasticsearch 和 Logstash 的启动脚本可以从环境值中附加额外的 JVM 选项
变量，允许用户调整每个组件可以使用的内存量：

|服务 |环境变量|
|---------------|----------------------|
|弹性搜索 | ES_JAVA_OPTS | ES_JAVA_OPTS |
|日志存储 | LS_JAVA_OPTS | LS_JAVA_OPTS |

为了适应内存稀缺的环境（Mac 版 Docker Desktop 默认只有 2 GB 可用空间），堆
默认情况下，大小分配有上限`docker-compose.yml`Elasticsearch 文件大小为 512 MB，Elasticsearch 文件大小为 256 MB
日志存储。如果要覆盖默认 JVM 配置，请编辑匹配的环境变量
`docker-compose.yml`文件。

例如，要增加 Logstash 的最大 JVM 堆大小：

```yml
logstash:

  environment:
    LS_JAVA_OPTS: -Xms1g -Xmx1g
```

当这些选项未设置时：

* Elasticsearch 从 [自动确定][es-heap] 的 JVM 堆大小开始。
* Logstash 启动时的 JVM 堆大小固定为 1 GB。

### 如何启用与服务的远程 JMX 连接

至于Java堆内存（见上文），您可以指定JVM选项来启用JMX并将JMX端口映射到Docker上
主持人。

更新`{ES,LS}_JAVA_OPTS`具有以下内容的环境变量（我已将 JMX 服务映射到端口上
18080，你可以改变它）。不要忘记更新`-Djava.rmi.server.hostname`选项与您的 IP 地址
Docker 主机（替换 **DOCKER_HOST_IP**）：

```yml
logstash:

  environment:
    LS_JAVA_OPTS: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=18080 -Dcom.sun.management.jmxremote.rmi.port=18080 -Djava.rmi.server.hostname=DOCKER_HOST_IP -Dcom.sun.management.jmxremote.local.only=false
```

## 更进一步

### 插件和集成

请参阅以下 Wiki 页面：

* [External applications](https://github.com/deviantony/docker-elk/wiki/External-applications)
* [Popular integrations](https://github.com/deviantony/docker-elk/wiki/Popular-integrations)

[elk-stack]：https://www.elastic.co/elastic-stack/
[弹性docker]：https://www.docker.elastic.co/
[订阅]：https://www.elastic.co/subscriptions
[es-security]：https://www.elastic.co/docs/reference/elasticsearch/configuration-reference/security-settings
[许可证设置]：https://www.elastic.co/docs/reference/elasticsearch/configuration-reference/license-settings
[许可证管理]：https://www.elastic.co/docs/deploy-manage/license/manage-your-license-in-self-management-cluster
[许可证-api]：https://www.elastic.co/docs/api/doc/elasticsearch/group/endpoint-license

[docker安装]：https://docs.docker.com/get-started/get-docker/
[撰写安装]：https://docs.docker.com/compose/install/
[linux-postinstall]：https://docs.docker.com/engine/install/linux-postinstall/
[桌面文件共享]：https://docs.docker.com/desktop/settings-and-maintenance/settings/#file-sharing

[引导检查]：https://www.elastic.co/docs/deploy-manage/deploy/self-management/bootstrap-checks
[es-sys-config]：https://www.elastic.co/docs/deploy-manage/deploy/self-management/important-system-configuration
[es-heap]：https://www.elastic.co/docs/deploy-manage/deploy/self-management/important-settings-configuration#heap-size-settings

[内置用户]：https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/built-in-users
[ls-monitoring]：https://www.elastic.co/docs/reference/logstash/monitoring-with-metricbeat
[sec-cluster]：https://www.elastic.co/docs/deploy-manage/security#cluster-or-deployment-security-features

[config-es]：./elasticsearch/config/elasticsearch.yml
[配置-kbn]：./kibana/config/kibana.yml
[配置-ls]：./logstash/config/logstash.yml

[es-docker]：https://www.elastic.co/docs/deploy-manage/deploy/self-management/install-elasticsearch-with-docker
[kbn-docker]：https://www.elastic.co/docs/deploy-manage/deploy/self-management/install-kibana-with-docker
[ls-docker]：https://www.elastic.co/docs/reference/logstash/docker-config

[升级]：https://www.elastic.co/docs/deploy-manage/upgrade/deployment-or-cluster/self-management

<!-- markdownlint 配置文件
{
“MD033”：{
“allowed_elements”：[“图片”，“来源”，“img”]
  }
}
-->
