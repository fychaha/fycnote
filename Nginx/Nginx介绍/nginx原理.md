# nginx原理

niginx 在启动后，有两个进程，master和worker

查看nginx进程

`````bash
ps -ef | nginx
`````

这代表nginx启动后有这两部分。master就像是个领导，管理与监控手下的worker。worker来执行任务。

具体工作流程是这样的：client（客户端）向nginx发送请求， 请求会先到master，master再通知worker有任务，然后worker再通过争抢机制得到任务，得到任务的worker可以进行反向代理，完成任务。

一个master和多个worker这样的设计有什么好处呢？

热部署,不需要重新启动，就可以生效修改完的配置

````bash
nginx -s reload
````

