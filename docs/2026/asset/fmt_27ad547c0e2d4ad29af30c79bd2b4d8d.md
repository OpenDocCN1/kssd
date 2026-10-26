
```
                    mov  eax,0x23ba078
                    mov  ecx,eax
                    mov  ebx,0x729a00
                    mov  eax,0x0e
                    call ebx
                    popad
                }

            hws2_32 = LoadLibrary( "ws2_32.dll");
            (unsigned long)::GetProcAddress( hws2_32, "send");

            __asm
            {
                push 0x00
                push 0x0e
                lea  ebx, nCmd
                push ebx
                mov  ebx,my_sendhandle
                push ebx
                call eax
            }
            */
            break;

             case    IDC_READ_TABLE:
             MessageBox( NULL, "write", "write", MB_OK);

             for( nindex;nindex<0xa7a9;nindex++)
              {
                    lpRead = 0x00c45878+nindex;
                    ReadProcessMemory( GetCurrentProcess(), (LPVOID)lpRead,lpBuffer, 0x01, &dwRead);
                    sprintf( szFormat,"0x%2x",lpBuffer[0]);
                    WriteFile( hFile, szFormat, dwWrite, &dwRead, 0);
              }
            break;

             case IDC_BTN_HOOK:
                  hws2_32 = LoadLibrary( "ws2_32.dll");
                  hookApi = (unsigned long)::GetProcAddress( hws2_32, "send");
                  MyApi = (unsigned long )GetSendPara;
                  _HOOK_APIN( MyApi, hookApi);
                 break;

        default:
            break;
        }

    case WM_INITDIALOG:
        hFile = CreateFile( "c:\\MYDebugLog.txt",GENERIC_READ | GENERIC_WRITE ,FILE_SHARE_READ|FILE_SHARE_WRITE,
            NULL,OPEN_ALWAYS ,FILE_ATTRIBUTE_NORMAL,0);
        break;

    case WM_CLOSE:
        EndDialog( hWnd, 0);
        break;

    default:
        DefWindowProc( hWnd, uMsg, wParam, lParam);
    }

    return 0;
}

/*取send句柄*/
void    GetSendPara(void)
{
     __asm
     {
        /*首先要堆栈平衡下 因为VC6前边会有压栈操作*/
        pop eax
        pop eax
        pop eax
        pop eax
        /*执行判断操作，取send的句柄*/
        mov eax, dword ptr [esp + 0x0c]
        cmp eax,0x0e
        jnz JMP_HOOM
        mov eax,dword ptr[esp+0x04]
        mov my_sendhandle,eax
     }

     WriteProcessMemory( GetCurrentProcess(), (void*)hookApi, lpResetSend, 0x05, &dwWrite);
     dwWrite = 0;

     __asm
     {
JMP_HOOM:
        /*跳会原来的地方*/
        sub ebp,0x04
        mov eax,hookApi
        mov edi,edi
        push ebp
        mov  ebp,esp
        add eax,5
        jmp eax
     }

     return;
}

void    Encode( BYTE* pCmd, int nLen)
{
       __asm{
               pushad
               mov  eax,dword ptr[esp+0x34]//长度
               push eax
               mov  eax, dword ptr[esp+0x34]//命令明文序列
               push eax
               mov  eax,0x2fd078 // 硬编码---编码表--和计数表
               mov  ecx,eax
               mov  ebx,0x729a00//加密call
               mov  eax,dword ptr[esp+0x08]//长度
               call ebx
               popad
             }

       return;
}

void    GoRight()
{
    BYTE lpCmd[RUN_RIGHT_LEN] = {RUN_RIGHT};
    Encode( lpCmd,RUN_RIGHT_LEN);
    /*发送封包*/
    hws2_32 = LoadLibrary( "ws2_32.dll");
    (unsigned long)::GetProcAddress( hws2_32, "send");

    __asm
     {
           push 0x00
           push RUN_RIGHT_LEN
           lea  ebx, lpCmd
           push ebx
           mov  ebx,my_sendhandle
           push ebx
           call eax
       }

}

void    Speck_Something(char* pSpeckBuffer)
{
    //MessageBox( NULL, "speck","speck",MB_OK);
    BYTE        packLen;
        int count = 0;
        WCHAR        wszSpec[MAX_PATH];
        RtlZeroMemory( wszSpec,MAX_PATH*2);

    count = strlen( pSpeckBuffer);
        long nwLong = MultiByteToWideChar( CP_ACP, 0, pSpeckBuffer, strlen(pSpeckBuffer),
                                                                                        wszSpec,sizeof(wszSpec));

    BYTE    temp[0x08]={SPECK_ONE};
    BYTE    *lpCmd;
    lpCmd = new BYTE [200] ; //申请一块封包的内存
        RtlZeroMemory( lpCmd, 200);

    memcpy( lpCmd,temp,0x08);                //com封包命令
    memcpy( lpCmd,(void*)&count,0x01);

        packLen = (BYTE)0x0c+nwLong*2+2;  //包头 封包整个长度
        lpCmd[0] = packLen;
        lpCmd [0x0A] = (BYTE)nwLong+1;//包的11个字节 字符串长度
        memcpy( (lpCmd+12), wszSpec, nwLong*2+1);//把字符放入消息
        Encode( lpCmd, (int)lpCmd[0]);

        hws2_32 = LoadLibrary( "ws2_32.dll");
    (unsigned long)::GetProcAddress( hws2_32, "send");

    __asm
     {
        push 0x00
        push packLen
        lea  ebx, lpCmd
        push ebx
        mov  ebx,my_sendhandle
        push ebx
        call eax
     }

   // delete [] lpCmd;
}
```

由于是测试代码，写的相当潦草。这是个dll代码，这个代码是注如到游戏进程的。有hook api操作。

在操作这前，先抓出明文封包动作

```
#ifndef     __COMMON_H_
#define     __COMMON_H_
#include    <windows.h>

BOOL _HookApi( unsigned long _My_Addr, unsigned long _Hook_Addr);

/*command数据*/
/*向右走*/
#define RUN_RIGHT_LEN   0x0e
#define RUN_RIGHT   0x0e,0x00,0xe0,0x55,0x8d,0xe2,0x00,0x00,0x00,0x00,0x06,0x00,0x00,0x00

/*喊话*/
//喊话封包的长度不固定-首部为封包长度-然后8个字节的命令.接着4个字节的字符长度.跟到字符串
#define SPECK_ONE   0x00,0x00,0xe0,0x55,0xb9,0x6f,0x00,0x00

#endif
```

然后就xxxx.....

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
