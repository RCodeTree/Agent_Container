# Docker_Container

## 项目介绍
- 这是一个基于Docker的容器项目，用于个人记录和分享。
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
   - 如果没有使用国内docker软件源，需要配置魔法上网
   - 首先，确保自己已经购买魔法上网服务，即梯子
   - 然后，使用Clash等工具配置魔法上网
   - 最后，确保自己的物理机已经连接到魔法上网服务
