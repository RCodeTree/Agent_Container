# 容器为基安装Claude-CLI

## 介绍

**这是一个基于Docker的容器，用于安装Claude-CLI。Claude-CLI是一个用于在命令行中与Claude进行交互的工具。**

### 基础镜像配置
- 镜像：`node:26-slim` 
    - 镜像配置方式(任选其一)
        - [Node官方网站](https://nodejs.org/zh-cn/download/current/)
        - docker pull node:26-slim

---

## 安装 Claude-CLI

### 编写 Dockerfile 构建镜像
1. 引入基础镜像
2. 更换镜像软件源 - 加速镜像构建
    - [清华源](https://mirrors.tuna.tsinghua.edu.cn/)(推荐)
3. 安装 Claude-CLI 必要依赖
4. npm/pnpm/bun 拉取 @anthropic-ai/claude-code@latest
5. 设置工作目录
6. 保持容器运行的默认指令
    - ["tail", "-f", "/dev/null"]
7. docker build -t claude-cli-env .

### 运行容器(关键步骤)
