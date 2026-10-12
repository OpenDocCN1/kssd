
```
008033CD   >  E9 EE200000    jmp     008054C0  
```

往上到了类函数：

```
006895D0  /$  55             push    ebp  
006895D1  |.  8BEC           mov     ebp, esp  
006895D3  |.  51             push    ecx  
006895D4  |.  894D FC        mov     dword ptr [ebp-4], ecx  
006895D7  |.  E8 E49D1700    call    008033C0  //实际这里直接来个 mov eax,755 代表执行成功，也算简单.  
006895DC  |.  8BE5           mov     esp, ebp  
006895DE  |.  5D             pop     ebp  
006895DF  \.  C3             retn  
```

我不说怎么跳了，麻烦死了~~大家自己看情况跳吧。。。只要不调用它就是了~~~~~~~

第四个函数 `Auth(DWORD dwAuthCode)`，也是最麻烦的一个，这个验证码我玩的游戏会用到的。

```
00803450  /$  8B0D 80062808  mov     ecx, dword ptr [8280680]  
00803456  |.  85C9           test    ecx, ecx  
00803458  |.  75 03          jnz     short 0080345D  
0080345A  |.  32C0           xor     al, al  
0080345C  |.  C3             retn  
0080345D  |>  8B4424 04      mov     eax, dword ptr [esp+4]  
00803461  |.  50             push    eax  
00803462  |.  E8 19220000    call    00805680  
00803467  \.  C3             retn  
```

真正函数体：

```
00805680  /$  56             push    esi  
00805681  |.  8BF1           mov     esi, ecx  
00805683  |.  803E 00        cmp     byte ptr [esi], 0  
00805686  |.  74 3B          je      short 008056C3  
00805688  |.  57             push    edi  
00805689  |.  8B7C24 0C      mov     edi, dword ptr [esp+C]  
0080568D  |.  57             push    edi  
0080568E  |.  68 14F48700    push    0087F414  
00805693  |.  E8 E8540000    call    0080AB80  
00805698  |.  83C4 04        add     esp, 4  
0080569B  |.  50             push    eax  
0080569C  |.  8D46 28        lea     eax, dword ptr [esi+28]  
0080569F  |.  50             push    eax  
008056A0  |.  E8 5BFAFFFF    call    00805100  
008056A5  |.  83C4 0C        add     esp, 0C  
008056A8  |.  8D8E 64100000  lea     ecx, dword ptr [esi+1064]  
008056AE  |.  57             push    edi  
008056AF  |.  68 16060000    push    616  
008056B4  |.  68 B1080000    push    8B1  
008056B9  |.  E8 E2390000    call    008090A0  
008056BE  |.  5F             pop     edi  
008056BF  |.  5E             pop     esi  
008056C0  |.  C2 0400        retn    4  
008056C3  |>  32C0           xor     al, al  
008056C5  |.  5E             pop     esi  
008056C6  \.  C2 0400        retn    4  
```

地方已经给大家了，大家看情况来做吧！是在那改也很容易的。

最后一个 `Exit()`，么什么了，这个函数跳不跳都不影响的。

```
008034B0  /$  8B0D 80062808  mov     ecx, dword ptr [8280680]  
008034B6  |.  85C9           test    ecx, ecx  
008034B8  |.  75 03          jnz     short 008034BD  
008034BA  |.  32C0           xor     al, al  
008034BC  |.  C3             retn  
008034BD  |>  53             push    ebx  
008034BE  |.  56             push    esi  
008034BF  |.  E8 BC220000    call    00805780  
008034C4  |.  8B0D 80062808  mov     ecx, dword ptr [8280680]  
008034CA  |.  8AD8           mov     bl, al  
008034CC  |.  85C9           test    ecx, ecx  
008034CE  |.  8BF1           mov     esi, ecx  
008034D0  |.  74 0E          je      short 008034E0  
008034D2  |.  E8 19000000    call    008034F0  
008034D7  |.  56             push    esi  
008034D8  |.  E8 99D7FDFF    call    007E0C76  
008034DD  |.  83C4 04        add     esp, 4  
008034E0  |>  8AC3           mov     al, bl  
008034E2  |.  5E             pop     esi  
008034E3  |.  C705 80062808 >mov     dword ptr [8280680], 0  
008034ED  |.  5B             pop     ebx  
008034EE  \.  C3             retn  
```

也许大家会说少个回调函数，说句很不好意思的话，那个玩意我还么怎么搞明白的。大家去试试吧~~

按这 5 步做下来，基本上 NP 会不见的。而且，有的游戏就完全可以玩了，有的不知道为什么，竟然会掉线~~

郁闷中~~~~~~~~

![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") 

大家就留着自己用吧，请不要转载了~~~![](img/32b12ba0f4c0e6a8843aaa06ba62fbdd.jpg "皱眉") 

免的以后不好使了哈哈~~~~~~~~~

-By EasyStudy  
2008 年 07 月 20 日 凌晨

类是这样的。

```cpp
class CNPGameLib
{
public:
explicit CNPGameLib(LPCTSTR lpszGameName)
{
#ifdef NO_GAMEGUARD
return;
#endif
PreInitNPGameMon(lpszGameName);
}

~CNPGameLib()
{
#ifdef NO_GAMEGUARD
return;
#endif
CloseNPGameMon();
}

public:
DWORD Init()
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (InitNPGameMon());
}

void SetHwnd(HWND hWnd)
{
#ifdef NO_GAMEGUARD
return;
#endif
SetHwndToGameMon(hWnd);
}

DWORD Check()
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (CheckNPGameMon());
}

DWORD Send(LPCTSTR lpszUserId)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendUserIDToGameMon(lpszUserId));
}

LPCSTR GetInfo()
{
#ifdef NO_GAMEGUARD
return NULL;
#endif
return (GetInfoFromGameMon());
}

DWORD Auth(DWORD dwAuth)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendCSAuthToGameMon(dwAuth));
}

DWORD Auth2(PGG_AUTH_DATA pAuth)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendCSAuth2ToGameMon(pAuth));
}

    void RunFAQ(DWORD dwErrcode)
    {
#ifdef NO_GAMEGUARD
return;
#endif
        RunFAQWebPage(dwErrcode);
    }

BOOL ResAuth(LPCTSTR lpszFilename, bool bCheckFilename = true)
{
#ifdef NO_GAMEGUARD
return TRUE;
#endif
return (ResourceAuth(lpszFilename, bCheckFilename));
}

private:
// No copies
CNPGameLib(const CNPGameLib &rhs);
CNPGameLib &operator=(const CNPGameLib &rhs);
};
```
