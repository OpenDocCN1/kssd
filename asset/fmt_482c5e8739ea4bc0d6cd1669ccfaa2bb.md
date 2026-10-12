
# svkp 1.43 分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-636.htm`](https://www.pediy.com/kssd/pediy08/pediy8-636.htm)

以下数据来自：

版本： `svk-protector` v1.43

样本: `xp` 下的 `notepad`，默认参数加壳

svkp 特点:

1. 大量花指令
2. 调试器检测
3. 抽入口
4. 抽 api
5. 驱动
6. 大量自修改 SMC，分块解密.
7. 大量构造运行栈. 运行栈指令
8. 异常处理
9. 一些小技巧.

## 整体流程

附加 SFX 节中:

1. SMC 几次后，建栈指令，栈中得到 `kernel.dll` 的 `imagebase`，出指令栈
2. SMC 几次后，建立 `kernel` 中的关键 api，如 `loadlibrary`，`virtualalloc` 等，加密之
3. 分配内存块 0，内存块 1，内存块 2，自解密 (内存块 1 貌似没使用)
   把 `fake_api_call` 的代码拷贝到内存块 1，其他代码数据拷贝到内存块 2.
   跑到内存块 2 继续执行.

内存块 2 中:

4. 建栈指令，栈中分配内存块 3，内存块 4，拷贝自己代码到内存块 3 和 4 中.
5. 在内存块 3 和 4 中分别建立 SEH，解密内存块 4，产生 `c0000005` 异常.
6. 在 4 的 SEH 中解密内存块 3，`SEH_CONTINUE_SEARCH` 到内存块 3
7. 在 3 的 SEH 中解密内存块 2，设置 `eip` 为内存块 2 解密后的代码，跑过去
8. 建栈指令，栈中取消 3,4 中的 SEH，通过便例 `kernel32.dll` 中的 export table
   中的 name table，hash 一把 name，查一张 hash 表，来判断这个 api 是否壳
   自己需要用，如果需要，则去找 `symbol_addr`，然后加密后写另外张地址表
9. 接密数据得到 `user32.dll`，加载后，用 8 同样的方法得到壳需要使用的 api
10.释放内存块 1,3,4，分配内存块 5，解码自己的其他数据到内存块 5 中.
   跑到内存块 5 继续执行.

内存块 5 中:

11.建栈指令，栈中建立 3 个 SEH 连，解码最后一个 seh 的 handle 代码.产生 `c00000005` 异常
12.seh 解密数据，跑到解密数据处继续执行.
13.建栈指令，栈中分配内存块 6，把内存块 1 拷贝到 6 中，取代原来的 `fake_api_call`，释放快 1
14.产生异常 `c000008c`(array_bound_exceeded)，等待 seh 处理
15.seh 一直提交到第一个 seh 处理,解密数据，跑到解密数据处继续执行.
16.再次解密数据，产生 `c0000005` 异常，是通过 `INT 1` 指令(异常会判断)产生的，等待 SEH 处理
17.seh 一直提交到第一个 seh 处理,设置新的 `eip`，跑到这个 `eip` 执行.
18.取消 3 个 seh 连. 接密数据得到 `advapi32.dll`，加载后，用 8 同样的方法得到壳需要使用的 api
19.打开驱动 `GetSystemDirectoryA\svkp.sys` 没有则生成改文件,驱动不存在则打开.
20.依次发送 `IoControlCode` 为 222000,222004,222014,22200c 的代码，判断返回值.
21.建栈指令，栈中根据记录信息循环解密节数据，资源和普通数据分开对待.
22.重建立 IAT，`SPECIAL.DLL` 这个 sdk 的 dll 另外处理.(抽 api 代码，需要还原,很简单)
23.运行抽去的 oep，然后跑到原始 `oep` +抽过的地方. (可还原,不过复杂)

至于具体步骤的详细流程，一会会详细解释.

to be continued.

## 第一部分： 花指令

以下是 svkp 常用的花指令. 可以直接在花指令后 `f4` 跳过.

注: 凡是 `db` 掉的，表示不会被执行，但里面的字节数不一定就是该值

由于 `rdtsc` 的泛用，可以用个让 `edx:eax` 递增的插件来解决.

