## C++高级教程


### 异常处理

通过try -- throw -- catch来完成（图片来自w3cschool）

![C++](https://7n.w3cschool.cn/statics/images/course/cpp_exceptions.jpg)

自定义异常：

```c++
//重写what
struct MyException : public exception
{
  const char * what () const throw ()
  {
    return "C++ Exception";
  }
};
```

### 命名空间

c++中的命名空间是为了区分不同的库中的同名函数

```c++
//声明,命名空间可以是不同的几个部分，分别在不同的文件中
namespace name {
   // 代码声明
}
//使用
using namespace name;
//嵌套
namespace namespace_name1 {
   // 代码声明
   namespace namespace_name2 {
      // 代码声明
   }
}
```

### 模板

模板是**泛型编程**的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

模板是创建泛型类或函数的蓝图或公式。

```c++
template <typename T>
inline T const& Max (T const& a, T const& b) 
{ 
    return a < b ? b:a; 
} 
```

每一个`template <class T>`或者`template <typename T>`用一次。

### 预处理器

用#开始的

1. 宏的使用
2. 条件编译

```c++
#ifdef DEBUG
   cerr <<"Variable x = " << x << endl; 
#endif

#if 0
   不进行编译的代码
#endif
//加了n就是非的意思如#ifndef
```

3. #和##运算符

   #运算符会把 replacement-text 令牌转换为用引号引起来的字符串。

```c++
#define MKSTR( x ) #x         //调用MKSTR函数会将参数进行替换
```

​		##运算符用于连接两个令牌，原理同上

4. 预定义宏

| 宏       | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| \__LINE__ | 这会在程序编译时包含当前行号。                               |
| \__FILE__ | 这会在程序编译时包含当前文件名。                             |
| \__DATE__ | 这会包含一个形式为 month/day/year 的字符串，它表示把源文件转换为目标代码的日期。 |
| \__TIME__ | 这会包含一个形式为 hour:minute:second 的字符串，它表示程序被编译的时间。 |

### 信号处理

1. signal

Ctrl+C产生中断，然后可以通过signal捕获并处理。

```c++
#include <iostream>
#include <csignal>
#include <unistd.h>

using namespace std;

void signalHandler( int signum ) //注意此处只有一个参数
{
    cout << "Interrupt signal (" << signum << ") received.\n";
    // 清理并关闭
    // 终止程序  
   exit(signum);  

}
int main ()
{
    // 注册信号 SIGINT 和信号处理程序
    signal(SIGINT, signalHandler);  //这个函数接收两个参数：第一个参数是一个整数，代表了信号的编号；第二个参数是一个指向信号处理函数的指针。
    while(1){
       cout << "Going to sleep...." << endl;
       sleep(1);
    }

    return 0;
}
```

2. raise

raise可以直接产生一个中断，类似于异常，可以被捕获并处理。raise的信号类型受限制，常用SIGINT（接收到交互注意信号）



### 多线程

直接看例子，其中struct用来传给线程的，thread_id传输id，用pthread_t

```c++
#include <iostream>
#include <cstdlib>
#include <pthread.h>

using namespace std;

#define NUM_THREADS     5

struct thread_data{
   int  thread_id;
   char *message;
};

void *PrintHello(void *threadarg)
{
   struct thread_data *my_data;

   my_data = (struct thread_data *) threadarg;

   cout << "Thread ID : " << my_data->thread_id ;
   cout << " Message : " << my_data->message << endl;

   pthread_exit(NULL);
}

int main ()
{
   pthread_t threads[NUM_THREADS];
   struct thread_data td[NUM_THREADS];
   int rc;
   int i;

   for( i=0; i < NUM_THREADS; i++ ){
      cout <<"main() : creating thread, " << i << endl;
      td[i].thread_id = i;
      td[i].message = "This is message";
      rc = pthread_create(&threads[i], NULL,
                          PrintHello, (void *)&td[i]);
      if (rc){
         cout << "Error:unable to create thread," << rc << endl;
         exit(-1);
      }
   }
   pthread_exit(NULL);
}
```

其他：

连接和分离线程

有以下两个例程，我们可以用它们来连接或分离线程：

```
pthread_join (threadid, status) 
pthread_detach (threadid) 
```

