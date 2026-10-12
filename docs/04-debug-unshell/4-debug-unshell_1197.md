
# yoda's Crypter V1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6238.htm`](https://www.pediy.com/kssd/pediy06/pediy6238.htm)

跟了几个`yoda'scrypter v1.2`的壳并参考了一下`fly`兄的脱壳文章(非常感谢`fly`), 现总结一下这个壳的脱壳要点.总的来说这个壳的保护只是一般,`anti-debug`的方法过于普通(用`createFilea`), 壳也没有多少花指令,脱壳要点:

一般加壳后你一直按`f8`(`f8`) 到了出现`Createfilea`(当然你也可以直接下`bpx createfilea`了)的下一行又一个经典`cmp eax,-1`这里的`eax`为`8`,改成`-1`就跳过了,再`f8`可以看到又一个经典

```
0040C345   8D85 A3214000    LEA EAX,DWORD PTR SS:[EBP+4021A3]
0040C34B   50               PUSH EAX
```

这里可以说是这个壳的标志吧??(这个不会因程序的变化而变化)

这后的`ret`到的地方的下一行

```
0040C41C   039D 68254000    ADD EBX,DWORD PTR SS:[EBP+402568]
```

就`push`了`oep`

`EBX=oep`

`4dd466d0e1e0cc123afbe504da2462ca`
