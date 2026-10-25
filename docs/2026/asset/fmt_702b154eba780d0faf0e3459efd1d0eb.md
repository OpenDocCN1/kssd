
`or dl, 41`

`:00437644 881408`

```
mov byte ptr [eax+ecx],
dl
```
* Referenced by a
(U)nconditional or (C)onditional Jump at Address:
|:0043763F(C)
|
`:00437647`
```
8A1408 mov dl, byte ptr [eax+ecx]
```
`:0043764A 80FA5A`
```
cmp dl, 5A
```
`:0043764D 7E06`
```
jle 00437655
```
`:0043764F 80E25A`
```
and dl, 5A
```
`:00437652 881408`
```
mov byte ptr [eax+ecx],
dl
```
* Referenced by a
(U)nconditional or (C)onditional Jump at Address:
|:0043764D(C)
|
`:00437655`
```
8A543004 mov dl, byte ptr [eax+esi+04]
```
`:00437659 8A1C08`
```
mov bl, byte ptr [eax+ecx]
```
`:0043765C 80F238`
```
xor dl, 38
```
`:0043765F 3ADA`
```
cmp bl, dl
```
上面这段算法产生注册码后十位并比较，由于这里是直接与真码逐位比较，所以要设法把真码的每一位保存下来，最后调用一个Messagebox来显示，但这个调用不能放在这里，因为软件每次启动时都要调用这个call来验证注册码，把Messagebox插在这里每次启动都会弹出。所以在这里要插入的就是把真码保存的代码。我在数据区找了一个地方`4B1B61`,这里我们先让程序跳到添加的代码处：

―――――――――――――――――――――――――――――――――

修改后的代码：

*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043764D(C)
|
`:00437655`
```
8A543004 mov dl, byte ptr [eax+esi+04]
```
`:00437659 8A1C08`
```
mov bl, byte ptr [eax+ecx]
```
`:0043765C 684E894800`
```
push 0048894E
```
`:00437661 C3`
```
ret
```
这两步跳到添加的代码处
`:00437662`
```
90 nop
```
`:00437663`
```
40 inc eax
```
`:00437664 EBC8`
```
jmp 0043762E
```
`:00437666 33FF`
```
xor edi, edi
```
以上是在原代码中的改动，下面是加在原`.text`块末尾的代码，从`48894E`处开始，执行结束后返回：

`:0048894E`
```
8898611B4B00 mov byte ptr [eax+004B1B61], bl
```
`:00488954 90`
```
nop
```
`:00488955 6863764300`
```
push 00437663
```
`:0048895A C3`
```
ret
```
―――――――――――――――――――――――――――――――――

`:00437661`
```
7503 jne 00437666
```
`:00437663 40`
```
inc eax
```
`:00437664 EBC8`
```
jmp 0043762E
```
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00437620(C),
:00437661(C)
|
`:00437666 33FF`
```
xor edi, edi
```
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00437627(C),
:00437631(C)
|
`:00437668 8B4510`
```
mov eax, dword ptr [ebp+10]
```
`:0043766B 85C0`
```
test eax, eax
```
`:0043766D`
```
7406 je 00437675
```
`:0043766F`
```
893D5C454B00 mov dword ptr [004B455C], edi
```
* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
|:0043766D(C)
|
`:00437675`
```
C745FCFFFFFFFF mov [ebp-04], FFFFFFFF
```
`:0043767C`
```
E81FA60200 call 00461CA0
```
`:00437681`
```
8BC7 mov eax, edi
```
`:00437683 8B4DF4`
```
mov ecx, dword ptr [ebp-0C]
```
`:00437686 64890D00000000`
```
mov dword ptr fs:[00000000], ecx
```
`:0043768D 5F`
```
pop edi
```
`:0043768E`
```
5E pop esi
```
`:0043768F 5B`
```
pop ebx
```
`:00437690 8BE5`
```
mov esp, ebp
```
`:00437692 5D`
```
pop ebp
```
`:00437693 C20C00`
```
ret 000C
```
********************************************************************************

最后讲一下如何取得字符串“注册码”在内存中的形式

在输入注册码时，我们注意到如果输入的注册码是错误的，程序会弹出一个`Messagebox`说：注册号码不对！

我们就从这个`Messagebox`的参数入手来取得“注册号码”的存放形式。

下断点`bpx Messageboxa`来到下面的地方：

* Referenced by a (U)nconditional or (C)onditional Jump
at Address:
|:0046CA9A(U)
|
`:0046CAB6 53`
```
push ebx
```
`:0046CAB7 57`
```
push edi
```
`:0046CAB8`
```
FF7508 push [ebp+08]
```
这里压入`lpText`，用`d (ebp+8)`查看，得到
`0187:01234700`
```
D7 A2 B2 E1 BA C5 C2 E0
```
这就是“注册号码”在内存中的形式，我们把第1、2、3、4、7、8个字节复制到`000B1B56`（`Raw Offset`）处
`:0046CABB FF75F4`
```
push [ebp-0C]
```
*
Reference To: `USER32.MessageBoxA`, Ord:01BEh
|
`:0046CABE`
```
FF1584964800 Call dword ptr [00489684]
```
`:0046CAC4`
```
85F6 test esi, esi
```
`:0046CAC6 8BF8`
```
mov edi, eax
```
`:0046CAC8 7405`
```
je 0046CACF
```
`:0046CACA 8B45F8`
```
mov eax, dword ptr [ebp-08]
```
`:0046CACD`
```
8906 mov dword ptr [esi], eax
```
这样我们就完成了对程序的修改。运行一下试试！

`cyclotron`
`2003.4.26`
