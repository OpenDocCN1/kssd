
```
00D15689  83E8 02        sub     eax, 2  
00D1568C  66:8338 00     cmp     word ptr [eax], 0  
00D15690  75 10          jnz     short 00D156A2               ; ④改 jmp  
```

IAT 处理完毕，撤消修改，以免后面出现内存保护错误

### 四、ASProtect 2.0 Alpha

#### 两个分支的处理

```
00A8CEAD  8B5424 0C      mov     edx, dword ptr [esp+C]    
00A8CEB1  8902           mov     dword ptr [edx], eax       ; ①ebp 里才是 IAT，把 eax 改成 ebp  
00A8CEB3  8B4424 0C      mov     eax, dword ptr [esp+C]  
00A8CEB7  8906           mov     dword ptr [esi], eax  
00A8CEB9  0FB74424 04    movzx   eax, word ptr [esp+4]  
00A8CEBE  0143 08        add     dword ptr [ebx+8], eax  
```

```
00A83780  53             push    ebx                       ; ②找到这，Patch 这里  
00A83781  8BD8           mov     ebx, eax  
00A83783  8BC3           mov     eax, ebx  
00A83785  E8 56FFFFFF    call    00A836E0  
00A8378A  C603 E8        mov     byte ptr [ebx], 0E8       ; 这里是 IAT 加密，给改成 EB 了  
00A8378D  43             inc     ebx  
00A8378E  8903           mov     dword ptr [ebx], eax  
00A83790  5B             pop     ebx  
00A83791  C3             retn 3  
```

自己 Patch 代码

```
00A83780  66:C700 FF15   mov     word ptr [eax], 15FF      ; Call 类型的，如果是 jmp 类型的把 15FF 改成 25FF  
00A83785  83C0 02        add     eax, 2  
00A83788  8910           mov     dword ptr [eax], edx  
00A8378A  892A           mov     dword ptr [edx], ebp      ; ebp 是正确的 IAT  
00A8378C  90             nop  
00A8378D  90             nop  
00A8378E  90             nop  
00A8378F  90             nop  
00A83790  90             nop  
00A83791  C3             retn  
```

IAT 处理完毕，最好是撤消修改，以免后面出现内存保护错误

### 五、MoleBox

上面我们说的都是比较明显的加密 IAT，现在来看个不明显的

MoleBox 这个就不用多说了，很容易到达 OEP，我们看看 IAT 表，部分 IAT 是加密的

```
00406DF8  - FF25 04756300   jmp     dword ptr [637504]               ; flashfxp.0073BED3  
00406DFE    8BC0            mov     eax, eax  
00406E00  - FF25 00756300   jmp     dword ptr [637500]               ; flashfxp.0073BE34  
00406E06    8BC0            mov     eax, eax  
00406E08  - FF25 FC746300   jmp     dword ptr [6374FC]               ; flashfxp.0073BFCE  
```

重新加载程序，`DD 637504`，下内存访问断点，几次 `F9`

```
00715B6F  FF96 20CB3200   call    dword ptr [esi+32CB20]           ; 从这里出来的  
00715B75  09C0            or      eax, eax  
00715B77  74 07          je      short 00715B80  
00715B79  8903           mov     dword ptr [ebx], eax             ; 到达这里，注意 eax  
00715B7B  83C3 04        add     ebx, 4  
00715B7E  ^ EB E1         jmp     short 00715B61  
```

```
EAX 0073BED3 flashfxp.0073BED3      ；填充无效指针  
ECX 0013FFE0  
EDX 00CA09D8  
EBX 00637504 flashfxp.00637504  
ESP 0013FFA4  
EBP 7C800000 kernel32.7C800000  
ESI 00401000 flashfxp.00401000  
EDI 007128E0 flashfxp.007128E0  
```

`he 00715B6F`

重载，`F9`，中断下来后 `F7`，跟踪发现

