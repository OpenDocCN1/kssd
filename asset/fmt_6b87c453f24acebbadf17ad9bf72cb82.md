
## 汇编代码分析

`0046E95A` `. DD45 DC        FLD QWORD PTR SS:[EBP-24]                //取:[EBP-24] 值送 EAX。`

`0046E95D` `. DC0D 10314000  FMUL QWORD PTR DS:[403110]              //与:[403110] 内的浮点数形式的常数'7'相乘。`

`0046E963` `. DD5D DC        FSTP QWORD PTR SS:[EBP-24]              //保存回原位置。`

`0046E966` `. DFE0           FSTSW AX                                 `

`0046E968` `. A8 0D          TEST AL,0D                               `

`0046E96A` `. 0F85 8F000000  JNZ BOOKEXPO.0046E9FF                    `

`0046E970` `. ^EB B6         JMP SHORT BOOKEXPO.0046E928              //继续循环上去。`

如果经过乘 `7` 后的注册码串长达到 `8` 位，则从 `0046E958` 蹦到这里来。

`0046E972` `> 8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]           //16 进制浮点数形式的结果分别送 EAX（高 4 字节），ECX（低 4 字节）。`

`0046E975` `. 8B4D DC        MOV ECX,DWORD PTR SS:[EBP-24]           `

`0046E978` `. 6A 08          PUSH 8                                    `

`0046E97A` `. 50             PUSH EAX                                   `

`0046E97B` `. 51             PUSH ECX                                   `

`0046E97C` `. FFD3           CALL EBX                                    `

`0046E97E` `. 8BD0           MOV EDX,EAX                                  `

`0046E980` `. 8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]             `

`0046E983` `. FFD7           CALL EDI                                    `

`0046E985` `. 50             PUSH EAX                                   `

`0046E986` `. FF15 AC124000  CALL DWORD PTR DS:[<&MSVBVM60.#616>];    MSVBVM60.rtcLeftCharBstr  //复制已经转换为字符的注册码到另一个位置。`

`0046E98C` `. 8BD0           MOV EDX,EAX                        //真注册码地址在 EAX 内。                     `

`0046E98E` `. 8D4D E4        LEA ECX,DWORD PTR SS:[EBP-1C]             `

`0046E991` `. FFD7           CALL EDI                                    `

`0046E993` `. 8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]             `

`0046E996` `. FF15 08134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr`

`0046E99C` `> 9B             WAIT                                        `

`0046E99D` `. 68 D8E94600    PUSH BOOKEXPO.0046E9D8                   `

`0046E9A2` `. EB 33          JMP SHORT BOOKEXPO.0046E9D7              `

`========================================================================================`

## 后记

这个小软件虽然是 `VB` 写的，界面非常简单。但是注册码的计算都在主程序内，所以，相对比较简单。通过跟踪这个软件，我明白了浮点数用法和指令，以前很少看到这方面的资料，一般都是整数运算，可是这个软件都是浮点运算。这个注册码的计算是根据用户名得到的，如果得到的注册码的长度不够 `8` 位则继续乘以 `7`，如果到达 `8` 位，则得到正确注册码了。简单计算过程如下，注册机可以很容易写出：

假如用户名为 `iloveu`，则首先在前面加上 `2` 个字符小写的`"l"`和大写的`"M"`。形成一个新串`"lMiloveu"`。

```
SUM=89715；

for(ESI=3;ESI<=8;ESI++)    

{

1.计算用户名第 i 位和第 i+2 位乘积，得结果 EAX;

2.用 SUM 除以 ESI，得余数送 EDX;

3.EDX=EDX+1，ESI=ESI+1;

4.EDX=EAX*EDX。

5.SUM+=EDX。

}

do{

断 SUM 的位数，如果小于 8，则乘以 7。如果等于 8 则结束循环。

}while(1);
```

## 结论

| Username | RegisterCode |
| :--- | :--- |
| `wanggang` | `24105501` |
| `iloveu` | `14669865` |

该程序在注册表内的 `H.L.M\\Software\\KLMsoft\\BookmarkExporter` 键内写入 `RegCode` 和 `RegName` 子键。

感谢您浪费时间阅读此文！恭祝所有看雪坛友新春快乐！！

`qduwg`

`qduwg@163.com`

`2006` 年 `1` 月 `25` 日结稿 Ethiopia

`4dd466d0e1e0cc123afbe504da2462ca`
