## activiti的特性

1. 七大service接口
   这七个接口是整个的activiti的核心,利用他们对对应的表进行操作。

| 接口名            | 作用                                                         |
| :---------------- | ------------------------------------------------------------ |
| RepositoryService | 流程仓库Service，用于管理流程仓库，例如：部署，删除，读取流程资源 |
| IdentityService   | 身份Service，可以管理，查询用户，组之间的关系 查询任务的候选组/人 |
| HistoryService    | 历史Service，可以查询所有历史数据，例如：流程实例，任务，活动，变量，附件等 |
| RuntimeService    | 运行时Service，可以处理所有正在运行状态的流程实例，任务等    |
| TaskService       | 任务Service，用于管理、查询任务，比如查询某人待办任务，完成某个任务等 |
| FormService       | 表单Service，用于读取和流程，任务相关的表单数据              |
| ManagementService | 引擎管理Service，和具体业务无关，主要是可以查询引擎配置，数据库，作业等 |



2. 涉及的表

   ```sql
   #RepositoryService
   select * from `act_ge_bytearray`;#二进制文件表
   select * from `act_re_deployment`;#流程部署表
   select * from `act_re_procdef`;#流程定义
   select * from `act_ge_property`;#工作流的ID算法和版本信息表
   
   # RuntimeService TaskService
   select * from `act_ru_execution`;#流程启动一次只要没有执行完，就会有一条数据
   select * from `act_ru_task`;#可能会有多条数据
   select * from `act_ru_variable`;#记录流程运行时的流程变量
   select * from `act_ru_identitylink`;#存放流程办理人的信息
   
   # HistoryService
   select * from `act_hi_procinst`;#历史流程实例
   select * from `act_hi_taskinst`;#历史任务实例
   select * from `act_hi_actinst`;#历史活动节点数
   select * from `act_hi_varinst`;#历史流程变量表
   select * from `act_hi_identitylink`;#历史办理人数
   select * from `act_hi_comment`;#批注表
   select * from `act_hi_attachment`;#附件表
   ```

   