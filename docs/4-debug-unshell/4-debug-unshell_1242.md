

# 修理外挂克星主程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-639.htm`](https://www.pediy.com/kssd/pediy07/pediy7-639.htm)

fly 大侠在  
外挂克星脱壳——江湖 OnLine V1.05 中秋版   
http://bbs.pediy.com/showthread.php?s=&threadid=17155  
中说道  
"看来作者对 OllyDBD 研究了不少，不过对 SoftICE+IceExt 却不作抵抗，可惜。"  
  
主程序下载  
http://www.szleyu.com/download/BSLYtools.zip  
  
思路:  
用 SI 跟外挂克星主程序到 oep, dump 出来, 写 1 段小程序创建一个新的输入表  
, 写 1 段小程序修正 jmp 走的 IAT 操作  
  
开始  
  
2ksp4+ds2.7+iceext  
  
  
一   到 oep  
  
ep 是 2e1d0  
bpmb 42e1d0 x  
运行 断下  
  
     42E1D0        CALL   \[1000B194\]        断下  
     42E1D6        ADD    \[EAX\], AL  
     42E1D8        ADD    \[EAX\], AL  
     42E1DA        ADD    \[EAX\], AL  
     42E1DC        ADD    \[EAX\], AL  
  
bpmb 42e1d6 x  
bpx.p readprocessmemory  
ctrl-d 跑  
第三次 readprocessmemory 断下后，F11 返回  
       
     9EE556        CALL   \[KERNEL32!ReadProcessMemory\]  
     9EE55C        MOV    ESI, 009F7214    F11 返回到这儿  
     9EE561        LEA    EAX, \[ESP+08\]  
  
清除 readprocessmemory 断点  
F10 走啊走啊走啊返回这个 CALL 到这儿  
  
     9EFBC7        CALL   009EE3A0  
     9EFBCC        TEST   EAX, EAX         到这儿  
我这里 EAX 的返回值是 0, 所以不行  
r eax 改成 1  
  
然后一路 F10  断在 ep 的下一行 42E1D6 了  
     42E1D0        CALL   \[1000B194\]  
     42E1D6        JMP    40D8CA           断在这儿, 飞向光明之颠   
  
  
  
二   dump  
  
基址 400000 总大小 40000  
!dump \\??\\e:\\1.bin 400000 40000  
  
看到 oep 开头几行有  
40D8EB   PUSH  ESI  
40D8EC   PUSH  EDI  
40D8ED   MOV   \[EBP-18\], ESP  
40D8F0   JMP   00CD145C    壳把调用 API 的地方偷走了  
到 CD145C 向上看看，向下看看 发现一张表  
     CD0048   E1 ED E6 77  FF 15 48 00  CD 00 E9 8D  9F 74 FF 00  
     ......  
     CD4708   00 92 9B 77  FF 15 08 47  CD 00 E9 1F  00 75 FF AA  
反汇编一下 CD0048 处，意思就是  
     CD004C   CALL \[CD0048\]   就是 CALL  77E6EDE1  
     CD0052   JMP  00149FE4   跳回主代码段  
这个表也要了  
!dump \\??\\e:\\api.bin cd0048 46d0  
  
这时可以清除所有断点, si 放掉后, 程序就跑起来了, 程序开在那里  
一直会不断的 MessageBox, 没办法,忍一下, 我们还要借原程序取输入表  
  
  
三   修复 1.bin 的 PE 格式  
  
LordPE 打开 1.bin  
EntryPoint: 2E1D0 -> D8CA  
.data  (30000, 7CC8, 30000, 4000)->(30000, 7CC8, 30000, 8000)  
.rsrc  (38000, 71F8, 34000, 8000)->(38000, 71F8, 38000, 8000)  
ImportTable(27004, 28) -> (0,0)  
IAT(27000, 544) -> (0,0)  
  
1.bin 改名 1.exe  
  
  
四   输入表  
  
先做一件事情, 任务管理器关掉 Explorer.exe 再打开 Explorer.exe, 这样别的进程就能看到它了  
原来的 IAT 是(27000, 544), 借用这块地方  
  
\-----------------------------第一个程序---------------------------------  
#include <windows.h>  
#include <stdio.h>  
  
