
# Access Folder v1.2

> 原文：`www.pediy.com/kssd/pediy04/kanxue124.htm`

软件简介：`Access Folders` 是一个“打开/保存”对话框专用的切换目录工具，只要用鼠标在视窗右上角的？图标上按下鼠标右键，便可以打开自定的目录列表，迅速切换到想要的目录，不需再用鼠标慢慢的切换。

`url: http://dl1.51soft.com:8080/soft/105/afolders12.zip`

下载后发现 `EVC` 这个破解版最大只能添加 5 个文件夹，再破如下:

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00402A06(C)

:00402A3F \- `cmp` `ebx`, `00000004` \<===== 比较

:00402A42 \- `mov` `dword ptr [0042B6D0]`, `ebx`

:00402A48 \- `jle` `00402A5D`

:00402A4A \- `cmp` `dword ptr [0042B6D8]`, `00000002`

:00402A51 \- `je` `00402A5D`

:00402A53 \- `mov` `dword ptr [0042B6D0]`, `00000005`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :004029D1(C), :00402A3D(U), :00402A48(C), :00402A51(C)

:00402A5D \- `push` `ebp`

:00402A5E \- `call` `0040441E`

:00402A63 \- `mov` `eax`, `dword ptr [esp+14]`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004024BE(C)

:00402552 \- `mov` `eax`, `dword ptr [esp+000002C8]`

:00402559 \- `mov` `edx`, `eax`

:0040255B \- `shr` `edx`, `10`

:0040255E \- `jne` `00402814`

:00402564 \- `and` `eax`, `0000FFFF`

:00402569 \- `sub` `eax`, `000003E9`

:0040256E \- `je` `004026DA`

:00402574 \- `dec` `eax`

:00402575 \- `jne` `00402814`

:0040257B \- `cmp` `dword ptr [0042B6D8]`, `00000002`

:00402582 \- `je` `00402623`

:00402588 \- `cmp` `dword ptr [0042B6D0]`, `00000004` \<===== 比较

:0040258F \- `jle` `00402623` \<==== 不跳死

:00402595 \- `mov` `esi`, `dword ptr [esp+000002C0]`

:0040259C \- `push` `00000034`

:0040259E \- `push` `0040A2B4`

:004025A3 \- `push` `0040A238`

:004025A8 \- `push` `esi`

:004025A9 \- `call` `dword ptr [00408168]`

:004025AF \- `cmp` `eax`, `00000006`

:004025B2 \- `jne` `00402814`

:004025B8 \- `mov` `ecx`, `dword ptr [0042B608]`

:004025BE \- `lea` `eax`, `dword ptr [esp+000000B4]`

:004025C5 \- `push` `00000104`

:004025CA \- `push` `eax`

:004025CB \- `push` `ecx`

:004025CC \- `call` `dword ptr [00408114]`

:004025D2 \- `lea` `edx`, `dword ptr [esp+000000B4]`

:004025D9 \- `push` `0040A120`

:004025DE \- `push` `edx`

:004025DF \- `call` `0040430E`

:004025E4 \- `push` `eax`

:004025E5 \- `lea` `eax`, `dword ptr [esp+000000C0]`

:004025EC \- `push` `0040A118`

:004025F1 \- `push` `eax`

:004025F2 \- `call` `dword ptr [00408174]`

:004025F8 \- `add` `esp`, `00000014`

:004025FB \- `lea` `ecx`, `dword ptr [esp+000000B4]`

:00402602 \- `push` `000007D1`

\* Possible Ref to Menu : `Menu ID_0014`, Item: `""`

:00402607 \- `push` `00000000`

=====================================================================

`402A3F` `83FB04` ----> `83FB44`

`402588` `833DD0B6420004` ----> `833DD0B6420044`

完成. `4dd466d0e1e0cc123afbe504da2462ca`
