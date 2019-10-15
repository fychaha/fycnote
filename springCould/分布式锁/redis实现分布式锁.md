#### Redis 实现分布式锁原理

​	多个客户端（jvm），使用setnx 命令，同时在redis上创建相同的一个key，因为redis 中key是唯一的，只要谁能创建成功就能够获取到锁，没能创建的则等待。

解释：setnx命令作用：类似于set 操作，可以写入key，并返回操作结。但是与set的返回结果形式不同，set在写入已经存在的key时，则会覆盖原来的key，并返回ok。而setnx命令在写入已经存在的key时则会保留原来的key，返回0表写入失败，写入成功时则返回1，这个特性让它能做分布式锁。

````shell
setnx student fyc 
````

 

#### 那么怎么释放锁呢？

在执行完操作后，删除对应得应的key。但是这种方法有很大的缺陷。如果删除操作失败的话，就会发生死锁。为了避免这种现象，要为这个key加上时效。





获取，释放锁代码

获取锁

````java
    //redis线程池
    private JedisPool jedisPool;
    private String lockKey="lock_key";//定义key，多个服务均用这个key
    //获取锁
    public String getRedisLock(Long acquireTimeout, Long timeOut){
        //1.建立redis连接
        //2.定义redis对应key的value(uuid),这个value的作用是在释放锁的时候要用到。
        //3.两个获取锁的超时时间-----
        // 1.在尝试获取锁的时候，等待超时，就放弃。
        // 2.在获取锁之后，对应的key有有效期，对应的key在规定时间内失效。
        //5. 定义两个超时时间
        //6. 使用循环保证重复进行尝试获取锁（乐观锁）
        //7. 使用setnx命令插入key，返回1则获取锁。
        Jedis conn = jedisPool.getResource();
        String value = UUID.randomUUID().toString();
        Long endTime = System.currentTimeMillis()+acquireTimeout;
        while(endTime>System.currentTimeMillis()){
            //获取锁
            if (conn.setnx(lockKey,value)==1){
                conn.expire(lockKey, Math.toIntExact(timeOut));
                return value;
            }
        }
        //超时返回空。
        return null;
    }
````

释放锁

```java
    //释放锁
    public void unRedisLock(String value){
        Jedis conn = jedisPool.getResource();
        //确保删除的是自己生成的key
        if (conn.get(lockKey).equals(value)){
            conn.del(lockKey);
        }

    }

}
```

注意：不能直接用conn.del(key)来删除key，，要保证是删除自己创建的key，分布式中会删除其他人的创建的key，所以要通过唯一的value值来删除。

