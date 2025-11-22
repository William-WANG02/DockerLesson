# Docker 实践课程完整使用说明

## 📚 项目概述

本仓库是一个全面的 Docker 实践教程，旨在帮助学习者从零开始掌握 Docker 容器化技术。通过循序渐进的课程设计和丰富的实践案例，您将学会如何使用 Docker 构建、部署和管理容器化应用。

### 🎯 适用人群

- Docker 初学者，希望系统学习容器化技术
- 开发人员，需要掌握应用容器化部署
- 运维工程师，希望提升容器管理能力
- 架构师，需要了解微服务容器化实践

### 💡 学习目标

完成本课程后，您将能够：

1. 理解 Docker 核心概念（镜像、容器、仓库）
2. 编写标准的 Dockerfile 构建自定义镜像
3. 管理容器的存储和网络配置
4. 使用 Docker Compose 编排多容器应用
5. 监控和管理生产环境中的容器
6. 将实际项目进行容器化部署

### 🚀 快速开始

1. **Fork 本仓库**到您的 GitHub 账号
2. **点击"一键学习"按钮**进入云原生开发环境（CNB）
3. **按照课程顺序**逐步学习和实践
4. **完成实战项目**巩固所学知识

---

## 📂 仓库结构说明

```
DockerLesson/
├── 1_foundation/          # 第一章：Docker 基础
├── 2_dockerfile/          # 第二章：Dockerfile 详解
├── 3_storage/             # 第三章：存储管理
├── 4_network/             # 第四章：网络管理
├── 5_compose/             # 第五章：Docker Compose
├── 6_management/          # 第六章：容器监控与管理
├── extended/              # 扩展内容：高级主题
├── project/               # 实战项目：RAG 智能问答系统
├── assets/                # 图片和资源文件
├── README.md              # 项目主页
└── USAGE.md               # 本说明文档
```

---

## 📖 详细课程内容

### 第一章：Docker 基础 (`1_foundation/`)

#### 使用场景
学习 Docker 的基本概念和操作，包括容器生命周期管理、镜像操作、仓库使用等。

#### 核心内容

1. **查看 Docker 信息**
   ```bash
   docker version  # 查看 Docker 版本
   docker info     # 查看 Docker 系统信息
   ```

2. **运行第一个容器**
   ```bash
   docker run hello-world
   ```
   - `docker run`: Docker 的核心命令，用于创建并启动容器
   - `hello-world`: 官方提供的最简单测试镜像
   - 执行流程：检查本地镜像 → 下载镜像（如不存在）→ 创建容器 → 运行容器

3. **镜像操作详解**
   
   **拉取镜像**
   ```bash
   docker pull alpine
   ```
   - `pull`: 从镜像仓库下载镜像
   - `alpine`: 轻量级 Linux 发行版（约 8MB）
   - 默认从 Docker Hub 拉取 latest 标签

   **查看镜像**
   ```bash
   docker image ls
   ```
   - 显示本地所有镜像
   - 镜像格式：`<repository>/<image>:<tag>`
   - 示例：`docker.cnb.cool/coldenn/docker-open-camp/docker-exercises/my-alpine:latest`

   **删除镜像**
   ```bash
   docker image rm <image_id>
   ```
   - 删除指定镜像
   - 如果镜像被容器使用，需先删除容器

   **查看镜像历史**
   ```bash
   docker history <image_id>
   ```
   - 显示镜像的构建历史
   - 可以看到每一层的大小和创建命令

