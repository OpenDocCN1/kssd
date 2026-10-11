
## 二、注册码验证逻辑分析

:004CA3EA E8C9D7FFFF call 004C7BB8 -->注册码转换为字串 2

:004CA3EF 663BF8 cmp di, ax

:004CA3F2 0F8574010000 jne 004CA56C -->注册码为字母或长度为 1,则跳转至注册失败

:004CA3F8 A1344E5400 mov eax, dword ptr [00544E34]

:004CA3FD BAFF010000 mov edx, 000001FF

:004CA402 E87DD6FFFF call 004C7A84 -->计算字串 1 累加和

:004CA407 8BF8 mov edi, eax

:004CA409 A1C44C5400 mov eax, dword ptr [00544CC4]

:004CA40E BAFF010000 mov edx, 000001FF

:004CA413 E86CD6FFFF call 004C7A84 -->计算字串 2 累加和

:004CA418 3BF8 cmp edi, eax

:004CA41A 0F854C010000 jne 004CA56C -->是否相等？ ***

- 不相等，则跳转至注册失败；
- 相等，继续则注册 OK！

*(修改为 909090909090 nop(6 个）*)

:004CA420 8D55CC lea edx, dword ptr [ebp-34]

:004CA423 8B06 mov eax, dword ptr [esi]

:004CA425 8B80E4020000 mov eax, dword ptr [eax+000002E4]

:004CA42B E87CF3F6FF call 004397AC

:004CA430 8B55CC mov edx, dword ptr [ebp-34]

:004CA433 A16C505400 mov eax, dword ptr [0054506C]

:004CA438 E83F9BF3FF call 00403F7C

:004CA43D 8D55C8 lea edx, dword ptr [ebp-38]

:004CA440 8B06 mov eax, dword ptr [esi]

:004CA442 8B80E8020000 mov eax, dword ptr [eax+000002E8]

:004CA448 E85FF3F6FF call 004397AC

:004CA44D 8B55C8 mov edx, dword ptr [ebp-38]

:004CA450 A1244D5400 mov eax, dword ptr [00544D24]

:004CA455 E8229BF3FF call 00403F7C

:004CA45A 8BC3 mov eax, ebx

:004CA45C E89BF9FFFF call 004C9DFC

*Possible StringData Ref from Code Obj -> "Thank You for registering"*

:004CA461 B8C8A64C00 mov eax, 004CA6C8

:004CA466 E81569F9FF call 00460D80

:004CA46B B201 mov dl, 01

:004CA46D A11CB84100 mov eax, dword ptr [0041B81C]

:004CA472 E81115F5FF call 0041B988

:004CA477 8BD8 mov ebx, eax

:004CA479 B101 mov cl, 01

*(略)*

:004CA56A EB14 jmp 004CA580

*Possible StringData Ref from Code Obj -> "Sorry, but Name or Registration number is wrong !"*

:004CA56C B8DCA74C00 mov eax, 004CA7DC

:004CA571 E80A68F9FF call 00460D80

:004CA576 A1E44B5400 mov eax, dword ptr [00544BE4]

:004CA57B 8B55FC mov edx, dword ptr [ebp-04]

:004CA57E 8910 mov dword ptr [eax], edx

从以上程序段可看出，只要用 `ULTRAEDIT` 等工具将 `***` 处作相应修改即可任意注册。

*(注册码不能为字母、注册码长度不能为 1)*

## 三、启动画面校验逻辑分析

注册成功后，重新运行程序，出现启动画面，显示 `self test` 后，出现提示 `Program was corrupted !` 画面，只有退出程序。

再在 `pwasm32` 反汇编文件中查找以上字符，相关程序如下：

:0053E19A 8B55E8 mov edx, dword ptr [ebp-18]

:0053E19D A1944C5400 mov eax, dword ptr [00544C94]

:0053E1A2 8B00 mov eax, dword ptr [eax]

:0053E1A4 8B80D0020000 mov eax, dword ptr [eax+000002D0]

:0053E1AA 8B8008020000 mov eax, dword ptr [eax+00000208]

:0053E1B0 8B08 mov ecx, dword ptr [eax]

:0053E1B2 FF5134 call [ecx+34] -->提示 "self test"

:0053E1B5 E89A45F8FF call 004C2754 -->程序代码校验

:0053E1BA 8B15244C5400 mov edx, dword ptr [00544C24]

:0053E1C0 3B82B4000000 cmp eax, dword ptr [edx+000000B4]

:0053E1C6 740F je 0053E1D7 -->比较校验结果：***

- 不符，则提示出错信息；
- 相符，则跳转程序正常初始化。

*(修改为 EB0f jmp 0053e1d7)*

*Possible StringData Ref from Code Obj -> "Program was corrupted !"*

:0053E1C8 B81CEA5300 mov eax, 0053EA1C

:0053E1CD E8AE2BF2FF call 00460D80

:0053E1D2 E996070000 jmp 0053E96D

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

`|:0053E1C6(C)`

:0053E1D7 68E4E95300 push 0053E9E4

:0053E1DC E82FD4ECFF call 0040B610

:0053E1E1 83C4F8 add esp, FFFFFFF8

:0053E1E4 DD1C24 fstp qword ptr [esp]

:0053E1E7 9B wait

:0053E1E8 8D45DC lea eax, dword ptr [ebp-24]

:0053E1EB E834E0ECFF call 0040C224

:0053E1F0 FF75DC push [ebp-24]

*Possible StringData Ref from Code Obj -> "] App Init .." -->程序初始化*

:0053E1F3 683CEA5300 push 0053EA3C

:0053E1F8 8D45E0 lea eax, dword ptr [ebp-20]

:0053E1FB BA03000000 mov edx, 00000003

:0053E200 E86360ECFF call 00404268

将上段程序 `***` 处作相应修改，程序即可正常运行。

## 四、注册比较逻辑分析

发现程序启动时，在下列程序段进行注册比较：（查找 `4C7A84` 时发现）

:0053C7C6 8B45F4 mov eax, dword ptr [ebp-0C]

:0053C7C9 E8CEB2F8FF call 004C7A9C

:0053C7CE 8B45F0 mov eax, dword ptr [ebp-10]

:0053C7D1 E8E2B3F8FF call 004C7BB8

:0053C7D6 A1344E5400 mov eax, dword ptr [00544E34]

:0053C7DB BAFF010000 mov edx, 000001FF

:0053C7E0 E89FB2F8FF call 004C7A84 -->计算用户名字串 1 累加和

:0053C7E5 8BF0 mov esi, eax

:0053C7E7 A1C44C5400 mov eax, dword ptr [00544CC4]

:0053C7EC BAFF010000 mov edx, 000001FF

:0053C7F1 E88EB2F8FF call 004C7A84 -->计算注册码字串 2 累加和

:0053C7F6 3BF0 cmp esi, eax

:0053C7F8 751C jne 0053C816 -->比较是否相等？ ***

- 不相等在 `license` 窗口显示 `Unregistration`
- 相等则在 `license` 窗口显示注册信息。

*(修改为 9090 nop（2 个）)*

:0053C7FA A16C505400 mov eax, dword ptr [0054506C]

:0053C7FF 8B55F4 mov edx, dword ptr [ebp-0C]

:0053C802 E87577ECFF call 00403F7C

:0053C807 A1244D5400 mov eax, dword ptr [00544D24]

:0053C80C 8B55F0 mov edx, dword ptr [ebp-10]

:0053C80F E86877ECFF call 00403F7C

:0053C814 EB2D jmp 0053C843

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

`|:0053C7F8(C)`

:0053C816 A1344E5400 mov eax, dword ptr [00544E34]

:0053C81B 33C9 xor ecx, ecx

:0053C81D BA00020000 mov edx, 00000200

:0053C822 E89565ECFF call 00402DBC

从以上程序段可看出，只要将 `***` 处作相应修改即可。

错误之处请指正，谢谢！！

`<Crack123>` `4dd466d0e1e0cc123afbe504da2462ca`
