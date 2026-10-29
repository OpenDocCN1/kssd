
最后看关键的判断炸弹是否爆炸的逻辑。`4011DF`这里会按照重新排列好的链表顺序，判断它们的`n2`成员是否从大到小排列，不是则爆炸。

```assembly
.text:00000000004011DF loc\_4011DF: ; CODE XREF: phase\_6+101↓j
.text:00000000004011DF mov rax, [rbx+8] ; rbx = arrLinkListNodes[0]
.text:00000000004011DF ; rax = rbx->next
.text:00000000004011E3 mov eax, [rax]
.text:00000000004011E5 cmp [rbx], eax ; if (rbx->n2 < rbx->next->n2) bomb
.text:00000000004011E7 jge short loc\_4011EE
.text:00000000004011E9 call explode\_bomb
```

让我们再回看一下数据窗口的链表：

```
hex view:
00000000006032D0 000000010000014C 00000000006032E0
00000000006032E0 00000002000000A8 00000000006032F0
00000000006032F0 000000030000039C 0000000000603300
0000000000603300 00000004000002B3 0000000000603310
0000000000603310 00000005000001DD 0000000000603320
0000000000603320 00000006000001BB 0000000000000000
```

它们的`n2`依次为`14c`, `a8`, `39c`, `2b3`, `1dd`, `1bb`, 写个脚本排列一下：

```python
dictNodes = {
0x14C : 1,
0xa8 : 2,
0x39c : 3,
0x2b3 : 4,
0x1dd : 5,
0x1bb : 6
}
def dict_val(item):
return item[0]
dictNewNodes = {k : v for k, v in sorted(dictNodes.items(), key=dict_val, reverse=True)}
print("input key: 7-index:")
for k, v in dictNewNodes.items():
# print("0x%04x-" % k, v, end="")
print("%d "% (7-v), end="") # 4 3 2 1 6 5
```

key : 4 3 2 1 6 5

## 目录

- phase 1
- phase 2
- phase 3
- phase 4
- phase 5
- phase 6

`Bomb`这个程序是《深入理解计算机系统》的配套实验的一部分，一共6关。

原地址：

- `498K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4K6j5i4m8H3i4K6u0W2j5%4y4Q4x3X3g2U0L8i4g2Q4x3X3g2W2k6s2g2Q4x3V1j5K6k6g2)9J5c8X3I4S2j5Y4y4Q4x3X3g2Z5N6r3#2D9`
- `029K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4K6j5i4m8H3i4K6u0W2j5%4y4Q4x3X3g2U0L8i4g2Q4x3X3g2W2k6s2g2Q4x3V1j5K6k6g2)9J5c8X3u0G2L8h3u0Q4x3X3g2@1j5i4t1`.

环境：

- Ubuntu 18.04
- IDA 远程调试

# phase 1

第一关热身题，输入一段明文保存的字符串：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `.text:``0000000000400EE4` `mov     esi, offset aBorderRelation     ;` `"Border relations with Canada have never"``...`  `.text:``0000000000400EE9` `call    strings_not_equal`  `.text:``0000000000400EEE` `test    eax, eax`  `.text:``0000000000400EF0` `jz      short loc_400EF7`  `.text:``0000000000400EF2` `call    explode_bomb` |

key: Border relations with Canada have never been better.

# phase 2

要求输入6个数：

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `.text:``000000000040145C` `public read_six_numbers`  `.text:``000000000040145C` `read_six_numbers proc near`  `...`  `.text:``0000000000401480` `mov     esi, offset aDDDDDD             ;` `"%d %d %d %d %d %d"`  `.text:``0000000000401485` `mov     eax,` `0`  `.text:``000000000040148A` `call    ___isoc99_sscanf` |

输入的6个数字会存在数组里，临时取名`input`。接下来`rbx`执行`input[1]`，`rbp`指向`input[6]`（用来结束循环）：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `.text:``0000000000400F30` `loc_400F30:`  `.text:``0000000000400F30`                                         `; phase_2``+``19``↑j`  `.text:``0000000000400F30` `lea     rbx, [rsp``+``38h``+``var_34]           ; rbx` `=` `pNum` `=` `&``input``[``1``]`  `.text:``0000000000400F35` `lea     rbp, [rsp``+``38h``+``var_20]           ; rbp` `=` `&``input``[``6``]` `=` `pEnd`  `.text:``0000000000400F3A` `jmp     short loopCompare` |

