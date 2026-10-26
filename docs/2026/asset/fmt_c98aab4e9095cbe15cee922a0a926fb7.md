
```text
00420345            >  8BF9             mov edi,ecx                          ;  将序号转为2进制
00420347            >  8B4424 28        mov eax,dword ptr ss:[esp+28]
0042034B            .  33C9             xor ecx,ecx
0042034D            .  8D49 00          lea ecx,dword ptr ds:[ecx]
00420350            >  85C0             test eax,eax
00420352            .  7C 55            jl short CamAudio.004203A9
00420354            .  3B46 08          cmp eax,dword ptr ds:[esi+8]
00420357            .  7D 50            jge short CamAudio.004203A9
00420359            .  8B5E 04          mov ebx,dword ptr ds:[esi+4]
0042035C            .  BA 01000000      mov edx,1
00420361            .  D3E2             shl edx,cl
00420363            .  85D7             test edi,edx                         ;  如果条件为真
00420365            .  0F95C2           setne dl                             ;  则这里为真，d1就等于1否则dl=0
00420368            .  41               inc ecx
00420369            .  881418           mov byte ptr ds:[eax+ebx],dl         ;  这里是保存2进制数
0042036C            .  40               inc eax
0042036D            .  83F9 05          cmp ecx,5
00420370            .^ 7C DE            jl short CamAudio.00420350
00420372            .  8B5424 28        mov edx,dword ptr ss:[esp+28]
00420376            .  83C2 05          add edx,5
00420379            .  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
0042037D            .  45               inc ebp
0042037E            .  895424 28        mov dword ptr ss:[esp+28],edx
00420382            .  FF15 FCD84800    call dword ptr ds:[<&MFC71.#2902>]   ;  MFC71.7C146AB0
00420388            .  3BE8             cmp ebp,eax
0042038A            .^ 7C 99            jl short CamAudio.00420325           ;  起初这里是95位，第二次注册码又去掉了一位移位码，所以这里只有90位了
0042038C            >  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
00420390            .  FF15 10D94800    call dword ptr ds:[<&MFC71.#578>]    ;  MFC71.7C1771B1
00420396            .  8B4C24 14        mov ecx,dword ptr ss:[esp+14]
0042039A            .  5F               pop edi
0042039B            .  5E               pop esi
0042039C            .  5D               pop ebp
0042039D            .  5B               pop ebx
0042039E            .  64:890D 00000000 mov dword ptr fs:[0],ecx
004203A5            .  83C4 10          add esp,10
```

```c
void change(char *get,char *tem)//get 为注册码,tem为存放二进制的数组
{
        char *m="CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8";
        int length,i,j,le=0;
        length=strlen(get);
        for(i=0 ;i<length; i++)
        {
                for(j=0;j<32;j++)
                {
                        if(get[i]==m[j])
                        {
                                le=j;
                                break;
                        }
                        else
                                le=0xff;
                }
                for(j=0;j<5;j++)
                {
                        if((1&(le>>j))==1)
                        {
                                tem[i*5+j]=1;
                        }
                }
        }
}
```

----------->这下面是移位

