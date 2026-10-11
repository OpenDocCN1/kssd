
# NDIS6 X86 通用 hook 代码

> 原文：[`www.pediy.com/kssd/pediy12/143507.html`](https://www.pediy.com/kssd/pediy12/143507.html)

原理参见[`www.opscn.com/index.php?actio...rum=1&topic=15`](http://www.opscn.com/index.php?action=vthread&forum=1&topic=15)

代码:

```
/*
 * 安装 ndis 协议 hook
 */
int init_ndis_hook(void* ProtoBlock)
{
        PNDIS60_PROTOCOL_BLOCK protocol = ProtoBlock;
        PNDIS_OPEN_BLOCK TcpOpenBlock = NULL;
        PUNICODE_STRING BindDeviceName = NULL;
        PUNICODE_STRING RootDeviceName = NULL;
        unsigned char *buf = NULL;

        KdPrint(("init_ndis_hook:%x %x\n", protocol, protocol->NextProtocol));
        ndisFindMiniportOnGlobalList = search_ndisFindMiniportOnGlobalList();

        while (protocol = protocol->NextProtocol) {
                PUNICODE_STRING Name = NULL;
                Name = &protocol->Name;

                KdPrint(("Enume protocol %wZ\n", Name));
                if (0 == wcsncmp(Name->Buffer, (const wchar_t*)&L"TCPIP", Name->Length>>1)){
                        TcpOpenBlock = protocol->OpenQueue;
                        break;
                }
        }
        
        if (!TcpOpenBlock)
                return -1;

        KdPrint(("TcpOpenBlock %x %x\n", TcpOpenBlock, MmUserProbeAddress));
        ndis_hook_info.OpenBlock = TcpOpenBlock;

        /* 搜索 BindDeviceName 以及 RootDeviceName */
        for (buf = (unsigned char*)TcpOpenBlock; buf < (unsigned char*)TcpOpenBlock + 0x500; buf+=4) {
                if (*(PULONG)(buf) > MmUserProbeAddress
                        && **((PULONG*)(buf)) == 0x005e005c 
                        && **((PULONG*)(buf + 4)) == 0x005e005c) {
                        BindDeviceName = *(PUNICODE_STRING*)buf;
                        RootDeviceName = *(PUNICODE_STRING*)(buf + 4);
                        ndis_hook_info.RootDeviceName = RootDeviceName;
                        KdPrint(("tcp root dev %wZ\n", RootDeviceName));
                        break;
                }
        }

        /* 搜索 ReceiveNetBufferLists 地址 */
        for (buf = (unsigned char*)TcpOpenBlock; buf < (unsigned char*)TcpOpenBlock + 0x500; buf+=4) {
                if (*(PULONG)(buf) > MmUserProbeAddress
                        && *(PULONG)(buf) == *(PULONG)(buf + 4)
                        && *(PULONG)(buf) == *(PULONG)(buf + 12)) {
                        /* buf->tcpip!FlReceiveNetBufferListChain */
                        ndis_hook_info.POpenBlockReceiveHandler = (PVOID*)(buf + 8);
                        ndis_hook_info.ReceiveNetBufferLists = *(PVOID*)(buf + 8);
                        KdPrint(("ReceiveNetBufferLists:%x\n", *(PVOID*)(buf + 8)));
                        break;
                }
        }

        /* 搜索底层 miniblock 数据发送函数 */
        if (ndisFindMiniportOnGlobalList && RootDeviceName) {
                PNDIS_MINIPORT_BLOCK miniBlock = ndisFindMiniportOnGlobalList(RootDeviceName);
                if (miniBlock) {
                        /* 0x16c NextSendNetBufferListsHandler  偏移地址 NDIS6 中固定 */
                        PVOID NextSendNetBufferListsHandler = *(PVOID*)((char*)miniBlock + 0x16c);
                        ndis_hook_info.MiniBlock = miniBlock;
                        ndis_hook_info.PMiniBlockNextSndHandler = (PVOID*)((char*)miniBlock + 0x16c);
                        ndis_hook_info.NextSendNetBufferListsHandler = NextSendNetBufferListsHandler;
                        KdPrint(("miniBlock: %x %x\n", miniBlock, NextSendNetBufferListsHandler));
                        
                } else {
                        KdPrint(("ndisFindMiniportOnGlobalList failed!\n"));
                }
        }

        /* 进行 NDIS60 的 HOOK 处理 */
        if (ndis_hook_info.POpenBlockReceiveHandler) 
                *ndis_hook_info.POpenBlockReceiveHandler = HookReceiveNetBuferLists; 

        if (ndis_hook_info.PMiniBlockNextSndHandler)
                *ndis_hook_info.PMiniBlockNextSndHandler = HookSendNetBufferLists;

        return 0;
}
```

![](img/42392cfb58333b0489700352cdd0fdc2.jpg)

WDK7600 编译，Windows7 旗舰版 SP1 测试通过

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || ndis6_hook.rar  
|
