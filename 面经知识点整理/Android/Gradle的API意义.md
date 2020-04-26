## minSdkVersion

最低的API等级。如果设置为19，如果在代码中使用sdk版本为21的API的话，编译器会给出警告

## CompileSdkVersion

当前的编译sdk版本

## TargetSdkVersion

目标Sdk版本，性能最佳or其他，兼容性问题的解决。可以调用TargetApi()

比如系统升级到Android 4.4(API 19)之后，AlarmManager API设置时间的精确度行为发生了变化，由精确变成了非精确。假设APP的targetSdkVersion是17，那么还是会保持原来的精确行为，当targetSdkVersion>=19时才会使用新特性.