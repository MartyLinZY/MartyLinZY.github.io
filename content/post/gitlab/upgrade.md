+++
title = 'GitLab 升级实录：踩坑记录与排查指南（PostgreSQL、Alertmanager、配置错误）'
date = 2025-07-28T16:15:46+08:00
draft = false

categories = [
    "Linux运维"
]

tags = [
    "GitLab",
    "PostgreSQL",
    "排查实录",
    "Linux运维",
]
+++

# GitLab升级

<https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/?current=14.4.1&auto=true&edition=ce>

从14.4升级上来
遇到的问题

## 📝 背景

在生产环境中，为了部署最新的安全补丁与功能，我们计划将 GitLab 从旧版本升级至较新的 omnibus 版本（GitLab 17+）。由于本地 GitLab使用内置 PostgreSQL、Prometheus、Alertmanager、Redis 等服务，升级过程中踩到了多个典型的坑。本篇文章记录下整个过程的排查路径与修复方式，供自己回顾，也帮助遇到相同问题的开发者。

---

## 🧱 系统环境与初始状态

- 系统：Debian 11
- GitLab：从旧版升级至 `17.8.7-ce.0`
- 数据库：自带 PostgreSQL（原为 v12，升级为 v13）
- 问题集中爆发时间：2025-07-23 至 07-24

---

## 🚧 问题一：PostgreSQL 版本不兼容导致数据库无法启动

### ❗错误日志

```text
FATAL:  database files are incompatible with server
DETAIL:  The data directory was initialized by PostgreSQL version 12, which is not compatible with this version 13.11.
```

### 🧠 分析原因

GitLab 升级时自动将 PostgreSQL 从 12 升级到 13，但由于旧数据目录未正确迁移或升级，导致服务无法启动。

### ✅ 解决方法

1. 确认数据目录为 `/var/opt/gitlab/postgresql/data`
2. 执行 GitLab 提供的升级工具：

```bash
gitlab-ctl pg-upgrade
```

3. 若失败，可参考官方步骤手动迁移：

```bash
gitlab-ctl stop
sudo -u gitlab-psql /opt/gitlab/embedded/bin/initdb -D /var/opt/gitlab/postgresql/data13
# 使用 pg_upgrade 工具迁移数据
```

---

## 🚧 问题二：Alertmanager 启动失败，端口冲突 9094

### ❗错误日志

```text
Failed to start TCP listener on "0.0.0.0" port 9094: bind: address already in use
unable to initialize gossip mesh
```

### 🧠 分析原因

Alertmanager 被多次启动，导致 9094 被占用，无法进行 gossip mesh 通信。

### ✅ 解决方法

1. 查找占用 9094 端口的进程：

```bash
lsof -i :9094
```

2. 杀掉残留进程：

```bash
kill -9 <PID>
```

3. 再次启动 Alertmanager：

```bash
gitlab-ctl restart alertmanager
```

---

## 🚧 问题三：`gitlab-ctl reconfigure` 报错：`Reading unsupported config value grafana`

### ❗错误日志

```text
FATAL: Mixlib::Config::UnknownConfigOptionError: Reading unsupported config value grafana.
```

### 🧠 分析原因

新版 GitLab 已不支持通过 `gitlab.rb` 管理 `grafana` 配置，但我们原来的配置仍然保留了如下字段：

```ruby
grafana['enable'] = true
```

### ✅ 解决方法

1. 编辑配置文件 `/etc/gitlab/gitlab.rb`
2. 注释掉或删除关于 `grafana` 的所有配置项：

```ruby
# grafana['enable'] = true
```

3. 执行重新配置：

```bash
gitlab-ctl reconfigure
```

---

## 🚧 问题四：Eureka 客户端连接失败，微服务无法注册

### ❗错误日志

```text
Connect to 47.98.191.109:8008 failed: Connection refused
DiscoveryClient was unable to refresh its cache!
```

### 🧠 分析原因

升级期间 Eureka Server 服务未启动或监听地址错误，客户端 `alert-center` 无法完成服务注册。

### ✅ 解决方法

- 检查 Eureka Server 是否已监听 8008 端口：

```bash
netstat -tlnp | grep 8008
```

- 检查 `application.yml` 中 `eureka.client.service-url.defaultZone` 配置是否正确

---

## 🔚 收尾与总结

升级 GitLab 是一项系统工程，尤其当涉及到 PostgreSQL 等状态型组件时，稍有不慎就会导致服务无法恢复。本次升级过程中暴露出以下关键经验：

### ✅ 升级建议总结

| 项目                    | 建议                                                         |
| ----------------------- | ------------------------------------------------------------ |
| PostgreSQL 升级         | 使用 GitLab 提供的 pg-upgrade 工具，或手动迁移时备份数据     |
| Alertmanager 等内部服务 | 升级前清理旧进程，避免端口冲突                               |
| 配置文件兼容性          | 每次升级前都应查看 [GitLab 官方变更说明](https://docs.gitlab.com/ee/update/#check-for-deprecated-settings) |
| 服务监控                | 保持 `gitlab-ctl status` 清洁可读，排查问题更方便            |
| 服务迁移方案            | 考虑将 PostgreSQL 与 Redis 外置，便于版本控制和灾备          |

---

## 🧠 后记

每次系统升级都是对生产系统的“耐压测试”。稳定升级的关键不在于避免所有问题，而在于能**高效排查 + 快速恢复**。希望本文能为大家在生产环境中部署 GitLab 升级提供实用参考。

---

📌 如有更复杂情况，欢迎在评论区补充交流。
