## APK文件构建过程

https://www.jianshu.com/p/c8ccf7ffa79e

1. java文件生成

   通过aapt工具生成java文件，如果文件为res目录下的文件和AndroidManifest.xml的文件

   通过aidl工具把.aidl文件生成java文件

2. 通过aapt工具来生成生成资源索引文件

   生成的文件名是resources.ap_

3. 用javac命令编译java文件
   就是使用jdk中的javac工具，做java开发的应该都知道怎么使用。

4. 通过dx工具生成dex文件，dx工具与aapt存放目录一致。

5. 通过apkbuilder打包apk

6. 签名，可以使用jarsigner工具签名和apksigner工具签名

7. zipaligin，它位于/android-sdk/build-tools/$build-tools-version 目录，是一个zip文件整理工具用来优化apk文件。它的主要工作是将apk包进行对齐处理，使apk包中的所有资源文件距离文件起始偏移为4字节整数倍，这样通过内存映射访问apk文件时速度会更快。

## APK文件结构

https://blog.csdn.net/winceos/article/details/37874383

APK文件本质上是一个ZIP文件

- **Manifest文件**

  AndroidManifest.xml是每个应用都必须定义和包含的，它描述了应用的名字、版本、权限、引用的库文件等信息，

- **META-INF目录**

  META-INF目录下存放的是签名信息，用来保证apk包的完整性和系统的安全。

- **classes.dex文件**

  classes.dex是java源码编译后生成的java字节码文件。但由于Android使用的dalvik虚拟机与标准的java虚拟机是不兼容的，dex文件与class文件相比，不论是文件结构还是opcode都不一样。

- **res目录**

  res目录存放资源文件。

## 全局整理ID的原因

在生成apk的时候，会和插件的资源的ID发生冲突，解决冲突的方式就是修改ID

资源的ID生成是有规则的，**规则：0xPPTTNNNN**，由8位16进制组成，其中： 

- PP段：表示资源的包空间：0x01表示系统资源空间，0x7f表示应用资源空间。
-  TT段：表示资源类型。 
- NNNN段：4个16进制表示资源id，一个apk中同一类型资源从0000开始递增。

两种方式修改，一种是编译阶段直接传入ID直接决定一个包的ID（不推荐）

另外一种就是冲突之后后期重新整理ID