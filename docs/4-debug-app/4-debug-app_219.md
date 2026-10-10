

# 蜘蛛纸牌底牌数据结构图及辅助代码利用

> 原文：[`www.pediy.com/kssd/pediy12/129478.html`](https://www.pediy.com/kssd/pediy12/129478.html)

1。从 RegisterClass 找到窗口回调函数处理地址。  
2。从 WM\_COMMAND 消息分支中找到菜单 ID，比如“开局”ID=9C45，  
  
![](img/d9f7d61e3a646d419bf3e2f2f8f4cfb0.jpg)  
3。代码利用显示底牌效果图  
![](img/c3c4c9b59a77ca51e38c20d43e005655.jpg)  
  
4。原来的循环代码，显示底牌代码直接拼接利用。  
    在 spider .text 上找一片空地，即搜索一大片"00 00.."  
我找的是 010021C1。  
  
原代码：发 44 张牌，不亮底牌。  
01005B7E       mov dword ptr ss:\[ebp+8\],ebx  
01005B81       mov dword ptr ss:\[ebp-8\],ebx  
01005B84       mov dword ptr ss:\[ebp-4\],ebx  
01005B87       cmp dword ptr ss:\[ebp-8\],4  ；最后 4 张牌。  
01005B8B       jnz short spider.01005B93  
01005B8D       cmp dword ptr ss:\[ebp-4\],3  
01005B91       jg short spider.01005BB9  
01005B93       mov ecx,dword ptr ds:\[esi+4\]  
01005B96       call spider.01007545  
01005B9B       mov ecx,dword ptr ds:\[esi+4\]  
01005B9E       push ebx        ；ebx=0，不亮底牌  
01005B9F       push dword ptr ss:\[ebp+8\]    
01005BA2       call spider.01007400  
01005BA7       mov ecx,dword ptr ds:\[esi+8\]  
01005BAA       push ebx  
01005BAB       push dword ptr ss:\[ebp+8\]  
01005BAE       push dword ptr ss:\[ebp-4\]  
01005BB1       call spider.01008069  
01005BB6       inc dword ptr ss:\[ebp+8\]  
01005BB9       inc dword ptr ss:\[ebp-4\]  
01005BBC       cmp dword ptr ss:\[ebp-4\],0A  ；内循环 10 次  
01005BC0       jl short spider.01005B87  
01005BC2       inc dword ptr ss:\[ebp-8\]  
01005BC5       cmp dword ptr ss:\[ebp-8\],5  ；外循环 5 次。  
01005BC9       jl short spider.01005B84  
  
原来代码：初始发牌的最后 10 张扑克是亮牌的。即第 44-54 张是明牌  
01005C0F       xor eax,eax  
01005C11       cmp edi,4  
01005C14       setl al  
01005C17       mov ecx,esi  
01005C19       add eax,4    ；第 5 行，or 第 6 行  
01005C1C       push eax      ；行  
01005C1D       push edi      ；列  
01005C1E       call spider.01005115    ；亮牌。  
  
  
改进后的代码：  
01005B9E   E9 1EC6FFFF  jmp spider.010021C1；修改成 jmp 10021C1                 
01005BA3   90      nop  
  
在 10021C1 写入利用代码：  
010021C1           6A 01        push 1  
010021C3           FF75 08      push dword ptr ss:\[ebp+8\]  
010021C6           E8 35520000  call spider.01007400  
010021CB           8B4E 08      mov ecx,dword ptr ds:\[esi+8\]  
010021CE           6A 01        push 1  
010021D0           FF75 08      push dword ptr ss:\[ebp+8\]  
010021D3           FF75 FC      push dword ptr ss:\[ebp-4\]  
010021D6           E8 8E5E0000  call spider.01008069  
010021DB           FF75 08      push dword ptr ss:\[ebp+8\]；行  
010021DE           FF75 FC      push dword ptr ss:\[ebp-4\]；列  
010021E1           8BCE         mov ecx,esi  
010021E3           E8 2D2F0000  call spider.01005115 ；亮 1 张底牌  
010021E8           E9 C9390000  jmp spider.01005BB6 ；跳回  
  
10021C1：二进制代码  
6A 01 FF 75 08 E8 35 52 00 00 8B 4E 08 6A 01 FF 75 08 FF 75 FC E8 8E 5E 00 00 FF 75 08 FF 75 FC 8B CE E8 2D 2F 00 00 E9 C9 39 00 00  
  
5。剩下的 50 张牌也能看，但我不知道该放在原程序的什么位置显示，就算了。  
   修改后的蜘蛛纸牌就像是没有颜色限制的高级空当接龙了：）  
  
6。蜘蛛纸牌辅助工具   
spider\_fix.exe  
  
void Getit()  
{  
  ULONG \*q=NULL,\*p=NULL;  
  unsigned char patch1\[\]={0xE9,0x1E,0xC6,0xFF,0xFF};  
  unsigned char patch2\[\]={0x6A,0x01,0xFF,0x75,0x08,0xE8,0x35,0x52,0x00,0x00,0x8B,0x4E,0x08,0x6A,0x01,0xFF,0x75,0x08,0xFF,0x75,0xFC,0xE8,0x8E,0x5E,0x00,0x00,0xFF,0x75,0x08,0xFF,0x75,0xFC,0x8B,0xCE,0xE8,0x2D,0x2F,0x00,0x00,0xE9,0xC9,0x39,0x00,0x00};  
    
    
  HWND hwnd=FindWindow("蜘蛛","蜘蛛");  
  if(!hwnd)  
  {  
    MessageBoxA(NULL,"蜘蛛纸牌没打开！","information",NULL);  
    return ;  
  }  
  DWORD dwProcessID;  
  GetWindowThreadProcessId(hwnd,&dwProcessID);  
  HANDLE hProcess=OpenProcess(PROCESS\_VM\_READ | PROCESS\_CREATE\_THREAD | PROCESS\_VM\_WRITE | PROCESS\_VM\_OPERATION,FALSE,dwProcessID);  
  if (!hProcess  )  
    return ;  
  WriteProcessMemory(hProcess,(LPBYTE)0x1005B9E,patch1,sizeof(patch1),p);    
  WriteProcessMemory(hProcess,(LPBYTE)0x10021C1,patch2,sizeof(patch2),p);  
    
  MessageBoxA(NULL,"ok","information",NULL);  
  CloseHandle(hProcess);    
    
}  
  
各位下载试用的请报一下在 Vista or win7 下有没有问题,我的机器老,只能跑 XP,谢谢.  
Spider\_fix.rar