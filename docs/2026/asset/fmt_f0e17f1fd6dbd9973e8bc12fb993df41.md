
# 反汇编与 VB 逻辑对照

## 循环 1：匹配字符 “f”

```asm
:00403287  F401                            LitI2_Byte                 ;Push 01
:00403289  0464FF                          FLdRfVar                   ;Push LOCAL_009C
:0040328C  6C70FF                          ILdRf                      ;Push DWORD [LOCAL_0090]
:0040328F  4A                              FnLenStr                   ;vbaLenBstr
:00403290  E4                              CI2I4                      ;Verify [stack] high word is 0000, ECX=[ECX]
:00403291  FE63A0FEF30A                    ForI2                      ;// FOR J 循环
:00403297  2828FF0100                      LitVarI2                   ;PushVarInteger 0001    \ 取长度
:0040329C  6B64FF                          FLdI2                      ;Push WORD [LOCAL_009C] | 变量j的值
:0040329F  E7                              CI4UI1                     ;                       | // MID函数参数入栈
:004032A0  0470FF                          FLdRfVar                   ;Push LOCAL_0090        / 文本内容
:004032A3  4D48FF0840                      CVarRef                    ;// 创建临时变量
:004032A8  0418FF                          FLdRfVar                   ;Push LOCAL_00E8 // 加载临时变量
**********Reference To->msvbvm60.rtcMidCharVar
                               |
:004032AB  0A05001000                      ImpAdCallFPR4              ;Call ptr_00401036; check stack 0010; Push EAX // MID()
:004032B0  0418FF                          FLdRfVar                   ;Push LOCAL_00E8 // 将MID()的字符入栈
******Possible String Ref To->"f"
                               |
:004032B3  3A08FF5800                      LitVarStr                  ;PushVarString ptr_00402574 // "f"入栈
:004032B8  5D                              HardType                   ;
:004032B9  FB33                            EqVarBool                  ;比较字符是否相等
:004032BB  36040028FF18FF                  FFreeVar                   ;Free 0004/2 variants
:004032C2  1CEB0A                          BranchF                    ;If Pop=0 then ESI=004032CF // 不相等则跳004032CF        \ 到下面也就是不做字符连接
:004032C5  6C74FF                          ILdRf                      ;Push DWORD [LOCAL_008C] // 加载code变量                 |
******Possible String Ref To->"f"
                               |
:004032C8  1B5800                          LitStr                     ;Push ptr_00402574 // "f"入栈                            |
:004032CB  2A                              ConcatStr                  ;vbaStrCat //连接字符串                                  |
:004032CC  3174FF                          FStStr                     ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=Pop //释放空间 |
:004032CF  0464FF                          FLdRfVar                   ;Push LOCAL_009C                                         /
:004032D2  64A0FEB30A                      NextI2                      ;// 循环结束
```

对应 VB 原逻辑：

```vb
For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "f" Then code = code & "f"
Next j
```

往下看看下面分别是 `edcba9876543210` 的判断结构；如果 `Mid()` 到的字符等于判断的字符则作连接；作完这些后我们到最后看看，记住 `[LOCAL_008C]` = `code` 变量。

## 循环 2：匹配字符 “e”

```asm
:004032D7  F401                            LitI2_Byte                 ;Push 01
:004032D9  0464FF                          FLdRfVar                   ;Push LOCAL_009C
:004032DC  6C70FF                          ILdRf                      ;Push DWORD [LOCAL_0090]
:004032DF  4A                              FnLenStr                   ;vbaLenBstr
:004032E0  E4                              CI2I4                      ;Verify [stack] high word is 0000, ECX=[ECX]
:004032E1  FE639CFE430B                    ForI2                      ;
:004032E7  2828FF0100                      LitVarI2                   ;PushVarInteger 0001
:004032EC  6B64FF                          FLdI2                      ;Push WORD [LOCAL_009C]
:004032EF  E7                              CI4UI1                     ;
:004032F0  0470FF                          FLdRfVar                   ;Push LOCAL_0090
:004032F3  4D48FF0840                      CVarRef                    ;
:004032F8  0418FF                          FLdRfVar                   ;Push LOCAL_00E8
**********Reference To->msvbvm60.rtcMidCharVar
                               |
:004032FB  0A05001000                      ImpAdCallFPR4              ;Call ptr_00401036; check stack 0010; Push EAX
:00403300  0418FF                          FLdRfVar                   ;Push LOCAL_00E8
******Possible String Ref To->"e"
                               |
:00403303  3A08FF5600                      LitVarStr                  ;PushVarString ptr_00402560
:00403308  5D                              HardType                   ;
:00403309  FB33                            EqVarBool                  ;
:0040330B  36040028FF18FF                  FFreeVar                   ;Free 0004/2 variants
:00403312  1C3B0B                          BranchF                    ;If Pop=0 then ESI=0040331F
:00403315  6C74FF                          ILdRf                      ;Push DWORD [LOCAL_008C]
******Possible String Ref To->"e"
                               |
:00403318  1B5600                          LitStr                     ;Push ptr_00402560
:0040331B  2A                              ConcatStr                  ;vbaStrCat
:0040331C  3174FF                          FStStr                     ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=Pop
:0040331F  0464FF                          FLdRfVar                   ;Push LOCAL_009C
:00403322  649CFE030B                      NextI2
```

对应 VB 原逻辑：

```vb
For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "e" Then code = code & "e"
Next j
```

## 循环 3：匹配字符 “d”

```asm
:00403327  F401                            LitI2_Byte                 ;Push 01
:00403329  0464FF                          FLdRfVar                   ;Push LOCAL_009C
:0040332C  6C70FF                          ILdRf                      ;Push DWORD [LOCAL_0090]
:0040332F  4A                              FnLenStr                   ;vbaLenBstr
:00403330  E4                              CI2I4                      ;Verify [stack] high word is 0000, ECX=[ECX]
:00403331  FE6398FE930B                    ForI2                      ;
:00403337  2828FF0100                      LitVarI2                   ;PushVarInteger 0001
:0040333C  6B64FF                          FLdI2                      ;Push WORD [LOCAL_009C]
:0040333F  E7                              CI4UI1                     ;
:00403340  0470FF                          FLdRfVar                   ;Push LOCAL_0090
:00403343  4D48FF0840                      CVarRef                    ;
:00403348  0418FF                          FLdRfVar                   ;Push LOCAL_00E8
**********Reference To->msvbvm60.rtcMidCharVar
                               |
:0040334B  0A05001000                      ImpAdCallFPR4              ;Call ptr_00401036; check stack 0010; Push EAX
:00403350  0418FF                          FLdRfVar                   ;Push LOCAL_00E8
******Possible String Ref To->"d"
                               |
:00403353  3A08FF5400                      LitVarStr                  ;PushVarString ptr_0040254C
:00403358  5D                              HardType                   ;
:00403359  FB33                            EqVarBool                  ;
:0040335B  36040028FF18FF                  FFreeVar                   ;Free 0004/2 variants
```
