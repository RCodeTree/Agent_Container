# Docker_Container

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
 - macvlan
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