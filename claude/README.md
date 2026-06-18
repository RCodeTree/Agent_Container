# 容器为基安装Claude-CLI

## 介绍

**这是一个基于Docker的容器，用于安装Claude-CLI。Claude-CLI是一个用于在命令行中与Claude进行交互的工具。**

### 基础镜像配置
- 镜像：`node:26-slim` 
    - 镜像配置方式(任选其一)
        - [Node官方网站](https://nodejs.org/zh-cn/download/current/)
        - docker pull node:26-slim

---

## 安装 Claude-CLI 和 OpenClaw

### 编写 Dockerfile 构建镜像
1. 引入基础镜像
2. 更换镜像软件源 - 加速镜像构建
    - [清华源](https://mirrors.tuna.tsinghua.edu.cn/)(推荐)
3. 安装 Claude-CLI 必要依赖
4. npm/pnpm/bun 拉取 @anthropic-ai/claude-code@latest
5. npm/pnpm/bun 拉取 @anthropic-ai/open-claw-cli@latest
6. 设置工作目录
7. 保持容器运行的默认指令
    - ["tail", "-f", "/dev/null"]
8. docker build -t claude-cli-env .

### 运行容器(关键步骤)

#### bridge 模式 (默认网络模式) 
**在 bridge 模式下，容器会与宿主机网络隔离，通过宿主机的IP映射容器的虚拟IP和端口访问外部网络(虚拟网桥)，容器无法直接访问宿主机的网络。**
``` bash
# 显性指定网络模式为 bridge 模式
docker run -itd --name container-name -v $(pwd):your-workdir --network bridge claude-cli-env

# 隐性指定网络模式为 bridge 模式
docker run -itd --name container-name -v $(pwd):your-workdir claude-cli-env
```
1. `Claude-CLI`
    - 国内不能直接访问Claude
    - **使用魔法上网(Clash 或者 Clash Verge 等工具)使用工具 TUN模式 并开启系统代理**
2. `OpenClaw-CLI`
    

