
```assembly
:0042D801 8D9568FFFFFF           lea edx, dword ptr [ebp+FFFFFF68]  
:0042D807 51                    push ecx  
:0042D808 8D8578FFFFFF          lea eax, dword ptr [ebp+FFFFFF78]  
:0042D80E 52                    push edx  
:0042D80F 8D4D88                lea ecx, dword ptr [ebp-78]  
:0042D812 50                    push eax  
:0042D813 8D5598                lea edx, dword ptr [ebp-68]  
:0042D816 51                    push ecx  
:0042D817 8D45A8                lea eax, dword ptr [ebp-58]  
:0042D81A 52                    push edx  
:0042D81B 8D4DB8                lea ecx, dword ptr [ebp-48]  
:0042D81E 50                    push eax  
:0042D81F 8D55C8                lea edx, dword ptr [ebp-38]  
:0042D822 51                    push ecx  
:0042D823 52                    push edx  
:0042D824 6A0A                  push 0000000A
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h  
                                  |  
:0042D826 FF1530104000          Call dword ptr [00401030]  
:0042D82C 83C42C                add esp, 0000002C  
:0042D82F C3                   ret
```

### 算法总结

一、取用户号，将前 3 位换到最后。如 `894112337` -> `112337894`

二、依次求循环次数的立方值

三、依次取变化后的用户号的数字，求其立方值

四、两者依次相加

五、相加之和 依次 与 `A` 求模！每次求模的结果连接起来就是注册码了！

### 完美爆破

`0042BC85`
```assembly
1BF6                  sbb esi, esi  
```
改为：
```assembly
33F6                  xor esi, esi  
```

### KeyMake 之{68th}内存注册机

`中断地址`：`0042BC6C`  
`中断次数`：`1`  
`第一字节`：`8B`  
`指令长度`：`2`

`内存方式`：`EAX`  

`宽字符串`

### 注册信息保存

`C:\WINDOWS\SYSTEM` 下的 `vfnlubfvt.sys` 文件

### 整理

`用户号`：`894112337`  
`注册码`：`295129513`

```text
        ,     _/   
      /| \_.-~/           \_     ,        青春都一饷  
      ( /~   /              ~-._ |
      `\\   /_/               ~\\ )          忍把浮名   
      _-~~~-. )  )\_/;;,.          \_  //'  
      /'\_,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.          换了破解轻狂  
      `~ _( ,_..--\ (      ,;'' /    ~--   /._`\  
      /~~//'   /' `~\           ) /--.._ , )_  `~  
      "  `~"  "       `"       /~'`\   `\\~~\   
                               "      "   "~'  ""
```

Cracked By 巢水工作坊——fly [OCN][FCG]

2003-04-23 2:00

```pascal
var  
 L, i, Key1, Key: Integer;  
 S, Temp, Temp1, Temp2, RegCode: String;

begin  
 S := Edit1.Text;  
 L := Length(S);  
 Temp := Copy(S, 1, 3);  
 Temp1 := Copy(S, 4, L);  
 Temp2 := Temp1 + Temp;  
 S := Temp2;  
 SetLength(RegCode, L);  
 RegCode := '';  
 for i := 1 to L do  
 begin  
 Key1 := StrToInt(s[i]);  
 Key := i * i * i + Key1 * Key1 * Key1;  
 Key := Key mod $0a;  
 RegCode := RegCode + IntToStr(Key);  
 end;  
 Edit2.Text := RegCode;  
end; 
```

`4dd466d0e1e0cc123afbe504da2462ca`
