
```
110744AD  MOV EAX,1                                        ; i=1
110744B2  MOV DWORD PTR SS:[EBP-58],EAX                    ；i
110744B5  MOV EDI,DWORD PTR DS:[11001388]                  ; MSVBVM60.__vbaFreeStr
	110744BB  MOV ECX,0A                                       ; 外循环==>
	110744C0  CMP EAX,ECX                                      ; i>10?
	110744C2  JG 11074637
	110744C8  MOV ECX,0FF                                      ; FF
	110744CD  CALL NEAR DWORD PTR DS:[110011F0]                ; MSVBVM60.__vbaUI1I2
	110744D3  MOV BYTE PTR SS:[EBP-48],AL                      ; tmp=FF
	110744D6  MOV ESI,1                                        ; j=1
		110744DB  MOV EAX,0A                                       ; 内循环==>
		110744E0  CMP ESI,EAX                                      ; j>10?
		110744E2  JG 110745A4
		110744E8  MOV DWORD PTR SS:[EBP-6C],1
		110744EF  MOV DWORD PTR SS:[EBP-74],2
		110744F6  LEA ECX,DWORD PTR SS:[EBP-54]
		110744F9  MOV DWORD PTR SS:[EBP-C0],ECX
		110744FF  MOV DWORD PTR SS:[EBP-C8],4008
		11074509  LEA EDX,DWORD PTR SS:[EBP-74]
		1107450C  PUSH EDX
		1107450D  PUSH ESI
		1107450E  LEA EAX,DWORD PTR SS:[EBP-C8]
		11074514  PUSH EAX
		11074515  LEA ECX,DWORD PTR SS:[EBP-84]
		1107451B  PUSH ECX
		1107451C  CALL NEAR DWORD PTR DS:[11001158]                ; 读取一位
		11074522  LEA EDX,DWORD PTR SS:[EBP-84]
		11074528  PUSH EDX
		11074529  LEA EAX,DWORD PTR SS:[EBP-94]
		1107452F  PUSH EAX
		11074530  CALL NEAR DWORD PTR DS:[1100117C]                ; 转化为大写
		11074536  LEA ECX,DWORD PTR SS:[EBP-94]
		1107453C  PUSH ECX
		1107453D  LEA EDX,DWORD PTR SS:[EBP-64]
		11074540  PUSH EDX
		11074541  CALL NEAR DWORD PTR DS:[1100125C]                ; MSVBVM60.__vbaStrVarVal
		11074547  PUSH EAX
		11074548  CALL NEAR DWORD PTR DS:[1100106C]                ; 取ASCII码
		1107454E  MOV ECX,EAX
		11074550  CALL NEAR DWORD PTR DS:[110011F0]                ; MSVBVM60.__vbaUI1I2
		11074556  MOV BL,AL                                        ; 转移
		11074558  LEA ECX,DWORD PTR SS:[EBP-64]
		1107455B  CALL NEAR EDI
		1107455D  LEA EAX,DWORD PTR SS:[EBP-94]
		11074563  PUSH EAX
		11074564  LEA ECX,DWORD PTR SS:[EBP-84]
		1107456A  PUSH ECX
		1107456B  LEA EDX,DWORD PTR SS:[EBP-74]
		1107456E  PUSH EDX
		1107456F  PUSH 3
		11074571  CALL NEAR DWORD PTR DS:[1100105C]                ; MSVBVM60.__vbaFreeVarList
		11074577  ADD ESP,10
		1107457A  CMP BL,BYTE PTR SS:[EBP-48]                      ; str<=tmp 谁大？
		1107457D  JNB SHORT 1107458D
			1107457F  MOV BYTE PTR SS:[EBP-48],BL                      ; tmp<==str 小的替换大的
			11074582  MOV ECX,ESI
			11074584  CALL NEAR DWORD PTR DS:[110011A0]                ; MSVBVM60.__vbaI2I4
			1107458A  MOV DWORD PTR SS:[EBP-44],EAX                    ; 暂存j
		1107458D  MOV EAX,1
		11074592  ADD EAX,ESI
		11074594  JO 11074B1E
		1107459A  MOV ESI,EAX                                      ；j++
		1107459C  MOV EBX,DWORD PTR SS:[EBP+8]
		1107459F  JMP 110744DB                                     ; 内循环<==
	110745A4  MOV AX,WORD PTR SS:[EBP-44]                      ; j
	110745A8  CWD
	110745AA  MOV CX,0A
	110745AE  IDIV CX
	110745B1  MOVSX ESI,DX                                     ; j mod 10
	110745B4  CMP ESI,0B
	110745B7  JB SHORT 110745BF
	110745B9  CALL NEAR DWORD PTR DS:[11001184]                ; MSVBVM60.__vbaGenerateBoundsError
	110745BF  MOV EAX,DWORD PTR SS:[EBP-58]                    ; i
	110745C2  CDQ
	110745C3  MOV ECX,0A
	110745C8  IDIV ECX
	110745CA  MOV ECX,EDX                                      ; i mod 10
	110745CC  CALL NEAR DWORD PTR DS:[11001210]                ; MSVBVM60.__vbaUI1I4
	110745D2  MOV EDX,DWORD PTR DS:[EBX+194]                   ; 数组地址
	110745D8  MOV BYTE PTR DS:[EDX+ESI],AL                     ; 保存索引到数组
	110745DB  PUSH 0FF
	110745E0  LEA EAX,DWORD PTR SS:[EBP-74]
	110745E3  PUSH EAX
	110745E4  CALL NEAR DWORD PTR DS:[11001248]                ; MSVBVM60.rtcVarBstrFromAnsi
	110745EA  LEA ECX,DWORD PTR SS:[EBP-54]
	110745ED  PUSH ECX
	110745EE  MOVSX EDX,WORD PTR SS:[EBP-44]
	110745F2  PUSH EDX
	110745F3  PUSH 1
	110745F5  LEA EAX,DWORD PTR SS:[EBP-74]
	110745F8  PUSH EAX
	110745F9  CALL NEAR DWORD PTR DS:[1100104C]                ; MSVBVM60.__vbaStrVarMove
	110745FF  MOV EDX,EAX
	11074601  LEA ECX,DWORD PTR SS:[EBP-64]
	11074604  CALL NEAR DWORD PTR DS:[1100133C]                ; MSVBVM60.__vbaStrMove
	1107460A  PUSH EAX
	1107460B  PUSH 0
	1107460D  CALL NEAR DWORD PTR DS:[1100137C]                ; FF替换第j位
	11074613  LEA ECX,DWORD PTR SS:[EBP-64]
	11074616  CALL NEAR EDI
	11074618  LEA ECX,DWORD PTR SS:[EBP-74]
	1107461B  CALL NEAR DWORD PTR DS:[11001038]                ; MSVBVM60.__vbaFreeVar
	11074621  MOV EAX,1
	11074626  ADD EAX,DWORD PTR SS:[EBP-58]
	11074629  JO 11074B1E
	1107462F  MOV DWORD PTR SS:[EBP-58],EAX
	11074632  JMP 110744BB                                     ; 外循环<==
```

