## ListView和RecyclerView的区别

https://www.jianshu.com/p/cd8244d1c19a

|                        | ListView                                                     | RecyclerView                                                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Adapter**            | 1. 继承重写BaseAdapter类；<br /> 2. 自定义ViewHolder与convertView的优化； | 1. 继承重写RecyclerView.Adapter与RecyclerView.ViewHolder <br />2. 设置LayoutManager，以及layout的布局效果 |
| **布局**               | 纵向                                                         | Manager可以设置横纵向，瀑布流                                |
| HeaderView和FooterView | 自带add函数                                                  | 需要自己实现，但是可能会影响其他view的position               |
| **刷新**               | 仅notifyDataSetChanged()                                     | 支持notifyItemChanged()                                      |
| 动画                   | 无                                                           | 有接口可以实现setItemAnimator()                              |
| 点击事件               | 因为Header和Footer会导致position不同，需要特输处理           | addOnItemTouchListener()                                     |
| 事件分发               | 无                                                           | NestedScrollingChild接口（父view和子view可以一起接受事件）   |
| **缓存**               | 二级缓存                                                     | 四级缓存（屏幕外多一层cache，无需再次bind，还有一个pool存储所有的view，需bind） |

### 缓存

#### ListView

![img](https://pic1.zhimg.com/80/v2-8745a78e15b09d7652ef9faa28eb1180_720w.jpg)

#### RecyclerView

四级缓存：

1. mAttachView和mChangedView 
2. mCacheView 
3. ViewCacheExtension（用户自定义缓存）
4.  ViewPool（缓存池）

![img](https://pic2.zhimg.com/80/v2-a8d1b3f8f1d3b96db61ef34b3934c7b9_720w.jpg)