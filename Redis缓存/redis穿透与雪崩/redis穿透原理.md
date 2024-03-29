## redis穿透原理

​	一般情况下，客户端发送请求，先去查询redis，判断是否存在，如何存在就直接执行业务逻辑。如果不存在，才去查询数据库，然后缓存到redis，并执行业务逻辑。

​	此时有一种情况，假设在数据库中某id字段只有1,2,3三条数据，而客户端那里发送请求要查询id为4的数据，这时按照正常流程，客户端先去redis查数据，没查到，然后再去数据库里查询，也没查到，返回结果为null。这种情况下，如果有个黑客想要攻击你的数据库，他可以让客户端不断地发送请求去查询你数据库不可能存在的数据，那么这每一条请求都会去访问数据库，绕过了redis，造成数据库的雪崩。这样的现象称为redis穿透。

那么如何解决穿透现象？

​	在使用key查询数据库无果后，用对应的key存入redis中，并赋值为""(空字符串或者null)，这样redis内有了这样一个key的值，那么当客户端再次请求查询这个key时，，就会在redis内查询到"",表示是数据库内没有这个值。

​	但是管事这样还有一个问题，如果数据库更新了这个key的值，那么再去查这个key，客户端在这个key从redis里消失前依然查不到，所以我们需要在每次数据库内有更新操作时，把redis内对应的key做相应的更新。