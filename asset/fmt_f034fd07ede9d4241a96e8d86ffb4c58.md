
# 360 比赛第 5 题-我的解法

> 原文：[`www.pediy.com/kssd/pediy10/78997.html`](https://www.pediy.com/kssd/pediy10/78997.html)

这题得了 100 分哦。

驱动加载后，直接对对 `IoCreateFile` 函数进行 `PATCH`,

`PATCH` 前函数为：

引用: `8056ba8c` `ff7508`          `push`    `dword ptr [ebp+8]`

`8056ba8f` `e882f2ffff`        `call`    `nt!IopCreateFile (8056ad16)`

`8056ba94` `3bc6`            `cmp`     `eax,esi`

`8056ba96` `7d15`            `jge`     `nt!IoCreateFile+0xa7 (8056baad)`

`8056ba98` `f6c301`          `test`    `bl,1`

`8056ba9b` `0f8574ffffff`      `jne`     `nt!IoCreateFile+0xf (8056ba15)`

`PATCH` 后函数为：

引用: `8056ba8c` `ff7508`          `push`    `dword ptr [ebp+8]`

`8056ba8f` `e8421a7b78`      `call`    `fileprot+0x4d6 (f8d1d4d6)`

`8056ba94` `3bc6`            `cmp`     `eax,esi`

`8056ba96` `7d15`            `jge`     `nt!IoCreateFile+0xa7 (8056baad)`

`8056ba98` `f6c301`          `test`    `bl,1`

`8056ba9b` `0f8574ffffff`    `jne`     `nt!IoCreateFile+0xf (8056ba15)`

之后所有调用 `IoCreateFile` 函数的都要经 `fileprot` 进行过滤；

代码:

```
00010522    call    dword\_10804    //  先调用原来的 nt!IopCreateFile                                           
00010528    mov     [ebp+var\_210], eax                                                 
0001052E    mov     eax, [ebp+var\_210]                                                 
00010534    test    eax, eax                                                           
00010536    jl      short loc\_105AD                                                    
00010538    mov     edx, [esi+8]                                                       
0001053B    cmp     word ptr [edx], 206h                                               
00010540    ja      short loc\_105AD                                                    
00010542    cmp     dword ptr [edx+4], 0                                               
00010546    jz      short loc\_105AD                                                    
00010548    cmp     word ptr [edx], 0                                                  
0001054C    jz      short loc\_105AD                                                    
0001054E    mov     eax, edx                                                           
00010550    movzx   ecx, word ptr [eax]                                                
00010553    mov     esi, [eax+4]                                                       
00010556    mov     eax, ecx                                                           
00010558    shr     ecx, 2                                                             
0001055B    lea     edi, [ebp+var\_20C]                                                 
00010561    rep movsd                                                                  
00010563    mov     ecx, eax                                                           
00010565    and     ecx, 3                                                             
00010568    rep movsb                                                                  
0001056A    movzx   eax, word ptr [edx]                                                
0001056D    and     [ebp+eax+var\_20C], 0                                               
00010576    lea     eax, [ebp+var\_20C]                                                 
0001057C    push    eax             ; wchar\_t \*                                        
0001057D    call    ds:\_wcsupr                                                         
00010583    lea     eax, [ebp+var\_20C]                                                 
00010589    mov     [esp+220h+var\_220], offset a360game360game ;
 "360GAME\\\\360GAME.TXT"
00010590    push    eax             ; wchar\_t \*                                        
00010591    call    ds:wcsstr                                                          
00010597    test    eax, eax                                                           
00010599    pop     ecx                                                                
0001059A    pop     ecx                                                                
0001059B    jz      short loc\_105A7                                                    
0001059D    and     dword ptr [ebx], 0                                                 
000105A0    mov     eax, 0C0000022h  
```

当发现文件名路径中含有`"360GAME\\\\360GAME.TXT"`，直接返回 `0C0000022h`，即 拒绝访问，导致访问失败。

实际上，该驱动不仅保护 `c:\\360game\\360game.Txt` 不被打开，所有文件名路径中含有`"360GAME\\\\360GAME.TXT"`都将被保护，如 `d:\\360game\\360game.Txt` 等等。

要突破其保护，只要文件名路径中不含有`"360GAME\\\\360GAME.TXT"`即可，方法是设置当前路径到 `c:\\360game`，然后直接对 `360game.Txt` 进行写就可以了。

另外，如果驱动其作用后，试图在通过双击直接打开 `c:\\360game\\360game.Txt` 的动作会导致 `explorer.exe` 占用 `c:\\360game\\360game.Txt` 的文件句柄，因此，在对 `c:\\360game\\360game.Txt` 进行写操作之前，先应该释放所有可能占用的文件句柄。在本程序中，是通过 `CloseFileHandle()` 来实现的，详细实现参见附件。

程序运行后，写入的内容为：

`“360game by windsun,and the tickCount is %d”`

其中`%d` 为 `GetTickCount()` 的返回值。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `AntiFile_by_windsun.rar` |

