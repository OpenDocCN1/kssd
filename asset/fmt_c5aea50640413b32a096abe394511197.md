
# EasyPDF v1.5.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1052.htm`](https://www.pediy.com/kssd/pediy03/forum1052.htm)

`EasyPDF` `v1.5.2` 是用 `Visual Protect 3` 加的壳。

被重定向的 `import table` 都是 `kernel32.dll`, `user32.dll` 等。

其重定向方式为：

`IAT [0068C208] = 01BFAC18`

`01BFAC18:`

```
B84CC39200    mov eax, 0092C34C
FFE0          jmp eax
```

`0092C34C:`

```
B88818F7BF    mov eax, BFF71888    <-- GetCurrentThreadId
FFE0          jmp eax
```

看上去很简单, 但 `Import REConstructor v1.2` 的 `Trace Level1 (Disasm)` 是不能分析这种重定向方式的, 因此我们只能手动重建了, 逐个将 `BFxxxxxx` 填回去是一件非常累人的事情, 幸好其 `redirect` 方式很固定, 都是 `mov eax, xxxxxxxx, jmp eax` 每次的代码不同之处只是 `xxxxxxxx`, 我们可以写个程序来读 `IAT` 指向的第 1, 6, 7 字节是否分别为 `B8`, `FF`, `E0`, 是就认为这是个有效的 `redirect`, 再根据第 5 个字节比 `B0` 大, 不是就以 2--5 字节为指向下一个 `jmp` 的地址再读, 是就认为这个 `DWORD` 是真正的 `Import Table Address`, 将其写到 `IAT` 处 (如上例的`[0068C208]`). 然后处理下一个 `IAT`, 直到 `IAT size / 4` 个为止.

下面附上源程序, 四个参数 `m_processid` (进程 ID), `m_imagebase` (IMAGE BASE), `m_iatrva` (IAT RVA), `m_iatsize` (IAT SIZE)

(为什么是 `m_xxxxx` ? , 呵呵是 `VC` 里搬过来的(:)

这样处理完以后就剩下一个被重定向为 `0089EF0C` 的 `IAT`, `debug` 看一下, 看其 `push` 进去的参数和返回结果就知道这是 `GetProcAddress`.

`EasyPDF.exe` 重建好了, `vspdf.dll` 也差不多, `Process ID` 就是 `EasyPDF.exe` 的, 关键是找到 `IAT RVA` 和 `IAT SIZE`, 其 `OEP` 是 `0x1000`, 不过 `Import REConstructor` 说找不到 `IAT`, 我试了 `OEP` 为 `0x1CBC0` 就能发现 `IAT`, 但 `size` 好像也不对, 我也没仔细看, 反正我是把它改大了.

对那些没用的 `IAT` 都 `cut` 掉. 还有就是 `EasyPDF.exe` 的 `662082 jz near xxxxxxxx`. 好, 大功告成。

// file name `exp.cpp`

// compile with "cl exp.cpp user32.lib"

```cpp
#include <windows.h>

void main( void )
{
    DWORD m_processid, m_imagebase, m_iatrva, m_iatsize;
    m_processid = 0xFFF529C6;
    m_imagebase = 0x00400000;
    m_iatrva    = 0x0028C204;
    m_iatsize  = 0x00000A0C;

    unsigned char buf[MAX_PATH];
    HANDLE h_process;
    DWORD dw_ImageBase = 0x400000;
    unsigned int *p_iatva, *p_first_iatrva, *p_NextRedirect;
    unsigned long *p_oldprotection=0;
    int num, NestedCount, i;
    dw_ImageBase = m_imagebase;
    num = m_iatsize/4;

__try
{
    p_first_iatrva = (unsigned int*) m_iatrva;
    if ( (h_process = OpenProcess(PROCESS_VM_OPERATION | PROCESS_VM_READ | PROCESS_VM_WRITE, false, m_processid)) != NULL )
    {
        p_iatva  = (unsigned int *) ((unsigned int)p_first_iatrva + dw_ImageBase);
        for(i=0; i<num; i++)
        {
            if( ReadProcessMemory( h_process, (void *)p_iatva, buf, 4, 0 ) )
            {
                p_NextRedirect = (unsigned int*) buf;
                p_NextRedirect = (unsigned int*) *p_NextRedirect;
                for( NestedCount=0; NestedCount<5; NestedCount++)
                    if( ReadProcessMemory( h_process, (void *)p_NextRedirect, buf, 10, 0 ) )
                    {
                        if (*buf == 0xB8 && *(buf+5) == 0xff && *(buf+6) == 0xe0)    //mov eax, hhxxxxxx
                        {                                                                             //jmp eax
                            p_NextRedirect = (unsigned int *)(void *) (buf + 1);
                            p_NextRedirect = (unsigned int*) *p_NextRedirect;
                            if ( *(buf+4) < 0xB0 )                                           //when hh >= 0xb0 assume it is the real import address, only test on my ME.
                                continue;                                                    //Visual Protect seem to redirect kernel32.dll, user32.dll ... only.
                            else
                            {
                                if(!VirtualProtectEx(h_process, (void *)p_iatva, 4, PAGE_READWRITE, p_oldprotection))
                                {
                                    MessageBox(0, "Error changing access protection of process!", "exp", MB_OK);
                                    break;
                                }
                                if(!WriteProcessMemory(h_process, (void *)p_iatva, (void *)(buf+1), 4, 0))
                                {
                                    wsprintf( (char *) buf,"Write process memory error at %x , error code : %x", p_iatva, GetLastError());
                                    MessageBox(0, (char *) buf, "exp", MB_OK | MB_ICONERROR);
                                    break;
                                }
                                else
                                    break;  //next iat
                            }
                        }
                        else
                            break; //does not match format or already write
                    }
                    else
                    {
                        //MessageBox(0, "Read process memory error!", "exp", MB_OK | MB_ICONERROR);
                        break;
                    }
                p_iatva ++;  //get next iat
            }
            else
                MessageBox(0, "Read process IAT memory error!", "exp", MB_OK | MB_ICONERROR);
        }
        MessageBox(0, "Write process memory end.", "exp", MB_OK | MB_ICONINFORMATION);
    }
    else
        MessageBox(0, "Can not open process!", "exp", MB_OK | MB_ICONERROR);
}

__except(1)
{
    MessageBox(0, "Found exception!!!", "exp", MB_OK | MB_ICONERROR);
}

    CloseHandle(h_process);
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
