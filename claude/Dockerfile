# 基础 Node.js 镜像
FROM node:26-slim

# 清除旧版源配置，并写入清华源 (DEB822 格式)
RUN echo "" > /etc/apt/sources.list && \
    cat <<'EOF' > /etc/apt/sources.list.d/debian.sources
Types: deb
URIs: http://mirrors.tuna.tsinghua.edu.cn/debian
Suites: trixie trixie-updates trixie-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# 以下安全更新软件源为官方源配置
Types: deb
URIs: http://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
EOF

# 更新系统并安装必要依赖，同时切换 npm 镜像源以加速 CLI 下载
RUN apt-get update && apt-get install -y curl git vim iproute2 && \
    rm -rf /var/lib/apt/lists/* && \
    npm config set registry http://registry.npmmirror.com

# 通过 npm 全局安装 Claude Code CLI
RUN npm install -g @anthropic-ai/claude-code@latest

# 设置工作目录
WORKDIR /project

# 保持容器运行的默认指令
CMD ["tail", "-f", "/dev/null"]
