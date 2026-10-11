
### 汇编代码片段
```
00406368  |.  8B4C24 08      mov ecx,dword ptr ss:[esp+8]  
0040636C  |.  64:890D 00000>mov dword ptr fs:[0],ecx  
00406373  |.  83C4 14        add esp,14  
00406376  |.  C2 0800        retn 8  
```

注册码的算出没有仔细跟，粗略看了一下，在以下三处（省略部分代码）：

### 第一处
```
00406255  |.  E8 B6720000   call 儿童快乐.0040D510               ; 获取 C 盘信息以便计算注册码  
```
**F7 跟进：**
```
0040D52F  |.  6A 0A          push 0A              ; /pFileSystemNameSize = 0000000A  
0040D531  |.  50            push eax             ; |pFileSystemNameBuffer  
0040D532  |.  8D4424 10     lea eax,dword ptr ss:[esp+10]       ; |  
0040D536  |.  50            push eax             ; |pFileSystemFlags  
0040D537  |.  51            push ecx             ; |pMaxFilenameLength  
0040D538  |.  52            push edx             ; |pVolumeSerialNumber  
0040D539  |.  6A 0C         push 0C              ; |MaxVolumeNameSize = C (12.)  
0040D53B  |.  56            push esi             ; |VolumeNameBuffer  
0040D53C  |.  68 F4384100    push 儿童快乐.004138F4             ; |c://  
0040D541  |.  FF15 60004100  call dword ptr ds:[<&KERNEL32.GetVolumeI>; \GetVolumeInformationA 获取 C 盘信息以便计算注册码  
```

### 第二处
```
00406261  |.  E8 2A650000   call 儿童快乐.0040C790               ; 第一次计算注册码  
```
**F7 跟进：**
```
0040C7E3  |> /8D3C00          /lea edi,dword ptr ds:[eax+eax]  
0040C7E6  |. |83FF 18        |cmp edi,18  
0040C7E9  |. |7E 05          |jle short 儿童快乐.0040C7F0  
0040C7EB  |. |BF 18000000    |mov edi,18  
0040C7F0  |> |B3 31          |mov bl,31                      ; 数字 1  
0040C7F2  |. |BE 01000000    |mov esi,1  
0040C7F7  |. |885C24 1C      |mov byte ptr ss:[esp+1C],bl  
0040C7FB  |> |8B5424 1C      |/mov edx,dword ptr ss:[esp+1C]  
0040C7FF  |. |8D4C24 14      ||lea ecx,dword ptr ss:[esp+14]  
0040C803  |. |52            ||push edx  
0040C804  |. |56            ||push esi  
0040C805  |. |E8 2C140000    ||call <jmp.&MFC42.#CString::Insert_6778>      ; 每隔两位插入递加数字 1、2、3、4、5……  
0040C80A  |. |FEC3          ||inc bl                           ; 递加  
0040C80C  |. |80FB 3A      ||cmp bl,3A  
0040C80F  |. |885C24 1C    ||mov byte ptr ss:[esp+1C],bl  
0040C813  |. |75 06        ||jnz short 儿童快乐.0040C81B  
0040C815  |. |B3 41        ||mov bl,41  
0040C817  |. |885C24 1C    ||mov byte ptr ss:[esp+1C],bl  
0040C81B  |> |8B4424 14    ||mov eax,dword ptr ss:[esp+14]  
0040C81F  |. |83C6 02      ||add esi,2  
0040C822  |. |8B48 F8      ||mov ecx,dword ptr ds:[eax-8]  
0040C825  |. |3BCF        ||cmp ecx,edi  
0040C827  |.^|7C D2        |\jl short 儿童快乐.0040C7FB  
0040C829  |. |8BC1        |mov eax,ecx  
0040C82B  |. |83F8 18     |cmp eax,18  
0040C82E  |.^\7C B3       \jl short 儿童快乐.0040C7E3  
0040C830  |.  5F           pop edi                       ; 经过以上计算，补足 18 位（即 24 位）字符  
0040C831  |>  68 18394100  push 儿童快乐.00413918  
0040C836  |.  8D4C24 10     lea ecx,dword ptr ss:[esp+10]  
0040C83A  |.  E8 9D130000   call <jmp.&MFC42.#CString::CString_537>  
0040C83F  |.  C64424 28 02  mov byte ptr ss:[esp+28],2  
0040C844  |.  33F6          xor esi,esi  
0040C846  |.  BB 04000000   mov ebx,4  
0040C84B  |>  6A 06         push 6  
0040C84D  |.  8D4424 1C     lea eax,dword ptr ss:[esp+1C]  
0040C851  |.  56            push esi  
0040C852  |.  50            push eax  
0040C853  |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  
0040C857  |.  E8 D4130000   call <jmp.&MFC42.#CString::Mid_4278>         ; 每 6 个一组取出计算注册码  
0040C85C  |.  50            push eax  
0040C85D  |.  8D4C24 10     lea ecx,dword ptr ss:[esp+10]  
0040C861  |.  C64424 2C 03  mov byte ptr ss:[esp+2C],3  
0040C866  |.  E8 59130000   call <jmp.&MFC42.#CString::operator_858>  
0040C86B  |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]  
0040C86F  |.  C64424 28 02  mov byte ptr ss:[esp+28],2  
0040C874  |.  E8 03130000   call <jmp.&MFC42.#CString::~CString_800>  
0040C879  |.  6A 06         push 6  
0040C87B  |.  6A 06         push 6  
0040C87D  |.  8D4C24 14     lea ecx,dword ptr ss:[esp+14]  
0040C881  |.  E8 A4130000   call <jmp.&MFC42.#CString::GetBuffer_2915>  
0040C886  |.  50            push eax  
0040C887  |.  8BCD          mov ecx,ebp  
0040C889  |.  E8 52FEFFFF   call 儿童快乐.0040C6E0                   ; 计算注册码，转化为字母。跟进，见下  
0040C88E  |.  6A FF        push -1  
0040C890  |.  8D4C24 10     lea ecx,dword ptr ss:[esp+10]  
0040C894  |.  E8 8B130000   call <jmp.&MFC42.#CString::ReleaseBuffer_557>  
0040C899  |.  6A 01         push 1  
0040C89B  |.  53            push ebx  
0040C89C  |.  8D4C24 14     lea ecx,dword ptr ss:[esp+14]  
0040C8A0  |.  E8 31130000   call <jmp.&MFC42.#CString::Delete_6648>  
0040C8A5  |.  8D4C24 0C     lea ecx,dword ptr ss:[esp+C]  
0040C8A9  |.  8D5424 14     lea edx,dword ptr ss:[esp+14]  
0040C8AD  |.  51            push ecx  
0040C8AE  |.  8D4424 20     lea eax,dword ptr ss:[esp+20]  
0040C8B2  |.  52            push edx  
0040C8B3  |.  50            push eax  
0040C8B4  |.  E8 65130000   call <jmp.&MFC42.#operator+_922>  
0040C8B9  |.  50            push eax  
0040C8BA  |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]  
0040C8BE  |.  885C24 2C     mov byte ptr ss:[esp+2C],bl  
0040C8C2  |.  E8 FD120000   call <jmp.&MFC42.#CString::operator_858>  
0040C8C7  |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  
0040C8CB  |.  C64424 28 02  mov byte ptr ss:[esp+28],2  
0040C8D0  |.  E8 A7120000   call <jmp.&MFC42.#CString::~CString_800>  
0040C8D5  |.  83C6 06       add esi,6  
0040C8D8  |.  83FE 18       cmp esi,18  
0040C8DB  |.^. 0F8C 6AFFFFFF   jl 儿童快乐.0040C84B                 ; 循环计算 5 次，得到 5 组字母，再经第二次计算即得注册码  
```

