
# 入门习作2：HOSTMONITOR 1.31 运行自校验及注册破解过程 (11千字)

程序名称：`HOSTMONITOR 1.31`

保护方式：注册码、运行时代码自校验

破解过程如下：

## 一、启动程序后，进入licence 的注册窗口

任意输入用户名及注册码，点击确定后出现“Sorry, but Name or Registration number is wrong !”画面，

用`pwdasm32`反汇编主程序`hostmonitor.exe`后，查找上面的字符串，发现下列代码段与注册有关：

```
:004CA33C 8B45E8
mov eax, dword ptr [ebp-18]
:004CA33F E86C81FFFF              call 004C24B0
-->判断用户名是否在HACKER
:004CA344 84C0
  test al, al      名单中
:004CA346 7441
  je 004CA389  -->不是则跳转4CA389处验证
      用户名及注册码有效性***
（*修改为 EB41
jmp 004ca389 *)
* Possible StringData Ref from Code Obj ->"Sorry, but your registration name"
                  ->"("
            |
:004CA348 68ECA54C00              push 004CA5EC
:004CA34D 8D55E0
lea edx, dword ptr [ebp-20]
:004CA350 8B06
  mov eax, dword ptr [esi]
:004CA352 8B80E4020000            mov eax, dword
ptr [eax+000002E4]
:004CA358 E84FF4F6FF              call 004397AC
:004CA35D FF75E0
push [ebp-20]
* Possible StringData Ref from Code Obj ->") found in "black list".
"
            |
:004CA360 6818A64C00              push 004CA618
* Possible StringData Ref from Code Obj ->"Should you have any questions, "
                  ->"please don`t
hesitate to let us "
                  ->"know.
"
            |
:004CA365 6840A64C00              push 004CA640
* Possible StringData Ref from Code Obj ->"E-Mail: line1@ks-soft.net; line2@ks-soft.net"
            |
:004CA36A 6890A64C00              push 004CA690
:004CA36F 8D45E4
lea eax, dword ptr [ebp-1C]
:004CA372 BA05000000              mov edx,
00000005
:004CA377 E8EC9EF3FF              call 00404268
:004CA37C 8B45E4
mov eax, dword ptr [ebp-1C]
:004CA37F E8FC69F9FF              call 00460D80
:004CA384 E9F7010000              jmp 004CA580
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CA346(C)
|
:004CA389 8D55DC
lea edx, dword ptr [ebp-24]
:004CA38C 8B06
  mov eax, dword ptr [esi]
:004CA38E 8B80E4020000            mov eax, dword
ptr [eax+000002E4]
:004CA394 E813F4F6FF              call 004397AC
-->计算字符串长度
:004CA399 837DDC00                cmp
dword ptr [ebp-24], 00000000
:004CA39D 0F84C9010000            je 004CA56C
-->用户名长度是否为0
      （跳转则注册失败）
:004CA3A3 8D55D8
lea edx, dword ptr [ebp-28]
:004CA3A6 8B06
  mov eax, dword ptr [esi]
:004CA3A8 8B80E8020000            mov eax, dword
ptr [eax+000002E8]
:004CA3AE E8F9F3F6FF              call 004397AC
:004CA3B3 837DD800                cmp
dword ptr [ebp-28], 00000000
:004CA3B7 0F84AF010000            je 004CA56C
-->注册码长度是否为0
      （跳转则注册失败）
:004CA3BD 8D55D4
lea edx, dword ptr [ebp-2C]
:004CA3C0 8B06
  mov eax, dword ptr [esi]
:004CA3C2 8B80E4020000            mov eax, dword
ptr [eax+000002E4]
:004CA3C8 E8DFF3F6FF              call 004397AC
:004CA3CD 8B45D4
mov eax, dword ptr [ebp-2C]
:004CA3D0 E8C7D6FFFF              call 004C7A9C
-->用户名转换为字串1
:004CA3D5 8BF8
  mov edi, eax
:004CA3D7 8D55D0
lea edx, dword ptr [ebp-30]
:004CA3DA 8B06
  mov eax, dword ptr [esi]
:004CA3DC 8B80E8020000            mov eax, dword
ptr [eax+000002E8]
:004CA3E2 E8C5F3F6FF              call 004397AC
:004CA3E7 8B45D0
mov eax, dword ptr [ebp-30]
:004CA3EA E8C9D7FFFF              call 004C7BB8
-->注册码转换为字串2
:004CA3EF 663BF8
cmp di, ax
:004CA3F2 0F8574010000            jne 004CA56C
-->注册码为字母或长度
      为1,则跳转至注册失败
:004CA3F8 A1344E5400              mov eax,
dword ptr [00544E34]
:004CA3FD BAFF010000              mov edx,
000001FF
:004CA402 E87DD6FFFF              call 004C7A84
-->计算字串1累加和
:004CA407 8BF8
  mov edi, eax
:004CA409 A1C44C5400              mov eax,
dword ptr [00544CC4]
:004CA40E BAFF010000              mov edx,
000001FF
:004CA413 E86CD6FFFF              call 004C7A84
-->计算字串2累加和
:004CA418 3BF8
  cmp edi, eax
:004CA41A 0F854C010000            jne 004CA56C
-->是否相等？  ***
      不相等，则跳转至注册失败；
      相等，继续则注册OK！
