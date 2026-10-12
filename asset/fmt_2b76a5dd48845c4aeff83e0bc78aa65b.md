
```
005B3C6C    55               push    ebp                     ; 保存现场
005B3C6D    53               push    ebx                     ; 保存现场
005B3C6E    33DB             xor     ebx, ebx                ; 计数器
005B3C70    8BC5             mov     eax, ebp                ; 取得载入的图片数所在的位置
005B3C72    8B40 FC          mov     eax, dword ptr [eax-4]
005B3C75    8B80 CC050000    mov     eax, dword ptr [eax+5CC]
005B3C7B    8B80 EC020000    mov     eax, dword ptr [eax+2EC]
005B3C81    E8 363AE7FF      call    004276BC                ; 取得载入的图片数
005B3C86    48               dec     eax
005B3C87    3BD8             cmp     ebx, eax                ; 跟计数器比较
005B3C89    7F 0C            jg      short 005B3C97          ; 处理完了就跳走
005B3C8B    8BC3             mov     eax, ebx
005B3C8D    43               inc     ebx                     ; 计数器+1
005B3C8E    55               push    ebp
005B3C8F    E8 B8FCFFFF      call    005B394C                ; 处理图片
005B3C94    59               pop     ecx
005B3C95   ^ EB D9          jmp     short 005B3C70          ; 返回处理下一张
005B3C97    5B               pop     ebx                     ; 恢复现场
005B3C98    5D               pop     ebp                     ; 恢复现场
```

### 【题外的话】

软件的关键代码不但需要运行中解压出来，而且还加了花指令。偶是壳盲，对 `dump` 代码部分一窍不通。想到的解决方案有 3：

1. 找到源代码的解码方法，把自己需要修改的地方逆回去写入源代码，嘿嘿。。其实看了下好像不是很困难。
2. 用 `loader` 解决。好用的 `loader` 还真难找。樱花补丁制作工具似乎和系统有冲突或者其他原因，对代码的搜索替换似乎晚了一步，有时候 `Nag` 都出来了那边代码才修改完。。。时灵时不灵的。
3. 自己写一个 `loader`。呵呵。。。可惜自己 `windows` 编程部分还不熟，要写出来不是不可能，不过要查阅很多资料，慢慢试验，可是我很懒。。。

### 【版权声明】

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！
