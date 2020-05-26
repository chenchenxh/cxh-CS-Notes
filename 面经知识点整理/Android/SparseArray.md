基本原理： https://juejin.im/entry/57c3e8c48ac24700634bd3cf

和HashMap区别： https://juejin.im/post/5d71fc9651882535db6de6ac

SparseArray优点：

- 不需要装箱和拆箱操作

- 节省空间

- 数据量少的话，插入和查询速度较快



如果是int，long，boolean这类key的话，且数据量少（10000以内）可以考虑使用SparseArray