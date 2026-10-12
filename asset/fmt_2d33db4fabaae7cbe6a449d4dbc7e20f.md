
:100011E0 890DAC910010                     mov dword ptr [100091AC], ecx

:100011E6 83F801                         cmp eax, 00000001

:100011E9 7E20                           jle 1000120B

//也不知是什么判断，反正总是跳。

:100011EB 0FBE8180910010                 movsx eax, byte ptr [ecx+10009180]

:100011F2 6A04                           push 00000004

:100011F4 50                             push eax

:100011F5 E8C4070000                     call 100019BE

:100011FA 8B15A8910010                   mov edx, dword ptr [100091A8]

:10001200 8B0DAC910010                   mov ecx, dword ptr [100091AC]

:10001206 83C408                         add esp, 00000008

:10001209 EB13                           jmp 1000121E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100011E9(C)

|

:1000120B 0FBE8180910010                 movsx eax, byte ptr [ecx+10009180]    //表首址

//这里是第一张表

* Possible StringData Ref from Data Obj ->"                   (((((                    "

                                                                 ->"  H 剟剟剟剟剟亖亖亖"

                                                                 ->"倐倐倐"

                                                                 ->" "

                                                                 |

:10001212 8B3560810010                   mov esi, dword ptr [10008160]

:10001218 8A0446                         mov al, byte ptr [esi+2*eax]        //查表

:1000121B 83E004                         and eax, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001209(U)

|

:1000121E 85C0                           test eax, eax

:10001220 741D                           je 1000123F        //如果输入的不是数字，则这里会跳。

:10001222 83F908                         cmp ecx, 00000008  //满了 8 个则跳。

:10001225 7D18                           jge 1000123F

:10001227 8A8180910010                   mov al, byte ptr [ecx+10009180]

:1000122D 240F                           and al, 0F

:1000122F 8882B0910010                   mov byte ptr [edx+100091B0], al

:10001235 42                             inc edx

:10001236 8915A8910010                   mov dword ptr [100091A8], edx

:1000123C 41                             inc ecx

:1000123D EB9C                           jmp 100011DB

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001220(C), :10001225(C)

|

:1000123F 83FA08                         cmp edx, 00000008

:10001242 5E                             pop esi

:10001243 7D34                           jge 10001279

:10001245 4A                             dec edx

:10001246 B807000000                     mov eax, 00000007

:1000124B 8915A8910010                   mov dword ptr [100091A8], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000126C(C)

|

:10001251 85D2                           test edx, edx

:10001253 7D09                           jge 1000125E

:10001255 C680B091001000                 mov byte ptr [eax+100091B0], 00

:1000125C EB0D                           jmp 1000126B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001253(C)

|

:1000125E 8A8AB0910010                   mov cl, byte ptr [edx+100091B0]

:10001264 4A                             dec edx

:10001265 8888B0910010                   mov byte ptr [eax+100091B0], cl

//这里 100091b0 处执行完后就存放了前 8 位数字的实际值，也就是

05 01 00 00 00 00 09 09

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000125C(U)

|

:1000126B 48                             dec eax

:1000126C 79E3                           jns 10001251

:1000126E A3AC910010                     mov dword ptr [100091AC], eax

:10001273 8915A8910010                   mov dword ptr [100091A8], edx

//这里存放俩处理过的字节数，都是 8。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001243(C)

|

:10001279 C3                             ret

下面是第二个 CALL 的内容，这个 CALL 把第一个 CALL 计算出来的数字再查一次表，结果覆盖了 100091b0 处的 8 个字节：

* Referenced by a CALL at Address:

|:1000113B

|

:10001280 33C0                           xor eax, eax

:10001282 A3AC910010                     mov dword ptr [100091AC], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012A7(C)

|

:10001287 0FBE88B0910010                 movsx ecx, byte ptr [eax+100091B0]

:1000128E 8B148590800010                 mov edx, dword ptr [4*eax+10008090]   //又查一次表

:10001295 40                             inc eax

