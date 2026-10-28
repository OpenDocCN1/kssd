
`DelBmp proc dwIdx: DWORD`
```
LOCAL @Bmp: Bmp
LOCAL @BmpAndPage[10h]: BmpAndPage ;不会有断点占10个页吧
LOCAL @PageMod: PageMod
LOCAL @i: DWORD
LOCAL @j: DWORD
LOCAL @dwOld: DWORD

mov @j, 0
mov @i, 0
invoke RtlZeroMemory, addr @Bmp, size @Bmp
invoke RtlZeroMemory,addr @BmpAndPage, 10h * size BmpAndPage
invoke RtlZeroMemory, addr @PageMod, size PageMod

;获取序号指定的断点结构体
mov eax, dwIdx

;断点序号超过范围
.if eax > 256
ret
.endif

mov ecx, size Bmp
mul ecx
add eax, offset g_arrBmp
assume eax: ptr Bmp
mov esi, eax
;暂存断点
invoke crt_memcpy, addr @Bmp, eax, size Bmp

;删除断点
invoke RtlZeroMemory, esi, size Bmp

;删除断点-内存页
.while @i < 256
mov eax, @i
mov ecx, size BmpAndPage
mul ecx
add eax, offset g_arrBmpAndPage
mov esi, eax
assume esi: ptr BmpAndPage

mov eax, @Bmp.address
.if [esi].bmp_addr == eax
;此内存断点对应的内存页，先拷贝到局部数组中保存
mov edx, @j
invoke crt_memcpy, addr @BmpAndPage[ edx * size BmpAndPage], esi, size BmpAndPage
inc @j

;删除此项
invoke RtlZeroMemory, esi, size BmpAndPage
.endif
inc @i
.endw

;遍历此断点对应的内存页，看有没有其他的断点在使用
;如果有，就不用删除内存页表中的项
;如果没有，则删除内存页表中的项
lea ebx, @BmpAndPage
mov @i, 0
mov edx, @j
.while @i < edx
mov eax, @i
mov ecx, size BmpAndPage
mul ecx
add eax, ebx
mov esi, eax
assume esi: ptr BmpAndPage

invoke IsInarrBmpAndPage, [esi].page_base
.if eax == 1 ;如果其他断点也占用了这块内存，那就不需要删除，不需要恢复内存属性
.continue
inc @i
.endif

;从内存页表中删除内存页，恢复内存属性
invoke FindAndDelPageInArrPageMod, [esi].page_base
mov edi, eax
invoke VirtualProtectEx, g_hProcess, [esi].page_base, 1000h, edi, addr @dwOld
inc @i
.endw

ret
DelBmp endp
```

到这里调试器主要的软件、硬件、内存断点、trace、单步已经实现完成了，剩下的功能就是显示反汇编、显示修改数据、寄存器、运行到返回等小功能了，那么这个简易的调试器就算是完成了，感谢阅读。

科锐44期学员

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-3-26 17:56

被st0ne编辑，原因： 修改内容

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#问题讨论](https://bbs.kanxue.com/forum-4-1-197.htm)

上传的附件：

- [stDbg.7z](javascript:void(0);)

（2.95MB，213次下载）
