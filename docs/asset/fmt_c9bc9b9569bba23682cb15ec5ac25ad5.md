
#### `KERNEL32.GetPrivateProfileStringA`, Ord: 013Ah

```
:00401CF6 8B3534E14300        mov esi, dword ptr [0043E134]        <== 准备功能调用
```

`"winamp.ini"` <== 文件名

```
:00401CFC 687C214400          push 0044217C  
:00401D01 AA                  stosb                                 <== 到此将 Ebp-600 到 Ebp-200 清为 0  
:00401D02 8D8500FEFFFF        lea eax, dword ptr [ebp+FFFFFE00]     <== 缓冲区 1 首址(句柄?)  
:00401D08 6800020000          push 00000200                         <== 缓冲区长度为 200  
:00401D0D BB50914400          mov ebx, 00449150  
:00401D12 50                  push eax 
```

`"RegisteredTo"`

```
:00401D13 BF6C214400          mov edi, 0044216C                    <== 读内容  
:00401D18 53                  push ebx  
:00401D19 57                  push edi 
```

`"WinampRegInfo"`

```
:00401D1A 685C214400          push 0044215C                          <== 小节名  
:00401D1F FFD6                call esi 
```

这个 `call` 将注册信息（RI）读到 `Ebp-200` 处，后面以 `0` 补足至 `Ebp`。当然，`WA` 没有注册时，`RI` 是不存在的，自己输入的方式是：

在 `WIN9X` 系统目录下，找 `winamp.ini` 文件并打开（什么，没有？不会自己生成一个！），添加 `[WinampRegInfo]` 小节，输入 `RI`。我的 `winamp.ini` 相关内容如下：

```ini
[WinampRegInfo] 
RegisteredTo=Fpc!!!123456 
```

注意 `RI` 中，`Fpc` 是 `Rn`，`123456` 是 `Irc`，中间的 `"!!!"` 是必须的，在下面的分析中你会找到原因。

```
:00401D21 80BD00FEFFFF00      cmp byte ptr [ebp+FFFFFE00], 00       <== 检查 RI 是否为空  
:00401D28 7524                jne 00401D4E                          <== 正确则向下跳  
:00401D2A 68A0794500          push 004579A0  
:00401D2F 8D8500FEFFFF        lea eax, dword ptr [ebp+FFFFFE00]  
:00401D35 6800020000          push 00000200  
:00401D3A 50                  push eax  
:00401D3B 53                  push ebx  
:00401D3C 57                  push edi  
:00401D3D FF35E0594500        push dword ptr [004559E0]  
:00401D43 FFD6                call esi                              <== 不用关心是在读什么  
:00401D45 80BD00FEFFFF00      cmp byte ptr [ebp+FFFFFE00], 00  
:00401D4C 7466                je 00401DB4                           <== 这里千万不能去
```

#### Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address: `:00401D28(C)`

```
:00401D4E 8D8500FEFFFF        lea eax, dword ptr [ebp+FFFFFE00] <== 缓冲区 1 首址 
```

`"!!!"`

```
:00401D54 6858214400          push 00442158                          <== 准备检查 RI 中是否含有"!!!"  
:00401D59 50                  push eax  
:00401D5A E8F1330300          call 00435150                          <== 这个 Call 有意思，错误则 eax=0，正确则 eax 指向 RI 的!!!123456，我这里返回值是 6DC4FB  
:00401D5F 59                  pop ecx                                 <== 对这个 Call 的分析见（二）  
:00401D60 85C0                test eax, eax  
:00401D62 59                  pop ecx  
:00401D63 744F                je 00401DB4                           <== Eax=0 就去死  
:00401D65 802000              and byte ptr [eax], 00                <== 使得 Rn 以'\0'结束  
:00401D68 33F6                xor esi, esi  
:00401D6A 83C003              add eax, 00000003                     <== Eax 指向 Irc：123456 
```

#### Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address: `:00401D84(U)`

```
:00401D6D 8A08                mov cl, byte ptr [eax]                <== 取一个字符  
:00401D6F 80F930              cmp cl, 30                            <== 数字为有效字符，非数字表示完成  
:00401D72 7C12                jl 00401D86  
:00401D74 80F939              cmp cl, 39  
:00401D77 7F0D                jg 00401D86  
:00401D79 0FBEC9              movsx ecx, cl                         <== Ecx 中无用位清 0  
:00401D7C 8D14B6              lea edx, dword ptr [esi+4*esi]        <= Edx=5*Esi  
:00401D7F 40                  inc eax                               <== Edx 的初始值为 0，这三条指令将字符串变为数字  
:00401D80 8D7451D0            lea esi, dword ptr [ecx+2*edx-30]     <= 很有意思  
:00401D84 EBE7                jmp 00401D6D 
```

