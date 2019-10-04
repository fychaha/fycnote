## dockerfile脚本编写

###### FROM: 指定基础镜像(相当于父类)，我们要创建的镜像是基于这个镜像的。这个指令必须在第一条，而且是必备的。

###### RUN:是用来执行命令行命令的,格式有两种:

1. shell格式:RUN <命令>，就像在命令行中输入命令一样

   ```dockerfile
   RUN yum install iproute nginx && yum clean all
   ```

2. exec格式(待定)

   ```
   RUN 
   ```

   

###### WORKDIR:指定工作目录，当启动容器时以此目录为工作目录

```dockerfile
WORKDIR /usr/local/
```

##### 构建镜像上下文

  在进行构建镜像时,并非所有制定都是用run指令完成的，经常需要将一些文件复制进镜像，比如copy与add指令，而docker build 命令构建镜像时，其实并非在本地构建，而是在服务端，也就是时在docker引擎中构建的，在这种c-s架构中，如何才能让服务器获得本地构建文件呢？

​	这就需要引入上下文这个概念，当构建时，用户指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下所有内容全部打包，然后上传给docker引擎。这样docker引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

```dockerfile
CPOY ./package.json/app/
```

这并不是要复制执行docker build 命令所在的目录下的package.json,也不是复制dockerfile所在目录下的package.json,而是复制上下文（context）目录下的package.json.

因此，copy这类指令中的源文件的路径都是相对路径，如果并不是的话，docker引擎就找不到文件。

##### EXPOSE:暴露端口





