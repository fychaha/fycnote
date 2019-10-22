# ArrayList

ArrayList的底层是数组队列，相当于动态数组。他的容量能动态增长。

所继承的父类是 AbstractList ，实现了List，RandomAccess，Cloneable，java.io.Serializable这些接口。他是个数组队列，提供了相关的添加，删除，修改，遍历等功能。

ArrayList实现了RandomAccess接口，他是个标志接口，表明实现这个接口的List集合是支持快速访问的，在ArrayList中，我们即可以通过元素的序号快速获取元素对象。这就是快速随机访问。

和Vector不同，ArrayList中的操作不是线程安全的，所以，建议在单线程中才去使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

