
# 雪花屏保

> 原文：[`www.pediy.com/kssd/pediy03/forum483.htm`](https://www.pediy.com/kssd/pediy03/forum483.htm)

Crack the snowflake3d screen save doc.

\----------------------

雪椰

2000,12,28am

Email:`wocy@263.net`

Http://wocy.yeah.net

\------------------------

Use `W32asm` open the file `snowflake3d.scr`

1, find the string "Registration" will go to...

Name: `DialogID_006C`, # of Controls=`002`, Caption:"Registration", ClassName:""

    `001` - ControlID:`0001`, Control Class:"BUTTON" Control Text:"OK"
    `002` - ControlID:`FFFF`, Control Class:"STATIC" Control Text:"Registration Successful"

Name: `DialogID_006D`, # of Controls=`003`, Caption:"Registration", ClassName:""

    `001` - ControlID:`0002`, Control Class:"BUTTON" Control Text:"OK"
    `002` - ControlID:`FFFF`, Control Class:"STATIC" Control Text:"Registration Code Invalid!"
    `003` - ControlID:`FFFF`, Control Class:"STATIC" Control Text:"You must enter the exact same information as you used to purchase this softwar"

Name: `DialogID_006E`, # of Controls=`003`, Caption:"Message", ClassName:""

2, find the dialog `6c&6d` you go to ....

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00401412(C)` <----------c |
| :--- |
| |
| `:004014CA 8B1598654200 mov edx, dword ptr [00426598]` |
| `:004014D0 6A00 push 00000000` |
| `:004014D2 6820124000 push 00401220` |
| `:004014D7 53 push ebx` |
| |
| \* Possible Reference to Dialog: `DialogID_006D` |
| |
| `:004014D8 6A6D push 0000006D` |
| `:004014DA 52 push edx` |
| \---->c |
| `:00401408 E883100000 call 00402490` |
| `:0040140D 83C410 add esp, 00000010` |
| `:00401410 85C0 test eax, eax` |
| `:00401412 0F84B2000000 je 004014CA <----------(1)` |

3, modify `1` with `jne xxxx`

4, runing,regist,press any code,you will ok

5, the world cleaned. `4dd466d0e1e0cc123afbe504da2462ca`
