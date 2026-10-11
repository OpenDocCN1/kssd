
### 1. 清除使用过的 DLL 名、函数名

```
00408C3D    C1CB 10         ror ebx,10
00408C40    66:0BDB        or bx,bx
00408C43    74 0E          je short 00408C53
00408C45    C1CB 10         ror ebx,10
00408C48    803B 00        cmp byte ptr ds:[ebx],0
00408C4B    74 06          je short 00408C53
```

//修改⑥：`jmp 00408C53`

```
00408C4D    C603 43        mov byte ptr ds:[ebx],43
00408C50    43             inc ebx
00408C51    EB F5          jmp short 00408C48
00408C53    C3             retn
```

---

### 2. 填充系统函数地址或者加密地址

```
00408C54    80BD 1D120000 0>cmp byte ptr ss:[ebp+121D],1
00408C5B    74 09          je short 00408C66
00408C5D    80BD 1A120000 0>cmp byte ptr ss:[ebp+121A],0
00408C64    75 03          jnz short 00408C69
00408C66    8907           mov dword ptr ds:[edi],eax
```

//修改⑦：`NOP` 填充系统函数地址或者加密地址

```
00408C68    C3             retn
```

修改完毕后在 `00408BE0` 处下断，`Shift+F9` 中断后输入表处理完毕

现在我们直接在 `OllyDbg` 里面复制 `003C0000-003C0264` 处的 2 进制代码，粘贴进原来的 `ImPort Table VA＝00406000` 处。

---

## 四、结局：OEP、PE 修正、完成脱壳

搞定输入表后下面就开始走 `OEP` 了

```
00408BE3    8B85 47120000   mov eax,dword ptr ss:[ebp+1247]
00408BE9    EB 02           jmp short 00408BED
00408BED    3385 9F120000   xor eax,dword ptr ss:[ebp+129F]
```

//`EAX=11362854 XOR 113602DC=00002A88` 计算出 `OEP RVA` ★

```
00408BF3    EB 02           jmp short 00408BF7
00408BF7    0385 A7120000   add eax,dword ptr ss:[ebp+12A7]
00408BFD    EB 01           jmp short 00408C00
00408C00    894424 1C       mov dword ptr ss:[esp+1C],eax
00408C04    EB 03           jmp short 00408C09
00408C09    8DBD 3D0C0000   lea edi,dword ptr ss:[ebp+C3D]
00408C0F    EB 03           jmp short 00408C14
00408C14    B9 07070000     mov ecx,707
00408C19    EB 01           jmp short 00408C1C
00408C1C    32C0            xor al,al
00408C1E    EB 02           jmp short 00408C22
00408C22    F3:AA           rep stos byte ptr es:[edi]
```

//清扫战场

```
00408C24    EB 01           jmp short 00408C27
00408C27    8BFD            mov edi,ebp
00408C29    EB 02           jmp short 00408C2D
00408C2D    B9 350C0000     mov ecx,0C35
00408C32    EB 01           jmp short 00408C35
00408C35    F3:AA           rep stos byte ptr es:[edi]
```

//清扫战场

```
00408C37    61              popad
00408C38    EB 01           jmp short 00408C3B
00408C3B    FFE0            jmp eax  ; PESHIELD.00402A88
```

//飞向光明之巅

```
00402A88    E8 1EE7FFFF     call 004011AB
```

//OEP

```
00402A8D    E8 4BE7FFFF     call 004011DD
00402A92    E8 69E5FFFF     call 00401000
```

运行 `LordPE` 纠正 `ImageSize` 后完全 `Dump` 这个进程存为 `dumped.exe`

修正其 `OEP RVA=00002A88`、`ImPort Table RVA=00006000`

再删掉壳区段，清除壳数据。主程序没有资源，其它有的一般要恢复几个图标资源。

马马虎虎算是一个“完美脱壳”吧。

---

```
         ,     \_/ 
       /| \_.-~/          \\_     ,        青春都一晌
      ( /~   /             \\~-.\_ |\\
      \`\\\\  \_/             \\   ~\\ )           忍把浮名 
   \_-~~~-.)  )\_\_/;;,.        \\_  //\'
  /'\_,\   --~   \ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂 
  \`~ _( ,\_.--\ (     ,;'' /    ~--   /.\_\`\\ 
  /~~//'   / \`~\          ) /--..\_, )\_  \`~
  "  \`~"  "        `"      /~'\`\\    \`\\\\~~\\    
                           "    "   "~'  ""

               UnPacKed By :  fly
                  2005-05-17 18:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
