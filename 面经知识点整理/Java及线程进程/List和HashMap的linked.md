## ArrayList和LinkedList

通常情况下，ArrayList和LinkedList的区别有以下几点：

1. ArrayList是实现了基于动态数组的数据结构，而LinkedList是基于链表的数据结构；

2. 对于随机访问get和set，ArrayList要优于LinkedList，因为LinkedList要移动指针；

3. 对于添加和删除操作add和remove，一般大家都会说LinkedList要比ArrayList快，因为ArrayList要移动数据。但是实际情况并非这样，对于添加或删除，LinkedList和ArrayList并不能明确说明谁快谁慢，



## HashMap和LinkedHashMap

在HashMap的put方法中，会有

`tab[i] = newNode(hash, key, value, null);`

方法，而LinkedHashMap继承自HashMap并重写了newNode方法

最终：

1. LinkedHashMap内是链式结果
2. LinkedHashMap保存着插入顺序