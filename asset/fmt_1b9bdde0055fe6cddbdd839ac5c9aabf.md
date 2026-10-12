
```
mov oep, eip  
alloc 1000  
mov mem, $RESULT  
mov xxx, mem  
mov org\_esp, esp  
@continue:  
add esp, 4  
cmp \[esp\], 0000ffff  
jnz @continue  
@next:  
mov \[xxx\], #68#,1  
inc xxx  
mov \[xxx\], \[esp\]  
sub esp,4  
add xxx,4  
cmp esp, org\_esp  
jae @next  
mov \[xxx\], #ff25#, 2  
add xxx, 2  
mov \[xxx\], iataddress  
add xxx, 4  
sub xxx, mem  
mov \[iataddress\], oep  
add iataddress, 4  
mov eip, iataddress  
mov yyy, mem  
@orz:  
cmp xxx, 0  
je @exit  
mov \[iataddress\], \[yyy\], 1  
inc iataddress  
inc yyy  
dec xxx  
jmp @orz  
@exit:  
msg "修补完成,可以 dump"  
free mem  
ret  
```

## overlay 处理

直接把原文件的 overlay 贴到脱壳后的文件末尾就可以了。

## 注意事项

1. 我的虚拟机里 XP 版本过老，木马运行后系统崩溃。可能是 XP 版本问题，换了个新版本的 XP 运行正常，但是因为新版本的 XP 没有弄测试环境，所以不能肯定。
2. 在某个系统下调试就不要换其它系统，修复堆栈里存放的函数地址是直接从堆栈中读取的而不是自己写动态获取过程得到的，所以肯定不能跨系统版本运行。脚本运行过程中会有个断点下在数据段的警告，确定就可以了。
3. 木马有危险，切记虚拟机下调试。稍微看了下没有新技术，以后有时间再仔细分析。字符串都是加密的，不停的拷贝自身。

## 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | 分析.rar [请到论坛下载：[`bbs.pediy.com/showthread.php?t=79532`](http://bbs.pediy.com/showthread.php?t=79532)] |
| :--- | :--- |
