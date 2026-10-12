
# VProtect1.70-1.88 过试用及密码脚本

> 原文：[`www.pediy.com/kssd/pediy12/129668.html`](https://www.pediy.com/kssd/pediy12/129668.html)

这个可能是 VP 的一个逻辑漏洞吧，可以跳过试用框和密码框！（不管是否有试用按钮）

不提供样本了只提供脚本吧！只支持 1.88 及以下的版本。因为更高版本的有校验，这个方法是过不了的！

注意：不支持无 key 不解码的类型！

其实这个也可以做个 LPK 的 因为 VP 的相对偏移是固定的 好像同一个版本是相同的偏移 所以 只要断 `VirtualAlloc` 然后 根据偏移和特征码应该可以用 LPK 实现的 呵呵 这个我就不写了以前发过一个 LPK 想写的可以参看那个吧

代码:

```
///////////////////////////////////////////////////////////////////
//VP 1.70-1.88 过试用及密码框脚本
//不支持无 KEY 不解码的类型
//CODE BY wuqing1501(笨笨鼠)
//////////////////////////////////////////////////////////////////////
var VirtualAlloc
var bypassaddr
var deax
var deax1 //保存第二次申请的地址
var deax2  //保存第三次申请的地址
var deip
var logfile
var n
bc
bpmc
bphwc
mov n,0
mov logfile,"VProtect1.7x-1.88 过试用及密码框地址.txt"
wrta logfile,"\\r\\n"
gpa "VirtualAlloc", "kernel32.dll"
cmp $RESULT,0
je erro
mov VirtualAlloc, $RESULT
add VirtualAlloc,19
bp VirtualAlloc

run
run
mov deax1,eax
run
mov deax2,eax
cmp deax1,deax2
mov deax,deax2
jnz findAllocaddr
run
mov deax,eax

findAllocaddr:
run
run
run
bc VirtualAlloc

findbypassaddr:
find deax,#85C00F84????????8B??08C7??????????8B??08C7????????????8B??08C7????????????8B??08C7????????????8B??08C7????????????#
cmp $RESULT,0
jnz VP1.7x

find deax,#558BECE8#
cmp $RESULT,0
je finded
add n,1
cmp n,4
jb next
cmp n,7
ja finded
bphws $RESULT,"x"
wrta logfile,"可能的地址为:"
wrta logfile,$RESULT
wrta logfile,"\\r\\n"
next:
mov deax,$RESULT
add deax,1
jmp findbypassaddr

VP1.7x:
mov deax,$RESULT
loop:
sub deax,1
find deax,#558BEC#,3
cmp $RESULT,0
je loop
bphws deax,"x"
run
jmp logaddr

finded:
run 
find eip,#558BECE8#,4
cmp $RESULT,0
je erro


logaddr:
mov deip,eip
mov deax,eax
wrta logfile,"by pass addr:"
wrta logfile,deip
wrta logfile,"\\r\\n"
mov eip,\[esp\]
jmp finded


erro:
msg "something wrong!"
ret

exit:
ret
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `VProtect1.70-1.88 过试用及密码脚本.rar`
|
