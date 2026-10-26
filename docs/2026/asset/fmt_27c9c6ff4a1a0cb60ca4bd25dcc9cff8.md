
```asm
00420180            .  7D 40            jge short CamAudio.004201C2
00420182            .  8B40 04          mov eax,dword ptr ds:[eax+4]
00420185            .  8B5424 14        mov edx,dword ptr ss:[esp+14]
00420189            .  8A143A           mov dl,byte ptr ds:[edx+edi]
0042018C            .  03C1             add eax,ecx
0042018E            .  45               inc ebp
0042018F            .  41               inc ecx
00420190            .  3BEE             cmp ebp,esi
00420192            .  8810             mov byte ptr ds:[eax],dl
00420194            .^ 7C CA            jl short CamAudio.00420160
00420196            >  8B4424 14        mov eax,dword ptr ss:[esp+14]
0042019A            .  85C0             test eax,eax
0042019C            .  C74424 2C FFFFFF>mov dword ptr ss:[esp+2C],-1
004201A4            .  74 09            je short CamAudio.004201AF
004201A6            .  50               push eax                             ; /block
004201A7            .  E8 E4140600      call <jmp.&MFC71.#266>               ; \free
004201AC            .  83C4 04          add esp,4
004201AF            >  8B4C24 24        mov ecx,dword ptr ss:[esp+24]
004201B3            .  5F               pop edi
004201B4            .  5E               pop esi
004201B5            .  5D               pop ebp
004201B6            .  5B               pop ebx
004201B7            .  64:890D 00000000 mov dword ptr fs:[0],ecx
004201BE            .  83C4 20          add esp,20
```

```c
void move(char *tem,int le,int f)//tem为二进制字符串,le移位总数,f为移位量
{
        int i,j;
        int tem1[100];
        if(le==-1)
                le=0x5a;
        for(i=0;i<100;i++)
                tem1[i]=tem[i];
        for(i=0;i<(le-f);i++)
                tem[i]=tem[i+f];
        for(i=(le-f),j=0;i<le;i++,j++)
                tem[i]=tem1[j];
}
```

------------------->这下面的功能是将二进制转为注册码