4. **容器操作详解**

   **运行容器**
   ```bash
   docker run alpine
   ```
   - 创建并启动容器
   - 容器执行完命令后会自动退出

   **查看容器**
   ```bash
   docker ps       # 查看运行中的容器
   docker ps -a    # 查看所有容器（包括已停止）
   ```

   **交互式运行容器**
   ```bash
   docker run -it alpine /bin/sh
   ```
   - `-i`: 保持标准输入打开（interactive）
   - `-t`: 分配伪终端（TTY）
   - `/bin/sh`: 要执行的命令（Alpine 的 shell）
   - 用途：进入容器内部进行调试或操作

   **后台运行容器**
   ```bash
   docker run -d alpine sleep 3600
   ```
   - `-d`: 后台运行模式（detached）
   - `sleep 3600`: 让容器保持运行 1 小时
   - 返回容器 ID

   **进入运行中的容器**
   ```bash
   docker exec -it <container_id> /bin/sh
   ```
   - `exec`: 在运行中的容器内执行命令
   - 创建新的 shell 进程，退出不影响容器运行

   **attach 方式（不推荐）**
   ```bash
   docker attach <container_id>
   ```
   - 直接连接到 PID=1 的进程
   - 退出会导致容器停止

   **查看容器日志**
   ```bash
   docker logs <container_id>
   docker logs -f <container_id>  # 持续跟踪日志
   ```

   **查看容器详情**
   ```bash
   docker inspect <container_id>
   ```
   - 返回容器的 JSON 格式配置信息

#### 核心概念

- **镜像层（Image Layers）**: Docker 使用 OverlayFS 联合文件系统，镜像由多个只读层组成
- **容器层（Container Layer）**: 容器运行时在镜像之上添加一个可写层
- **写时复制（Copy-on-Write）**: 修改文件时才复制到容器层，提高效率

---

### 第二章：Dockerfile 详解 (`2_dockerfile/`)

#### 使用场景
学习如何编写 Dockerfile 自动化构建自定义镜像，实现可重复、标准化的镜像构建过程。

#### 核心内容

1. **命令式创建镜像（不推荐用于生产）**

   ```bash
   # 1. 启动容器
   docker run -it --name alpine alpine
   
   # 2. 在容器内安装软件
   apk update
   apk add figlet
   exit
   
   # 3. 提交容器为镜像
   docker commit <container_id> alpine-figlet
   
   # 4. 使用新镜像
   docker run alpine-figlet figlet "Hello Docker"
   ```

   **局限性：**
   - 不可重复：依赖人工操作
   - 镜像臃肿：包含临时文件和缓存
   - 难以维护：无法版本控制
   - 安全风险：无法追溯构建过程

