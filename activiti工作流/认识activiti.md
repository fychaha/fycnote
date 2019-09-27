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

3. 部署流程定义的两种方法
   读取单个流程定义文件

   ```java
     @Test
       public void test() {
           //得到流程引擎对象，这里是拿到默认的流程引擎
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           //创建流程部署对象
           DeploymentBuilder deploymentBuilder = processEngine.getRepositoryService().createDeployment();
   
           //读取单个指定的流程定义文件
           deploymentBuilder.addClasspathResource("/process/test.bpmn");
           deploymentBuilder.addClasspathResource("/process/test.png");
           Deployment deployment = deploymentBuilder.deploy();//部署流程
       }
   ```

   使用zip部署,将test.bpmn和test.png放到同一个zip压缩包中。(建议使用这种方法)

   ```java
   @Test
       public void test() {
           //使用默认配置文件创建流程引擎
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           //创建流程部署对象
           DeploymentBuilder deploymentBuilder = processEngine.getRepositoryService().createDeployment();
   
           //读取ZIP压缩文件
           //读取资源文件
           ZipInputStream zipInputStream = new ZipInputStream(this.getClass().getClassLoader().getResourceAsStream("/process/process.zip"));
           deploymentBuilder.addZipInputStream(zipInputStream);
           deploymentBuilder.name("请假流程部署");//设置流程定义名称
           Deployment deployment1 = deploymentBuilder.deploy();//部署流程
       }
   ```

4. 流程变量

   ```java
   
   /**
   	 * 设置流程变量数据
   	 */
   	@Test
   	public void setVariableValues(){
   		TaskService taskService=processEngine.getTaskService();//获取任务
   		String taskId="";//任务id知道是哪个人物，设置流程变量.
   		//下面设置任务的内容，比如请假流程
   		taskService.setVariable(taskId, "days", 2);//请假天数
   		taskService.setVariable(taskId, "date", new Date());//请假日期
   		taskService.setVariable(taskId, "reason", "发烧");//请假原因
   		
   		//下面我们再测试一个额外的知识点，就是流程传输变量，这里我们再新建一个student对象，对象有id 和name两个属性,还有就是序列化传输
   		Student student=new Student();
   		student.setId(1);
   		student.setName("zhangsan");
   		taskService.setVariable(taskId, "student",student);//序列化对象
   		taskService.complete(task.getId(),variables);//完成任务
   		
   	}
   ```

   使用map一次设置多个流程变量

   ```java
   @Test
   	public void setVariableValue1(){
   		TaskService taskService=processEngine.getTaskService();//获取任务
   		String taskId="25004";//更加任务id知道是哪个人物，设置流程变量。可以更加查看任务方法查看任务的id，可以到数据库直接看
   		//下面设置任务的内容，比如请假流程，任务的第一节点也就是申请人要写请假的原因
   //		taskService.setVariable(taskId, "days", 2);//请假天数
   //		taskService.setVariable(taskId, "date", new Date());//请假日期
   //		taskService.setVariable(taskId, "reason", "faShao");//请假原因
   		
   		//下面我们再测试一个额外的知识点，就是流程传输变量，这里我们再新建一个student对象，对象有id 和name两个属性,还有就是序列化传输
   		Student student2=new Student();
   		student2.setId(1);
   		student2.setName("zhangsan");
   		taskService.setVariable(taskId, "student",student2);//序列化对象
   	//创建流程变量map
   		Map<String, Object> variables=new HashMap<String,Object>();
   		variables.put("days", 3);
   		variables.put("date",new Date());
   		variables.put("reason", "faShao2");
   		variables.put("student", student2);
   		taskService.setVariables(taskId, variables);
   		taskService.complete(task.getId(),variables);
   	}
   ```

5. 排他网关
   即单线流程

6. 并行网关

   ​    注意：此时整个流程有两个执行实例，这两个实例又属于整个流程实例，所以execution表里有三条数据（父流程实例，与两个执行实例）

   

   如图：

   ![1569413630534](C:\Users\admi\AppData\Roaming\Typora\typora-user-images\1569413630534.png)

   

7. 启动流程

   ```java
   public void startProcess(){
       RuntimeService runtimeService =processEngine.getRuntimeService();
       runtimeService.startProcessInstanceById("myProcess:1:4", bussinessKey);//用流程id启动
   }
   ```

8. 接收任务（receiveTask,等待活动）

   说明：与用户任务（UserTask）不同的是，接收任务（ReceiveTask）创建后，会进入一个等待状态，一般指机器自动完成，但需要耗费一定时间的工作，当完成工作后后，向后推移流程。

   与UserTask不同的还有，在task表中是没有数据的

   //部署省略。。。。。

   启动流程

   ```java
   //3.查询执行对象表,使用流程实例ID和当前活动的名称（receivetask1）
   		String processInstanceId = pi.getId();//得到流程实例ID
   				//拿到流程实例
   		Execution execution1 = processEngine.getRuntimeService()
   				.createExecutionQuery()
   				.processInstanceId(processInstanceId)//流程实例ID
   				.activityId("receivetask1")//当前活动的id
               String value="流程变量";//这个数据在数据库中查找，是个耗时操作。
           runtimeService.setVariable(execution1.getId(),"流程变量",value)
   ```

   之后向后执行一步,如果流程处于等待状态，使流程继续进行

   ````java
   runtimeService.signal(execution1.getId())
   ````

   此时活动节点到了第二个任务,执行第二个任务

   直到该任务完成后再向后执行一步

   ````java
   //第二个任务代码略
   runtimeService.signal(execution1.getId());
   ````

9. 组任务

   

   任务拾取（指定该任务的办理人），实际开发中由上级执行，指派办理，为他们拾取任务。被指派者去执行任务。

   ```java
   @Test
   public void claim(){
       String taskId="任务id";
       String person="办理人";
        taskService.claim(taskId,person);
   }
   ```

   任务拾取后，在indentity表中，该办理人的type属性会从"candidate"()被修改为"assignee"

   

   任务回退(重新指定办理人)

   ```java
   @Test
   public void claim(){
       String taskId="任务id";
       String person="办理人";
        taskService.setAssidnee(taskID,null);//把之前在task表里设定的办理人清空。
       //之后再重新拾取
   }
   ```

10. 用户与用户组
    创建用户与用户组

    ```java
    //newGroup方法创建Group实例
    Group group = identityService.newGroup("1");
    group.setName("经理组");
    group.setType("manager");
    // 自定义方法保存用户组
    public void createGroup(IdentityService identityService, String id,String name, String type) {
      Group group = identityService.newGroup(id);
      group.setName(name);
      group.setType(type);
      identityService.saveGroup(group);}
    createGroup(identityService, "1", "经理组", "typeManager");
    ```

    