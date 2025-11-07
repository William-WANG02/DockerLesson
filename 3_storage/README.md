# Docker 存储管理详解

Docker 容器在运行时会产生大量数据，这些数据如何持久化和管理是一个重要的话题。
本节我们将通过一个 Nginx Web 服务器的案例，来深入探讨 Docker 的三种数据管理方式。

## Docker 存储基础

Docker 提供了三种主要的数据管理方式：

1. **默认存储**：容器内的数据随容器删除而丢失
2. **Bind Mounts（绑定挂载）**：将主机上的目录或文件直接挂载到容器中
3. **Volumes（卷）**：由 Docker 管理的持久化存储空间，完全独立于容器的生命周期

让我们通过一个 Nginx Web 服务器的例子来理解这三种方式的区别。我们将在每种方式下执行相同的操作：创建一个 HTML 文件，然后测试数据的持久性。

### 场景一：默认存储（非持久化）

在这个场景中，我们直接在容器内创建文件，看看数据会发生什么：

```shell
# 运行一个 nginx 容器
docker run -d --name web-default -p 8000:80 nginx

# 在容器中创建一个测试页面
docker exec -it web-default /bin/sh
echo "<h1>Hello from Default Storage</h1>" > /usr/share/nginx/html/index.html
exit

# 访问页面验证内容
curl http://localhost:8000

# 容器中文件目录获取
# 背景介绍：
#  容器目录是在宿主机目录下的一个子目录，通过chroot的linux技术实现；
#  容器的目录对宿主机隔离，一般禁止访问；
# 该命令用于获取 Docker 容器在宿主机上的实际存储路径。当容器使用联合文件系统（如 overlay2）时，容器文件系统由多个层组成， MergedDir 就是这些层最终合并后在宿主机上的实际目录路径
docker inspect -f '{{.GraphDriver.Data.MergedDir}}' <container_id>
cat 容器目录/usr/share/nginx/html/index.html

# 删除容器
docker stop web-default
docker rm web-default

# 用同样的配置重新运行容器
docker run -d --name web-default-2 -p 8000:80 nginx

# 再次访问页面，内容不存在
curl http://localhost:8000
```

### 场景二：使用 Bind Mount

在这个场景中，我们将主机上的目录直接挂载到容器中：

```shell
# 创建本地目录
mkdir nginx-content

# 运行 Nginx 容器并挂载本地目录
docker run -d --name web-bind \
   -p 8081:80 \
   -v $(pwd)/nginx-content:/usr/share/nginx/html nginx

# 在容器中创建一个测试页面
docker exec -it web-bind sh -c 'echo "<h1>Hello from Bind Mounts</h1>" > /usr/share/nginx/html/index.html'

# 访问页面验证内容
curl http://localhost:8081

# 删除容器
docker rm -f web-bind

# 用同样的配置重新运行容器
docker run -d --name web-bind-2 -p 8081:80 \
   -v $(pwd)/nginx-content:/usr/share/nginx/html nginx

# 再次访问页面，内容仍然存在
curl http://localhost:8081
```

### 场景三：使用 Volume

在这个场景中，我们使用 Docker 管理的卷来存储数据：

```shell
# 创建一个 Docker volume
docker volume create nginx_data

# 运行 Nginx 容器并挂载卷
docker run -d --name web-volume -p 8082:80 -v nginx_data:/usr/share/nginx/html nginx

# 在容器中创建一个测试页面
docker exec -it web-volume sh -c 'echo "<h1>Hello from Volume Storage</h1>" > /usr/share/nginx/html/index.html'

# 访问页面验证内容
curl http://localhost:8082

# 删除容器
docker rm -f web-volume

# 用同样的配置重新运行容器
docker run -d --name web-volume-2 -p 8082:80 \
   -v nginx_data:/usr/share/nginx/html nginx

# 再次访问页面，内容仍然存在
curl http://localhost:8082

# 查看卷的详细信息
docker volume ls
docker volume inspect nginx_data
```

### 三种方式的对比

1. **默认存储**
   - 数据随容器删除而丢失
   - 适合存储临时数据
   - 容器间数据隔离
   - 无需额外配置

2. **Bind Mount**
   - 数据持久化，存储在主机指定位置
   - 可以直接在主机上修改文件
   - 不足：目录权限不对等，有安全风险
   - 不足：依赖主机文件系统结构

3. **Volume**
   - 数据持久化，独立于容器生命周期
   - 数据存储在 Docker 管理区域，安全性好

### 清理操作

完成实验后，可以进行清理：

```shell
# 清理容器
docker rm -f web-default web-volume web-volume-2 web-bind web-bind-2

# 清理卷
docker volume rm nginx_data

# 清理本地目录
rm -rf nginx-content
```






## 实践案例：使用 Bind Mount 运行 DeepSeek-R1

在这个案例中，我们将演示如何使用 Bind Mount 来运行 DeepSeek-R1 大语言模型。
这个案例很好地展示了 Bind Mount 在处理大型文件时的优势：模型文件通常很大（几十GB），
如果打包到镜像中会导致镜像体积过大，而使用 Bind Mount 可以直接挂载主机上的模型文件。

### 运行 ollama 容器

```shell
# 查看本地的模型文件是否存在
ls -l deepseek-models

# 运行 ollama 容器
docker run \
    --name deepseek-r1 \
    -it -d \
    -v $(pwd)/deepseek-models:/root/.ollama/models \
    ollama/ollama

# 查看容器状态
docker ps
docker logs deepseek-r1
```

### 运行 deepseek r1 模型

``` shell
# 查看模型是否被识别
docker exec -it  deepseek-r1 ollama ls
# 运行 deepseek r1 模型
docker exec -it  deepseek-r1 ollama run deepseek-r1:8b-0528-qwen3-q8_0
```
>   💡 `ollama run deepseek-r1:8b-0528-qwen3-q8_0` 用于在本地部署并运行 DeepSeek-R1 系列的 8B 参数版本模型。





## 实践案例：使用 Volume 部署 MySQL 数据库

我们将通过一个 MySQL 数据库的例子来演示如何使用 Volume 持久化数据。

### 创建并管理 Volume

```shell
# 创建一个数据卷，名称为 mysql_data
docker volume create mysql_data

# 列出所有卷
docker volume ls

# 查看卷信息
docker volume inspect mysql_data
```

### 运行 MySQL 容器,并挂载卷

```shell
# 运行 MySQL 容器并挂载卷
# 备注：MYSQL_ROOT_PASSWORD 是环境变量，用于设置 MySQL 的 root 用户密码
docker run -d \
  --name mysql_db \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

# 进入容器创建测试数据
docker exec -it mysql_db mysql -uroot -pmysecret -h127.0.0.1

# 在 MySQL 中创建测试数据
mysql> CREATE DATABASE test_db;
mysql> USE test_db;
mysql> CREATE TABLE users (id INT, name VARCHAR(50));
mysql> INSERT INTO users VALUES (1, 'John Doe');
mysql> exit
```

### 验证数据持久化

```shell
# 删除原容器
docker rm -f mysql_db

# 使用同一个卷启动新容器
docker run -d \
  --name mysql_db2 \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

# 验证数据是否存在
docker exec -it mysql_db2 \
   mysql -uroot -pmysecret -e "USE test_db; SELECT * FROM users;"
```

