# Docker 容器监控与管理

在本章节中，我们将学习如何有效地监控和管理 Docker 容器，包括使用命令行工具和图形化界面（Portainer）进行容器管理。

注意: 该部分所需要的容器我们可以利用上一节 docker compose 出来的容器来演示。

## 1. 容器管理基础

### 1.1 容器生命周期管理

以下是一些最常用的容器管理命令：

```bash
# 列出所有容器（包括停止的容器）
docker ps -a

# 仅列出运行中的容器
docker ps

# 启动容器
docker start <container_id>

# 停止容器
docker stop <container_id>

# 重启容器
docker restart <container_id>

# 删除容器（需要先停止）
docker rm <container_id>

# 强制删除运行中的容器
docker rm -f <container_id>
```

### 1.2 容器资源监控

Docker 提供了多种方式来监控容器的资源使用情况：

```bash
# 实时查看容器资源使用状态
docker stats

# 例：查看容器资源使用情况（仅显示 CPU 和内存）
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# 查看容器详细信息
docker inspect <container_id>

# 查看容器内进程
docker top <container_id>

# 查看容器端口映射
docker port <container_id>
```

## 2. 容器日志与调试

### 2.1 日志查看

```bash
# 查看容器日志
docker logs <container_id>

# 实时查看最新日志
docker logs -f <container_id>

# 查看最近 100 行日志
docker logs --tail 100 <container_id>

# 显示时间戳
docker logs -t <container_id>
```



## 3. 实践练习: 使用 Portainer 对 docker 进行可视化管理

Portainer 是一个轻量级的 Docker 管理工具，提供了直观的 Web 界面来管理 Docker 环境。

### 3.1 安装 Portainer

```bash
# 创建 Portainer 数据卷
docker volume create portainer_data

# 运行 Portainer 容器
docker run -d -p 9000:9000 \
    --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:2.29.2
```


   `-v /var/run/docker.sock:/var/run/docker.sock`

>    /var/run/docker.sock 是一个实现 Docker 客户端 与 Docker 服务器端 之间通信的“桥梁”，一个特殊的套接字文件，用于进程间通信
>   其核心作用是让容器内的进程能够直接与主机的 Docker 守护进程（dockerd）通信，从而操作主机上的 Docker 资源（容器、镜像、网络等）。相当于赋予了 Portainer 容器与宿主机 Docker 引擎同等的控制权。


安装完成后，在 cnb 上我们可以通过添加一个 9000 的端口映射来实现外网访问, 可以按照如下步骤来配置。

![port_forward](../assets/port_forward.png)

点击这个浏览器图标，就可以访问 Portainer 了。

### 3.2 Portainer 主要功能

1. **仪表盘概览**
   - 查看环境整体状态
   - 监控资源使用情况
   - 查看事件日志

2. **容器管理**
   - 创建、启动、停止、删除容器
   - 查看容器日志和统计信息
   - 进入容器终端
   - 修改容器配置

3. **镜像管理**
   - 拉取和删除镜像
   - 构建新镜像
   - 推送镜像到仓库

4. **网络管理**
   - 创建和管理 Docker 网络
   - 配置容器网络连接

5. **数据卷管理**
   - 创建和删除数据卷
   - 管理数据卷权限
