基本使用:https://www.jianshu.com/p/3b0d0469cf3e

查三门课都在80分以上的学生的名字

```sql
select distinct name from stu where name not in(select distinct name from stu where score<80)
```



### SQL左连接右连接

https://blog.csdn.net/plg17/article/details/78758593