## HashMap

[https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/](https://yikun.github.io/2015/04/01/Java-HashMap工作原理及实现/)

- 扩容是2倍，需不需要修改索引看这个数字在扩容位置是1还是0,0则不需要，1则需要（原位置+gap）
- 冲突则先链表，如果链表超过阈值则转换为红黑树

### hash函数的实现

可以看到这个函数大概的作用就是：高16bit不变，低16bit和高16bit做了一个异或。



### 总结

**以Entry[]数组实现的哈希桶数组，用Key的哈希值取模桶数组的大小可得到数组下标。**

插入元素时，如果两条Key落在同一个桶（比如哈希值1和17取模16后都属于第一个哈希桶），我们称之为哈希冲突。

JDK的做法是链表法，Entry用一个next属性实现多个Entry以单向链表存放。查找哈希值为17的key时，先定位到哈希桶，然后链表遍历桶里所有元素，逐个比较其Hash值然后key值。

在JDK8里，新增默认为8的阈值，**当一个桶里的Entry超过閥值，就不以单向链表而以红黑树来存放以加快Key的查找速度。**

当然，最好还是桶里只有一个元素，不用去比较。所以默认当Entry数量达到桶数量的75%时，哈希冲突已比较严重，就会成倍扩容桶数组，并重新分配所有原来的Entry。扩容成本不低，所以也最好有个预估值。

**取模用与操作（hash & （arrayLength-1））会比较快，所以数组的大小永远是2的N次方**， 你随便给一个初始值比如17会转为32。默认第一次放入元素时的初始值是16。

iterator（）时顺着哈希桶数组来遍历，看起来是个乱序



## concurrentHashMap原理

HashMap是线程不安全的，HashTable是线程安全的（synchronize）。

而HashTable是锁整个map，性能低。

Java1.5之后，concurrentHashMap诞生，分段锁，每个segment都会锁

Java1.8之后，concurrentHashmap使用链表+红黑树，只锁第一个节点**（Hashmap多大，性能提高多少倍）**



## HashMap的节点问题

在查看java1.8之后的版本的源码可以发现，以hashmap的put为例：

put调用putVal方法，里面有的逻辑是：

```java
if (p.hash == hash &&   //key的位置上没有entry
    ((k = p.key) == key || (key != null && key.equals(k))))  
    e = p;
else if (p instanceof TreeNode)  //如果位置上是一个TreeNode
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
else {  //否则按照链表插入
    for (int binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            break;
        }
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        p = e;
    }
}
```

根据上面就会有一个疑问：**这个node到底是什么结构，为什么又是节点又是链表又是TreeNode的？**

这可以看node的结构：

```java
static class Node<K,V> implements Map.Entry<K,V> {  //node是实现了Map的Entry
    final int hash;
    final K key;
    V value;
    Node<K,V> next;  //而且本身就是一个链表
    ...
}
```

那TreeNode的结构呢，我们可以查看TreeNode的结构：

```java
//继承自LinkedHashMap的Entry，将其改为红黑树实现
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    ...
}
```

再看LinkedHashMap.Entry的实现：

```java
//直接继承自HashMap的Node，加上before和after的引用而已
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

会发现回到了HashMap的Node上了。

**总结：TreeNode -> LinkedHashMap.Entry -> HashMap.Node -> Map.Entry**，就是这么个继承关系，所以直接可以直接判断HashMap上的节点是不是instanceof TreeNode，如果不是，那么就是普通的链表了。



## hashmap的hash

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

上面是java8中的hash值的求法，这就是 **扰动函数，32位的hashcode，高16位和低16位异或**，Java8之前采用的是4次异或。优点就是 **低位也能有高位的信息，能够更好的混淆多个对象**

而在获得这个hash值之后是通过右移来获取index的： `tab[i = (n - 1) & hash]`，所以使用2 的幂。