## LinkList简介

LinkList是一个实现了List接口和Deque接口，使得LinkList底层的链表结构使他支持高效地插入和删除操作。，同时也有队列的特性。LinkList不是线程安全的，不过可以调用静态类Collections类中的synchronizeList方法：

`````java
List list = Collections.synchronizeList(new LinkList());
`````

## <font face="楷体" id="2">内部结构分析</font>

**如下图所示：**
![LinkedList内部结构](https://user-gold-cdn.xitu.io/2018/3/19/1623e363fe0450b0?w=600&h=481&f=jpeg&s=18502)
看完了图之后，我们再看LinkedList类中的一个<font color="red">**内部私有类Node**</font>就很好理解了：

```java
private static class Node<E> {
        E item;//节点值
        Node<E> next;//后继节点
        Node<E> prev;//前驱节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

这个类就代表双端链表的节点Node。这个类有三个属性，分别是前驱节点，本节点的值，后继结点。