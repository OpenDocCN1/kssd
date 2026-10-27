
# [原创]反NP监视原理(+Bypass NP in ring0)

NP=nProtect GameGuard(如果你不知道这是什么，请不要往下看)

- 标题:【原创】反NP监视原理
- 作者:堕落天才
- 日期:2007年1月3号
- 版权声明:请保存文章的完整，转载请注明出处

## 一、NP用户层监视原理

NP启动后通过`WriteProcessMemory`跟`CreateRemoteThread`向所有进程注入代码（除了系统进程`smss.exe`），代码通过np自己的`LoadLibrary`向目标进程加载`npggNT.des`。`npggNT.des`一旦加载就马上开始干“坏事”，挂钩(HOOK)系统关键函数如`OpenProcess`,`ReadProcessMemory`,`WriteProcessMemory`,`PostMessage`等等。挂钩方法是通过改写系统函数头，在函数开始`JMP`到`npggNT.des`中的替换函数。用户调用相应的系统函数时，会首先进入到`npggNT.des`模块等待NP的检查，如果发现是想对其保护的游戏进行不轨操作的话，就进行拦截，否则就调用原来的系统函数，让用户继续。

下面是NP启动前`user32.dll`中的`PostMessageA`的源代码（NP版本900,XP sp2）

```
8BFF            MOV EDI,EDI
55              PUSH EBP
8BEC            MOV EBP,ESP
56              PUSH ESI
57              PUSH EDI
8B7D 0C         MOV EDI,DWORD PTR SS:[EBP+C]
8BC7            MOV EAX,EDI
2D 45010000     SUB EAX,145
74 42           JE SHORT USER32.77D1CBDA
83E8 48         SUB EAX,48
74 3D           JE SHORT USER32.77D1CBDA
2D A6000000     SUB EAX,0A6
0F84 D4530200   JE USER32.77D41F7C
8B45 10         MOV EAX,DWORD PTR SS:[EBP+10]
8B0D 8000D777   MOV ECX,DWORD PTR DS:[77D70080]
F641 02 04      TEST BYTE PTR DS:[ECX+2],4
0F85 03540200   JNZ USER32.77D41FBE
8D45 10         LEA EAX,DWORD PTR SS:[EBP+10]
50              PUSH EAX
57              PUSH EDI
E8 FBFEFFFF     CALL USER32.77D1CAC0
FF75 14         PUSH DWORD PTR SS:[EBP+14]
FF75 10         PUSH DWORD PTR SS:[EBP+10]
57              PUSH EDI
FF75 08         PUSH DWORD PTR SS:[EBP+8]
E8 ACBFFFFF     CALL USER32.77D18B80
5F              POP EDI
5E              POP ESI
5D              POP EBP
C2 1000         RETN 10
```

而下面是NP启动后`user32.dll`中的`PostMessageA`的源代码（NP版本900,XP sp2）

```
E9 A69AB8CD     JMP npggNT.458A6630
56              PUSH ESI
57              PUSH EDI
8B7D 0C         MOV EDI,DWORD PTR SS:[EBP+C]
8BC7            MOV EAX,EDI
2D 45010000     SUB EAX,145
74 42           JE SHORT USER32.77D1CBDA
83E8 48         SUB EAX,48
74 3D           JE SHORT USER32.77D1CBDA
2D A6000000     SUB EAX,0A6
0F84 D4530200   JE USER32.77D41F7C
8B45 10         MOV EAX,DWORD PTR SS:[EBP+10]
8B0D 8000D777   MOV ECX,DWORD PTR DS:[77D70080]
F641 02 04      TEST BYTE PTR DS:[ECX+2],4
0F85 03540200   JNZ USER32.77D41FBE
8D45 10         LEA EAX,DWORD PTR SS:[EBP+10]
50              PUSH EAX
57              PUSH EDI
E8 FBFEFFFF     CALL USER32.77D1CAC0
FF75 14         PUSH DWORD PTR SS:[EBP+14]
FF75 10         PUSH DWORD PTR SS:[EBP+10]
57              PUSH EDI
FF75 08         PUSH DWORD PTR SS:[EBP+8]
E8 ACBFFFFF     CALL USER32.77D18B80
5F              POP EDI
5E              POP ESI
5D              POP EBP
C2 1000         RETN 10
```