```
\===============================

花指令 0_2:

    jmp +2

db ?? ??

花指令 0_3:

    jmp +3

db ?? ?? ??

花指令 0_4:

    jmp +4

db ?? ?? ?? ??

\===============================

花指令 1:

    call +1

db e8h          

    call +2

db cdh, 20h

    add dword ptr[esp], 0bh

    add dword ptr[esp+4], 13h

    ret

db e9h

\===============================

花指令 2:

    call +1

db 9ah

    add esp, 4

\================================ 

花指令 3:

    pushad

    call @1

db d2h

    jmp @4

@1:

    pop eax

    jmp @2

db 48h

@2:

    inc eax

    jmp @3

db 35h

@3:

    jmp eax

@4:

    popad

\================================ 

花指令 4:

    jl  @1

@2:

    jmp @3

db 8bh

@1:

    je  @2

@3:

\================================= 

花指令 5:

    call +2

db cdh, 02h

    add dword ptr[esp], 8

    ret

db e8h

\================================== 

花指令 6:  带 rdtsc  长 df 字节   == push ecx

    push eax

    push edx

    [花指令 0_2]

    rdtsc

    [花指令 0_2]

    [花指令 0_2]

    call @sub_eip_add_1

db c3h

    [花指令 0_4]

    push eax

    push ebx

    call +4

db 8dh, b5h, d4h, 46h

    pop ax

    cmp esi, esi

    pop bx

    jnz ??? = nop

    [花指令 2]

    pop eax

    [花指令 0_3]

    call @sub_eip_add_2

db ffh, e4h

    cmp edi, edi

    pop ebx

    xchg eax, ebx   

    jnz ??? = nop

    push ebx

    [花指令 0_2]

    mov ebx, edx

    [花指令 5]

    ;执行到这, rdtsc 的 edx 保存在 ebx 里面, ebx 给压栈保存了. 下面再次 rdtsc 比较

    rdtsc

    [花指令 2]

    sub edx, ebx  ; 这里需要注意, edx-ebx 只能为 0 或者 1, 要不然就飞了

    [花指令 0_3]

    pop ebx

    [花指令 4]

    sub edx, 2

    [花指令 4]

    jns ???       ;这里跳就飞了

    [花指令 4]

    pop edx

    [花指令 0_3]

    pop eax

    [花指令 0_3]

    [花指令 4]

    pop ebp

    jmp over

//////////////////

@sub_eip_add_1:

    call @eip_inc

db ffh

    inc dword ptr[esp]

    [花指令 0_2]

    ret

//////////////////

eip_inc:

    push ebp

    mov ebp, esp

    sub ebp, 80

    jnz +1 = jmp

db ffh      

    add dword ptr[esp+84], 1

    jnz +2 = jmp 

db 68h, 58h

    mov esp, ebp

    pop ebp

    ret

//////////////////

@sub_eip_add_2:

    call eip_inc

db ffh

    sub dword ptr[esp], -2

    jnz +2 = jmp

db 8dh, 87h

    [花指令 0_2]

    ret

///////////////////   

over:

\================================== 

花指令 7:               长 68 字节

push eax     

push ebx

call +4

db ?? ?? ?? ??

pop ax

cmp esi, esi

pop bx

jnz ?? = nop

pop eax

jmp +3

db ?? ?? ??

add_eip_2

db ?? ??

cmp edi, edi

pop ebx

xchg eax, ebx

jnz ?? = nop

jmp add_eip_2_over

add_eip_2

add_eip_2_over:

\==================================== 

花指令 8:    带 rtdsc 和自修改功能 长度 137  ==push eax

9c9c93:

pushad

call +0             ;ebp

pop ebp

\------------------------------ 

call +2                    |

db ?? ??                   |

add dword ptr [esp], 8     |

ret                       |

db ??                     |

\------------------------------ 

sub ebp, 5  

\------------------------------ 

jmp +2                    |

db ?? ??                  |

\------------------------------ 

\------------------------------ 

jmp +2                    |

db ?? ??                  |

\------------------------------ 

rdtsc

\------------------------------ 

call +2                    |

db ?? ??                   |

add dword ptr[esp], 8       |

ret                       |

db ??                     |

\------------------------------ 

mov eax, 5c

\------------------------------ 

jl +3                     |

jmp +3                    |

db ??                     |

je -5                     |

\------------------------------ 

add eax, ebp

\------------------------------ 

call +1                    |

db ??                     |

add esp, 4                |

\------------------------------ 

mov ecx, 5e; 0    <----------------- 

                         | |

\------------------------------   | |

jmp +3                    |   | |

db ?? ?? ??               |   | |

\------------------------------   | |

                         | |

\------------------------------   | |

jl +3                     |   | |

jmp +3                    |   | |

db ??                     |   | |

je -5                     |   | |

\------------------------------   | |

                         | |

mov dword ptr[eax], ecx 5e,0---> | |

                       | | |

\------------------------------ | | |

jmp +2                    | | | |

db ?? ??                  | | | |

\------------------------------ | | |

                       | | |

call +5e == @1;0         <------ | |

                         | |

pop eax                  | |

                         | |

\------------------------------   | |

jl +3                     |   | |

jmp +3                    |   | |

db ??                     |   | |

je -5                     |   | |

\------------------------------   | |

                         | |

mov ecx, edx             | |

                         | |

\------------------------------   | |

call +1                    |   | |

db ??                     |   | |

add esp, 4                |   | |

\------------------------------   | |

                         | |

rdtsc                     | | 

                         | |  
```
