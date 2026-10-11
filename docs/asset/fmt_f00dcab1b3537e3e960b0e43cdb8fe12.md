
```
0045AF66  0000  add  [eax], al
0045AF68  0000  add  [eax], al
0045AF6A  56  push  esi
0045AF6B  57  push  edi
0045AF6C  53  push  ebx
.........
.........
0045AFF3  0FB6C3  movzx  eax, bl
0045AFF6  0FB6D7  movzx  edx, bh
0045AFF9  29D0  sub  eax, edx
0045AFFB  5B  pop  ebx
0045AFFC  5F  pop  edi
0045AFFD  5E  pop  esi
0045AFFE  C3  retn
0045AFFF  00  ???  ; 命令置于内存块尾
```

之后再把 `00407FDC` 处改一下，跳过来：

```
00407FDC  /E9 892F0500  jmp  0045AF6A
00407FE1  |C3  retn
00407FE2  |B7 68  mov  bh, 68
```

然后保存修改到文件中，运行一下。终于 OK 了。

### 四、后记

感谢你这么有耐心地看完这篇文章，希望对你有帮助。在这里更要感谢论坛里的牛人们，无数牛人的文章使我受益良多。

关于那段 `stolen code`，其实这段代码可以恢复成和原来一样的，但是如果没有原来的代码我想很难分析出来，上面的分析我也是照着我的源码分析的，联系到今后更多的情况时未知源码的，这种贴回来的方法更实用。

`4dd466d0e1e0cc123afbe504da2462ca`
