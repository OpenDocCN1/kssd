
# [原创]某公司DLP产品通过驱动注入DLL分析

最近在分析某公司 DLP 产品，里边文档控制相关功能，出于好奇分析了一下原理。

应用层的权限控制一般通过全局 Hook 注入 DLL 进行权限控制，使用工具 PCHunder 发现，除了 `WH_KEYBOARD_LL` 消息 Hook，没有其它消息 Hook，最后通过 PCHunder 发现是通过驱动层进行设置 `PsSetCreateProcessNotifyRoutine` 回调注入 Shellcode 实现。

## IDA 分析

具体 IDA 分析如下图：

```c
bool __stdcall sub_15D40(HANDLE ProcessHandle, wchar_t *wzDllPath)

{

  int v3; // [sp+0h] [bp-488h]@21

  int v4; // [sp+4h] [bp-484h]@11

  char v5; // [sp+8h] [bp-480h]@11

  int v6; // [sp+14h] [bp-474h]@12

  int v7; // [sp+18h] [bp-470h]@13

  unsigned __int8 v8; // [sp+240h] [bp-248h]@2

  int v9; // [sp+241h] [bp-247h]@2

  unsigned __int8 v10; // [sp+245h] [bp-243h]@2

  int v11; // [sp+246h] [bp-242h]@2

  unsigned __int16 v12; // [sp+24Ah] [bp-23Eh]@2

  int v13; // [sp+24Ch] [bp-23Ch]@2

  int v14; // [sp+250h] [bp-238h]@2

  int v15; // [sp+254h] [bp-234h]@3

  __int16 v16; // [sp+25Dh] [bp-22Bh]@5

  __int16 v17; // [sp+25Fh] [bp-229h]@5

  int v18; // [sp+261h] [bp-227h]@5

  wchar_t v19; // [sp+265h] [bp-223h]@5

  int v20; // [sp+46Dh] [bp-1Bh]@5

  int v21; // [sp+471h] [bp-17h]@5

  unsigned __int8 v22; // [sp+478h] [bp-10h]@8

  int v23; // [sp+479h] [bp-Fh]@11

  bool v24; // [sp+483h] [bp-5h]@1

  int v25; // [sp+484h] [bp-4h]@1



  v24 = 0;

  v25 = (int)MyAllocateVirtualMemory(ProcessHandle, 0x2BBu, 0);

  if ( v25 )

  {

    v8 = 0xB8u;                                  // MOV EAX,XXXXXXXX

    v9 = v25;

    v10 = 0xB9u;                                // MOV ECX,XXXXXXXX

    v11 = v25 + 565;

    v12 = 0xD1FFu;                            // CALL ECX

    v13 = 0x4C69636D;                     // 特征码

    v14 = 0;

    v15 = pfnNtTestAlert ? pfnNtTestAlert : sub_15570(ProcessHandle);

    v16 = 2 * wcslen(wzDllPath);                // UNICODE_STRING ++0 Length

    v17 = v16 + 2;                                    // UNICODE_STRING ++2 MaxmumLength

    v18 = v25 + 0x25;                              // UNICODE_STRING ++4 Buffer

    wcscpy(&v19, wzDllPath);                    // 将wzDllPath拷贝进ShellCode

    v20 = pfnNtProtectVirtualMemory;

    v21 = pfnLdrLoadDll;

    if ( v15 )

    {

      if ( MyWriteVirtualMemory((int)ProcessHandle, v25, (int)&v8, 566)

        && MyWriteVirtualMemory((int)ProcessHandle, v25 + 0x235, (int)sub_10420, 133)

        && MyReadVirtualMemory((int)ProcessHandle, v15, (int)&v22, 5)// 读取pfnNtTestAlert函数前5个字节备份到ShellCode中

        && MyWriteVirtualMemory((int)ProcessHandle, v25 + 0x18, (int)&v22, 5) )

      {

        if ( v22 == 0xE9 )                      // 构造要插入dll的链表ShellCode

        {

          v4 = v15 + v23 + 5;

          if ( MyReadVirtualMemory((int)ProcessHandle, v4, (int)&v5, 566) && v6 == 0x4C69636D )

          {

            while ( v7 )

            {

              v4 = v7;

              if ( !MyReadVirtualMemory((int)ProcessHandle, v7, (int)&v5, 566) || v6 != 0x4C69636D )

              {

                v4 = 0;

                break;

              }

            }

            if ( v4 )

            {

              v7 = v25;

              v24 = MyWriteVirtualMemory((int)ProcessHandle, v4, (int)&v5, 566);

            }

          }

        }

        if ( !v24 && MyProtectVirutalMemory((int)ProcessHandle, v15, 5, 64, (int)&v3) )// 修改pfnNtTestAlert函数前面5个字节为jmp XXXX,其中XXXX为Shellcode入口地址

        {

          v22 = 0xE9u;

          v23 = v25 - v15 - 5;

          MyWriteVirtualMemory((int)ProcessHandle, v15, (int)&v22, 5);

          MyProtectVirutalMemory((int)ProcessHandle, v15, 5, v3, (int)&v3);

          v24 = 1;

        }

      }

    }

  }

  return v24;

}
```

## Windbg 验证

使用 Windbg 启动 `notepad`，反汇编发现 `ntdll!NtTestAlert` 函数前 5 个字节被修改为 `jmp` 指令，随后将 `jmp` 位置的内存进行转储分析如下：
