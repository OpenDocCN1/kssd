
```
000680B8E  |.  0FB74C24 18   movzx   ecx, word ptr [esp+18]
000680B93  |.  0FB75424 14   movzx   edx, word ptr [esp+14]
000680B98  |.  0FB77C24 24   movzx   edi, word ptr [esp+24]
000680B9D  |.  83C0 03       add     eax, 3
000680BA0  |.  0FB7C0        movzx   eax, ax
000680BA3  |.  894424 20     mov     dword ptr [esp+20], eax
000680BA7  |.  894C24 3C     mov     dword ptr [esp+3C], ecx
000680BAB  |.  0FB7C0        movzx   eax, ax
000680BAE  |.  895424 38     mov     dword ptr [esp+38], edx
000680BB2  |.  8D4C24 4C     lea     ecx, dword ptr [esp+4C]
000680BB6  |.  51            push    ecx
000680BB7  |.  894424 34     mov     dword ptr [esp+34], eax
000680BBB  |.  8D5424 24     lea     edx, dword ptr [esp+24]
000680BBF  |.  52            push    edx
000680BC0  |.  8D4424 2C     lea     eax, dword ptr [esp+2C]
000680BC4  |.  50            push    eax
000680BC5  |.  8D4C24 20     lea     ecx, dword ptr [esp+20]
000680BC9  |.  51            push    ecx
000680BCA  |.  8D5424 28     lea     edx, dword ptr [esp+28]
000680BCE  |.  52            push    edx
000680BCF  |.  8D4424 40     lea     eax, dword ptr [esp+40]
000680BD3  |.  50            push    eax
000680BD4  |.  8D4C24 40     lea     ecx, dword ptr [esp+40]
000680BD8  |.  51            push    ecx
000680BD9  |.  8D5424 38     lea     edx, dword ptr [esp+38]
000680BDD  |.  52            push    edx
000680BDE  |.  6A 10         push    10                  //读狗
000680BE0  |.  E8 5B06F5FF   call    00601240            ;  计算3 (RY_CALCULATE3)   功能码：10H
```

&H10 号功能，是狗中运算功能，这个东西是要有点运气，想去找出他的算法的话，我是运气好，一下给蒙出来了，当然如果找不到，就去查出所有调用他 的地方，统统的用JZ替代JNZ，也是个办法！

下面看看我的蒙的过程
```
000680BE5  |.  8B4C24 58     mov     ecx, dword ptr [esp+58]
000680BE9  |.  83C4 28       add     esp, 28
000680BEC  |.  0FB7C0        movzx   eax, ax
000680BEF  |.  51            push    ecx
000680BF0  |.  8BCD          mov     ecx, ebp
000680BF2  |.  894424 44     mov     dword ptr [esp+44], eax
000680BF6  |.  E8 A5FCFFFF   call    0006808A0      // 对狗计算的数据进行变换，
000680BFB  |.  3D 8A0C0000   cmp     eax, 0D23     //比较是否大？
000680C00      0F8D 3E010000 jge     000680D44
000680C06  |.  66:395C24 40  cmp     word ptr [esp+40], bx
000680C0B  |.  0F85 33010000 jnz     000680D44
000680C11  |.  8B5424 10     mov     edx, dword ptr [esp+10]
000680C15  |.  8B4424 30     mov     eax, dword ptr [esp+30]
000680C19  |.  83C2 DF       add     edx, -21
000680C1C  |.  50            push    eax
000680C1D  |.  8BCD          mov     ecx, ebp
000680C1F  |.  66:895424 46  mov     word ptr [esp+46], dx
000680C24  |.  E8 77FCFFFF   call    0006808A0    //对狗计算的数据进行变换，
000680C29  |.  3D F90C0000   cmp     eax, 0FF5     //比较是否大？
000680C2E      0F8D 10010000 jge     000680D44
```

这个狗中的计算取的数据简单，通过跟踪后，发现他取的是一个时间数据变量，取到后，与现在的机器时间进行相差， 结果通过一个容不得错区间比较，如果在这个差域内，则说明通过了！

以下为最后一个狗函数！ 关闭（CLOSE DOG）狗！4号功能
```
000680E4A  |.  6A 04         push    4
000680E4C  |.  E8 EF03F5FF   call    00601240                        ;  close
```

了解上面的过程后，我们开始对软件做手术，就是在每个调用点上，给他正确的返回扬数据，所以这点和微狗的给值是类似的，只有你给他了，狗才会听话哟！

我们进入调狗代码，拟合下数据：这个CALL： `call 00601240` 读狗哟！

在 `00601240` 键入以下代码：
```
00601240      8B4424 04           mov     eax, dword ptr [esp+4]
//取得狗功能码给EAX，为什么是ESP+4，因为调用读狗前PUSH的就是功能号；
00601244      83F8 05             cmp     eax, 5
00601247      74 12               je      short 0060125B  //5号向读狗
00601249      83F8 0A             cmp     eax, 0A
0060124C      74 08               je      short 00601256 //10号取狗ID
0060124E      83F8 10             cmp     eax, 10
00601251      74 29               je      short 0060127C //16号狗中计算
00601253      33C0                xor     eax, eax     //其他号调用狗的都让他返回0；如打开锁、查找锁、关闭锁、检验密码等；
00601255      C3                  retn
```

10 号取狗的USERID：
```
00601256      C74424 54 88888888  mov     dword ptr [esp+54], XXXXXXXX
0060125E      33C0                xor     eax, eax
00601260      C3                  retn
```

5号读狗中的数据：
```
00601261      C74424 7C 88888888  mov     dword ptr [esp+7C], 数据1
00601269      C78424 80000000 888>mov     dword ptr [esp+80], 数据2
                    。。。。。。。。。。。。。。。。。。。。。。 数据N
00601274      33C0                xor     eax, eax            返回0
00601276      C3                  retn
```

16号取狗中的动态数据（计算数据）
```
0060127C      FF15 54925200       call    dword ptr [<&KERNEL32.GetTickCount>]   //取时间
00601282      35 CF000000         xor     eax, 0CF                               //计算
00601287      894424 5C           mov     dword ptr [esp+5C], eax                //存放
0060128B      33C0                xor     eax, eax                               //EAX回0
0060128D      C3                  retn
```

以上写完后，保存为一个新文件，运行测试，一切OK，狗驯服了！，收工！

写得可能有点乱，我已经很努力去说清楚了，实在文字水平这样，有看不懂的童鞋，一定相信是我的文字组织的不好，狗一定是你可以征服的！

--------------------------------------------------------------------------------

### 【经验总结】

现在的狗不象早期的锁了，就那个几个判断，强跳后就解决了，现的狗总是搞得那么花，不光是代码花，数据也花，所以 甚至有作者吹出“不可破解”的神话，但对于软件原理总就是那么回事，所以我总觉得要看锁，先去看作者的SDK，函数接 口，事半功倍是肯定的了，由于本人水平有限，写的不好，请不要向我扔石头！

--------------------------------------------------------------------------------

### 【版权声明】

谢谢观看, 转载请注明作者并保持文章的完整, 谢谢!

2009年11月15日 15:02:31

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
