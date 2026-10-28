
调用`GetImageFilePath`函数去取得创建或者销毁进程的完整映像路径名，如果成功，在`us.Buffer`中保存的就是Unicode格式的完整映像路径名，我们调用`RtlUnicodeStringToAnsiString`函数将其转换成Ansi格式，同时不要忘记释放`GetImageFilePath`函数为`us.Buffer`分配的内存空间。

```
end else

begin

{ If we fail to get process's image file path }

{ just use only process name from EPROCESS. }

if g_dwImageFileNameOffset <> 0 then

  begin

	memcpy(@g_ProcessData.szProcessName, PAnsiChar(DWORD(peProcess) + g_dwImageFileNameOffset), 16);

end;

end;
```

如果`GetImageFilePath`返回失败，我们就无法取得进程的完整映像路径名，这时我们就只能提取`EPROCESS`结构里保存的进程名了，因为`EPROCESS`结构中定义的进程名字段长度只有16个字节，如果进程名长度超过16个字节，多余的部分会被截掉，因此这个字段就不是以空结束的字节串（ASCIIZ）了，因此我们把复制了全部的16个字节。

```
KeSetEvent(g_pkEventObject, 0, False);
```

然后你就可以给客户端管理程序发送信号唤醒客户端线程，客户端线程向驱动程序发送`IOCTL_GET_PROCESS_DATA`控制码收集进程信息。

**未完待续**

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [processmon_12.jpg](javascript:void(0);)

  （58.70kb，647次下载）

- [objecttype_12.jpg](javascript:void(0);)

  （59.24kb，642次下载）

- [memdump_12_02.jpg](javascript:void(0);)

  （100.39kb，645次下载）

- [objectproperty_12_03.jpg](javascript:void(0);)

  （36.08kb，637次下载）
