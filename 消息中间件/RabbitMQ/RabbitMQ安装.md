# RabbitMQ 安装（centOS7）

1. 下载 erlong环境

   ```linux
   wget  http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm
   ```

2. 下载rabbitmq-server

   ```
   wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el7.noarch.rpm
   ```

3. 安装erlong

   ```
   rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm
   ```

   检查是否安装成功 

   ```
   [root@localhost ~]# erl
   Erlang/OTP 19 [erts-8.0.3] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]
   
   Eshell V8.0.3  (abort with ^G)
   1> 12+3.
   15
   1> halt().
   [root@localhost ~]# la
   
   ```

4.  安装rabbitmq

   ```
   rpm -ivh rabbitmq-server-3.6.6-1.el7.noarch.rpm
   ```

   在安装时会提示需要socat的依赖，所以先安装socat,再执行上一步.

   ```
   yum install socat
   ```

   至此安装完毕

5.  启动，关闭

   ```
   systemctl start rabbitmq-server # 启动
   systemctl stop rabbitmq-server # 关闭
   systemctl status rabbitmq-server # 查看状态
   ```

6. 使用管理命令

   ```
   cd /sbin 
   查看插件列表：./rabbitmq-plugins list
   查看rabbitmq状态：./rabbitmqctl status
   添加用户 ：./rabbitmqctl add_user admin admin
   设置用户角色：./rabbitmqctl set_user_tags admin administrator
   列出所有用户：./rabbitmqctl list_users
   启用rabbitmq管理控制台：rabbitmq-plugins enable rabbitmq_management
   
   ```

7. 关于防火墙开放端口

   ```
   # 开放端口
   firewall-cmd --zone=public --add-port=15672/tcp --permanent
   #重启防火墙
   systemctl restart rabbitmq-server
   ```

8. 访问管理控制平台
   在浏览器输入 <http://10.6.51.230:15672/>

   管理控制台访问默认端口15672

   Admin授权

    

   ![img](file:///C:\Users\admi\AppData\Local\Temp\ksohtml3592\wps1.jpg) 

   单击上图中admin用户，进入下图

    

   ![img](file:///C:\Users\admi\AppData\Local\Temp\ksohtml3592\wps2.jpg) 

   单击上图中 Set permission