## 基本规律：

对字符串中的字符在本字符串中的大小按原序存放求出顺序号。如：`ABCDE`，则顺序号为`12345`；`BACDE`，则顺序号为`21345`等等。结果填充数组。

利用上述方法分别对注册码的第四段、第三段求出对应的数组，为后续的换位提供依据。

## 换位方法：

这部分的代码较长，免去代码部分，直接给出仿真代码：

假设：

```
    B=array(1,2,3,4,5)
    n = Len(in2)   'in2为数字串
    ztmp = Space(n)'作等长的空格串
    t = 1
    For j = 1 To 5
        index = b(j)
        For i = 1 To n
            If index <= 6 Then
                Mid(ztmp, index, 1) = Mid(in2, t, 1)
                index = index + 5
                t = t + 1
            End If
        Next
        If t = n + 1 Then Exit For '如果全部做过交换则退出
    Next
```

## 关于字符索引表：

字符索引表就是每个单元存放的是字符串的数组，其存放的数字串右1位的，也有两位的，但并非规律存放，也不是所有的数字串都存在。这个数组的产生代码较长，但对于一个ActiveX来说是固定的。不过对于VB的东西来说，观察起来稍微麻烦。如果不找到合适的位置，很难知道数组中的元素是什么。

建立字符索引表使用一个字符串和一个数字串数组：

