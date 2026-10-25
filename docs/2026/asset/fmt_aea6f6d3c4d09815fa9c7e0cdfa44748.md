
### 汇编分析

`:004BB01C 8AD0`
`mov dl, al`
将取出的字符 ascii 给 `dl`

`:004BB01E 8BCA`
`mov ecx, edx`
再给 `ecx`

`:004BB020 0FAFCA`
`imul ecx, edx`
`ecx * edx`，也就是将取出字符的 ascii 码开方！

`:004BB023 8D4317`
`lea eax, dword ptr [ebx+17]`
`ebx+17` 中依次是 `18`、`19`……传给 `eax`

`:004BB026 F7EE`
`imul esi`
乘 `esi`，这里第一次 `esi` 是 `0`,

`:004BB028 03C8`
`add ecx, eax`
`ecx + eax` 相加

`:004BB02A 8BF1`
`mov esi, ecx`
保存运算结果到 `esi` 中

`:004BB02C 43`
`inc ebx`
计数，循环多少次？名字的字符串个数！

`:004BB02D FF4DF0`
`dec [ebp-10]`

`:004BB030 75CB`
`jne 004BAFFD`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004BAFF3(C)

`:004BB032 8B45F8`
`mov eax, dword ptr [ebp-08]`
这个是将输入的注册码 Hex 值给 `eax`

`:004BB035 2D4D050000`
`sub eax, 0000054D`
`eax - 0x54D` 传给 `eax`

`:004BB03A 3BF0`
`cmp esi, eax`
看结果是不是等于 `esi`（上面运算出的值）

`:004BB03C 7508`
`jne 004BB046`
不能跳哟！

`:004BB03E 85F6`
`test esi, esi`

`:004BB040 7404`
`je 004BB046`

`:004BB042 B301`
`mov bl, 01`
看到这个了吧！哈哈哈哈！舒服呀！

`:004BB044 EB02`
`jmp 004BB048`

### 算法

以姓名 `TAE!` 为例

`A = 0x54^2 + 0x18 * 0`
`B = 0x41^2 + 0x19 * A`
`C = 0x45^2 + 0x20 * B`
`D = 0x21^2 + 0x21 * C`
`SN = D + 0x54D`

### 得到注册码的简单方法

到了 `004BB032` 那一行，可以用下命令 `? esi + 54D`，即可得到正确注册码！！

### 一个可用的注册码

Name：`TAE!`
Sn：`126929743`

下面这个 ascii 涂鸦就是用这个软件做的，呵呵！

                    .----.
                \_.'\_\_    `.
            .--(#)(##)---/#\
          .' @          /###\
          :        ,  #####
            `-..\_\_.-' \_.-\###/
                  `;\_:
`"'
                .'"""""`.
              /,  JOE  ,\
              //  COOL!  \\
              `-.\_\_\_\_\_\_\_.-'
              \_\_\_`. | .'\_\_\_
            (\_\_\_\_\_\_|\_\_\_\_\_\_)
