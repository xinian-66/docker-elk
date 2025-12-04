
---

# **Elastic Stack（ELK）在 Docker 上的部署 — **

[![Elastic Stack 版本](https://img.shields.io/badge/Elastic%20Stack-9.2.1-00bfb3?style=flat\&logo=elastic-stack)]
运行最新版本的 [Elastic Stack][elk-stack]（Elasticsearch、Logstash、Kibana）使用 Docker 与 Docker Compose。

你可以使用 Elasticsearch 的搜索/聚合功能以及 Kibana 的可视化能力分析任何数据集。

本项目基于 Elastic 官方 Docker 镜像：

* Elasticsearch
* Logstash
* Kibana

还有其他变体：

* `tls`：启用 Elasticsearch、Kibana（可选）、Fleet 的 TLS 加密

---

# **重要提示**

默认启用 **Elastic Platinum 订阅功能（30 天试用）**。试用结束后将自动切换到免费 Basic License，不需要手动操作，也不会丢数据。
如需关闭付费功能，请参考下文 *“如何禁用付费功能”*。

---

# **项目理念**

docker-elk 的目标并不是提供生产级部署方案，而是提供最简化、便于理解的模板，适合学习、测试与探索。

默认配置极简、没有多余自动化，也不依赖外部组件。

---

# **目录**

1. 系统需求
2. 使用方法
3. 配置
4. 扩展
5. JVM 调优
6. 高级主题

---

# **系统需求**

### 主机环境

* Docker Engine ≥ 18.06
* Docker Compose ≥ 2.0
* 至少 1.5 GB 内存

默认开放的端口：

| 服务                 | 端口    |
| ------------------ | ----- |
| Logstash Beats 输入  | 5044  |
| Logstash TCP 输入    | 50000 |
| Logstash 监控 API    | 9600  |
| Elasticsearch HTTP | 9200  |
| Elasticsearch TCP  | 9300  |
| Kibana             | 5601  |

⚠️ **Linux 环境请确保当前用户已加入 docker 组，否则无法与 Docker daemon 通信。**

---

# **使用方法**

## **1. 克隆仓库**

```bash
git clone https://github.com/deviantony/docker-elk.git
```

## **2. 初始化 Elasticsearch 用户与权限**

```bash
docker compose up setup
```

## **3.（可选）生成 Kibana 加密密钥**

```bash
docker compose up kibana-genkeys
```

将输出内容复制到 `kibana/config/kibana.yml`

## **4. 启动 ELK 堆栈**

```bash
docker compose up
```

等待约 1 分钟，在浏览器访问：

```
http://localhost:5601
```

默认登录信息：

* 用户名：`elastic`
* 密码：`changeme`

---

# **初始化步骤**

## 1. 设置用户认证密码

默认密码 `changeme` 非常不安全，请修改：

### 重置 elastic 用户密码

```bash
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic
```

重置 logstash_internal 用户：

```bash
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal
```

重置 kibana_system 用户：

```bash
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
```

## 2. 将密码写入配置文件

修改 `.env` 文件中的：

* `ELASTIC_PASSWORD`
* `LOGSTASH_INTERNAL_PASSWORD`
* `KIBANA_SYSTEM_PASSWORD`

并同步修改：

* `kibana/config/kibana.yml`
* `logstash/pipeline/logstash.conf`

## 3. 重启 Logstash 与 Kibana

```bash
docker compose up -d logstash kibana
```

---

# **导入数据**

访问 Kibana 后，可以：

### 通过 logstash 的 50000 端口导入日志

```bash
cat /path/to/log.log | nc -q0 localhost 50000     # BSD
cat /path/to/log.log | nc -c localhost 50000      # GNU
cat /path/to/log.log | nc --send-only localhost 50000  # nmap
```

也可以在 Kibana 内加载示例数据。

---

# **清理（删除所有数据）**

```bash
docker compose --profile=setup down -v
```

---

# **版本选择**

默认 main 分支为 9.x 系列。

如需切换版本，修改 `.env` 内的版本号：

```
ELK_VERSION=8.15.0
```

然后重新构建：

```
docker compose build
```

可选的旧版本：

* release-8.x
* release-7.x（停止维护）
* release-6.x（停止维护）
* release-5.x（停止维护）

---

# **配置 Elasticsearch、Kibana、Logstash**

配置文件位置：

| 服务            | 配置文件                                     |
| ------------- | ---------------------------------------- |
| Elasticsearch | `elasticsearch/config/elasticsearch.yml` |
| Kibana        | `kibana/config/kibana.yml`               |
| Logstash      | `logstash/config/logstash.yml`           |

可通过 compose 文件注入环境变量，如：

```yaml
elasticsearch:
  environment:
    cluster.name: my-cluster
```

---

# **禁用付费功能**

如需关闭试用版：

### 方法 1（在 Kibana 内修改 license）

在 Kibana → Management → License Management 切换到 basic。

### 方法 2（使用 API）

```bash
POST /_license/start_basic
```

---

# **扩展插件**

### 添加插件步骤：

1. 在对应 Dockerfile 里添加：

```Dockerfile
RUN logstash-plugin install logstash-filter-json
```

2. 修改配置文件
3. 执行构建：

```bash
docker compose build
```

---

# **JVM 调优**

服务 JVM 内存配置环境变量：

| 服务            | 变量           |
| ------------- | ------------ |
| Elasticsearch | ES_JAVA_OPTS |
| Logstash      | LS_JAVA_OPTS |

示例：

```yaml
logstash:
  environment:
    LS_JAVA_OPTS: -Xms1g -Xmx1g
```

---

# **远程 JMX 调试**

示例：

```yaml
logstash:
  environment:
    LS_JAVA_OPTS: -Dcom.sun.management.jmxremote ...
```

---

# **深入学习**

参考 Wiki：

* 外部应用
* 常用集成

---

