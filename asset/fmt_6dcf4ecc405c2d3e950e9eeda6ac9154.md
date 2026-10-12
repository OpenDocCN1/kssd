
`wsprintf(tcCaption,TEXT("\[OEP 被更改为 0x%08X\]我难过~~~！"),_ntHeader->OptionalHeader.AddressOfEntryPoint);`

`if(MessageBox(NULL,TEXT("小湔雪，我已经被人修改了，如果运行会有麻烦的，点取消吧！"),`

`tcCaption,MB_YESNO|MB_ICONSTOP)!=IDYES)`

```
{
  ::ExitProcess(0);
}
```

`};`

`//恢复入口点代码:`

`memcpy((PBYTE)(_ntHeader->OptionalHeader.ImageBase+0x00005944),&uiData1[0],15);`

```
}
```

`//----------------------------------------------------------------------------------------`

大家还可以做的更难点，比如对代码进行处理！动态修复等等，看大家自己的了~~~~~~~~~~~~

吃饭去了哈哈！！！
