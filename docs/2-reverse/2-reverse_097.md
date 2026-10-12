
# Armadillo3.70a IMT 修复脚本

> 原文：[`www.pediy.com/kssd/pediy06/pediy6643.htm`](https://www.pediy.com/kssd/pediy06/pediy6643.htm)

![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)

这是我第一接触 Armadillo，按 `fxyang` 兄加自己一丝调试偷懒写的，没测试很多，大家帮忙吧。。。`3.70a` 是我在跟一个加 Armadillo 加壳的 flash 工具进壳里偷看的。。。`test`... `test with you...`

```
/*
Script written by NewHand...

  Armadillo3.70a 双进程非远地址（非乱序）IMT 修复脚本      

        OS : winXP, no test on others

       Tool: OD1.1b, OllyScript 0.81

Debugging options: Tick all items in Debugging Options-Exceptions

                   and add C000001D..C000001E in custom exceptions

Note:   It's hard to write it with me, because my brain is poor with 

  idea and fist time touch Armadillo, and test... and rewritten, 

  after long time, after all appeared...at best to design it with intellgences by me. 

  my English is badly, I'm sorry with it. 

  希望所有使用这个脚本的朋友都能 pass-)Enjoy! 

Thanks : fxyang-The firstman talk it as I See in bbs.pediy.com and get me his methor, 

  Oleh Yuschuk, SHaG, jingulong, fly...over more oges essay's author, and you! 

*/

var exchange

var push_eax

var push_edx

var check

var min

var max

var test

gpa "OpenMutexA", "kernel32.dll"

cmp $RESULT,0

je end

bp $RESULT

run

mov test,401000

redone:

find test,#0000000000000000000000000000000000000000000000#

cmp $RESULT,0

jne kill_father

add test,1000

jmp redone

kill_father:

go $RESULT

mov [$RESULT],#609C680000000033C05050E8B4B2A5779D61E933F7A577#

mov exchange,esp

add exchange,0C

mov exchange,[exchange]

add $RESULT,3

mov [$RESULT],exchange

mov eip,401000

run

run

mov [test],#0000000000000000000000000000000000000000000000#

bc eip

gpa "VirtualProtect", "kernel32.dll"

cmp $RESULT,0

je end

bp $RESULT

run

run

run

run

run

bc $RESULT

rtr

rtr

sti

mov exchange,eip

add exchange,20

mov $RESULT,[exchange]

mov $RESULT,[$RESULT]

bp $RESULT

run

bc $RESULT

find eip,#558BEC515333DB# //Find Special Hex Code!

cmp $RESULT,0

je end

add $RESULT,1E

bp $RESULT // Set break First Anti Address!

run

xor eax,eax // Fix the First Anti!

cmt eip,"Fixed the First Anti!"

bc $RESULT

gpa "VirtualAlloc", "kernel32.dll"

cmp $RESULT,0

je end

bp $RESULT

run

run

rtr

bc $RESULT

gpa "memcpy", "msvcrt.dll"

cmp $RESULT,0

je end

bp $RESULT

run

rtr

bc $RESULT

sti

findop eip,#0FB68550CEFFFF85C00F842A010000#

cmp $RESULT,0

je end

add $RESULT,7

bp $RESULT

run

xor eax,eax //Fix The Second Anti!

cmt eip,"Fixed the Second Anti!"

bc $RESULT

sti

sti

findop eip,#83C004898590E5FFFFEBCA#

cmp $RESULT,0

je end

add $RESULT,0B

bp $RESULT

run

bc $RESULT

sti

mov $RESULT,eip

add $RESULT,0B  

mov $RESULT,[$RESULT]

bp $RESULT

run

run

run

bc $RESULT

find eip,#0FB68548CEFFFF#

cmp $RESULT,0

je end

add $RESULT,7

bp $RESULT

run

xor eax,eax ///Fix The Third Anti!

cmt eip,"Fixed the Third Anti!"

bc $RESULT

pause

findop eip,#8B8C95A0E6FFFF# //Start Searching and Fixing!

cmp $RESULT,0

je end

mov [$RESULT],#33C99090909090#

findop eip,#83BD80E2FFFF00#

cmp $RESULT,0

je end

add $RESULT,7

mov [$RESULT],#9090#

findop $RESULT,#FFB5C4CDFFFFFFB5B8E4FFFF#

cmp $RESULT,0

je end

mov exchange,$RESULT

add exchange,11

mov [exchange],#9090#

gpa "GetProcAddress", "kernel32.dll"

cmp $RESULT,0

je end

sub $RESULT,exchange

sub exchange,4

mov [exchange],$RESULT //Last Fixing!

serching_somthing:

findop eip,#59EB03D6D6#

cmp $RESULT,0

je end

mov check,$RESULT

bp $RESULT

find eip,#898570E8FFFF8B8570E8FFFF89851CE8FFFF83BDF0E6FFFF00#

cmp $RESULT,0

je end

bp $RESULT

run

mov max,eax

mov min,eax

check_address:

run

cmp eip,check

je final

mov test,eax

mov exchange,eax

check_max:

cmp test,max

ja save_max

check_min:

cmp exchange,min

jb save_min

jmp check_address

save_max:

mov max,test

jmp check_min

save_min:

mov min,test

jmp check_address

final:

bc $RESULT

bc eip

mov push_eax,eax

mov push_edx,edx

mov eax,min

mov edx,max

msg "Look at EAX, May be: Imt's Starting!"

msg "Look at EDX, May be: Imt's Ending!"

mov eax,push_eax

mov edx,push_edx

cmt eip,"The Imt's operation is finished!"

msg "Now why not to get THE CORRET IMT!"

end:

ret
```

![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
