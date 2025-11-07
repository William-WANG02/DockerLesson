## 实战：查看 RootFS 内容

方法1：直接查看容器文件系统

```bash
# 运行Ubuntu容器
docker run -it --name test ubuntu:20.04 bash

# 在容器内查看RootFS
ls -la /
drwxr-xr-x   1 root root 4096 Jan 1 00:00 bin   ← RootFS内容
drwxr-xr-x   2 root root 4096 Jan 1 00:00 boot  ← 空目录（无BootFS）
drwxr-xr-x   5 root root  360 Jan 1 00:00 dev   ← 虚拟设备
drwxr-xr-x   1 root root 4096 Jan 1 00:00 etc
drwxr-xr-x   2 root root 4096 Jan 1 00:00 home
drwxr-xr-x   1 root root 4096 Jan 1 00:00 lib
...

# 注意：/boot 目录是空的！
ls /boot/
# 输出为空，因为没有BootFS
```

方法2：对比宿主机内核

```bash
# 宿主机内核版本
uname -r
5.4.241-1-tlinux4-0023.2

# 容器内内核版本（与宿主机相同）
docker run ubuntu:20.04 uname -r
5.4.241-1-tlinux4-0023.2  ← 共享宿主机内核！

# 容器的RootFS版本
docker run ubuntu:20.04 cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"  ← 容器的RootFS
```

## 实战：创建最小化 Docker 镜像

在`/extended` 目录下
```dockerfile
# 使用 scratch（空镜像）
FROM scratch

# 只复制单个可执行文件（静态编译）
COPY hello /

# 运行
CMD ["/hello"]
```

编译静态二进制：
```bash
# Go 语言示例（天然支持静态编译）
cat > hello.go <<EOF
package main
import "fmt"
func main() {
    fmt.Println("Hello from minimal rootfs!")
}
EOF

# 静态编译
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o hello hello.go

# 构建镜像
docker build -t minimal:v1 .

# 查看大小
docker images minimal
REPOSITORY   TAG    SIZE
minimal      v1     2MB  ← 只有可执行文件！
```

这个镜像：
- ❌ 没有完整的RootFS
- ❌ 没有/bin, /lib, /etc
- ✅ 只有一个可执行文件
- ✅ 依然共享宿主机BootFS


