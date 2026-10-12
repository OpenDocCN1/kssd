
:10001136 `E895000000`                call `100011D0`

:1000113B `E840010000`                call `10001280`

:10001140 `33C9`                      xor `ecx`, `ecx`

:10001142 `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

//这两个 `CALL` 会根据序列号的前 8 位数字经过查表运算得来一系列值，也就内定了前八个字符必须是数字。详细内容见后面。

计算出来八个值：`04 0C 09 04 06 0A 0A 0B`，放在 `1000CBB0` 处。

:10001148 `8A81B0CB0010`              mov `al`, byte ptr `[ecx+1000CBB0]`    //取第一个

:1000114E `8A91B4CB0010`              mov `dl`, byte ptr `[ecx+1000CBB4]`    //取第四个

:10001154 `32C2`                      xor `al`, `dl`                  //先异或

//八个值中，第一个值和第四个值异或，再和最后一个值或，结果如不是数字或者字母则加 7 变成字母，比较后通过则继续计算第二个和第五个，并且仍和最后一个或，依此类推。

:10001156 `8A15B7CB0010`              mov `dl`, byte ptr `[1000CBB7]`         //一直都是取最后一个

:1000115C `0AC2`                      or `al`, `dl`                   //再或

:1000115E `0C30`                      or `al`, `30`

:10001160 `3C39`                      cmp `al`, `39`

:10001162 `8881B8CB0010`              mov byte ptr `[ecx+1000CBB8]`, `al`    //存结果

:10001168 `7E08`                      jle `10001172`

:1000116A `0407`                      add `al`, `07`                  //变字母

:1000116C `8881B8CB0010`              mov byte ptr `[ecx+1000CBB8]`, `al`    //也是存结果

//这里的 `AL` 是算出来的一个字符，算出一个后马上跟输入的序列号的最后四位比较。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001168(C)

|

:10001172 `8A81B8CB0010`              mov `al`, byte ptr `[ecx+1000CBB8]`

:10001178 `8A9188CB0010`              mov `dl`, byte ptr `[ecx+1000CB88]`

//这是输入序列号的后四位地址。前八位数字计算出来的四个字符必须和序列号末尾的四个字母一样才行。

:1000117E `3AC2`                      cmp `al`, `dl`

:10001180 `7406`                      je `10001188`                //比！

:10001182 `0C20`                      or `al`, `20`

:10001184 `3AC2`                      cmp `al`, `dl`

:10001186 `7514`                      jne `1000119C`               //不等就完蛋了

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001180(C)

|

:10001188 `41`                        inc `ecx`

:10001189 `83F904`                    cmp `ecx`, `00000004`         //比完四对就……

:1000118C `7CBA`                      jl `10001148`

:1000118E `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

:10001194 `B801000000`                mov `eax`, `00000001`         //正确标志

:10001199 `5F`                        pop `edi`

:1000119A `5E`                        pop `esi`

:1000119B `C3`                        ret

这里程序算出来的四个字符是“`BFBF`”

——————————————————————————————

这里是第一个 `CALL` 的内容，它的功能也就是先根据输入的前 8 个字符查一张表，如果查出来的东西符合要求（也就是前 8 个字符都是数字）则把前 8 个字符的 `ASCII` 码都转换成实际的数字存放在 `1000CBB8` 处：

* Referenced by a `CALL` at Address:

|:10001136

|

:100011D0 `53`                        push `ebx`

:100011D1 `33D2`                      xor `edx`, `edx`

:100011D3 `33DB`                      xor `ebx`, `ebx`

:100011D5 `56`                        push `esi`

:100011D6 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

:100011DC `33C9`                      xor `ecx`, `ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001240(U)

|

:100011DE `A18C930010`                mov `eax`, dword ptr `[1000938C]`

:100011E3 `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

:100011E9 `83F801`                    cmp `eax`, `00000001`

:100011EC `7E20`                      jle `1000120E`

//也不知是什么判断，反正总是跳。

:100011EE `0FBE8180CB0010`            movsx `eax`, byte ptr `[ecx+1000CB80]`

:100011F5 `6A04`                      push `00000004`

:100011F7 `50`                        push `eax`

:100011F8 `E883080000`                call `10001A80`

:100011FD `8B15A8CB0010`              mov `edx`, dword ptr `[1000CBA8]`

:10001203 `8B0DACCB0010`              mov `ecx`, dword ptr `[1000CBAC]`

:10001209 `83C408`                    add `esp`, `00000008`

