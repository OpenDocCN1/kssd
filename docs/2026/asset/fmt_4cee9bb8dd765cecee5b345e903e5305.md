
```
nt!PspExitProcess:
fffff800`04191288  48895c2408          mov     qword ptr [rsp+8],rbx
fffff800`0419128d  48896c2410          mov     qword ptr [rsp+10h],rbp
fffff800`04191292  4889742418          mov     qword ptr [rsp+18h],rsi
fffff800`04191297  57                  push    rdi
fffff800`04191298  4154                push    r12
fffff800`0419129a  4155                push    r13
fffff800`0419129c  4156                push    r14
fffff800`0419129e  4157                push    r15
fffff800`041912a0  4883ec30            sub     rsp,30h
fffff800`041912a4  488bda              mov     rbx,rdx
fffff800`041912a7  448af9              mov     r15b,cl
fffff800`041912aa  f0838a4004000004    lock or dword ptr [rdx+440h],4
fffff800`041912b2  65488b3c2588010000  mov     rdi,qword ptr gs:[188h]
fffff800`041912bb  4533ed              xor     r13d,r13d
fffff800`041912be  41be01000000        mov     r14d,1
fffff800`041912c4  413acd              cmp     cl,r13b
fffff800`041912c7  747c                je      nt!PspExitProcess+0xbd (fffff800`04191345)
fffff800`041912c9  ba02030000          mov     edx,302h
fffff800`041912ce  488bcb              mov     rcx,rbx
fffff800`041912d1  e842d6feff          call    nt!EtwTraceProcess (fffff800`0417e918)
fffff800`041912d6  66ff8fc4010000      dec     word ptr [rdi+1C4h]
```

看了一下`PspExitProcess`，在``fffff800`041912d1 call nt!EtwTraceProcess``，这里距离`PspExitProcess`开头非常近，基本上保存了参数，经过应该一个判断，就进入了etw。

而系统调用的事件，进入etw是在函数快结束的位置，返回就继续返回就结束了。

据此猜想，process事件的要替换的函数指针应该是`PspExitProcess`里`call`进etw时保存函数返回地址，而不是想系统调用一样`PspExitProcess`的一个新开始地址指针。如果是新的会有重入问题，而且既然在头部进入etw记录，返回就继续执行就行了，没必要那么做，当然这不是百分之百的确定，要把`PspExitProcess`逆一遍，看看返回后是否真的在继续执行`PspExitProcess`，或者经过一个判断直接跳到末尾然后返回（当然在ida里看跳转图，不能一次到末尾），如果是返回继续执行，那么替换函数返回地址，自己构造一个`fakePspExitProcess`函数，应该也是可行的。

# 0x4结束

以上均为个人的逆向分析，如果有错误或是不同观点，欢迎指出。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
