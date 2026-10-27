
最后要注意的就是有时候`FirstThunk`指向的区域是不能写的，所以当我们做完全部工作后要将所有节的属性赋予它可写的权限。

```asm
;; 修改所有节为可写属性
mov esi, dwNTHeaderAddr
assume esi : ptr IMAGE_NT_HEADERS
mov cx, word ptr [esi].FileHeader.NumberOfSections
movzx ecx, cx
add esi, sizeof IMAGE_NT_HEADERS
assume esi : ptr IMAGE_SECTION_HEADER
.WHILE ecx != 0
    mov eax, dword ptr [esi].Characteristics
    or eax, IMAGE_SCN_MEM_WRITE
    mov dword ptr [esi].Characteristics, eax
    dec ecx
    add esi, sizeof IMAGE_SECTION_HEADER
.ENDW
```

`IMAGE_SCN_MEM_WRITE`是写属性的常量值。

终于写完了。累。。。![](https://bbs.kanxue.com/view/img/face/50.gif)

这节的原理也可以用在R3下的HOOK方面，大家应该很轻松的想到如何做R3下的HOOK了，如果有刚刚接触研究病毒的朋友也可以联想到一种模糊入口点的方式。呵呵！

...才知道看雪论坛一贴只能传两个附件，我将列出引入表的附件穿到自己的沙发了 呵呵。。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [CryptIID_bin.rar](javascript:void(0);) （2.23kb，554次下载）
- [CryptIID_src.rar](javascript:void(0);) （7.36kb，646次下载）