通过对比我们可以发现，NP把`PostMessageA`函数头原来的`8BFF558BEC`五个字节改为了`E9A69AB8CD`，即将`MOV EDI,EDI  PUSH EBP  MOV EBP,ESP`三条指令改为了`JMP npggNT.458A6630`。所以用户一旦调用`PostMessageA`的话，就会跳转到`npggNT.des`中的`458A6630`中去。

## 二、用户层反NP监视方法

### 1. 把被NP修改了的函数头改回去

上面知道NP是通过在关键系统函数头写了一个`JMP`来进行挂钩的，因此，在理论上我们可以通过把函数头写回去来进行调用。在实际操作的时候，这种方法并不理想。因为`npggNT.des`也挂钩了把函数头改写回去的所有函数，还有它的监视线程也会进行检校判断它挂钩了的函数是不是被修改回去。因此实现起来很困难，随时都会死程序。

### 2. 构建自己的系统函数(感谢JTR提供)

这种方法适用于代码比较简单的系统函数。下面我们看看`keybd_event`的函数源码

```
8BFF            MOV EDI,EDI                              ; USER32.keybd_event
55              PUSH EBP
8BEC            MOV EBP,ESP
83EC 1C         SUB ESP,1C
8B4D 10         MOV ECX,DWORD PTR SS:[EBP+10]
8365 F0 00      AND DWORD PTR SS:[EBP-10],0
894D EC         MOV DWORD PTR SS:[EBP-14],ECX
66:0FB64D 08    MOVZX CX,BYTE PTR SS:[EBP+8]
66:894D E8      MOV WORD PTR SS:[EBP-18],CX
66:0FB64D 0C    MOVZX CX,BYTE PTR SS:[EBP+C]
66:894D EA      MOV WORD PTR SS:[EBP-16],CX
8B4D 14         MOV ECX,DWORD PTR SS:[EBP+14]
894D F4         MOV DWORD PTR SS:[EBP-C],ECX
6A 1C           PUSH 1C
33C0            XOR EAX,EAX
8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
40              INC EAX
51              PUSH ECX
50              PUSH EAX
8945 E4         MOV DWORD PTR SS:[EBP-1C],EAX
E8 9B8DFCFF     CALL USER32.SendInput
C9              LEAVE
C2 1000         RETN 10
```

由上面我们看到`keybd_event`进行了一些参数的处理最后还是调用了`user32.dll`中的`SendInput`函数。而下面是`SendInput`的源代码

```
B8 F6110000     MOV EAX,11F6
BA 0003FE7F     MOV EDX,7FFE0300
FF12            CALL DWORD PTR DS:[EDX]          ; ntdll.KiFastSystemCall
C2 0C00         RETN 0C
```

`SendInput`代码比较简单吧？我们发现`SendInput`最终是调用了`ntdll.dll`中的`KiFastSystemCall`函数，我们再跟下去，`KiFastSystemCall`就是这个样子了

```
8BD4            MOV EDX,ESP
0F34            SYSENTER
```

通过上面的代码我们发现一个`keybd_event`函数构建并不复杂因此我们完全可以把上面的代码COPY到自己的程序，用来替代原来的`keybd_event`。NP启动后依然会拦截原来的那个，但已经没关系啦，因为我们不需要用原来那个`keybd_event`了。

这种方法适用于源代码比较简单的系统函数，复杂的话实现起来就比较麻烦了。我是没有信心去重新构建一个`PostMessageA`,因为其中涉及到N个`jmp`和`Call`,看起来头都大。 还有在VC6里嵌入汇编经常死VC(这种事太烦人了)，我想不会是我用了盗版的原因吧？

### 3. 进入ring0(感谢风景的驱动鼠标键盘模拟工具)

由上面可以看到，NP用户层的监视不过是修改了一下系统的函数头，进行挂钩监视。因此，要反NP用户层监视的话，进入ring0的话很多问题就可以解决了。比如`WinIO`在驱动层进行键盘模拟，`npggNT.des`是拦截不到的。但是由于NP用了特征码技术，再加上`WinIO`名气太大了，所以`WinIO`在NP版本8××以后都不能用了。但是如果熟悉驱动开发的话，自己写一个也不是很困难的事。

说了那么多看起来很“高深”的东西，现在说一些象我这样的菜鸟都能明白的东西，呵呵，因为这是菜鸟想出来的菜办法。

### 4. 断线程