字符串：`"ECGRAPHDDD"`。这个是软换合成的。

数字串数组：

```
11073F64  MOV DWORD PTR SS:[EBP-C0],11008170               ；"0"
11073F9A  MOV DWORD PTR SS:[EBP-D0],1100CEC8               ："1"
11073FD2  MOV DWORD PTR SS:[EBP-E0],1100D090               ; UNICODE "31"
11073FFE  MOV DWORD PTR SS:[EBP-F0],1100CD08               ; UNICODE "32"
1107402A  MOV DWORD PTR SS:[EBP-100],1100C444              ; UNICODE "33"
11074056  MOV DWORD PTR SS:[EBP-110],1100B810              ; "6"
11074082  MOV DWORD PTR SS:[EBP-120],1100D09C              ; UNICODE "34"
110740AE  MOV DWORD PTR SS:[EBP-130],1100D0A8              ; UNICODE "35"
110740DA  MOV DWORD PTR SS:[EBP-140],1100D0B4              ; UNICODE "36"
11074103  MOV DWORD PTR SS:[EBP-150],110094E4              ; "9"
1107412F  MOV DWORD PTR SS:[EBP-160],1100D0C0              ; UNICODE "37"
1107415B  MOV DWORD PTR SS:[EBP-170],1100D0CC              ; UNICODE "38"
11074187  MOV DWORD PTR SS:[EBP-180],1100D0D8              ; UNICODE "39"
110741B3  MOV DWORD PTR SS:[EBP-190],1100D0E4              ; UNICODE "30"
110741DF  MOV DWORD PTR SS:[EBP-1A0],1100D0F0              ; "5"
1107420B  MOV DWORD PTR SS:[EBP-1B0],1100D0F8              ；"4"
11074237  MOV DWORD PTR SS:[EBP-1C0],1100D100              ; UNICODE "71"
11074263  MOV DWORD PTR SS:[EBP-1D0],1100D10C              ; UNICODE "72"
1107428F  MOV DWORD PTR SS:[EBP-1E0],11008FC0              ; "0"
110742BB  MOV DWORD PTR SS:[EBP-1F0],1100D118              ; "8"
110742E7  MOV DWORD PTR SS:[EBP-200],1100D120              ; "2"
11074313  MOV DWORD PTR SS:[EBP-210],1100D128              ; UNICODE "73"
1107433F  MOV DWORD PTR SS:[EBP-220],1100D134              ; UNICODE "74"
1107436B  MOV DWORD PTR SS:[EBP-230],1100D140              ; UNICODE "75"
11074397  MOV DWORD PTR SS:[EBP-240],1100D14C              ; UNICODE "76"
110743C3  MOV DWORD PTR SS:[EBP-250],1100D158              ; UNICODE "77"
110743EF  MOV DWORD PTR SS:[EBP-260],1100D164              ; UNICODE "78"
1107441B  MOV DWORD PTR SS:[EBP-270],1100D170              ; UNICODE "79"
11074447  MOV DWORD PTR SS:[EBP-280],1100D17C              ; UNICODE "70"
```

其中一个数字的字符串在OD中观察不到，是我添加进来的。

上述数据不是在最终结果中直接使用，而是构造字符索引数组使用的。构造出的字符索引数组的内容仍是数字串，内容对这个控件来说是固定的。

## 产生最终结果字符串方法：

假设字符索引数组已经制作好，下面的代码将利用字符索引数组来产生最终结果：
