[[Android热修复技术原理详解（最新最全版本）](https://www.cnblogs.com/popfisher/p/8543973.html)](https://www.cnblogs.com/popfisher/p/8543973.html)

## 热修复

正常开发

![img](https://images2018.cnblogs.com/blog/823551/201803/823551-20180311132422132-630335315.png)热修复

![img](https://images2018.cnblogs.com/blog/823551/201803/823551-20180311132434276-1651642452.png)

## 框架分类介绍

![img](https://images2018.cnblogs.com/blog/823551/201803/823551-20180313230533833-1720866500.png)

或者这样子分类

![img](https://images2018.cnblogs.com/blog/823551/201803/823551-20180313230542094-924942701.png)

## 技术特点

Dexposed和Andfix都是native层的

- 缺：会导致原有类内容或者结构变化
- 优：速度快，即时生效；可以补上Google的坑



qq空间的Dex插拔技术

- 缺：需要下次启动才生效；性能较差；
- 优：不需要考虑对dalvik虚拟机和art虚拟机做适配



美团的Robust

- 缺点
  - 几乎不会影响性能（方法调用，冷启动）
  - 高兼容性（Robust只是在正常的使用DexClassLoader）、高稳定性，修复成功率高达99.9%
  - 补丁实时生效，不需要重新启动
  - 支持方法级别的修复，包括静态方法
  - 支持增加方法和类
- 优点
  - 代码是侵入式的，会在原有的类中加入相关代码
  - 会增大apk的体积，平均一个函数会比原来增加17.47个字节，10万个函数会增加1.67M。



阿里Sophix

- 优化Andfix，全量合成dex技术优化版本

![img](https://images2018.cnblogs.com/blog/823551/201803/823551-20180311132855847-745121352.png)