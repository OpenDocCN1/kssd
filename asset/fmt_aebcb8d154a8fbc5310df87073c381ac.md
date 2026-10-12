
```asm
01008770     8B45 0C           mov     eax, dword ptr [ebp+C]  ;让 eax = uMsg
01008773     83E8 4E           sub     eax, 4E       ;WM\_NOTIFY 消息的值是 0x004E
01008776     75 0E             jnz     short 01008786    ;如果不是 WM\_NOTIFY 消息就跳走，否则先进行预览。
01008778     8B45 14           mov     eax, dword ptr [ebp+14]
0100877B     50               push    eax
0100877C     8B45 08          mov     eax, dword ptr [ebp+8]
0100877F     50               push    eax
01008780     FF15 40300101    call    dword ptr <&PrevText.HandleNoti>;  PrevText.HandleNotify
01008786     33C0             xor     eax, eax
01008788     ^ E9 0294FFFF    jmp     01001B8F
```

最后保存文件，再看看保存文件对话框，也可以进行内容预览了。

![6.补充说明 要得到上面图中的效果，我们要先修改记事本的资源，要手工添加 3 个控件用于信息的显示，两个静态控件和一个文本框。 我是使用 Restorator 导出资源后手动修改的，调整位置的时候要有点小技巧。 在附件中包含了修改好的 `.rc` 资源文件，直接用 Restorator 导入替换原来的 NPENCODINGDIALOG 对话框再进行上面的操作就可以看到效果了。 祝大家玩的愉快！上传的附件| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 工程源码和对话框资源.rar\n\n |](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)

增加了文本预览功能的记事本.rar
