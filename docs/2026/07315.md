
# [原创][分享]用户APC调用过程学习笔记

## 用户APC调用过程中的堆栈操作

![](https://bbs.kanxue.com/upload/attach/202008/881392_2NQDMYS3QHSXNZY.jpg)

## 用户APC调用：IDA分析内核模块，`KiDeliverApc`函数执行流程

![](https://bbs.kanxue.com/upload/attach/202008/881392_Z6QXBHBZ55U4MRH.jpg)

经过以上分析，得知：`KiDeliverApc`函数的作用如下：

1. 判断用户APC链表是否为空
2. 判断第一个参数是为1
3. 判断`ApcState.UserApcPending`是否为1
4. 将`ApcState.UserApcPending`设置为0
5. 链表操作将当前APC从用户队列中拆除
6. 调用函数(`KAPC.KernelRoutine`)释放`KAPC`结构体内存空间
7. 调用`KiInitializeUserApc`函数

## 分析`KiInitializeUserApc`函数

![](https://bbs.kanxue.com/upload/attach/202008/881392_QFG5HQV3MHFWJCT.jpg)

![](https://bbs.kanxue.com/upload/attach/202008/881392_YYX577S7C7839V9.jpg)

![](https://bbs.kanxue.com/upload/attach/202008/881392_JHM7F97VW42US53.jpg)

### 总结：

1. 内核APC函数在线程切换的时候就执行了，不需要换栈，比较简单，一个循环就可以执行完毕
2. 用户APC函数在系统调用，中断，异常返回3环的时候，会判断是否有需要执行的用户APC函数，如果有再执行APC函数
3. 每个用户APC函数执行前都会先执行内核APC函数

### 调用步骤总结：

1. 判断用户APC链是否为空
2. 判断第一个参数是否为1，为1时才会处理用户APC
3. 判断`_KTHREAD Thread->_KAPC.STATE ApcState -> UChar UserApcPending`是否为1，
   为1则当前APC有3环APC函数
4. 将`_KTHREAD Thread->_KAPC.STATE ApcState -> UChar UserApcPending`设置为0
5. 链表操作，将当前APC从用户APC队列中摘除
6. 调用`KiInitializeUserApc`函数，初始化用户APC的执行环境
    - 申请堆栈空间`0x2DC`字节(`Context`大小+APC相关16字节)，备份当前`TrapFrame`到堆栈中的`ConText`
    - 给APC相关的16字节赋值(APC总入口或内核函数、用户APC函数、参数1，参数2)
    - 修改当前`TrapFrame`，将`TrapFrame`修改成3环堆栈，`EIP`修改成`KeUserDispatcher`函数
    - 执行用户APC总入口，它会调用用户APC函数
    - 返回内核，循环操作

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-11-29 12:52 被三一米田编辑，原因：