void main(void)  
{  
  FILE \*fp = fopen("e:\\\\api.bin", "rb");  
  HANDLE h = OpenProcess(PROCESS\_ALL\_ACCESS, 0, 1360); // 打开外挂克星进程  
  
  char apibuf\[16\];  
  
  DWORD dwPrevApiAddr = 0;  
  DWORD dwApiAddr = 0;  
  DWORD dwIatAddr = 0x400000 + 0x27000;                // IAT 地址  
  DWORD dwWrite;  
    
  int r = fread(apibuf, 16, 1, fp);  
  while (r !=EOF && r != 0)  
  {  
    dwApiAddr = \*(LPDWORD)apibuf;  
    if (dwPrevApiAddr != dwApiAddr)  
    {  
      // 不带重复的一个个写进去  
      WriteProcessMemory(h, (LPVOID)dwIatAddr, apibuf, 4, &dwWrite);  
      dwIatAddr += 4;  
    }  
    dwPrevApiAddr = dwApiAddr;  
    r = fread(apibuf, 16, 1, fp);  
  }  
  
  fclose(fp);  
  CloseHandle(h);  
}  
  
写完后, 用 ImportREC 打开外挂克星进程 RVA 27000, SIZE 514   
得到的指针就 1 个, 显示是假的, 别理它  
ImportREC 的选项 选中 "创建新的 IAT", 去掉 "修复 ep 到 oep"  
修复 1.exe, 得到 1\_.exe  
  
  
  
五   修正 jmp 走的 IAT 操作  
可以关掉程序了  
\---------------------------第二个程序---------------------------------  
#include <windows.h>  
#include <stdio.h>  
  
void main(void)  
{  
  
  FILE \*fp = fopen("e:\\\\1\_.exe", "rb");  
  char \*mapoffile = new char\[0x42000\];      //文件大小 0x42000  
  fread(mapoffile, 0x42000, 1, fp);         //文件内容全读进来  
  fclose(fp);  
  
  fp = fopen("e:\\\\api.bin", "rb");  
  
  char apibuf\[16\];  
  
  DWORD dwPrevApiAddr = 0;  
  DWORD dwApiAddr = 0;  
  DWORD dwIatAddr = 0x400000 + 0x40000;  
  DWORD dwRCall;  
  
    
  int r = fread(apibuf, 16, 1, fp);  
  dwPrevApiAddr = \*(LPDWORD)apibuf;  
  while (r !=EOF && r != 0)  
  {  
    // 得到 IAT 的位置  
    dwApiAddr = \*(LPDWORD)apibuf;  
    if (dwPrevApiAddr != dwApiAddr)  
      dwIatAddr += 4;  
    if (\*(LPDWORD)(dwIatAddr-0x400000+mapoffile) == 0)  
      dwIatAddr += 4;  
  
    dwPrevApiAddr = dwApiAddr;  
  
    // 找偷掉 API 操作的字节长度  
    // 比如 CALL \[0x00CD\*\*\*\*\]  是 6 个字节  
    // MOV  EAX, \[0x00CD\*\*\*\*\]  是 5 个字节  
    for (int i=4; i<16; i++)  
    {  
      if (\*(LPWORD)&apibuf\[i\] == 0x00CD)  
        break;  
    }  
  
    //通过跳回去的地址算出跳进来的地址  
    dwRCall = \*(LPDWORD)&apibuf\[i-2\] + 4 + 5 + \*(LPDWORD)&apibuf\[i+3\];  
      
    //把 jmp CD\*\*\*\* 改回 CALL \[IAT\]. MOV EDI, \[IAT\] 等等  
    memcpy((LPVOID)(dwRCall-0x400000+mapoffile), &apibuf\[4\], i-2-4);  
    memcpy((LPVOID)(dwRCall-0x400000+mapoffile+i-2-4), &dwIatAddr, 4);  
  
    r = fread(apibuf, 16, 1, fp);  
  }  
  
  fclose(fp);  
  
  // 2.exe 是最终文件  
  fp = fopen("e:\\\\2.exe", "wb");  
  fwrite(mapoffile, 0x42000, 1, fp);  
  fclose(fp);  
  
  delete\[\] mapoffile;  
}  
  
  
学习 

 4dd466d0e1e0cc123afbe504da2462ca