
```
00008136: add eax, ecx              ; 6603c1
00008139: pop dx                    ; 5a
0000813a: push dx                   ; 52
0000813b: mov bx, di                ; 8bdf
0000813d: mov cx, 0x0001            ; b90100
00008140: call .-1123               ; e89dfb
FB9D+8143=7CE0H
00008143: pop dx                    ; 5a
00008144: mov bx, dx                ; 8bda
00008146: ret                       ; c3
00008147: add byte ptr ds:[bx+si], al ; 0000
00008149: add byte ptr ds:[bx+si], al ; 0000
000081f9: add byte ptr ds:[bx+si], al ; 0000
000081fb: add byte ptr ds:[bx+si], al ; 0000
000081fd: add byte ptr ds:[di-86], dl ; 0055aa
```

接下来跳至`2000:00H`处执行的程序就有点大了，`20H`个扇区，而实际上这个程序的有效字节大约在`30000H`多。这就可以做很多事情，比如：进入保护模式等，离主题越来越远了。

好了，我们回到正题。上面程序中是如何计算表项地址的？

`FAT`表项所在扇区相对`FAT`表起始扇区偏移扇区数（对应扇区）=`对应簇号*4/200`，因为每个`FAT`表项占`4`个字节。

`FAT`表项相对对应扇区偏移字节=`对应簇号*4%200`。

操作系统就是通过这种方法利用`FDT`和`FAT`查找并将文件读入内存。

紧接`FDT`的就是逻辑盘的`DATA`区。具体存放各种文件（以簇为单位）。当`EBR`，`FAT`，甚至`FDT`损坏后，其实，我们仍然可以通过搜索`DATA`区的办法来恢复文件。

注：本帖由看雪论坛志愿者`PEstone` 重新将`DOC`整理排版，若和原文有出入，以原作者附件为准

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [硬盘.rar](javascript:void(0);)
  （0.98MB，201次下载）
- [1.png](javascript:void(0);)
  （144.60kb，7次下载）
- [2.png](javascript:void(0);)
  （206.69kb，5次下载）
- [3.png](javascript:void(0);)
  （56.48kb，5次下载）
- [4.png](javascript:void(0);)
  （135.08kb，5次下载）
- [6.png](javascript:void(0);)
  （130.45kb，5次下载）
- [5.png](javascript:void(0);)
  （174.86kb，3次下载）