2. **声明式创建镜像（推荐）**

   **简单 Dockerfile 示例** (`2_dockerfile/Dockerfile`)
   ```dockerfile
   FROM alpine:latest
   RUN apk update &&\
       apk add figlet
   ```

   **逐行解释：**
   - `FROM alpine:latest`: 指定基础镜像为 Alpine Linux 最新版
   - `RUN`: 在镜像构建时执行命令
   - `apk update`: 更新软件包索引
   - `apk add figlet`: 安装 figlet 工具
   - `&&\`: 使用 && 连接命令，\用于换行

   **构建镜像：**
   ```bash
   docker build -t alpine-figlet-from-dockerfile .
   ```
   - `build`: 构建镜像命令
   - `-t`: 指定镜像名称和标签
   - `.`: 构建上下文路径（当前目录）

3. **Jupyter Notebook 镜像实例**

详见 `2_dockerfile/jupyter_sample/Dockerfile`

**主要指令解释：**
- `FROM python:3.10-slim`: 使用精简版 Python 基础镜像
- `RUN apt-get update && apt-get install`: 安装系统依赖
- `--no-install-recommends`: 不安装推荐的额外包，减小镜像
- `rm -rf /var/lib/apt/lists/*`: 清理 apt 缓存
- `pip install --no-cache-dir`: 安装 Python 包，不缓存
- `WORKDIR /notebooks`: 设置工作目录
- `COPY sample-notebook.ipynb .`: 复制文件到镜像
- `EXPOSE 8888`: 声明监听端口（文档作用）
- `CMD`: 容器启动时默认执行的命令

**使用方法：**
```bash
docker build -t jupyter-sample jupyter_sample/
docker run -d -p 8888:8888 jupyter-sample
```

4. **Golang 多阶段构建**

**单阶段构建问题：**镜像包含完整 Go 开发环境，体积超过 1GB

**多阶段构建方案** (`2_dockerfile/golang_sample/Dockerfile.multi`)

```dockerfile
# 第一阶段：构建
FROM golang:1.23 AS builder
WORKDIR /app
COPY go.mod main.go ./
ENV CGO_ENABLED=0 GOOS=linux
RUN go mod tidy && go build -ldflags="-w -s" -o server .

# 第二阶段：运行
FROM alpine:latest
ARG PORT=8081
ENV PORT=${PORT}
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/server .
EXPOSE ${PORT}
CMD ["./server"]
```

**关键点：**
- `AS builder`: 为第一阶段命名
- `COPY --from=builder`: 从第一阶段复制文件
- `-ldflags="-w -s"`: 去除调试信息，减小二进制大小
- 最终镜像只有几十 MB

**构建对比：**
```bash
docker build -t golang-demo-single -f golang_sample/Dockerfile.single golang_sample/
docker build -t golang-demo-multi -f golang_sample/Dockerfile.multi golang_sample/
docker images | grep golang-demo  # 对比镜像大小
```

#### Dockerfile 关键指令

- `FROM`: 指定基础镜像
- `RUN`: 构建时执行命令
- `COPY`: 复制文件到镜像
- `ADD`: 复制并自动解压
- `WORKDIR`: 设置工作目录
- `ENV`: 设置环境变量
- `ARG`: 定义构建参数
- `EXPOSE`: 声明端口
- `CMD`: 容器启动默认命令（可被覆盖）
- `ENTRYPOINT`: 容器入口点（不可被覆盖）
- `VOLUME`: 声明挂载点

#### 最佳实践

1. **合并 RUN 指令减少层数**
2. **使用 .dockerignore 忽略文件**
3. **使用具体版本标签**
4. **清理缓存减小镜像**
5. **敏感信息用环境变量**

---

### 第三章：存储管理 (`3_storage/`)

#### 使用场景
学习 Docker 的数据持久化方案，确保容器删除后数据不丢失。

#### 三种存储方式

1. **默认存储**：数据随容器删除而丢失
2. **Bind Mount**：挂载主机目录，适合开发
3. **Volume**：Docker 管理的存储，适合生产

#### 实战案例：MySQL 数据库持久化

```bash
# 创建卷
docker volume create mysql_data

# 运行 MySQL
docker run -d \
  --name mysql_db \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

# 创建测试数据
docker exec -it mysql_db mysql -uroot -pmysecret -h127.0.0.1
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'John Doe');
exit

# 删除容器
docker rm -f mysql_db

# 重新启动，数据保留
docker run -d \
  --name mysql_db2 \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

# 验证数据
docker exec -it mysql_db2 mysql -uroot -pmysecret -e "USE test_db; SELECT * FROM users;"
```

**关键点：**
- `-e`: 设置环境变量
- `-v mysql_data:/var/lib/mysql`: 卷挂载

---

### 第四章：网络管理 (`4_network/`)

#### Docker 网络类型

1. **Bridge 网络（默认）**
   - 默认 bridge：不支持容器名解析
   - 自定义 bridge：支持 DNS，推荐使用

2. **Host 网络**
   - 直接使用主机网络
   - 最佳性能，无隔离

3. **None 网络**
   - 禁用网络
   - 适合批处理任务

#### 实战案例：Web + Redis 通信

**应用代码** (`4_network/web-app/app.py`)：
```python
from flask import Flask
import redis
import socket

app = Flask(__name__)
redis_client = redis.Redis(host='redis-server', port=6379)

@app.route('/')
def hello():
    count = redis_client.incr('hits')
    hostname = socket.gethostname()
    return f'Hello! I have been seen {count} times.\nHostname: {hostname}\n'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**部署流程：**
```bash
# 构建镜像
docker build -t web-app web-app

# 创建网络
docker network create my-bridge-network

# 启动服务
docker run -d --name redis-server --network my-bridge-network redis:alpine
docker run -d --name web-app --network my-bridge-network -p 5000:5000 web-app

# 测试
curl http://localhost:5000
```

**关键点：**
- 通过容器名 `redis-server` 连接
- 自定义网络自动 DNS 解析

---

### 第五章：Docker Compose (`5_compose/`)

#### 为什么需要 Docker Compose？

多容器应用管理复杂：
- 多个镜像构建
- 网络配置
- 存储配置
- 依赖关系
- 启动顺序

**Docker Compose 解决：**
- YAML 文件定义所有服务
- 一条命令启动应用

#### docker-compose.yml 详解

```yaml
services:
  nginx:
    build: ./nginx
    ports:
      - "8080:80"
    depends_on:
      - frontend
      - backend
    
  frontend:
    build: ./frontend
    expose:
      - "3000"
    environment:
      - REACT_APP_API_URL=/api
    depends_on:
      - backend
    volumes:
      - ./frontend:/app
      - /app/node_modules
    
  backend:
    build: ./backend
    expose:
      - "3001"
    environment:
      - MONGODB_URI=mongodb://mongodb:27017/todos
    depends_on:
      - mongodb
    volumes:
      - ./backend:/app
      - /app/node_modules

  mongodb:
    image: mongo:6
    expose:
      - "27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

**关键配置：**
- `build` vs `image`: 构建 vs 使用现有镜像
- `ports` vs `expose`: 暴露到主机 vs 仅容器网络
- `depends_on`: 定义启动顺序
- `environment`: 环境变量
- `volumes`: 数据持久化

#### 常用命令

```bash
docker compose up -d        # 后台启动
docker compose ps           # 查看状态
docker compose logs -f      # 查看日志
docker compose down         # 停止并删除
docker compose restart      # 重启服务
```

---

### 第六章：容器监控与管理 (`6_management/`)

#### 资源监控

```bash
docker stats                           # 实时监控
docker inspect <container_id>          # 详细信息
docker top <container_id>              # 查看进程
docker logs -f <container_id>          # 查看日志
```

#### Portainer 可视化管理

```bash
# 创建卷
docker volume create portainer_data

# 运行 Portainer
docker run -d -p 9000:9000 \
    --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:2.29.2
```

**关键点：**
- `-v /var/run/docker.sock:/var/run/docker.sock`: 
  - 挂载 Docker socket
  - 允许 Portainer 控制 Docker

**主要功能：**
- 容器管理
- 镜像管理
- 网络配置
- 卷管理
- 日志查看
- Web 终端

---

### 扩展内容 (`extended/`)

#### 最小化镜像

```dockerfile
FROM scratch                # 空镜像
COPY hello /
CMD ["/hello"]
```

**构建：**
```bash
# Go 静态编译
CGO_ENABLED=0 GOOS=linux go build -o hello hello.go

# 构建镜像（约 2MB）
docker build -t minimal:v1 .
```

---

### 实战项目：RAG 智能问答系统 (`project/`)

#### 技术栈

- 前端：Next.js
- 后端：Python
- LLM：DeepSeek/OpenAI
- 向量数据库：ChromaDB
- 关系型数据库：MySQL
- 对象存储：MinIO
- 编排：Docker Compose

#### RAG 工作流程

1. 用户提问
2. 问题向量化
3. 向量数据库检索
4. 构建增强提示
5. LLM 生成答案
6. 返回答案和来源

#### 项目任务

1. 编写各服务 Dockerfile
2. 编写 docker-compose.yml
3. 配置网络通信
4. 配置数据持久化
5. 实现一键部署

---

## 常见问题 FAQ

### 1. 容器启动后立即退出？
容器需要前台进程，使用 `-it` 或 `-d` 配合持久命令

### 2. 无法访问容器服务？
检查端口映射、网络配置、防火墙

### 3. 镜像构建缓慢？
使用国内源、.dockerignore、构建缓存

### 4. 容器间无法通信？
使用自定义 bridge 网络

### 5. 数据丢失？
使用 Volume 或 Bind Mount 持久化

---

## 最佳实践

### Dockerfile
- 使用官方镜像
- 明确版本标签
- 合并 RUN 指令
- 清理缓存
- 多阶段构建

### 容器运行
- 命名容器
- 限制资源
- 持久化数据
- 健康检查
- 自定义网络

### 生产部署
- 使用编排工具
- 日志监控
- 定期备份
- 安全扫描
- 私有仓库

---

## 学习路径

### 初学者（1-2周）
- Docker 基础
- Dockerfile 编写

### 进阶（3-4周）
- 存储管理
- 网络配置

### 高级（5-6周）
- Docker Compose
- 监控管理

### 实战（7-8周）
- RAG 项目
- 自己项目容器化

---

## 参考资源

- [Docker 官方文档](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile 最佳实践](https://docs.docker.com/develop/dev-best-practices/)

---

祝您学习愉快！🎉