这一段以 `C` 来表示:

```c
char Irc[6]="123456"        /* 当然注册码不止 6 位    */  
long Temp=0;  
int i;

for( i=0; i<6; i++)  
{  
    if( !isdigit( Irc[i] ) )  
        break;  
    Temp*=10;  
    Temp+=Irc[i]-'0';  
}
```

#### Referenced by a `(U)nconditional` or `(C)onditional` Jump at Addresses: `:00401D72(C)`, `:00401D77(C)`

```
:00401D86 8D8500FEFFFF        lea eax, dword ptr [ebp+FFFFFE00] <== Eax 指向 Rn  
:00401D8C 50                  push eax  
:00401D8D E83F350200          call 004252D1                         <== !! 关键 Call，由 Rn 计算得出 Rc，保存在 Eax。具体分析请看（三）  
:00401D92 3BC6                cmp eax, esi                          <== 比较 Rc 与 Irc  
:00401D94 59                  pop ecx  
:00401D95 751D                jne 00401DB4                          <== 不相等则 GAME OVER，大家将此处 NOP 掉，看会发生什么有趣的事情  
:00401D97 50                  push eax  
:00401D98 8D8500FEFFFF        lea eax, dword ptr [ebp+FFFFFE00]  
:00401D9E 50                  push eax  
:00401D9F 8D8500FAFFFF        lea eax, dword ptr [ebp+FFFFFA00] 
```

`"----- LICENSED TO: %s (%d) -----"`

```
:00401DA5 6834214400          push 00442134  
:00401DAA 50                  push eax 
```

#### `USER32.wsprintfA`, Ord: 02ACh

```
:00401DAB FF1568E34300        Call dword ptr [0043E368]           <== 打印授权信息  
:00401DB1 83C410              add esp, 00000010 
```

#### Referenced by a `(U)nconditional` or `(C)onditional` Jump at Addresses: `:00401D4C(C)`, `:00401D63(C)`, `:00401D95(C)`

```
:00401DB4 80BD00FAFFFF00      cmp byte ptr [ebp+FFFFFA00], 00  
:00401DBB 5F                  pop edi  
:00401DBC 5E                  pop esi  
:00401DBD 5B                  pop ebx  
:00401DBE 751C                jne 00401DDC  
:00401DC0 6A00                push 00000000  
:00401DC2 6A00                push 00000000 
```

`"UNREGISTERED SHAREWARE VERSION - PLEASE REGISTER"`

```
:00401DC4 6A12                push 00000012  
:00401DC6 E8C5A10200          call 0042BF90  
:00401DCB 50                  push eax  
:00401DCC 8D8500FAFFFF        lea eax, dword ptr [ebp+FFFFFA00]  
:00401DD2 50                  push eax 
```

#### `USER32.wsprintfA`, Ord: 02ACh

```
:00401DD3 FF1568E34300        Call dword ptr [0043E368]           <== 打印未授权信息  
:00401DD9 83C414              add esp, 00000014                   <== 向下很长一段与分析无关 
```

#### Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address: `:00401DBE(C)`

```
:00401DDC 8D8500FAFFFF        lea eax, dword ptr [ebp+FFFFFA00]  
:00401DE2 50                  push eax 
```

`DialogID_009C`, `CONTROL_ID:0483`, `"License"`

```
:00401DE3 6883040000          push 00000483  
:00401DE8 FF7508              push [ebp+08] 
```

#### `USER32.SetDlgItemTextA`, Ord: 022Ch

```
:00401DEB FF1534E34300        Call dword ptr [0043E334]  
:00401DF1 C9                  leave  
:00401DF2 C3                  ret 
```

.........

`"!!!"`

```
:00401EAC 6858214400          push 00442158  
:00401EB1 50                  push eax  
:00401EB2 E829330300          call 004351E0  
:00401EB7 59                  pop ecx  
:00401EB8 8D8580FBFFFF        lea eax, dword ptr [ebp+FFFFFB80]  
:00401EBE 59                  pop ecx
```
