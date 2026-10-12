
#### 对象遍历与消息分发

`0043730A`  `|.`  `FF15 B8167900`           `call dword ptr ds:[<&mss32.ss32._AIL_release_timer_>]`       //  `mss32.AIL_release_timer_handle`

//这个循环对对象数组内的所有对象 进行遍历 操作

`00437310`  `|>`  `C705 6C732F04 02000000`    `mov dword ptr ds:[42F736C],2`

`0043731A`  `|.`  `C705 AC6A2F04 00000000`    `mov dword ptr ds:[42F6AAC],0`

`00437324`  `|>`  `56`                       `push esi`

`00437325`  `|.`  `57`                       `push edi`

`00437326`  `|.`  `BF FFFF0000`               `mov edi,0FFFF`

`0043732B`  `|.`  `BE 886C1E01`               `mov esi,Client.011E6C88`

`00437330`  `|.`  `893D 58FFB000`             `mov dword ptr ds:[B0FF58],edi`

`00437336`  `|>`  `8B06`                      `/mov eax,dword ptr ds:[esi] //指针`

`00437338`  `|.`  `85C0`                      `|test eax,eax`

`0043733A`  `|.`  `74 3A`                     `|je short Client.00437376  //如果是零  下一个`

`0043733C`  `|.`  `3978 0C`                   `|cmp dword ptr ds:[eax+C],edi // 0xFFFF`

`0043733F`  `|.`  `74 35`                     `|je short Client.00437376`

`00437341`  `|.`  `8B0D 84172001`             `|mov ecx,dword ptr ds:[1201784]  //此值为零 则不进行处理`

`00437347`  `|.`  `85C9`                      `|test ecx,ecx`

`00437349`  `|.`  `74 1C`                     `|je short Client.00437367`

`0043734B`  `|.`  `8B0D 04ED1F01`             `|mov ecx,dword ptr ds:[11FED04]`

`00437351`  `|.`  `83B9 90000000 01`          `|cmp dword ptr ds:[ecx+90],1 //此值不为 1 则不进行处理`

`00437358`  `|.`  `75 0D`                     `|jnz short Client.00437367`

`0043735A`  `|.`  `8B48 04`                   `|mov ecx,dword ptr ds:[eax+4] //`

`0043735D`  `|.`  `83F9 2D`                   `|cmp ecx,2D`

`00437360`  `|.`  `74 14`                     `|je short Client.00437376 //youxi 人物`

`00437362`  `|.`  `83F9 06`                   `|cmp ecx,6`

`00437365`  `|.`  `74 0F`                     `|je short Client.00437376 //静态咚咚？`

`00437367`  `|>`  `6A 00`                     `|push 0`                          //  `/Arg4 = 00000000`

`00437369`  `|.`  `6A 00`                     `|push 0`                          //  `|Arg3 = 00000000`

`0043736B`  `|.`  `6A 08`                     `|push 8`                          //  `|Arg2 = 00000008`

`0043736D`  `|.`  `50`                         `|push eax`                        //  `|Arg1 //指向对象的指针`

`0043736E`  `|.`  `E8 DDFCFFFF`                `|call Client.00437050`            //  `\Client.00437050`