:1000120C `EB13`                      jmp `10001221`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100011EC(C)

|

:1000120E `0FBE8180CB0010`            movsx `eax`, byte ptr `[ecx+1000CB80]`    //表首地址

* Possible StringData Ref from Data Obj ->"             (((((              "

                                          ->"  H 剟剟剟剟剟亖亖亖"

//第一张表的内容                    ->"倐倐倐"

                                          ->" "

                                          |

:10001215 `8B3580910010`              mov `esi`, dword ptr `[10009180]`

:1000121B `8A0446`                    mov `al`, byte ptr `[esi+2*eax]`

:1000121E `83E004`                    and `eax`, `00000004`

//下面说明前八个字符必须都是数字。后四个字符根据这些数字来计算。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000120C(U)

|

:10001221 `3BC3`                      cmp `eax`, `ebx`

:10001223 `741D`                      je `10001242`          //如果输入的不是数字，则这里会跳。

:10001225 `83F908`                    cmp `ecx`, `00000008`    //满了 8 个则跳。

:10001228 `7D18`                      jge `10001242`

:1000122A `8A8180CB0010`              mov `al`, byte ptr `[ecx+1000CB80]`

:10001230 `240F`                      and `al`, `0F`

:10001232 `8882B0CB0010`              mov byte ptr `[edx+1000CBB0]`, `al`

:10001238 `42`                        inc `edx`

:10001239 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

:1000123F `41`                        inc `ecx`

:10001240 `EB9C`                      jmp `100011DE`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001223(C), :10001228(C)

|

:10001242 `83FA08`                    cmp `edx`, `00000008`

:10001245 `7D33`                      jge `1000127A`

:10001247 `4A`                        dec `edx`

:10001248 `B807000000`                mov `eax`, `00000007`

:1000124D `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000126D(C)

|

:10001253 `3BD3`                      cmp `edx`, `ebx`

:10001255 `7D08`                      jge `1000125F`

:10001257 `8898B0CB0010`              mov byte ptr `[eax+1000CBB0]`, `bl`

:1000125D `EB0D`                      jmp `1000126C`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001255(C)

|

:1000125F `8A8AB0CB0010`              mov `cl`, byte ptr `[edx+1000CBB0]`

:10001265 `4A`                        dec `edx`

:10001266 `8888B0CB0010`              mov byte ptr `[eax+1000CBB0]`, `cl`

//这里 `1000cbb0` 处执行完后就存放了前 8 位数字的实际值，也就是

`05 01 00 01 02 03 04 05`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000125D(U)

|

:1000126C `48`                        dec `eax`

:1000126D `79E4`                      jns `10001253`

:1000126F `A3ACCB0010`                mov dword ptr `[1000CBAC]`, `eax`

:10001274 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

//这里存放俩处理过的字节数，都是 8。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001245(C)

|

:1000127A `5E`                        pop `esi`

:1000127B `5B`                        pop `ebx`

:1000127C `C3`                        ret

下面是第二个 `CALL` 的内容，这个 `CALL` 把第一个 `CALL` 计算出来的数字再查一次表，结果覆盖了 `1000cbb0` 处的 8 个字节：

* Referenced by a `CALL` at Address:

|:1000113B

|

:10001280 `33C0`                      xor `eax`, `eax`

:10001282 `A3AC910010`                mov dword ptr `[100091AC]`, `eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012A7(C)

|

:10001287 `0FBE88B0910010`            movsx `ecx`, byte ptr `[eax+100091B0]`

:1000128E `8B148590800010`            mov `edx`, dword ptr `[4*eax+10008090]`    //又查一次表

:10001295 `40`                        inc `eax`

:10001296 `83F808`                    cmp `eax`, `00000008`               //处理 8 个字符

:10001299 `8A0C0A`                    mov `cl`, byte ptr `[edx+ecx]`

:1000129C `A3AC910010`                mov dword ptr `[100091AC]`, `eax`

:100012A1 `8888AF910010`              mov byte ptr `[eax+100091AF]`, `cl`    //存查表结果

:100012A7 `7CDE`                      jl `10001287`

:100012A9 `C3`                        ret                             //返回

查出来的结果仍然放在 `1000cbb0`，这里变成了 `04 0C 09 04 06 0A 0A 0B`

最后得可以用的序列号： `5101-2345BF-BF`

说明：此段和 `SoftICE` 的序列号破解过程除了指令定位地址不一样外，其他几乎一模一样，所以连说明文字都类似，`*_*`