:10001296 83F808                         cmp eax, 00000008        //处理 8 个字符

:10001299 8A0C0A                         mov cl, byte ptr [edx+ecx]

:1000129C A3AC910010                     mov dword ptr [100091AC], eax

:100012A1 8888AF910010                   mov byte ptr [eax+100091AF], cl      //存查表结果

:100012A7 7CDE                           jl 10001287

:100012A9 C3                             ret                          //返回

查出来的结果仍然放在 100091b0，这里变成了 04 0C 09 03 0D 07 06 08

最后得可以用的序列号： 5100-000099-9BFB

需要说明的是，这一大段是检验从注册表里读出来的序列号，不是检验输入的。——其实都大同小异嘛，不信，在反编译的 `sice405.txt` 中再查找一下“`function143`”看看，还有这一处被调用了：

`label185`:

00664E:0152:       `RegDBGetKeyValueEx(lString0, "Serial", lNumber1, lString1, lNumber3)`;

006665:0021:       `lNumber4 = LAST_RESULT`;

00666D:0128:       `lNumber5 = lNumber4 = 0`;

00667F:0022:       if (`lNumber5 = 0`) then

                            goto `label187`;

                      endif;

00668D:0013:       `lString3 = lString1`;

`label186`:

006699:0152:       `RegDBGetKeyValueEx(lString0, "Onomatopoeia", lNumber1, lString1, lNumber3)`;

0066B6:0021:       `lNumber4 = LAST_RESULT`;

0066BE:0128:       `lNumber5 = lNumber4 = 0`;

0066D0:0022:       if (`lNumber5 = 0`) then

                            goto `label188`;

                      endif;

0066DE:006D:       `StrToNum(lNumber0, lString1)`;

0066E6:0122:       `lNumber0 = lNumber0 >> 13`;

0066F3:006E:       `NumToStr(lString1, lNumber0)`;

//推测应该是序列号前 8 位转化成纯数字值再右移 13 位后成的数字必须能通过下面的这个函数检验。

`label187`:

0066FF:00B4:       `NMINST32.ValidCookie(2, 0, lString1)`;

//这是多出来的一重检验函数，但运行时 `nminst32.dll` 的这个输出函数无论如何都跟不进去（不断），也不知道是怎么回事。

006711:0021:       `lNumber5 = LAST_RESULT`;

006719:0128:       `lNumber5 = lNumber5 = 1`;

00672B:0022:       if (`lNumber5 = 0`) then

                            goto `label190`;

                      endif;

006739:00B5:       `function143(lString3)`;

//这里就是检验输入序列号是否合法的，检验形式基本一样，只不过前面多了一重束缚。

006744:0021:       `lNumber5 = LAST_RESULT`;

00674C:0128:       `lNumber5 = lNumber5 = 1`;

00675E:0022:       if (`lNumber5 = 0`) then

                            goto `label189`;

                      endif;

00676C:0013:       `string13 = lString3`;

006774:012F:       `return(1)`;

00677D:002C:       goto `label190`;

还有一点，前 8 位数字不是随便填什么都能在 `10001140` 处中断的，估计就是那个 `NMINST32.ValidCookie(2, 0, lString1);` 搞的鬼。我在跟踪过程中却不见程序执行到这个函数的入口，也不知道是什么原因。

另外 `SoftICE4.05` 是要找出能用的序列号才需要这么麻烦地跟，倘若只想用暴力跳过检验安装成功，只需来一个 `bpx messageboxA` 再看看上面第一个 `JNZ` 的指令就行，实在不难。

再写几个随便跟出来的、可以用的序列号（在已知某些可用的序列号的基础上，;P ）

`5107-0145BF-BB`

`5107-0045BB-BB`

`1107-0745FF-BF`

其实这个破得也不算完美，只是时间紧，写得不好，大伙儿包涵包涵。——这还是皮特陈大客“点将”的“恶果”，有气儿找他去，我躲了。哈！  `4dd466d0e1e0cc123afbe504da2462ca`
