
```asm
00435405  |> C74424 0C 010>MOV DWORD PTR SS:[ESP+C],1  

0043540D  |. 6A 08            PUSH 8                                     ; /Arg3 = 00000008  

0043540F  |. 8D5424 04       LEA EDX,DWORD PTR SS:[ESP+4]             ; |  

00435413  |. 52             PUSH EDX                                   ; |Arg2  

00435414  |. 8D4C24 18      LEA ECX,DWORD PTR SS:[ESP+18]            ; |  

00435418  |. 51             PUSH ECX                                   ; |Arg1  

00435419  |. E8 12E10600   CALL EXPLORER.004A3530                     ; \EXPLORER.004A3530  

0043541E  |. 83C4 0C        ADD ESP,0C  

00435421  |. C74424 18 020>MOV DWORD PTR SS:[ESP+18],2               ; 这个 2 就是 SE\_PRIVILEGE\_ENABLED, 改成 0 ********************************  

00435429  |. 6A 00          PUSH 0  

0043542B  |. 6A 00          PUSH 0  

0043542D  |. 6A 00          PUSH 0  

0043542F  |. 8D4424 18      LEA EAX,DWORD PTR SS:[ESP+18]  

00435433  |. 50             PUSH EAX  

00435434  |. 6A 00          PUSH 0  

00435436  |. 8B5424 1C      MOV EDX,DWORD PTR SS:[ESP+1C]  

0043543A  |. 52             PUSH EDX  

0043543B  |. FFD7           CALL EDI  

0043543D  |. 85C0           TEST EAX,EAX  

0043543F  |. 75 05          JNZ SHORT EXPLORER.00435446  

00435441  |. 83C8 FF        OR EAX,FFFFFFFF  

00435444  |. EB 02          JMP SHORT EXPLORER.00435448  

00435446  |> 33C0           XOR EAX,EAX  

00435448  |> 83C4 1C        ADD ESP,1C  

0043544B  |. 5F             POP EDI  

0043544C  |. 5E             POP ESI  

0043544D  |. 5B             POP EBX  

0043544E  \. C3             RETN  
```

加上这个，似乎也还是不够![](img/8f62d873668cbf759316096ebfcf749d.jpg)  

`ExeCryptor v2.2.6` 主程序是直接用 `GetTokenPrivilege` 检查当前进程的权限。  

`simonzh` 兄跟了这个主程序吗？我前段时间跟了一部分（我指那些线程）。  

主程序的线程函数只有 5 个，我跟了 3 个。地址是:  

`51A515` - 检测内存代码完整性,`anti-patch`  
`54329D` - 检测 `ExeCryptor.exe` 文件完整性  
`54AFCF` - 这个复杂,有检查 `SeDebugPrivilege`,`FindWindow` `GetTickCount`,`EnumWindows`  
有一点不明白，在以前的版本也见过，代码 `LoadLibrary("uxtheme.dll")` 然后立即 `FreeLibrary` 了，不明白是什么意思。  
这个线程最后注册了 1 个窗口类后退出。好象还是这个线程，再次启动时估计会创建窗口，我还没有跟，应该与 `anti` 有关。  

还有 2 个 `thread` 没有跟,是 `570F6D` 和 `563EA3`。  

这个东东我放下有一段时间了，盼望 `simonzh` 出手![](img/83bdd63c29485f7768335d7ba60e67a5.jpg)

`od` 用 `SE\_PRIVILEGE\_ENABLED` 用启用 `SE\_DEBUG\_NAME`,最好不要禁用。  

为什么要启用 `SE\_DEBUG\_NAME`？  

调试某些进程需要此权限。某些调试ＡＰＩ需要此权限。  

有没有更好的办法呢？有！  

`win32` 里面其实没有父进程的概念，只有从哪个进程继承数据的概念。  

同样这个继承了 `SE\_DEBUG\_NAME`  

所以，只要改变，让子进程从 `explorer` 继承就行了。  

我有打算写这么一个插件，  

只是近期没有时间,可能要过些日子才行。  

到时候反父进程方法和楼主这篇文章就算废了，因为没此必要了，呵呵。  

等不及想自己写的，如果有什么问题可以问我。 

> 引用:
>
> * * *
>
> *最初由 `goldenegg` 发布*  
>
> `od` 用 `SE\_PRIVILEGE\_ENABLED` 用启用 `SE\_DEBUG\_NAME`,最好不要禁用。  
> 为什么要启用 `SE\_DEBUG\_NAME`？  
> 调试某些进程需要此权限。某些调试ＡＰＩ需要此权限。  
>
> 有没有更好的办法呢？有！  
>
> ........ 

权限我是这样搞的:`Hook` 掉 `OD` 自己的 `WaitForDebugEvent`,在收到被调试进程的第 1 个 `int3` 后降低其权限。  

修改被调试进程的父进程,我用过这个办法:`Hook` `ntoskrnl!NtCreateProcessEx`,创建被调试进程后,修改 `EPROCESS.InheritedFromUniqueProcessId` 为 `explorer` 的 `pid`,但这样做 `xikuge` 的那个 `reverseme` 不能运行,不知为什么。  

`<加密技术内幕>`里的 `antiloader` 倒是可以通过。  

最后放弃了修改父进程的打算，换个方式:`Hook` `ntoskrnl!NtOpenProcess`,如果要打开的是 `OD`,就让它去打开 `explorer`

`4dd466d0e1e0cc123afbe504da2462ca`