**`0040C889` 处跟进如下：**
```
0040C6F9  |> /83F8 07          /cmp eax,7  
0040C6FC  |. |7E 02          |jle short 儿童快乐.0040C700  
0040C6FE  |. |33C0          |xor eax,eax  
0040C700  |> |33D2          |xor edx,edx  
0040C702  |. |33DB          |xor ebx,ebx  
0040C704  |. |8A1431        |mov dl,byte ptr ds:[ecx+esi]  
0040C707  |. |8A98 A4304100  |mov bl,byte ptr ds:[eax+4130A4]  
0040C70D  |. |0FAFD3        |imul edx,ebx  
0040C710  |. |03FA          |add edi,edx  
0040C712  |. |41            |inc ecx  
0040C713  |. |40            |inc eax  
0040C714  |. |3BCD          |cmp ecx,ebp  
0040C716  |.^\7C E1        \jl short 儿童快乐.0040C6F9  
0040C718  |>  57             push edi                       ; /<%ld>  
0040C719  |.  68 7C304100   push 儿童快乐.0041307C                 ; |%ld  
0040C71E  |.  56            push esi                       ; |s  
0040C71F  |.  FF15 6C034100 call dword ptr ds:[<&USER32.wsprintfA>]    ; \wsprintfA  
0040C725  |.  8A4E 02       mov cl,byte ptr ds:[esi+2]                   ;   
////经过以上计算转化为四位数字  
0040C728  |.  8A46 03       mov al,byte ptr ds:[esi+3]  
0040C72B  |.  83C4 0C       add esp,0C  
0040C72E  |.  3AC8          cmp cl,al  
0040C730  |.  7D 06         jge short 儿童快乐.0040C738  
0040C732  |.  8AD0          mov dl,al  
0040C734  |.  2AD1          sub dl,cl  
0040C736  |.  EB 04         jmp short 儿童快乐.0040C73C  
0040C738  |>  8AD1          mov dl,cl  
0040C73A  |.  2AD0          sub dl,al  
```