(*修改为 909090909090            nop(6个）*）
:004CA420 8D55CC
lea edx, dword ptr [ebp-34]
:004CA423 8B06
  mov eax, dword ptr [esi]
:004CA425 8B80E4020000            mov eax, dword
ptr [eax+000002E4]
:004CA42B E87CF3F6FF              call 004397AC
:004CA430 8B55CC
mov edx, dword ptr [ebp-34]
:004CA433 A16C505400              mov eax,
dword ptr [0054506C]
:004CA438 E83F9BF3FF              call 00403F7C
:004CA43D 8D55C8
lea edx, dword ptr [ebp-38]
:004CA440 8B06
  mov eax, dword ptr [esi]
:004CA442 8B80E8020000            mov eax, dword
ptr [eax+000002E8]
:004CA448 E85FF3F6FF              call 004397AC
:004CA44D 8B55C8
mov edx, dword ptr [ebp-38]
:004CA450 A1244D5400              mov eax,
dword ptr [00544D24]
:004CA455 E8229BF3FF              call 00403F7C
:004CA45A 8BC3
  mov eax, ebx
:004CA45C E89BF9FFFF              call 004C9DFC
* Possible StringData Ref from Code Obj ->"Thank You for registering"
            |
:004CA461 B8C8A64C00              mov eax,
004CA6C8
:004CA466 E81569F9FF              call 00460D80
:004CA46B B201
  mov dl, 01
:004CA46D A11CB84100              mov eax,
dword ptr [0041B81C]
:004CA472 E81115F5FF              call 0041B988
:004CA477 8BD8
  mov ebx, eax
:004CA479 B101
  mov cl, 01
  .
  .
  .
  .
(略)                    .
  .
  .
:004CA56A EB14
  jmp 004CA580
* Possible StringData Ref from Code Obj ->"Sorry, but Name or Registration "
                  ->"number is
wrong !"
            |
:004CA56C B8DCA74C00              mov eax,
004CA7DC
:004CA571 E80A68F9FF              call 00460D80
:004CA576 A1E44B5400              mov eax,
dword ptr [00544BE4]
:004CA57B 8B55FC
mov edx, dword ptr [ebp-04]
:004CA57E 8910
  mov dword ptr [eax], edx
```

从以上程序段可看出，只要用`ULTRAEDIT`等工具将***处作相应修改即可任意注册。

（*注册码不能为字母、注册码长度不能为1）

## 二、注册成功后，重新运行程序

重新运行程序，出现启动画面，显示“self test"后，出现提示“Program was corrupted !”画面，只有退出程序。

再在`pwasm32`反汇编文件中查找以上字符，相关程序如下：

```
:0053E19A 8B55E8
mov edx, dword ptr [ebp-18]
:0053E19D A1944C5400              mov eax,
dword ptr [00544C94]
:0053E1A2 8B00
  mov eax, dword ptr [eax]
:0053E1A4 8B80D0020000            mov eax, dword
ptr [eax+000002D0]
:0053E1AA 8B8008020000            mov eax, dword
ptr [eax+00000208]
:0053E1B0 8B08
  mov ecx, dword ptr [eax]
:0053E1B2 FF5134
call [ecx+34]  -->提示“self test"
:0053E1B5 E89A45F8FF              call 004C2754
-->程序代码校验
:0053E1BA 8B15244C5400            mov edx, dword
ptr [00544C24]
:0053E1C0 3B82B4000000            cmp eax, dword
ptr [edx+000000B4]
:0053E1C6 740F
  je 0053E1D7    -->比较校验结果：***
        不符，则提示出错信息；
        相符，则跳转程序正常初始化。
(*修改为  EB0f
  jmp 0053e1d7 *)
* Possible StringData Ref from Code Obj ->"Program was corrupted !"
            |
:0053E1C8 B81CEA5300              mov eax,
0053EA1C
:0053E1CD E8AE2BF2FF              call 00460D80
:0053E1D2 E996070000              jmp 0053E96D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0053E1C6(C)
|
:0053E1D7 68E4E95300              push 0053E9E4
:0053E1DC E82FD4ECFF              call 0040B610
:0053E1E1 83C4F8
add esp, FFFFFFF8
:0053E1E4 DD1C24
fstp qword ptr [esp]
:0053E1E7 9B
  wait
:0053E1E8 8D45DC
lea eax, dword ptr [ebp-24]
:0053E1EB E834E0ECFF              call 0040C224
:0053E1F0 FF75DC
push [ebp-24]
* Possible StringData Ref from Code Obj ->"] App Init .."
            |
          -->程序初始化
:0053E1F3 683CEA5300              push 0053EA3C
:0053E1F8 8D45E0
lea eax, dword ptr [ebp-20]
:0053E1FB BA03000000              mov edx,
00000003
:0053E200 E86360ECFF              call 00404268
```

将上段程序***处作相应修改，程序即可正常运行

## 三、发现程序启动时，在下列程序段进行注册比较

查找`4C7A84`时发现：

```
:0053C7C6 8B45F4
mov eax, dword ptr [ebp-0C]
:0053C7C9 E8CEB2F8FF              call 004C7A9C
:0053C7CE 8B45F0
mov eax, dword ptr [ebp-10]
:0053C7D1 E8E2B3F8FF              call 004C7BB8
:0053C7D6 A1344E5400              mov eax,
dword ptr [00544E34]
:0053C7DB BAFF010000              mov edx,
000001FF
:0053C7E0 E89FB2F8FF              call 004C7A84
-->计算用户名字串1累加和
:0053C7E5 8BF0
  mov esi, eax
```
