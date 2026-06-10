FROM node:26-slim

# 安装必要的依赖工具
RUN apt-get update && apt-get install -y curl git vim iproute2 && rm -rf /var/lib/apt/lists/*

# 使用 npm 全局安装 Claude Code CLI
RUN npm install -g @anthropic-ai/claude-code@latest

# 设置工作目录
WORKDIR /project

# 保持容器运行的默认指令
CMD ["tail", "-f", "/dev/null"] 