```
0073C4E9  FF75 E4         push    dword ptr [ebp-1C]               ; [ebp-1C]里才是真正的 IAT  
0073C4EC  8B0D 34707400   mov     ecx, dword ptr [747034]  
0073C4F2  E8 3E070000     call    0073CC35                          ; 加密 Call  
0073C4F7  8945 E0         mov     dword ptr [ebp-20], eax          ; call 会产生一个值给 eax，0 或 1  
0073C4FA  837D E0 00      cmp     dword ptr [ebp-20], 0            ; eax 再给[ebp-20]，再比较  
0073C4FE  74 08          je      short 0073C508                   ; 为 0 则不加密，为 1 则加密，Magic Jump，改 jmp  
0073C500  8B45 E0         mov     eax, dword ptr [ebp-20]          ; 加密则把值传回 eax  
0073C503  8B00            mov     eax, dword ptr [eax]             ; [eax]里就是无效指针，传给 eax  
0073C505  8945 E4         mov     dword ptr [ebp-1C], eax          ; 无效指针再传给[ebp-1C]  
0073C508  834D FC FF      or      dword ptr [ebp-4], FFFFFFFF  
0073C50C  E8 02000000     call    0073C513  
0073C511  EB 1C          jmp     short 0073C52F  
0073C513  A1 34707400     mov     eax, dword ptr [747034]  
0073C518  8945 D4         mov     dword ptr [ebp-2C], eax  
0073C51B  837D D4 00      cmp     dword ptr [ebp-2C], 0  
0073C51F  74 0D          je      short 0073C52E  
0073C521  8B45 D4         mov     eax, dword ptr [ebp-2C]  
0073C524  83C0 10        add     eax, 10  
0073C527  50             push    eax  
0073C528  FF15 18F07300   call    dword ptr [<&KERNEL32.LeaveCriti>; ntdll.RtlLeaveCriticalSection  
0073C52E  C3             retn  
0073C52F  8B45 E4         mov     eax, dword ptr [ebp-1C]          ; IAT 或无效指针再传给 eax  
0073C532  8B4D F0         mov     ecx, dword ptr [ebp-10]  
0073C535  64:890D 0000000>mov     dword ptr fs:[0], ecx  
0073C53C  5F             pop     edi  
0073C53D  5E             pop     esi  
0073C53E  5B             pop     ebx  
0073C53F  C9             leave  
0073C540  C2 0800        retn    8                                  ; 返回后把 eax 的值传给 IAT 表地址  
```

我们把 `Magic Jump` 改成 `jmp` 之后 `IAT` 不再加密：）

这里需要说明的是，这个法子跟我原先的文章里的法子是不一样的，但是可以达到相同的效果

于是，我们可以写个脚本

```c
/*
Script written by wynney  

Date:   2007-04-13
Script:MoleBox  
Environment : OllyDbg 1.1, ODBGScript 1.52, Winxp Sp2  
Debugging options: Tick all items in OllyDbg's Debugging Options-Exceptions  

Thanks :
          kanxue     - author of HideOD        
          hnhuqiong  - author of ODbgScript 1.52  
*/  
#log  
var temp  

log eip  
STI  
STO  
mov temp, esp  
bphws temp,"r"             //esp 定律,用来找 OEP  
esto  
bphwcall  
find eip,#8945??837D??0074088B45??8B008945??834D??FF#  
cmp $RESULT, 0  
je err  
mov temp,$RESULT  
repl temp,#8945??837D??0074088B45??8B008945??834D??FF#,#8945??837D??00EB088B45??8B008945??834D??FF#,21  
sti  
sti  
mov temp, eax  
bp temp  
esto  
bc temp  
sti  
mov temp, esp  
bphws temp,"r"  
esto  
bphwcall  
mov temp, eip  
add temp, 0D  
bp temp  
esto  
bc temp  
sti  
dpe "\\UnPacked.exe", eip  
cmt eip, "This is the OEP!  Found By: wynney "   

err:  
msg "error"  
ret
```

------------------------------------------------------------------------
【经验总结】

1、若有的壳有内存访问断点校验，可以换用内存写入断点，或者是硬件访问/写入断点
2、如果不是很明显的寄存器填充，那么一般 `IAT` 加密就在附近，多跟踪下：）
3、很多时候，并不是单纯的修改可以解决的，需要自己 `Patch` 代码，多了解下汇编吧，这个是别人教不会的
------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2007 年 04 月 14 日 wynney

4dd466d0e1e0cc123afbe504da2462ca