```
{
  00437050   /$  55                      push ebp
  00437051   |.  8BEC                    mov ebp,esp
  00437053   |.  8B45 0C                 mov eax,dword ptr ss:[ebp+C]  // eax 0x08
  00437056   |.  56                      push esi
  00437057   |.  3D 16040000             cmp eax,416                   //  Switch (cases 0..44E)
  0043705C   |.  7F 77                   jg short Client.004370D5
  0043705E   |.  3D 11040000             cmp eax,411
  00437063   |.  0F8D 86000000          jge Client.004370EF
  00437069   |.  83F8 0A                cmp eax,0A
  0043706C   |.  7F 48                   jg short Client.004370B6
  0043706E   |.  74 14                   je short Client.00437084
  00437070   |.  83F8 08                cmp eax,8
  00437073   |.  77 48                   ja short Client.004370BD
  00437075   |.  33C9                    xor ecx,ecx
  00437077   |.  8A88 10714300          mov cl,byte ptr ds:[eax+437110] // cl = byte ptr [437118]  //索引
  0043707D   |.  FF248D 08714300        jmp dword ptr ds:[ecx*4+437108] //跳到 8 那里
  00437084   |>  8B75 08                 mov esi,dword ptr ss:[ebp+8]          //  Case A of switch 00437057
  00437087   |.  8B45 10                 mov eax,dword ptr ss:[ebp+10]
  0043708A   |.  57                      push edi
  0043708B   |.  8B7D 14                 mov edi,dword ptr ss:[ebp+14]
  0043708E   |.  8B16                    mov edx,dword ptr ds:[esi]
  00437090   |.  57                      push edi
  00437091   |.  50                      push eax
  00437092   |.  6A 0A                   push 0A
  00437094   |.  8BCE                    mov ecx,esi
  00437096   |.  FF52 04                 call dword ptr ds:[edx+4]
  00437099   |.  81FF FF000000           cmp edi,0FF
  0043709F   |.  5F                      pop edi
  004370A0   |.  75 5E                   jnz short Client.00437100
  004370A2   |.  85F6                    test esi,esi
  004370A4   |.  74 5A                   je short Client.00437100
  004370A6   |.  8B16                    mov edx,dword ptr ds:[esi]
  004370A8   |.  6A 01                   push 1
  004370AA   |.  8BCE                    mov ecx,esi
  004370AC   |.  FF12                    call dword ptr ds:[edx]
  004370AE   |.  B8 01000000             mov eax,1
  004370B3   |.  5E                      pop esi
  004370B4   |.  5D                      pop ebp
  004370B5   |.  C3                      retn
  004370B6   |>  3D F9030000             cmp eax,3F9
  004370BB   |.  74 32                   je short Client.004370EF
  004370BD   |>  8B4D 14                 mov ecx,dword ptr ss:[ebp+14]         //  Default case of switch 00437057
  004370C0   |.  8B55 10                 mov edx,dword ptr ss:[ebp+10]
  004370C3   |.  51                      push ecx                              // /Arg4
  004370C4   |.  52                      push edx                              // |Arg3
  004370C5   |.  50                      push eax                              // |Arg2
  004370C6   |.  8B45 08                 mov eax,dword ptr ss:[ebp+8]          // |
  004370C9   |.  50                      push eax                              // |Arg1
  004370CA   |.  E8 C1B2FFFF             call Client.00432390                  // \Client.00432390
  004370CF   |.  83C4 10                 add esp,10
  004370D2   |.  5E                      pop esi
  004370D3   |.  5D                      pop ebp
  004370D4   |.  C3                      retn
  004370D5   |>  8D88 E7FBFFFF           lea ecx,dword ptr ds:[eax-419]
  004370DB   |.  83F9 35                 cmp ecx,35
  004370DE   |.^ 77 DD                   ja short Client.004370BD
  004370E0   |.  33D2                    xor edx,edx
  004370E2   |.  8A91 24714300           mov dl,byte ptr ds:[ecx+437124]
  004370E8   |.  FF2495 1C714300         jmp dword ptr ds:[edx*4+43711C]       //  Cases 0,1,2,3,6,7,8,3F9,411,412,413,414,415,416,419,42F,430,43A,44D,44E of sw>
  //调到这里
  004370EF   |>  8B75 14                 mov esi,dword ptr ss:[ebp+14]   //  0
  004370F2   |.  8B4D 08                 mov ecx,dword ptr ss:[ebp+8]    // 指向对象的指针
  004370F5   |.  56                      push esi  //  
  004370F6   |.  8B75 10                 mov esi,dword ptr ss:[ebp+10]   // 0
  004370F9   |.  8B11                    mov edx,dword ptr ds:[ecx]      // edx 是指向这个处理对象数据的函数指针
  004370FB   |.  56                      push esi  // 0  
  004370FC   |.  50                      push eax  // 8
  004370FD   |.  FF52 04                 call dword ptr ds:[edx+4]  // 调用这个函数
  {
    ecx = 指向对象的指针
    func(8,0,0)
  }
  00437100   |>  B8 01000000             mov eax,1
  00437105   |.  5E                      pop esi
}
```