```text
004200A0            $  64:A1 00000000   mov eax,dword ptr fs:[0]
004200A6            .  6A FF            push -1
004200A8            .  68 08524800      push CamAudio.00485208
004200AD            .  50               push eax
004200AE            .  64:8925 00000000 mov dword ptr fs:[0],esp
004200B5            .  83EC 14          sub esp,14
004200B8            .  53               push ebx
004200B9            .  55               push ebp
004200BA            .  8B6C24 38        mov ebp,dword ptr ss:[esp+38]
004200BE            .  83FD FF          cmp ebp,-1                           ;  如果这里为-1则移位总数为5a=90
004200C1            .  56               push esi
004200C2            .  57               push edi
004200C3            .  8B7C24 34        mov edi,dword ptr ss:[esp+34]
004200C7            .  75 04            jnz short CamAudio.004200CD
004200C9            .  8B6F 08          mov ebp,dword ptr ds:[edi+8]
004200CC            .  4D               dec ebp
004200CD            >  8B5C24 3C        mov ebx,dword ptr ss:[esp+3C]
004200D1            .  33C0             xor eax,eax
004200D3            .  8BF5             mov esi,ebp
004200D5            .  2BF3             sub esi,ebx
004200D7            .  46               inc esi
004200D8            .  C74424 10 A82849>mov dword ptr ss:[esp+10],CamAudio.0>
004200E0            .  894424 14        mov dword ptr ss:[esp+14],eax
004200E4            .  894424 20        mov dword ptr ss:[esp+20],eax
004200E8            .  894424 1C        mov dword ptr ss:[esp+1C],eax
004200EC            .  894424 18        mov dword ptr ss:[esp+18],eax
004200F0            .  6A FF            push -1
004200F2            .  56               push esi
004200F3            .  8D4C24 18        lea ecx,dword ptr ss:[esp+18]
004200F7            .  894424 34        mov dword ptr ss:[esp+34],eax
004200FB            .  E8 00FEFFFF      call CamAudio.0041FF00
00420100            .  33C9             xor ecx,ecx
00420102            .  3BDD             cmp ebx,ebp
00420104            .  8BC3             mov eax,ebx
00420106            .  7F 3F            jg short CamAudio.00420147
00420108            .  EB 06            jmp short CamAudio.00420110
0042010A            >  8B7C24 34        mov edi,dword ptr ss:[esp+34]
0042010E            .  8BFF             mov edi,edi
00420110            >  85C0             test eax,eax
00420112            .  0F8C AA000000    jl CamAudio.004201C2
00420118            .  3B47 08          cmp eax,dword ptr ds:[edi+8]
0042011B            .  0F8D A1000000    jge CamAudio.004201C2
00420121            .  8B7F 04          mov edi,dword ptr ds:[edi+4]
00420124            .  03F8             add edi,eax
00420126            .  85C9             test ecx,ecx
00420128            .  0F8C 94000000    jl CamAudio.004201C2
0042012E            .  3B4C24 18        cmp ecx,dword ptr ss:[esp+18]
00420132            .  0F8D 8A000000    jge CamAudio.004201C2
00420138            .  8A17             mov dl,byte ptr ds:[edi]
0042013A            .  8B7C24 14        mov edi,dword ptr ss:[esp+14]
0042013E            .  40               inc eax
0042013F            .  88140F           mov byte ptr ds:[edi+ecx],dl
00420142            .  41               inc ecx
00420143            .  3BC5             cmp eax,ebp
00420145            .^ 7E C3            jle short CamAudio.0042010A
00420147            >  8B4424 38        mov eax,dword ptr ss:[esp+38]
0042014B            .  99               cdq
0042014C            .  F7FE             idiv esi
0042014E            .  33ED             xor ebp,ebp
00420150            .  85F6             test esi,esi
00420152            .  8BCB             mov ecx,ebx
00420154            .  7E 40            jle short CamAudio.00420196
00420156            .  8BC2             mov eax,edx
00420158            .  2BC3             sub eax,ebx
0042015A            .  894424 40        mov dword ptr ss:[esp+40],eax
0042015E            .  EB 04            jmp short CamAudio.00420164
00420160            >  8B4424 40        mov eax,dword ptr ss:[esp+40]
00420164            >  03C1             add eax,ecx
00420166            .  99               cdq
00420167            .  F7FE             idiv esi
00420169            .  8BFA             mov edi,edx
0042016B            .  85FF             test edi,edi
0042016D            .  7C 53            jl short CamAudio.004201C2
0042016F            .  3B7C24 18        cmp edi,dword ptr ss:[esp+18]
00420173            .  7D 4D            jge short CamAudio.004201C2
00420175            .  85C9             test ecx,ecx
00420177            .  7C 49            jl short CamAudio.004201C2
00420179            .  8B4424 34        mov eax,dword ptr ss:[esp+34]
0042017D            .  3B48 08          cmp ecx,dword ptr ds:[eax+8]
```
