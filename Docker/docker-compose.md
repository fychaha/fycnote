# docker compose

- 前面我们使用 Docker 的时候，定义 Dockerfile 文件，然后使用 docker build、docker run 等命令操作容器。然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，那么效率之低，维护量之大可想而知
- **使用 Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具**

#### docker compose 安装

下载docker-compose

````bash
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
````

赋予权限

````bash
sudo chmod +x /usr/local/bin/docker-compose
````

- 查看安装是否成功

```bash
docker-compose -v
```

### 快速入门

- 打包项目，获得 jar 包 docker-demo-0.0.1-SNAPSHOT.jar

```go
mvn clean package
```

- 在 jar 包所在路径创建 Dockerfile 文件，添加以下内容

```bash
FROM java:8
VOLUME /tmp
ADD docker-demo-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
EXPOSE 9000
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
```

- 在 jar 包所在路径创建文件 docker-compose.yml，添加以下内容

```bash
version: '2' # 表示该 Docker-Compose 文件使用的是 Version 2 file
services:
  docker-demo:  # 指定服务名称
    build: .  # 指定 Dockerfile 所在路径
    ports:    # 指定端口映射
      - "9000:8761"
```

- 在 docker-compose.yml 所在路径下执行该命令 Compose 就会自动构建镜像并使用镜像启动容器

```cpp
docker-compose up
docker-compose up -d  // 后台启动并运行容器
```