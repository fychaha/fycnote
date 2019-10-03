## 部署springboot项目jar包

1. 准备一个springboot项目，打成jar包，设置好该项目的访问端口号，这里的例子用8721作为端口

   ```properties
   server.port=8721
   ```

2. 生产机（centos）上随意某处创建一个文件夹用来存放jar包和对应的dockfile文件。

   结构如图：

   ![1569828536922](C:\Users\admi\AppData\Roaming\Typora\typora-user-images\1569828536922.png)

3. 创建一个dockerfile （也可以是Dockerfile）的文件，进去进行配置

   ```dockerfile
   vim dockerfile
   ```

   进行如下配置:

   ````dockerfile
   FROM java:8
   COPY ./docker-0.0.1-SNAPSHOT.jar fycapp.jar
   EXPOSE 8721
   CMD java -jar fycapp.jar
   ````

   ##### from:表示要在哪个环境镜像的基础上创建你的项目镜像，就是你的项目依赖的环境的整合镜像，由于我这里只依赖java环境，所以只用java：8这个镜像。

   ##### copy: ./表示当前目录下,选中你复制进来的jar包， 后面再写你自定义的名字jar包

   ##### expose : 表示你要创建的容器的出口端口，与你的前面配置的项目端口一致.

   ##### cmd: 表示启动容器时他帮你执行的命令。

   ##### 其他配置略，这里只了解最基础的配置。

4. 准备好后创建镜像

   ```
   docker build -t fycapp
   ```

5. 创建容器，并运行

   ```
    docker run -d -p 8721:8721 fycapp
   ```

   -p: 指定你的端口，前面8721为你的容器内部开放的端口，后者为你外部访问的端口，可以任意，容器会自动帮你的端口映射成8721.

6. 做完以上工作，就可以在网页上访问啦~