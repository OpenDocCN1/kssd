
```asm
:004063A8 EB03                 jmp 004063AD
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0040639E(C)`

`|`

```asm
:004063AA 0F9CC0               setl al
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:004063A8(U)`

`|`

```asm
:004063AD 3A442410             cmp al, byte ptr [esp+10]
:004063B1 7401                 je 004063B4
:004063B3 4D                   dec ebp
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:004062DB(C), :004062F8(U), :00406397(C), :004063B1(C)`

`|`

```asm
:004063B4 807C2EFF00          cmp byte ptr [esi+ebp-01], 00  <==从 004062DB 跳到这里
:004063B9 0F95C0              setne al
:004063BC 0AD8                or bl, al
:004063BE 7407                je 004063C7
:004063C0 8B0424              mov eax, dword ptr [esp]
:004063C3 8928                mov dword ptr [eax], ebp
:004063C5 EB07                jmp 004063CE
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:004063BE(C)`

`|`

```asm
:004063C7 8B0424              mov eax, dword ptr [esp]
:004063CA 33D2                xor edx, edx
:004063CC 8910                mov dword ptr [eax], edx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:004061E8(U), :004063C5(U)`

`|`

```asm
:004063CE 8B442408            mov eax, dword ptr [esp+08]
```

<===如果前面位是 0x 的话则，这里 EAX 的值正好等于 787878，要 EAX=199FF2，到这里我们可以判断出注册码了，它就是 0x199ff2(均为小写)。退出程序一试，呵呵，“注册完成”

```asm
:004063D2 8B54240C            mov edx, dword ptr [esp+0C]
:004063D6 83C414              add esp, 00000014
:004063D9 5D                  pop ebp
:004063DA 5F                  pop edi
:004063DB 5E                  pop esi
:004063DC 5B                  pop ebx
:004063DD C3                  ret
```

------------------------------------------------------------------------------------------

4、软件还有一种是，前两位不是 0x 的情况，我跟踪出来，但不知怎么返回到注册码，把它的算法写在下面：

```c
eax=0;
string=输入的注册码;
for(i=0;i=strlen(string);i++)
{eax=eax*10+string[i]
}
printf("%d",eax);
```

最后 EAX 应该等于 EAX=199FF2(16 进制)=1679346(10 进制)

呵呵，算出来了，还有一个注册码就是 1679346

5、注册信息保存在注册表

`[HKEY_LOCAL_MACHINE\Software\CLASSES\CLSID\{1AE69D60-73D0-11D4-BD52-38A480C50000}]`

`"224951124"="224951124"`

这个键值为 1-10，是尚余的使用次数，如果是上面的数值时，就是注册版的了。

删除键值，就成为未注册版本了。

(我没学过汇编，C 语言自学了一段时间，没学完，如有不对的地方请大家指正)  4dd466d0e1e0cc123afbe504da2462ca
