
```asm
004DCD27    inc ebx                                   ; 计数器 EBX+1  
004DCD28    dec esi                                   ; 计数器 ESI-1  
004DCD29    jnz 2005gaok.004DCC50                     ; 继续大循环  
004DCD2F    mov eax,dword ptr ss:[ebp+8]              ; 设置缓存  
004DCD32    mov edx,dword ptr ss:[ebp-1C]             ; 得到的字符串  
004DCD35    call 2005gaok.00403D20                    ; 字符串转换移。  
004DCD3A    xor ecx,ecx                                ; 以下清空数据.....  
004DCD3C    mov edx,5  
004DCD41    mov eax,dword ptr ss:[ebp-4]  
004DCD44    call 2005gaok.004D9C6C  
004DCD49    xor eax,eax  
004DCD4B    pop edx  
004DCD4C    pop ecx  
004DCD4D    pop ecx  
004DCD4E    mov dword ptr fs:[eax],edx  
004DCD51    push 2005gaok.004DCD7B  
004DCD56    lea eax,dword ptr ss:[ebp-24]  
004DCD59    call 2005gaok.00403CCC  
004DCD5E    lea eax,dword ptr ss:[ebp-1C]  
004DCD61    call 2005gaok.00403CCC  
004DCD66    lea eax,dword ptr ss:[ebp-C]  
004DCD69    mov edx,2  
004DCD6E    call 2005gaok.00403CF0  
004DCD73    retn  
004DCD74    jmp 2005gaok.004036E0  
004DCD79    jmp short 2005gaok.004DCD56  
004DCD7B    pop edi  
004DCD7C    pop esi  
004DCD7D    pop ebx  
004DCD7E    mov esp,ebp  
004DCD80    pop ebp  
004DCD81    retn 4  
```

前四次计算结果每次得 2 位数（余数<99)，后 5 次计算结果每次得 1 位数（余数<9)，一共 13 位注册码，再与关键字符`**`连接。

`VB` 做的注册机（可以用 `Word` 宏运行）：

```vb
sub main() 

A = 机器码 

A = Right(A, 9) 

B = 密钥 

A1 = 0: B1 = 0 

For i = 0 To 8 
  A1 = A1 + Asc(Mid(A, i + 1, 1)) 
  B1 = B1 + Asc(Mid(B, i + 1, 1)) 

Next 

char = "" 

For i = 1 To 9 
   R2 = i + 5 
   R2 = R2 * Asc(Mid(B, i, 1)) 
   R2 = R2 + i 
   R2 = R2 * Asc(Mid(A, i, 1)) 
   D = i * 2 * 66 
   R2 = R2 + D 
   C = (2 * i * i + 13) * A1 
   R2 = R2 + C 
   D = i * 3 + 12 
   R2 = R2 + D * B1 
   C = 230 - i 
   D = R2 Mod C 
   If i <= 4 Then 
      R2 = D Mod 99 
      char = char & R2 
   Else 
      R2 = D Mod 9 
      char = char & R2 
   End If 

Next 

InputBox "您的注册码为：", "Greet", ** & char 

End Sub
```

```asm
004DCCA3    cdq                                       ; EDX 清零
```

这一句是不正确,

这是一个扩展数位的操作符

可以看看下面的:

### `CDQ` 指令

**指令格式**: `CDQ`

**指令功能**: 将 `EAX` 中的数符号扩展为 `EDX:EAX` 中的 64 位数, `EDX` 存放高 32 位, `Q` 表示 8 个字节.

**指令说明**: 该指令一般用在累加器参加算术运算之前, 以符合运算规则或防止溢出.

`CDQ` 是扩展指令，将 `EDX` 扩展为 `EAX` 的高位，除了上面兄弟说的之外，`CDQ` 在扩展时也分有符号数和无符号，如果 `EAX` 是有符号数，`EDX` 将被扩展成有符号数“`0FFFFFFFFh`”，如果 `EAX` 是无符号数，`EDX` 将被清 0。

`4dd466d0e1e0cc123afbe504da2462ca`
