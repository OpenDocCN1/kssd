
虽然实例3可以打开,但是进入游戏后,不能启动外挂的热键,

而实例1和实例2都可以,什么原因呢?难道实例3的代码还有什么问题?

这样来试试:先运行实例一,然后不运行实例2,因为实例2和实例3的代码完全一致,

区别仅在于数据(2变成3).

`00404816`       `75 7E`         `jnz short TJMan23.00404896`

这里是跳去实例2,跳到`00404896`后再来个长跳转`jmp 004157B0`,去运行实例3.

这里不在`00404816`直接长跳是为了避免数据再次搬家.

结果实例3还是不能启动热键,这可是和实例2一样的代码啊!

晕~~找不到原因了...

还是写下了这篇总结,理一理自己的思路....

## 【参考知识】从网上搜得,零零散散的

1.

在`InitApplication`中创建一个同步对象,如果`GETLASTERROR()`调用返回`ERROR_ALREADY_EXISTS`表明已经有一个了

在`InitInstance`里可以调用`::OpenMutex`函数判断互斥量是否存在,如果没有,调用`::CreateMutex`建立一个互斥量,反之表明已经有一个实例了。退出程序时调用`::ReleaseMutex`释放创建的互斥量。

2.

创建互斥量：

``` 
HANDLE CreateMutex(

  LPSECURITY_ATTRIBUTES lpMutexAttributes,// 安全信息

  BOOL bInitialOwner,  // 最初状态，

  //如果设置为真，则表示创建它的线程直接拥有了该互斥量，而不需要再申请

  LPCTSTR lpName       // 名字，可以为NULL，但这样一来就不能被其他线程/进程打开

);
```

打开一个存在的互斥量：

``` 
HANDLE OpenMutex(

  DWORD dwDesiredAccess,  // 存取方式

  BOOL bInheritHandle,    // 是否可以被继承

  LPCTSTR lpName          // 名字

);
```

释放互斥量的使用权,但要求调用该函数的线程拥有该互斥量的使用权：

``` 
BOOL ReleaseMutex(//作用如同LeaveCriticalSection

  HANDLE hMutex   // 句柄

);
```

关闭互斥量：

``` 
BOOL CloseHandle(

  HANDLE hObject   // 句柄

);
```

3.

`CreateFileMapping`的函数为指定的文件创建一个文件映射对象,该函数的原形如下：

``` 
HANDLE CreateFileMapping(HANDLE hFile,//用于映射的文件句柄 LPSECURITY_ATTRIBUTES FileMappingAttributes,//内存映射文件的安全描述符 DWORD

Flprotect,//文件映射对象的最大长度的高32位 DWORD dwMaximumSizelow,//最大长度的低32位 LPCTSTR IPNAME//指定这个内存映射文件的名字)
```

值得注意的是,参数如果是`OXFFFFFFFF`,将在操作系统虚拟内存页面替换文件中创建文件映射对象,而不是使用磁盘文件,同时必须给出这个映射对象的大小。

## 【破解总结】
None

## 【Greetings】
看雪论坛,FCG论坛,DFCG论坛等

## 【完稿时间等】
2005.05.13,22:31,天气:晴,广州

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