我们知道NP是通过`CreateRemoteThread`在目标进程创建远程线程的，还有一点，很重要的一点就是:NP向目标进程调用了`CreateRemoteThread`后就什么都不管了，也就是说，凭本事可以对除游戏外的所有进程`npggNT.des`模块进行任何“处置”。这样我们可以用一个很简单的方法就是检查自己的线程，发现多余的话（没特别的事情就是NP远程创建的）就马上结束了它，这样NP就无法注入了。但是由于windows系统是多任务系统，而`CreateRemoteThread`的执行时间又极短，要在这么短的时间内发现并结束它的话是一件很困难的事。一旦`CreateRemoteThread`执行完毕而我们的监视线程还没有起作用的话，后果就惨重了，`npggNT.des`马上把程序“搞死”。因为我们一直试图关闭它的线程，而`npggNT.des`又拦截了`TerminateThread`,所以我们就只能不断地“重复重复再重复”去试图关闭`npggNT.des`的监视线程。如果我们很幸运地在其执行注入代码时就能断了它地线程地话，`npggNT.des`就无法注入了。这种方法在NP早期版本大概有百分之五十的成功率，现在能有百分之一的成功率都不错了。

### 5. 断线程之线程陷阱

我知道“线程陷阱”这个词肯定不是我首创，但用“陷阱”这种方法来对付NP之前在网上是找不到的。为什么要叫“线程陷阱”？因为这确确实实是一个陷阱，在`npggNT.des`肯定要经过的地方设置一个“陷阱”，等它来到之后，掉进去自动就死掉了。而搭建陷阱的方法简单得令你难以相信。

上面我们从`npggNT.des`的监视原理可以看到，`npggNT.des`要来挂钩(HOOK)我们的系统函数，这种的方法我们也会，是不是？哪想想，这种挂钩方法需要用到哪些系统函数呢? 打开进程`OpenProcess`或`GetCurrentProcess`(因为`npggNT.des`已经进入了目标进程，所以没有必要再调用`OpenProcess`，肯定是用后者)、找模块地址`GetModelHandle`、找函数地址`GetProcAddress`、改写函数头的内存属性`VirtualQuery`&`VirtualProtect`、写内存`WriteProcessMemory`。嘿嘿，在这些地方设置陷阱就八九不离十了，肯定是`npggNT.des`干那坏勾当要经过的地方。

怎么设陷阱呢？选一个上面说的函数（我没有一一尝试），先自己挂钩（嘿嘿，NP会我们也会）。等到有人调用的时候，先判断当前的的线程是不是我们程序的，不是的话，那就断了它吧（一个`ExitThread`就可以了）。大概就像下面这个样子

```
HANDLE WINAPI MyGetCurrentProcess(VOID)//替换掉原来的GetCurrentProcess
{
   DWORD dwThreadId=GetCurrentThreadId();//得到当前线程ID
   if(!IsMyThread(dwThreadId)){//不是我们要保护的线程
      ExitThread(0);//断了它吧
   }
   UnhookGetCurrentProcess(); //是我们要保护的线程调用就恢复函数头
   HANDLE hProcess=GetCurrentProcess();//让它调用
   RehookGetCurrentProcess();//重新挂钩
   return hProcess;   //返回调用结果
}
```

这种方法去掉`npggNT.des`的监视是完全能够实现的，但是这个函数`IsMyThread(dwThreadId)`非常关键，要考虑周全，不然断错线程的话，就“自杀”了。

### 6. 更简单的陷阱

原理跟上面一样，但是我们将替换函数写成这个样子

```
HANDLE WINAPI MyGetCurrentProcess(VOID)//替换掉原来的GetCurrentProcess
{
   HMODLE hMod=GetModelHandle("npggNT.des");
   if(hMod!=NULL){
      FreeLibrary(hMod);      //直接Free掉它
   }
   UnhookGetCurrentProcess(); //是我们要保护的线程调用就恢复函数头
   HANDLE hProcess=GetCurrentProcess();//让它调用
   RehookGetCurrentProcess();//重新挂钩
   return hProcess;   //返回调用结果
}
```

这种方法就万无一失了，不用担心会“自杀”。

## 三、总结

由上面可以看到在用户层上反NP监视是不是很简单的事？最简单有效的就是第六种方法，短短的几行代码就可以搞定了。但是不要指望去掉了`npggNT.des`就可以为所欲为了，还有NP还在驱动层做了很多手脚，比如`WriteProcessMemory`在用户层用没问题，但是过不了NP的驱动检查，对游戏完全没效果。要在NP下读写游戏内存，说起来又另一篇文章了《如何在NP下读写游戏内存》，请继续关注。

---

## Bypass NP in ring0 (2007年3月16日)

1. Add MyService
