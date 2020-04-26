## Jar

- *.jar，JAR 文件就是 Java Archive File，顾名思意，它的应用是与 Java 息息相关的，是 Java 的一种文档格式。只包含了class文件与清单文件 ，不包含资源文件，如图片等所有res中的文件。找一个jar文件，然后修改后缀名为‘zip’或者‘rar’格式，然后解压该文件，打开解压后的文件夹，截图如下所示

![img](https://img-blog.csdn.net/20161010145203181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



- *.jar文件使用。

*.jar文件拷贝到libs目录，eclipse直接导入即可，AndroidStudio项目中添加：

dependencies { 
	compile fileTree(include: ['*.jar'], dir: 'libs')
} 
重新编译即可完成。



## AAR

 *.aar，AAR（Android Archive）包是一个Android库项目的二进制归档文件。我们随便找一个aar文件，然后修改后缀名为‘zip’或者‘rar’格式，然后解压该文件，打开解压后的文件夹，截图如下所示：（每个aar解压后的内容可能不完全一样，但是都会包含AndroidManifest.xml，classes.jar，res，R.txt）

![img](https://img-blog.csdn.net/20161010142010859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)