```asm
004201D0            $  6A FF            push -1
004201D2            .  68 29524800      push CamAudio.00485229               ;  SE handler installation
004201D7            .  64:A1 00000000   mov eax,dword ptr fs:[0]
004201DD            .  50               push eax
004201DE            .  64:8925 00000000 mov dword ptr fs:[0],esp
004201E5            .  83EC 08          sub esp,8
004201E8            .  53               push ebx
004201E9            .  55               push ebp
004201EA            .  56               push esi
004201EB            .  33F6             xor esi,esi
004201ED            .  57               push edi
004201EE            .  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
004201F2            .  897424 14        mov dword ptr ss:[esp+14],esi
004201F6            .  FF15 00D94800    call dword ptr ds:[<&MFC71.#310>]    ;  MFC71.7C173199
004201FC            .  8B5C24 2C        mov ebx,dword ptr ss:[esp+2C]
00420200            .  8B4B 08          mov ecx,dword ptr ds:[ebx+8]
00420203            .  B8 67666666      mov eax,66666667
00420208            .  F7E9             imul ecx
0042020A            .  D1FA             sar edx,1
0042020C            .  8BEA             mov ebp,edx
0042020E            .  C1ED 1F          shr ebp,1F
00420211            .  03EA             add ebp,edx
00420213            .  3BEE             cmp ebp,esi
00420215            .  897424 20        mov dword ptr ss:[esp+20],esi
00420219            .  897424 14        mov dword ptr ss:[esp+14],esi
0042021D            .  7E 59            jle short CamAudio.00420278
0042021F            .  90               nop                                  ;  这里的功能是把二进制转为密码序列
00420220            >  8D56 04          lea edx,dword ptr ds:[esi+4]
00420223            .  33FF             xor edi,edi                          ;  取决于edi
00420225            .  33C9             xor ecx,ecx
00420227            .  3BF2             cmp esi,edx
00420229            .  8BC6             mov eax,esi
0042022B            .  7F 28            jg short CamAudio.00420255
0042022D            .  8D49 00          lea ecx,dword ptr ds:[ecx]
00420230            >  85C0             test eax,eax
00420232            .  7C 74            jl short CamAudio.004202A8
00420234            .  3B43 08          cmp eax,dword ptr ds:[ebx+8]
00420237            .  7D 6F            jge short CamAudio.004202A8
00420239            .  8B5B 04          mov ebx,dword ptr ds:[ebx+4]
0042023C            .  803C18 00        cmp byte ptr ds:[eax+ebx],0          ;  02A10DF0
00420240            .  74 09            je short CamAudio.0042024B
00420242            .  BB 01000000      mov ebx,1
00420247            .  D3E3             shl ebx,cl                           ;  cl=3
00420249            .  0BFB             or edi,ebx
0042024B            >  8B5C24 2C        mov ebx,dword ptr ss:[esp+2C]
0042024F            .  41               inc ecx
00420250            .  40               inc eax
00420251            .  3BC2             cmp eax,edx
00420253            .^ 7E DB            jle short CamAudio.00420230
00420255            >  33C0             xor eax,eax
00420257            .  8A87 88284900    mov al,byte ptr ds:[edi+492888]
0042025D            .  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
00420261            .  50               push eax
00420262            .  FF15 74D54800    call dword ptr ds:[<&MFC71.#908>]    ;  MFC71.7C18B24E
00420268            .  8B4424 14        mov eax,dword ptr ss:[esp+14]
0042026C            .  40               inc eax
0042026D            .  83C6 05          add esi,5
00420270            .  3BC5             cmp eax,ebp
00420272            .  894424 14        mov dword ptr ss:[esp+14],eax
00420276            .^ 7C A8            jl short CamAudio.00420220
00420278            >  8B7424 28        mov esi,dword ptr ss:[esp+28]
0042027C            .  8D4C24 10        lea ecx,dword ptr ss:[esp+10]        ;  这里重要
00420280            .  51               push ecx
00420281            .  8BCE             mov ecx,esi
00420283            .  FF15 F0D84800    call dword ptr ds:[<&MFC71.#297>]    ;  MFC71.7C14E575
00420289            .  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
0042028D            .  FF15 10D94800    call dword ptr ds:[<&MFC71.#578>]    ;  MFC71.7C1771B1
00420293            .  8B4C24 18        mov ecx,dword ptr ss:[esp+18]
00420297            .  5F               pop edi
00420298            .  8BC6             mov eax,esi
0042029A            .  5E               pop esi
0042029B            .  5D               pop ebp
0042029C            .  5B               pop ebx
0042029D            .  64:890D 00000000 mov dword ptr fs:[0],ecx
004202A4            .  83C4 14          add esp,14
004202A7            .  C3               retn
```

```c
void back(char *be,char *kt,int length)//be为存放二进制的数组,kt为保存注册码的数组，length为注册码的长度
{
        char *m="CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8";
        int t,i,j;
        for(i=0;i<length;i++)
        {
                t=0;
                for(j=0;j<5;j++)
                {
                        if(be[i*5+j]==1)
                                t+=(1<<j);
                }
                kt[i]=m[t];
        }
}
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*下面为部分 keygen 的代码，完整代码请参见附件。

```c
#include "mode.h"

#pragma comment(linker, "/ENTRY:EntryPoint")

HINSTANCE    hi;

void EntryPoint()
{
     ExitProcess(WinMain(GetModuleHandle(NULL), NULL, NULL, SW_SHOWNORMAL));
}

int WINAPI WinMain(HINSTANCE hInstance,
                        HINSTANCE hPrevInstance,
                        LPSTR lpCmdLine,
                        int nCmdShow)
{
        hi=hInstance;
        DialogBoxParam(hInstance,MAKEINTRESOURCE(IDD_DIALOG),NULL,(DLGPROC)MyBox,NULL);
        ExitProcess(0);
        return 0;
}

LRESULT CALLBACK MyBox(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
        char sn[26]="8F8A9CACCBACCA724C6058687";
        switch(message)
        {
        case WM_INITDIALOG:
```