`input[i]`与`input[i-1] * 2`比较，相等即可，所以输入一个等比数列即可。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `.text:``0000000000400F17` `loopCompare:`  `.text:``0000000000400F17`                                         `; phase_2``+``3E``↓j`  `.text:``0000000000400F17` `mov     eax, [rbx``-``4``]                    ; eax` `=` `*``(``-``-``pNum)`  `.text:``0000000000400F1A` `add     eax, eax                        ; eax` `*``=` `2`  `.text:``0000000000400F1C` `cmp`     `[rbx], eax`  `.text:``0000000000400F1E` `jz      short loc_400F25                ;` `if` `*``pNum` `=``=` `2``*``eax` `continue`  `.text:``0000000000400F20` `call    explode_bomb`  `.text:``0000000000400F25` `;` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-`  `.text:``0000000000400F25`  `.text:``0000000000400F25` `loc_400F25:`  `.text:``0000000000400F25` `add     rbx,` `4`  `.text:``0000000000400F29` `cmp`     `rbx, rbp                        ;` `if` `i` `=``=` `len``(``input``)` `break`  `.text:``0000000000400F2C` `jnz     short loopCompare               ; eax` `=` `*``(``-``-``pNum)`  `.text:``0000000000400F2E` `jmp     short loc_400F3C` |

key: 1 2 4 8 16 32

# phase 3

输入两个数 `num1`, `num2`

|  |  |

| --- | --- |

| 1  2  3 | `.text:``0000000000400F51` `mov     esi, (offset aDDDDDD``+``0Ch``)       ;` `"%d %d"`  `.text:``0000000000400F56` `mov     eax,` `0`  `.text:``0000000000400F5B` `call    ___isoc99_sscanf` |

有一个长度为8的跳转表，所以会检查`num1<=7`, 之后跳转到`jpt_400F75[num1]`, 每个`case`都有一个数，与`num2`相等则通过

|  |  |

| --- | --- |

| 1  2  3  4  5 | `.text:``0000000000400F6A` `loc_400F6A:`  `.text:``0000000000400F6A` `cmp`     `[rsp``+``18h``+``var_10],` `7`             `; switch` `8` `cases`  `.text:``0000000000400F6F` `ja      short def_400F75                ; jumptable` `0000000000400F75` `default case`  `.text:``0000000000400F71` `mov     eax, [rsp``+``18h``+``var_10]`  `.text:``0000000000400F75` `jmp     ds:jpt_400F75[rax``*``8``]            ; switch jump` |

比如下面这个`num1==0`的`case`:

|  |  |

| --- | --- |

| 1  2  3  4 | `.text:``0000000000400F7C` `loc_400F7C:`  `.text:``0000000000400F7C`                                         `; DATA XREF: .rodata:jpt_400F75↓o`  `.text:``0000000000400F7C` `mov     eax,` `0CFh`                       `; jumptable` `0000000000400F75` `case` `0`  `.text:``0000000000400F81` `jmp     short loc_400FBE` |

所以输入 `0 207`（`0xcf==207`） 即可，其它7个`case`也可以，所以应该是有8对数作为`flag`。

# phase 4

还是输入两个数`num1 num2`, 要求`num1 <= 0xe`

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `.text:``000000000040101A` `mov     esi, (offset aDDDDDD``+``0Ch``)       ;` `"%d %d"`  `.text:``000000000040101F` `mov     eax,` `0`  `.text:``0000000000401024` `call    ___isoc99_sscanf`  `.text:``0000000000401029` `cmp`     `eax,` `2`  `.text:``000000000040102C` `jnz     short loc_401035`  `.text:``000000000040102E` `cmp`     `[rsp``+``18h``+``var_10],` `0Eh`           `; num` `1` `<``=` `0xe`  `.text:``0000000000401033` `jbe     short loc_40103A`  `.text:``0000000000401035`  `.text:``0000000000401035` `call    explode_bomb` |

然后调用一个`func4(0xe, 0, num1)`, 要求返回0，并且`num2`也需要为0：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `.text:``000000000040103A` `mov     edx,` `0Eh`  `.text:``000000000040103F` `mov     esi,` `0`  `.text:``0000000000401044` `mov     edi, [rsp``+``18h``+``var_10]           ; edi` `=` `num1`  `.text:``0000000000401048` `call    func4`  `.text:``000000000040104D` `test    eax, eax`  `.text:``000000000040104F` `jnz     short loc_401058                ;` `if` `func4() !``=` `0`    `bomb`  `.text:``0000000000401051` `cmp`     `[rsp``+``18h``+``var_C],` `0`              `;` `if` `num2!``=``0`  `bomb`  `.text:``0000000000401056` `jz      short loc_40105D` |

分析一下`func4`函数，这是一个递归函数：

|  |  |

| --- | --- |
