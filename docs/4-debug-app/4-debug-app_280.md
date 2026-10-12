
# 红心大战内存补丁

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-309.htm`](https://www.pediy.com/kssd/pediy07/pediy7-309.htm)

## 目标与功能

目标：`XP` 下的红心大战。
功能：玩一次就赢。
说明：红心大战类似拱猪游戏，得分最少为赢家。

在 `.DrawTextW` 下断进入得分关键算法功能处。

```
010054AE   .  FFD7          call edi                                      ;  USER32.DrawTextW
```

相关内存区域保存着分数数据：

```
0100D41C  00000000  ....
0100D420  00000011  ...
0100D424  00000028  (...
0100D428  00000034  4...
```

## 汇编代码分析

以下代码展示了得分计算与绘制的关键逻辑：

```
010054E0   > /8B45 CC       mov eax,dword ptr ss:\[ebp-34\]               ;  mshearts.0100D420

010053EC   .  8D45 A8       lea eax,dword ptr ss:\[ebp-58\]
010053EF   .  50           push eax
010053F0   .  8D8D 38FFFFFF lea ecx,dword ptr ss:\[ebp-C8\]
010053F6   .  E8 31540000   call <jmp.&MFC42u.#5785>
010053FB   .  8B3D F0130001 mov edi,dword ptr ds:\[<&USER32.DrawTextW>\]    ;  USER32.DrawTextW
01005401   .  8945 90       mov dword ptr ss:\[ebp-70\],eax
01005404   .  8B45 A4       mov eax,dword ptr ss:\[ebp-5C\]
01005407   .  83C0 0F       add eax,0F
0100540A   .  895D C4       mov dword ptr ss:\[ebp-3C\],ebx
0100540D   .  895D F0       mov dword ptr ss:\[ebp-10\],ebx
01005410   .  C745 E8 20D40>mov dword ptr ss:\[ebp-18\],mshearts.0100D420
01005417   .  C745 E4 FBFFF>mov dword ptr ss:\[ebp-1C\],-5
0100541E   .  8945 E0       mov dword ptr ss:\[ebp-20\],eax
01005421   >  A1 F0D40001   mov eax,dword ptr ds:\[100D4F0\]               ;  打牌次数
01005426   .  3BC3          cmp eax,ebx
01005428   .  7E 30         jle short mshearts.0100545A
0100542A   .  8B4D F0       mov ecx,dword ptr ss:\[ebp-10\]
0100542D   .  03C8          add ecx,eax
0100542F   .  8B45 C0       mov eax,dword ptr ss:\[ebp-40\]
01005432   .  39048D 1CD400>cmp dword ptr ds:\[ecx*4+100D41C\],eax
01005439   .  75 1F         jnz short mshearts.0100545A
0100543B   .  A1 F4D40001   mov eax,dword ptr ds:\[100D4F4\]
01005440   .  F7D8          neg eax
01005442   .  1BC0          sbb eax,eax
01005444   .  25 7F0001FF   and eax,FF01007F
01005449   .  05 0000FF00   add eax,0FF0000
0100544E   .  50           push eax
0100544F   .  8D8D 38FFFFFF lea ecx,dword ptr ss:\[ebp-C8\]
01005455   .  E8 CC530000   call <jmp.&MFC42u.#6168>
0100545A   >  FF75 C4       push dword ptr ss:\[ebp-3C\]
0100545D   .  8B0D 14D50001 mov ecx,dword ptr ds:\[100D514\]
01005463   .  8D45 98       lea eax,dword ptr ss:\[ebp-68\]
01005466   .  50           push eax
01005467   .  E8 DAFBFFFF   call mshearts.01005046
0100546C   .  50           push eax
0100546D   .  8D4D EC       lea ecx,dword ptr ss:\[ebp-14\]
01005470   .  C645 FC 09    mov byte ptr ss:\[ebp-4\],9
01005474   .  E8 A7530000   call <jmp.&MFC42u.#858>
01005479   .  8D4D 98       lea ecx,dword ptr ss:\[ebp-68\]
0100547C   .  C645 FC 08    mov byte ptr ss:\[ebp-4\],8
01005480   .  E8 15520000   call <jmp.&MFC42u.#800>
01005485   .  FF75 8C       push dword ptr ss:\[ebp-74\]                   ;  /Bottom
01005488   .  8D45 B0       lea eax,dword ptr ss:\[ebp-50\]                ;  |
0100548B   .  FF75 E0       push dword ptr ss:\[ebp-20\]                   ;  |Right
0100548E   .  6A 05         push 5                                         ;  |Top = 5
01005490   .  FF75 E4       push dword ptr ss:\[ebp-1C\]                   ;  |Left
01005493   .  50           push eax                                         ;  |pRect
01005494   .  FF15 B8130001 call dword ptr ds:\[<&USER32.SetRect>\]         ;  \SetRect
0100549A   .  68 01080000   push 801                                       ;  UINT uFormat     // text-drawing flags
0100549F   .  8D45 B0       lea eax,dword ptr ss:\[ebp-50\]
010054A2   .  50           push eax                                         ;   LPRECT lpRect,    // pointer to struct with formatting dimensions
010054A3   .  6A FF         push -1                                        ;  int nCount,       // string length, in characters
010054A5   .  FF75 EC       push dword ptr ss:\[ebp-14\]                   ;   name             LPCTSTR lpString, // pointer to string to draw
010054A8   .  FFB5 3CFFFFFF push dword ptr ss:\[ebp-C4\]                   ;  HDC hDC,           // handle to device context
010054AE   .  FFD7          call edi                                       ;   USER32.DrawTextW
010054B0   .  0145 B4       add dword ptr ss:\[ebp-4C\],eax               ;  返回值：成功返回字符高度，失败返回 0
010054B3   .  8945 94       mov dword ptr ss:\[ebp-6C\],eax
010054B6   .  8D45 9C       lea eax,dword ptr ss:\[ebp-64\]
010054B9   .  50           push eax
010054BA   .  8D8D 38FFFFFF lea ecx,dword ptr ss:\[ebp-C8\]
010054C0   .  E8 67530000   call <jmp.&MFC42u.#5785>
010054C5   .  8D4D EC       lea ecx,dword ptr ss:\[ebp-14\]
010054C8   .  E8 03520000   call <jmp.&MFC42u.#2606>
010054CD   .  A1 F0D40001   mov eax,dword ptr ds:\[100D4F0\]
010054D2   .  48           dec eax
010054D3   .  85C0         test eax,eax
010054D5   .  895D D0       mov dword ptr ss:\[ebp-30\],ebx
010054D8   .  7E 4C         jle short mshearts.01005526
010054DA   .  8B45 E8       mov eax,dword ptr ss:\[ebp-18\]
010054DD   .  8945 CC       mov dword ptr ss:\[ebp-34\],eax
010054E0   >  8B45 CC       mov eax,dword ptr ss:\[ebp-34\]               ;  *****!!!mshearts.0100D420
010054E3   .  FF30          push dword ptr ds:\[eax\]
010054E5   .  8D4D D8       lea ecx,dword ptr ss:\[ebp-28\]               ;  第一次得分记录
010054E8   .  68 14180001   push mshearts.01001814                       ;  UNICODE "%d
010054ED   .  6A 14         push 14
010054EF   .  E8 D0510000   call <jmp.&MFC42u.#2910>
010054F4   .  50           push eax                                         ; |s
010054F5   .  FF15 A0130001 call dword ptr ds:\[<&USER32.wsprintfW>\]      ; \wsprintfW
010054FB   .  83C4 0C       add esp,0C
010054FE   .  6A FF         push -1
01005500   .  8D4D D8       lea ecx,dword ptr ss:\[ebp-28\]
01005503   .  E8 B6510000   call <jmp.&MFC42u.#5568>
01005508   .  8D45 D8       lea eax,dword ptr ss:\[ebp-28\]
0100550B   .  50           push eax
0100550C   .  8D4D EC       lea ecx,dword ptr ss:\[ebp-14\]
0100550F   .  E8 00530000   call <jmp.&MFC42u.#940>
01005514   .  FF45 D0       inc dword ptr ss:\[ebp-30\]
01005517   .  8345 CC 04    add dword ptr ss:\[ebp-34\],4
0100551B   .  A1 F0D40001   mov eax,dword ptr ds:\[100D4F0\]
01005520   .  48           dec eax
01005521   .  3945 D0       cmp dword ptr ss:\[ebp-30\],eax
01005524   .^ 7C BA         jl short mshearts.010054E0
01005526   >  6A 01         push 1
01005528   .  8D45 B0       lea eax,dword ptr ss:\[ebp-50\]
0100552B   .  50           push eax
0100552C   .  6A FF         push -1
0100552E   .  FF75 EC       push dword ptr ss:\[ebp-14\]
01005531   .  FFB5 3CFFFFFF push dword ptr ss:\[ebp-C4\]
01005537   .  FFD7          call edi
01005539   .  A1 F0D40001   mov eax,dword ptr ds:\[100D4F0\]
0100553E   .  48           dec eax
0100553F   .  0FAF45 94    imul eax,dword ptr ss:\[ebp-6C\]
01005543   .  0145 B4       add dword ptr ss:\[ebp-4C\],eax
01005546   .  8D45 A8       lea eax,dword ptr ss:\[ebp-58\]
01005549   .  50           push eax
0100554A   .  8D8D 38FFFFFF lea ecx,dword ptr ss:\[ebp-C8\]
01005550   .  E8 D7520000   call <jmp.&MFC42u.#5785>
01005555   .  A1 F0D40001   mov eax,dword ptr ds:\[100D4F0\]
0100555A   .  3BC3          cmp eax,ebx
0100555C   .  7E 2F         jle short mshearts.0100558D
0100555E   .  8B4D F0       mov ecx,dword ptr ss:\[ebp-10\]
01005561   .  03C8          add ecx,eax
01005563   .  FF348D 1CD400>push dword ptr ds:\[ecx*4+100D41C\]             ;  100d41c 里保存着我的分数
0100556A   .  8D4D EC       lea ecx,dword ptr ss:\[ebp-14\]                 ;  第二次得分记录
```



```
0100556D   .  68 0C180001   push mshearts.0100180C                        ;  UNICODE "%d"  

01005572   .  6A 14         push 14  

01005574   .  E8 4B510000   call <jmp.&MFC42u.#2910>  

01005579   .  50           push eax                                        ; |s  

0100557A   .  FF15 A0130001 call dword ptr ds:[<&USER32.wsprintfW>]       ; \wsprintfW  

01005580   .  83C4 0C       add esp,0C  

01005583   .  6A FF        push -1  

01005585   .  8D4D EC      lea ecx,dword ptr ss:[ebp-14]  

01005588   .  E8 31510000  call <jmp.&MFC42u.#5568>  

0100558D   >  6A 01        push 1  

0100558F   .  8D45 B0      lea eax,dword ptr ss:[ebp-50]  

01005592   .  50           push eax  

01005593   .  6A FF        push -1  

01005595   .  FF75 EC      push dword ptr ss:[ebp-14]  

01005598   .  FFB5 3CFFFFFF push dword ptr ss:[ebp-C4]  

0100559E   .  FFD7         call edi  

//////////////////////////////////////////////////////////////////////////////////////////////////
```

```
  int buf[13]={0}; //积分数组为 13 个元素  

    int buf2[13]={0};  

    HWND wnd1;   

    HANDLE ID;   

    DWORD Pid;   

    int i,fen;   

    unsigned long len;   

    long *m_addr,*m_addr2; //Microsoft 网上红心大战的关键数组地址   

    m_addr = (long *) 0x100d41c;   

    m_addr2 = (long *) 0x100D4BC;   


    ::memset(buf,0,13);   

    ::memset(buf2,0,13);   

    if( (wnd1 = ::FindWindow(NULL,"Microsoft 网上红心大战"))==NULL)   

    {   

        AfxMessageBox("Microsoft 网上红心大战未运行!");   

        return;   

    }   

    ::GetWindowThreadProcessId(wnd1,&Pid);   

    ID=::OpenProcess(PROCESS_ALL_ACCESS,false,Pid);   

    if(ID)   

    {   

        ::ReadProcessMemory(ID,m_addr,buf,52,&len);//将 Microsoft 网上红心大战的数据读入 buf 数组中   

        ::ReadProcessMemory(ID,m_addr2,buf2,52,&len);  

    for(i=0;i<13;i++)//  
        {   
          buf[i]=0;//数组的值代表分数  

        }   

       for(i=0;i<13;i++)//  
        {   

          buf2[i]+=100;//数组的值代表分数  

        }   

      //::DrawText( const CString& str, LPRECT lpRect, DT_CENTER    );  

        ::WriteProcessMemory(ID,m_addr,buf,52,&len);//将处理过的值再次写入 Microsoft 网上红心大战中。  

        ::WriteProcessMemory(ID,m_addr2,buf2,52,&len);//将处理过的值再次写入 Microsoft 网上红心大战中。对方一定死翘翘了。   

    ::CloseHandle(ID);   

        ::BringWindowToTop(wnd1);   

    }
```

`4dd466d0e1e0cc123afbe504da2462ca`
