# Agent_Container

## 项目介绍
- 这是一个基于Docker的容器以及Docker基础的项目，用于个人记录和分享。
- 物理机系统：Ubuntu 26.04
- docker CLI 版本：Docker version 29.5.2, build 79eb04c
- 镜像：centos:8
**docker软件源参考链接：** [Docker 官方文档](https://docs.docker.com/)  或者  [阿里源](https://developer.aliyun.com/mirror)  或者  [清华源](https://mirrors.tuna.tsinghua.edu.cn/)
#### 个人配置案例(仅供参考)
``` bash
路径：/etc/apt/sources.list.d/docker.sources

Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu
Suites: noble
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
```
--- 

## 物理机配置
1. 配置docker软件源 ---- 参照项目介绍部分
2. 网络配置(可选) ---- 魔法上网
- docker安装软件源
   - 如果没有使用国内docker软件源，需要配置魔法上网
   - 首先，确保自己已经购买魔法上网服务，即梯子
   - 然后，使用[Clash](https://www.clashverge.dev) 或者 [Clash GitHub](https://github.com/clash-verge-rev/clash-verge-rev)等工具配置魔法上网
   - 最后，确保自己的物理机已经连接到魔法上网服务
- docker镜像源加速配置
   - 方式一：镜像加速配置文件
      - 在docker配置文件中添加镜像加速配置(/etc/docker/daemon.json)
      ``` json
      {
        "registry-mirrors": [
           "国内镜像地址1",
           "国内镜像地址2",
           "国内镜像地址3",
           "........"
        ]
      }
      ```
      - 重启docker服务
      - 验证镜像加速配置是否生效
        - 执行docker info命令，查看镜像加速配置是否生效
   - 方式二：配置物理机魔法网络端口映射
      - 配置路径：/etc/systemd/system/docker.service.d
      - 配置文件：proxy.conf
      ``` bash
      [Service]
      Proxy=1
      Environment=="HTTP_PROXY=http://127.0.0.1:clash_port"
      Environment=="HTTPS_PROXY=http://127.0.0.1:clash_port"
      Environment=="NO_PROXY=localhost,127.0.0.1,registry.fedoraproject.org"  
      ```
   - **备注：** 如果没有该目录和配置文件，需要手动创建

---

## 容器配置
1. 配置容器网络模式(常用为bridge和host)
 - host
 - bridge(默认)
 - none
 - overlay
 - macvlan
2. 更换容器软件源
- [阿里源](https://developer.aliyun.com/mirror/)
- [清华源](https://mirrors.tuna.tsinghua.edu.cn/)
``` bash
路径：/etc/yum.repos.d/CentOS-Base.repo

# 备份原始文件
mkdir -p /etc/yum.repos.d/bak
mv /etc/yum.repos.d/* /etc/yum.repos.d/bak/

# 配置新源示例
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
yum clean all
yum makecache
```

---

## Docker 命令速查手册

### 一、Docker 基础命令

#### 1.1 Docker 版本与信息

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker version` | 查看 Docker 客户端与服务端版本 | `--format '{{.Server.Version}}'` 格式化输出 |
| `docker info` | 查看 Docker 系统详细信息（镜像数、容器数、存储驱动等） | `--format '{{.RegistryConfig.Mirrors}}'` 查看镜像加速配置 |
| `docker system df` | 查看 Docker 磁盘使用情况 | `-v` 显示详细空间占用 |
| `docker system info` | 同 `docker info`，系统级信息 | — |

#### 1.2 Docker 系统管理

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker system prune` | 清理未使用的数据（停止的容器、未使用的网络、悬空镜像、构建缓存） | `-a` 清理所有未使用的镜像；`-f` 强制清理不提示；`--volumes` 同时清理未使用的卷 |
| `docker image prune` | 清理悬空镜像 | `-a` 清理所有未使用的镜像；`-f` 强制清理 |
| `docker container prune` | 清理所有停止的容器 | `-f` 强制清理 |
| `docker volume prune` | 清理所有未使用的本地卷 | `-f` 强制清理 |
| `docker network prune` | 清理所有未使用的网络 | `-f` 强制清理 |
| `docker builder prune` | 清理构建缓存 | `-a` 清理所有缓存；`-f` 强制清理 |

---

### 二、Docker 镜像命令

#### 2.1 镜像管理

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker images` | 列出本地所有镜像 | `-a` 显示所有（含中间层）；`-q` 仅显示镜像ID；`--filter "dangling=true"` 过滤悬空镜像；`--format` 格式化输出 |
| `docker search <镜像名>` | 搜索 Docker Hub 上的镜像 | `--limit 25` 限制返回数量；`--filter "stars=100"` 按星数过滤；`--filter "is-official=true"` 官方镜像 |
| `docker pull <镜像名>:<标签>` | 拉取镜像 | `--platform linux/amd64` 指定平台架构；`--all-tags` 拉取所有标签；`-q` 静默模式 |
| `docker push <镜像名>:<标签>` | 推送镜像到仓库 | `--disable-content-trust` 跳过内容信任签名 |
| `docker rmi <镜像ID/名>` | 删除镜像 | `-f` 强制删除；`--no-prune` 不删除未标记的父镜像 |
| `docker tag <源镜像> <目标镜像>:<标签>` | 给镜像打标签 | — |
| `docker save -o <文件名.tar> <镜像名>` | 导出镜像为 tar 文件 | `-o` 指定输出文件 |
| `docker load -i <文件名.tar>` | 从 tar 文件导入镜像 | `-i` 指定输入文件；`-q` 静默模式 |
| `docker image inspect <镜像名>` | 查看镜像详细信息（JSON） | `--format '{{.RootFS.Layers}}'` 格式化提取字段 |
| `docker image history <镜像名>` | 查看镜像构建历史（分层信息） | `--no-trunc` 显示完整输出；`--format` 格式化；`-H` 人类可读的时间 |

#### 2.2 镜像导入导出

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker export -o <文件.tar> <容器ID>` | 导出容器快照为 tar（导出的是容器文件系统，不含镜像元数据） | `-o` 指定输出文件 |
| `docker import <文件.tar> <镜像名>:<标签>` | 从容器快照创建镜像 | `-m "提交信息"` 添加提交信息；`-c "CMD ..."` 应用 Dockerfile 指令 |

---

### 三、Docker 容器命令

#### 3.1 容器生命周期

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker run <镜像名>` | 创建并启动一个容器（核心命令） | 见下方 **docker run 参数详解** |
| `docker start <容器ID/名>` | 启动已停止的容器 | `-a` 附加到容器输出；`-i` 交互模式 |
| `docker stop <容器ID/名>` | 优雅停止容器（SIGTERM → SIGKILL） | `-t <秒>` 等待超时时间，默认 10 秒 |
| `docker restart <容器ID/名>` | 重启容器 | `-t <秒>` 等待超时时间 |
| `docker kill <容器ID/名>` | 强制停止容器（直接 SIGKILL） | `-s <信号>` 指定发送的信号，如 `-s KILL` |
| `docker rm <容器ID/名>` | 删除已停止的容器 | `-f` 强制删除运行中的容器；`-v` 同时删除关联的匿名卷；`-l` 删除指定链接 |
| `docker pause <容器ID/名>` | 暂停容器内所有进程（使用 cgroups freezer） | — |
| `docker unpause <容器ID/名>` | 恢复被暂停的容器 | — |
| `docker create <镜像名>` | 创建容器但不启动（返回容器ID） | 参数同 `docker run` |

#### 3.2 docker run 参数详解

##### 命名与交互

| 参数 | 说明 |
|------|------|
| `--name <名称>` | 为容器指定名称，必须唯一 |
| `-d`, `--detach` | 后台运行容器，返回容器ID |
| `-i`, `--interactive` | 保持 STDIN 打开（交互模式） |
| `-t`, `--tty` | 分配一个伪终端（通常与 `-i` 一起使用：`-it`） |
| `--rm` | 容器退出后自动删除（不能与 `-d` 同时使用） |
| `-a`, `--attach` | 附加到 STDIN / STDOUT / STDERR |

##### 环境变量与工作目录

| 参数 | 说明 |
|------|------|
| `-e <KEY=VALUE>`, `--env <KEY=VALUE>` | 设置环境变量，可多次使用 |
| `--env-file <文件路径>` | 从文件中读取环境变量（每行 `KEY=VALUE` 格式） |
| `-w <路径>`, `--workdir <路径>` | 设置容器内工作目录 |
| `-u <用户>`, `--user <用户>` | 指定运行用户（用户名或 UID[:GID]） |
| `--entrypoint <可执行文件>` | 覆盖镜像默认的 ENTRYPOINT |
| `--hostname <主机名>` | 设置容器主机名 |

##### 端口映射

| 参数 | 说明 |
|------|------|
| `-p <宿主机端口>:<容器端口>` | 端口映射，如 `-p 8080:80` |
| `-p <IP>::<容器端口>` | 绑定到指定IP的随机端口 |
| `-p <宿主机端口>:<容器端口>/udp` | 绑定 UDP 端口 |
| `-p <宿主机端口>-<结束端口>:<容器端口>` | 端口范围映射 |
| `-P`, `--publish-all` | 将 EXPOSE 的所有端口随机映射到宿主机端口 |

##### 卷挂载与数据持久化

| 参数 | 说明 |
|------|------|
| `-v <宿主机路径>:<容器路径>` | **绑定挂载（Bind Mount）**，将宿主机目录/文件挂载到容器 |
| `-v <卷名>:<容器路径>` | **命名卷挂载（Named Volume）**，使用 Docker 管理的卷 |
| `-v <容器路径>` | **匿名卷挂载（Anonymous Volume）**，Docker 自动创建 |
| `-v <宿主机路径>:<容器路径>:ro` | 只读挂载（容器内不可写入） |
| `--mount type=bind,src=<宿主机路径>,dst=<容器路径>` | `--mount` 语法（推荐），绑定挂载 |
| `--mount type=volume,src=<卷名>,dst=<容器路径>` | `--mount` 语法，卷挂载 |
| `--mount type=tmpfs,dst=<容器路径>` | tmpfs 挂载（存储在内存中） |
| `--volumes-from <容器名>` | 从另一个容器挂载所有卷（数据卷容器模式） |

##### 资源限制

| 参数 | 说明 |
|------|------|
| `-m <数值>`, `--memory <数值>` | 内存限制，如 `-m 512m`、`-m 2g` |
| `--memory-swap <数值>` | 总内存+交换空间限制，`-1` 为无限制 |
| `--memory-reservation <数值>` | 内存软限制 |
| `--cpus <数值>` | CPU 核心数限制，如 `--cpus 1.5` |
| `--cpuset-cpus <核心编号>` | 绑定到指定 CPU 核心，如 `0-2` 或 `0,2` |
| `--cpu-shares <数值>` | CPU 相对权重（默认 1024） |
| `--blkio-weight <数值>` | 块 IO 相对权重（10-1000） |
| `--restart <策略>` | 重启策略：`no`、`on-failure[:次数]`、`always`、`unless-stopped` |
| `--ulimit <类型>=<软限制>:<硬限制>` | 设置 ulimit，如 `--ulimit nofile=1024:2048` |
| `--shm-size <数值>` | `/dev/shm` 大小，如 `--shm-size 256m` |
| `--oom-kill-disable` | 禁止 OOM Killer 杀死容器 |
| `--health-cmd` / `--health-interval` / `--health-timeout` / `--health-retries` | 健康检查配置 |

##### 网络相关

| 参数 | 说明 |
|------|------|
| `--network <网络名>` | 连接到指定网络（`bridge` / `host` / `none` / 自定义网络） |
| `--network-alias <别名>` | 添加网络别名（DNS 解析用） |
| `--add-host <主机名>:<IP>` | 添加自定义主机名到 IP 的映射（写入 /etc/hosts） |
| `--dns <IP>` | 设置 DNS 服务器 |
| `--dns-search <域名>` | 设置 DNS 搜索域 |
| `--link <容器名>:<别名>` | 连接到另一个容器（已废弃，推荐使用自定义网络） |
| `--expose <端口>` | 暴露端口（不映射到宿主机，仅容器间访问） |
| `--ip <IP地址>` | 指定容器 IP（需配合自定义网络使用） |
| `--mac-address <MAC>` | 指定容器 MAC 地址 |

##### 权限与安全

| 参数 | 说明 |
|------|------|
| `--privileged` | 授予容器扩展权限（近乎宿主机的 root 权限） |
| `--cap-add <能力>` | 添加 Linux 能力，如 `SYS_ADMIN`、`NET_ADMIN` |
| `--cap-drop <能力>` | 移除 Linux 能力，如 `--cap-drop ALL` 移除全部再按需添加 |
| `--security-opt <选项>` | 安全选项，如 `seccomp=unconfined`、`apparmor=unconfined` |
| `--read-only` | 将容器根文件系统挂载为只读 |
| `--tmpfs <路径>` | 在只读文件系统上创建 tmpfs 挂载点 |

##### 日志与容器生命周期

| 参数 | 说明 |
|------|------|
| `--log-driver <驱动>` | 日志驱动：`json-file`、`syslog`、`journald`、`none` 等 |
| `--log-opt <选项>` | 日志驱动选项，如 `max-size=10m`、`max-file=3` |
| `--init` | 在容器内运行一个 init 进程（处理僵尸进程） |
| `--stop-timeout <秒>` | 停止容器时的超时时间 |
| `--stop-signal <信号>` | 停止容器时发送的信号 |

#### 3.3 容器操作与调试

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker ps` | 列出运行中的容器 | `-a` 显示所有容器（含已停止）；`-q` 仅显示容器ID；`-s` 显示大小；`-l` 显示最近创建的；`-n 5` 显示最近 N 个；`--filter "status=exited"` 过滤；`--format` 格式化输出 |
| `docker logs <容器ID/名>` | 查看容器日志 | `-f` 实时跟踪；`--tail 100` 显示最后 N 行；`-t` 显示时间戳；`--since "2024-01-01"` 从指定时间开始；`--until "5m"` 截止到某时间 |
| `docker exec <容器ID/名> <命令>` | 在运行中的容器内执行命令 | `-i` 交互模式；`-t` 分配伪终端（常用 `-it`）；`-d` 后台执行；`-w` 指定工作目录；`-u` 指定用户；`-e` 设置环境变量 |
| `docker attach <容器ID/名>` | 附加到容器的主进程（stdin/stdout/stderr） | `--sig-proxy=false` 不转发信号 |
| `docker cp <容器ID>:<路径> <宿主机路径>` | 容器↔宿主机之间复制文件 | `-a` 归档模式（保留权限）；`-L` 跟随符号链接 |
| `docker top <容器ID/名>` | 显示容器内运行的进程 | — |
| `docker stats` | 实时显示容器资源使用统计（CPU/内存/网络/磁盘） | `--all` 显示所有容器（含已停止）；`--no-stream` 仅输出一次；`--format` 格式化 |
| `docker inspect <容器ID/名>` | 查看容器详细信息（JSON） | `--format '{{.NetworkSettings.IPAddress}}'` 提取特定字段；`-s` 显示文件大小 |
| `docker diff <容器ID/名>` | 检查容器文件系统的更改（A=新增, C=修改, D=删除） | — |
| `docker port <容器ID/名>` | 列出容器的端口映射 | — |
| `docker events` | 实时获取 Docker 守护进程事件 | `--since` / `--until` 时间过滤；`--filter "event=start"` 按事件类型过滤 |
| `docker wait <容器ID/名>` | 阻塞直到容器退出，打印退出码 | — |
| `docker rename <旧名> <新名>` | 重命名容器 | — |
| `docker update <容器ID/名>` | 更新运行中容器的配置 | `--restart` 更新重启策略；`--memory` 更新内存限制；`--cpus` 更新CPU限制 |

---

### 四、Docker 网络命令

#### 4.1 网络管理

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker network ls` | 列出所有网络 | `--filter "driver=bridge"` 按驱动过滤；`-q` 仅显示ID；`--no-trunc` 不截断输出 |
| `docker network create <网络名>` | 创建自定义网络 | `-d` 指定驱动（`bridge` / `overlay` / `macvlan`）；`--subnet` 指定子网；`--gateway` 指定网关；`--ip-range` IP 分配范围；`--internal` 限制外网访问；`--label` 添加标签；`--attachable` 允许容器接入（用于 swarm overlay 网络） |
| `docker network rm <网络名>` | 删除网络 | `-f` 强制删除 |
| `docker network inspect <网络名>` | 查看网络详细信息（挂载的容器、IP等） | `--format '{{.Containers}}'` 提取字段 |
| `docker network connect <网络名> <容器名>` | 将运行中的容器连接到网络 | `--ip` 指定IP；`--alias` 添加网络别名；`--link` 添加链接 |
| `docker network disconnect <网络名> <容器名>` | 断开容器与网络的连接 | `-f` 强制断开 |
| `docker network prune` | 清理未使用的网络 | `-f` 强制清理 |

#### 4.2 网络驱动类型

| 驱动 | 说明 | 适用场景 |
|------|------|----------|
| `bridge` | 默认网桥驱动，容器通过 NAT 与宿主机通信 | 单机容器互联 |
| `host` | 容器与宿主机共享网络栈，无网络隔离 | 高性能网络场景 |
| `none` | 容器无网络接口 | 完全隔离场景 |
| `overlay` | 跨多台 Docker 主机的覆盖网络 | Swarm 集群容器通信 |
| `macvlan` | 容器直接获取物理网络中的 MAC 地址和 IP | 需要容器直接暴露在物理网络 |
| `ipvlan` | 与 macvlan 类似但共享 MAC 地址 | 云环境有 MAC 限制时使用 |

---

### 五、Docker 卷（Volume）与数据持久化

#### 5.1 卷管理命令

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker volume ls` | 列出所有卷 | `--filter "dangling=true"` 过滤未使用的卷；`-q` 仅显示名称 |
| `docker volume create <卷名>` | 创建命名卷 | `-d` 指定驱动（默认 `local`）；`--opt type=nfs` 驱动选项；`--label` 标签 |
| `docker volume rm <卷名>` | 删除卷 | `-f` 强制删除（即使卷正在使用） |
| `docker volume inspect <卷名>` | 查看卷详细信息（挂载点等） | `--format '{{.Mountpoint}}'` 提取挂载路径 |
| `docker volume prune` | 清理所有未使用的本地卷 | `-f` 强制清理；`--filter "label=..."` 按标签过滤 |

#### 5.2 三种挂载方式对比

| 方式 | 语法 | 数据位置 | 适用场景 |
|------|------|----------|----------|
| **绑定挂载 (Bind Mount)** | `-v /host/path:/container/path` 或 `--mount type=bind,src=...,dst=...` | 宿主机任意路径 | 开发环境、共享配置文件、代码热更新 |
| **命名卷 (Named Volume)** | `-v volume_name:/container/path` 或 `--mount type=volume,src=...,dst=...` | `/var/lib/docker/volumes/` | 生产环境数据持久化、数据库存储 |
| **tmpfs 挂载** | `--tmpfs /container/path` 或 `--mount type=tmpfs,dst=...` | 内存中 | 临时数据、敏感信息安全存储（容器停止即消失） |

#### 5.3 卷驱动选项（--opt）

| 选项 | 说明 |
|------|------|
| `type=nfs` | NFS 远程挂载 |
| `o=addr=<IP>,rw` | NFS 服务器地址和读写权限 |
| `device=:<远程路径>` | NFS 远程路径 |
| `type=cifs` | CIFS/Samba 远程挂载 |
| `device=//<IP>/<共享>` | CIFS 远程共享路径 |
| `o=username=<用户>,password=<密码>` | CIFS 认证信息 |

---

### 六、Dockerfile 指令手册

#### 6.1 核心指令

| 指令 | 说明 | 格式与示例 |
|------|------|------------|
| `FROM` | 指定基础镜像（必须是第一条非注释指令） | `FROM centos:8` / `FROM ubuntu:22.04 AS builder`（多阶段构建） |
| `RUN` | 构建时执行命令（每执行一次新增一层） | `RUN apt-get update && apt-get install -y curl`（shell格式）；`RUN ["/bin/sh", "-c", "echo hello"]`（exec格式） |
| `CMD` | 容器启动时的默认命令（可被 `docker run` 后的命令覆盖） | `CMD ["nginx", "-g", "daemon off;"]`（exec格式，推荐）；`CMD echo hello`（shell格式）；`CMD ["param1","param2"]`（为 ENTRYPOINT 提供默认参数） |
| `ENTRYPOINT` | 容器入口点（不会被 `docker run` 后的命令覆盖，除非使用 `--entrypoint`） | `ENTRYPOINT ["python", "app.py"]`（exec格式）；`ENTRYPOINT top -b`（shell格式） |
| `LABEL` | 添加元数据标签 | `LABEL version="1.0" maintainer="user@example.com"` |
| `EXPOSE` | 声明容器运行时监听的端口（仅声明，不发布） | `EXPOSE 80` / `EXPOSE 80/tcp` / `EXPOSE 80/udp` |
| `ENV` | 设置环境变量（持久化到容器运行时） | `ENV APP_HOME=/app` / `ENV NODE_VERSION=18 PATH=$PATH:/opt/node/bin` |
| `ARG` | 定义构建时变量（仅构建时可用，`--build-arg` 传入） | `ARG VERSION=latest` |
| `COPY` | 将宿主机文件/目录复制到镜像中 | `COPY ./app /app` / `COPY --chown=user:group src dst` / `COPY --from=builder /app/build /app`（多阶段构建） |
| `ADD` | 同 COPY，但额外支持 URL 下载和自动解压 tar | `ADD https://example.com/file.tar.gz /tmp/` / `ADD archive.tar.gz /app/`（自动解压） |
| `WORKDIR` | 设置工作目录（后续指令在此目录执行） | `WORKDIR /app` |
| `VOLUME` | 创建挂载点（匿名卷，用于数据持久化） | `VOLUME /data` / `VOLUME ["/data", "/logs"]` |
| `USER` | 设置运行用户 | `USER nginx` / `USER 1000:1000` |
| `SHELL` | 设置 RUN/CMD/ENTRYPOINT 使用的默认 shell | `SHELL ["/bin/bash", "-c"]` |
| `HEALTHCHECK` | 容器健康检查 | `HEALTHCHECK --interval=30s --timeout=3s --retries=3 CMD curl -f http://localhost/ || exit 1` |
| `STOPSIGNAL` | 设置停止容器的信号 | `STOPSIGNAL SIGTERM` / `STOPSIGNAL 9` |
| `ONBUILD` | 当下游镜像以此镜像为基础时触发（延迟执行） | `ONBUILD COPY . /app` |

#### 6.2 RUN 命令优化技巧

| 技巧 | 说明 | 示例 |
|------|------|------|
| 合并 RUN 命令 | 减少镜像层数 | `RUN apt update && apt install -y curl && rm -rf /var/lib/apt/lists/*` |
| `--mount=type=cache` | 缓存包管理器数据，加速重复构建（BuildKit） | `RUN --mount=type=cache,target=/var/cache/apt apt install -y curl` |
| `--mount=type=secret` | 构建时安全使用敏感文件（BuildKit） | `RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret` |
| `--mount=type=ssh` | 构建时使用SSH密钥（BuildKit） | `RUN --mount=type=ssh git clone git@github.com:user/repo.git` |

#### 6.3 多阶段构建

```dockerfile
# 第一阶段：构建
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# 第二阶段：运行（极小运行镜像）
FROM alpine:3.19
COPY --from=builder /app/myapp /usr/local/bin/myapp
ENTRYPOINT ["myapp"]
```

| 特性 | 说明 |
|------|------|
| `FROM ... AS <名称>` | 为构建阶段命名 |
| `COPY --from=<阶段名>` | 从其他构建阶段复制文件 |
| `COPY --from=0` | 使用阶段索引（从 0 开始） |

---

### 七、Docker 构建命令

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker build -t <镜像名>:<标签> .` | 从 Dockerfile 构建镜像 | `-f <Dockerfile路径>` 指定 Dockerfile；`--build-arg KEY=VALUE` 传入构建参数；`--no-cache` 不使用缓存；`--pull` 始终拉取最新基础镜像；`--target <阶段>` 多阶段构建时停在指定阶段；`--platform linux/amd64,linux/arm64` 多平台构建；`--secret id=mysecret,src=./secret` 构建时密钥；`--ssh default` 构建时SSH代理；`--network host` 构建时网络模式；`-q` 静默模式 |
| `docker buildx build` | BuildKit 增强构建（多平台、缓存等） | `--platform linux/amd64,linux/arm64` 多平台；`--push` 构建后直接推送；`--cache-from` / `--cache-to` 缓存导入导出；`--output type=local,dest=./out` 导出构建结果 |
| `docker buildx create --name <构建器名>` | 创建构建器实例 | `--use` 设为默认；`--driver docker-container` 驱动类型 |
| `docker buildx ls` | 列出构建器 | — |
| `docker buildx prune` | 清理构建缓存 | `-a` 清理全部；`-f` 强制 |

---

### 八、Docker Compose 命令

#### 8.1 docker compose 常用命令（V2 语法）

> **注意**：V2 使用 `docker compose`（无连字符），V1 使用 `docker-compose`（已废弃）

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker compose up` | 创建并启动所有服务 | `-d` 后台运行；`--build` 启动前构建镜像；`--force-recreate` 强制重新创建容器；`--remove-orphans` 删除 compose 文件中未定义的服务容器；`--scale <服务名>=N` 扩展服务实例数；`--no-deps` 不启动依赖服务 |
| `docker compose down` | 停止并删除容器、网络 | `-v` 同时删除卷（**慎用！** 会删除持久化数据）；`--rmi all` 删除所有镜像；`--rmi local` 仅删除无标签镜像；`-t <秒>` 超时时间；`--remove-orphans` 删除孤立容器 |
| `docker compose start` | 启动已存在的服务容器 | — |
| `docker compose stop` | 停止服务容器（保留容器） | `-t <秒>` 超时时间 |
| `docker compose restart` | 重启服务容器 | `-t <秒>` 超时时间 |
| `docker compose ps` | 列出 compose 项目中的容器 | `-a` 显示所有（含已停止）；`--format` 格式化输出；`--services` 仅显示服务名 |
| `docker compose logs` | 查看服务日志 | `-f` 实时跟踪；`--tail 100` 最后N行；`-t` 时间戳；`--no-color` 无颜色输出 |
| `docker compose build` | 构建（或重建）服务镜像 | `--no-cache` 不使用缓存；`--pull` 拉取最新基础镜像；`--build-arg KEY=VALUE` 构建参数；`--parallel` 并行构建 |
| `docker compose pull` | 拉取服务镜像 | `--ignore-pull-failures` 忽略拉取失败；`--parallel` 并行拉取；`-q` 静默 |
| `docker compose push` | 推送服务镜像 | `--ignore-push-failures` 忽略推送失败 |
| `docker compose exec <服务名> <命令>` | 在运行中的服务容器内执行命令 | `-i` 交互；`-T` 禁用伪终端（脚本中使用）；`-u` 指定用户；`-w` 指定工作目录；`-e` 环境变量；`--index N` 指定副本索引 |
| `docker compose run <服务名> <命令>` | 对服务执行一次性命令（创建新容器） | `--rm` 退出后自动删除；`-v` 挂载卷；`-e` 环境变量；`-u` 用户；`-w` 工作目录；`--no-deps` 不启动依赖；`--entrypoint` 覆盖入口点；`--service-ports` 使用服务端口映射 |
| `docker compose pause` | 暂停服务容器 | — |
| `docker compose unpause` | 恢复暂停的服务容器 | — |
| `docker compose top` | 显示服务中运行的进程 | — |
| `docker compose config` | 验证并查看 compose 文件（解析后） | `-q` 仅检查语法不输出内容；`--services` 列出服务名；`--volumes` 列出卷名；`--format json` JSON格式输出 |
| `docker compose images` | 列出 compose 使用的镜像 | — |
| `docker compose cp` | 在服务容器和宿主机之间复制文件 | `-a` 归档模式；`-L` 跟随符号链接；`--index N` 指定副本 |
| `docker compose create` | 创建服务容器但不启动 | `--force-recreate` 强制重新创建；`--build` 先构建 |
| `docker compose kill` | 强制停止服务容器 | `-s <信号>` 指定信号 |
| `docker compose rm` | 删除已停止的服务容器 | `-f` 强制；`-v` 同时删除匿名卷；`-s` 在删除前停止（默认行为） |
| `docker compose port <服务名> <私有端口>` | 打印服务端口映射的公网端口 | `--protocol tcp` 指定协议；`--index N` 指定副本 |
| `docker compose events` | 接收项目中容器的实时事件 | `--json` JSON 格式输出 |

#### 8.2 docker-compose.yml 核心配置项

```yaml
version: "3.8"   # Compose 文件版本（V2之后 version 已非必填）
name: project_name   # Compose 项目名称

services:
  web:
    image: nginx:alpine            # 使用镜像
    # build:                       # 构建配置（与 image 互斥）
    #   context: ./web             # Dockerfile 上下文路径
    #   dockerfile: Dockerfile.prod # Dockerfile 文件名
    #   args:                      # 构建参数
    #     NODE_ENV: production
    #   target: builder            # 多阶段构建目标
    container_name: my-web         # 容器名
    hostname: web-server          # 主机名
    restart: unless-stopped       # 重启策略
    ports:
      - "8080:80"                 # 端口映射
    expose:
      - "80"                      # 仅暴露给链接的服务
    environment:                   # 环境变量
      - NODE_ENV=production
      - DB_HOST=db
    env_file:                     # 从文件读取环境变量
      - ./config/web.env
    volumes:                       # 卷挂载
      - type: volume
        source: web_data
        target: /usr/share/nginx/html
      - type: bind
        source: ./nginx.conf
        target: /etc/nginx/nginx.conf
        read_only: true
    networks:
      - frontend
      - backend
    depends_on:                   # 服务依赖
      - db
    command: ["nginx", "-g", "daemon off;"]  # 覆盖 CMD
    entrypoint: /entrypoint.sh    # 覆盖 ENTRYPOINT
    user: "1000:1000"             # 运行用户
    working_dir: /app             # 工作目录
    healthcheck:                  # 健康检查
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:                      # 日志配置
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    deploy:                       # 部署配置（仅 Swarm 模式）
      replicas: 3
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    profiles:                     # 配置文件（按需启动）
      - production

volumes:
  web_data:                       # 声明命名卷
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/export/web"

networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br-frontend
  backend:
    driver: bridge
```

#### 8.3 Compose 配置文件

| 配置项 | 说明 |
|--------|------|
| `depends_on` | 启动顺序依赖（注意：不会等待服务就绪） |
| `condition: service_healthy` | 等待服务健康检查通过（需版本 3.4+，`depends_on` 增强） |
| `profiles` | 按需启动：`docker compose --profile production up` |
| `extends` | 继承其他 compose 文件配置 |
| `x-` 扩展字段 | YAML 锚点复用配置块（如 `x-common: &common`） |
| `include` | 包含其他 compose 文件（V2.20+） |

---

### 九、Docker 仓库与注册表

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker login <注册表URL>` | 登录到 Docker 注册表 | `-u` 用户名；`-p` 密码（不推荐明文）；`--password-stdin` 从 STDIN 读取密码 |
| `docker logout <注册表URL>` | 退出登录 | — |
| `docker pull <镜像>` | 拉取镜像 | `--all-tags` 拉取所有标签；`--platform` 指定平台 |
| `docker push <镜像>` | 推送镜像 | `--disable-content-trust` 跳过签名 |
| `docker search <关键词>` | 搜索镜像 | `--filter "stars=100"`；`--filter "is-official=true"`；`--limit N` |

---

### 十、Docker Swarm 集群命令

#### 10.1 集群管理

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker swarm init` | 初始化 Swarm 集群 | `--advertise-addr <IP>` 广播地址；`--listen-addr <IP:PORT>` 监听地址 |
| `docker swarm join` | 加入 Swarm 集群 | `--token <令牌> <IP:PORT>` |
| `docker swarm leave` | 离开 Swarm 集群 | `-f` 强制离开（管理节点） |
| `docker swarm join-token <角色>` | 查看加入令牌 | `worker` / `manager`；`-q` 仅显示令牌 |
| `docker swarm update` | 更新集群配置 | `--task-history-limit N`；`--dispatcher-heartbeat` |

#### 10.2 服务与任务

| 命令 | 说明 | 常用参数 |
|------|------|----------|
| `docker service create` | 创建服务 | 参数与 `docker run` 类似，额外有 `--replicas N`、`--update-delay` 等 |
| `docker service ls` | 列出服务 | `--format` 格式化 |
| `docker service ps <服务名>` | 列出服务任务 | `--no-trunc` 不截断 |
| `docker service logs <服务名>` | 查看服务日志 | `-f` 跟踪；`--tail N` |
| `docker service scale <服务名>=N` | 扩缩容 | — |
| `docker service update <服务名>` | 更新服务配置 | `--image` 更新镜像；`--replicas`；`--force` 强制更新 |
| `docker service rm <服务名>` | 删除服务 | — |
| `docker node ls` | 列出集群节点 | — |
| `docker node promote <节点>` | 将工作节点提升为管理节点 | — |
| `docker node demote <节点>` | 降级管理节点 | — |

#### 10.3 栈（Stack）

| 命令 | 说明 |
|------|------|
| `docker stack deploy -c <compose文件> <栈名>` | 部署 Compose 栈到 Swarm |
| `docker stack ls` | 列出栈 |
| `docker stack services <栈名>` | 列出栈中的服务 |
| `docker stack ps <栈名>` | 列出栈中的任务 |
| `docker stack rm <栈名>` | 删除栈 |

---

### 十一、Docker 日志与监控

| 命令 | 说明 |
|------|------|
| `docker logs --details <容器>` | 查看日志详细信息（含标签） |
| `docker system events --since 1h` | 查看最近1小时的系统事件 |
| `docker stats --all --no-stream` | 一次性查看所有容器资源统计 |
| `docker inspect --format '{{.State.Health}}' <容器>` | 查看容器健康状态 |

---

### 十二、Docker 实用技巧

#### 12.1 批量操作

```bash
# 删除所有停止的容器
docker container prune -f

# 删除所有未使用的镜像
docker image prune -a -f

# 停止所有运行中的容器
docker stop $(docker ps -q)

# 删除所有容器（含运行中的）
docker rm -f $(docker ps -aq)

# 删除所有镜像
docker rmi -f $(docker images -q)

# 删除所有未使用的资源（清理所有）
docker system prune -a --volumes -f
```

#### 12.2 格式化输出示例

```bash
# 查看容器IP
docker inspect --format '{{.NetworkSettings.IPAddress}}' <容器名>

# 格式化列出容器（表格形式）
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# 查看镜像创建时间
docker images --format "{{.Repository}}:{{.Tag}} ({{.CreatedSince}})"
```

#### 12.3 调试技巧

| 技巧 | 命令 |
|------|------|
| 进入容器 Shell | `docker exec -it <容器> /bin/bash`（或 `/bin/sh`） |
| 以临时容器调试网络 | `docker run --rm -it --network <网络名> alpine sh` |
| 查看容器退出码 | `docker inspect --format '{{.State.ExitCode}}' <容器>` |
| 查看容器启动错误 | `docker logs <容器> 2>&1 \| tail -50` |
| 检查 Docker daemon 状态 | `systemctl status docker` |
| 重启 Docker 服务 | `sudo systemctl restart docker` |

---

### 十三、Docker 配置文件与路径

| 配置项 | 路径 | 说明 |
|--------|------|------|
| Docker 守护进程配置 | `/etc/docker/daemon.json` | 镜像加速、日志驱动、存储驱动等 |
| Docker 服务代理配置 | `/etc/systemd/system/docker.service.d/proxy.conf` | HTTP/HTTPS 代理 |
| Docker 数据存储目录 | `/var/lib/docker/` | 镜像、容器、卷等数据 |
| 卷存储位置 | `/var/lib/docker/volumes/<卷名>/_data/` | 命名卷的实际数据路径 |
| 容器配置与日志 | `/var/lib/docker/containers/<ID>/` | 容器配置 JSON 和日志文件 |
| Docker Compose 项目 | `./docker-compose.yml` 或 `compose.yaml` | 默认文件名 |
| 用户级 Docker 配置 | `~/.docker/config.json` | 登录凭证等 |
| Dockerfile | `./Dockerfile` | 默认文件名（构建时 `-f` 可指定其他） |
| `.dockerignore` | `./.dockerignore` | 构建时忽略的文件（类似 .gitignore） |

---

### 十四、.dockerignore 文件

用于排除不需要发送到 Docker 构建上下文的文件，提升构建速度。

```dockerignore
# 版本控制
.git
.gitignore

# 依赖
node_modules
__pycache__
*.pyc

# 构建产物
dist
build
*.class

# 文档与说明
README.md
LICENSE
*.md

# 临时文件
*.log
*.tmp
.DS_Store
Thumbs.db

# 环境配置
.env
.env.local

# IDE
.vscode
.idea

# docker 相关
Dockerfile
docker-compose.yml
.dockerignore

# 测试
test
tests
coverage
```

---

### 十五、常用 docker run 命令速查示例

```bash
# 基础运行（前台）
docker run --rm -it centos:8 bash

# 后台运行带端口映射
docker run -d --name my-web -p 8080:80 nginx:alpine

# 挂载宿主机目录
docker run -d --name my-app \
  -v /host/data:/container/data \
  -v /host/nginx.conf:/etc/nginx/nginx.conf:ro \
  my-image

# 资源限制运行
docker run -d --name limited-app \
  --memory 512m \
  --memory-swap 1g \
  --cpus 2 \
  --restart unless-stopped \
  my-image

# 使用自定义网络
docker run -d --name my-app \
  --network my-custom-network \
  --network-alias app-server \
  my-image

# 传递环境变量
docker run -d --name my-app \
  -e DB_HOST=db.example.com \
  -e DB_PASSWORD=secret123 \
  -e "TZ=Asia/Shanghai" \
  --env-file ./prod.env \
  my-image
```
