# Hash表简介

哈希表是基于哈希值的桶和链表。由于底层是由数组实现的，所以增删改的复杂度为O(1)，即与数据规模（多少）无关。但是hash表有个缺陷，就是可能会发生hash值的碰撞。