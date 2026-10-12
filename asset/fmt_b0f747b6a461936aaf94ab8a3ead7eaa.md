
```
:0045E6A3 A1649A4600                mov eax, dword ptr [00469A64]   
:0045E6A8 8B18                      mov ebx, dword ptr [eax]   
:0045E6AA FF13                      call dword ptr [ebx]            <==== 从 ini 文件中取得注册码   
:0045E6AC 8B95CCFDFFFF             mov edx, dword ptr [ebp+FFFFFDCC]    \   
:0045E6B2 8D85D0FDFFFF             lea eax, dword ptr [ebp+FFFFFDD0]    |   
:0045E6B8 B9FF000000               mov ecx, 000000FF            |   
:0045E6BD E85654FAFF               call 00403B18                \\ 将注册名信息存放到 ebp-c9 到 ebp-66 处，ebp-ca 是注册名长度   
:0045E6C2 8D95D0FDFFFF             lea edx, dword ptr [ebp+FFFFFDD0]    /   
:0045E6C8 8D8536FFFFFF             lea eax, dword ptr [ebp+FFFFFF36]    |   
:0045E6CE B164                     mov cl, 64             |   
:0045E6D0 E8AB41FAFF               call 00402880               
:0045E6D5 8A459B                   mov al, byte ptr [ebp-65]   
:0045E6D8 84C0                     test al, al            <==== 检查注册名长度是否为 0   
:0045E6DA 7409                     je 0045E6E5   
:0045E6DC 80BD36FFFFFF06           cmp byte ptr [ebp+FFFFFF36], 06    <==== 检查注册码长度是否为 6   
:0045E6E3 740C                     je 0045E6F1   
```

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E6DA(C)`

`:0045E6E5 C605609A460000             mov byte ptr [00469A60], 00      <==== 错误则置标志为 0`

`:0045E6EC E996000000               jmp 0045E787            <==== 跳到下面，返回`

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E6E3(C)`

`:0045E6F1 8BF0                     mov esi, eax`

`:0045E6F3 81E6FF000000             and esi, 000000FF`

`:0045E6F9 85F6                     test esi, esi`

`:0045E6FB 7C21                     jl 0045E71E`

`:0045E6FD 46                       inc esi            <==== esi 为注册名长度加 1`

`:0045E6FE 8D4D9B                   lea ecx, dword ptr [ebp-65]      <==== ecx 指向 注册名长度字节+注册名`

`:0045E701 8D9DD1FEFFFF             lea ebx, dword ptr [ebp+FFFFFED1]   <==== ebx 指向欲生成的注册码地址`

`Referenced by a (U)nconditional or (C)onditional Jump at Address:     <==== 下面这个循环生成注册码，设为 注册码 A`

`:0045E71C(C)`

`:0045E707 33C0                     xor eax, eax`

`:0045E709 8A01                     mov al, byte ptr [ecx]`

`:0045E70B BF0A000000               mov edi, 0000000A      <==== 除数为 a，即为 10（十进制）`

`:0045E710 33D2                     xor edx, edx`

`:0045E712 F7F7                     div edi            <==== 相除后，edx 中为余数`

`:0045E714 80C230                   add dl, 30              <==== 变为 ASCII 码`

`:0045E717 8813                     mov byte ptr [ebx], dl      <==== 按先后顺序存放于 [ebx]`

`:0045E719 43                       inc ebx`

`:0045E71A 41                       inc ecx`

`:0045E71B 4E                       dec esi`

`:0045E71C 75E9                     jne 0045E707            <==== 未完则继续`

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E6FB(C)`

`:0045E71E 8A459B                   mov al, byte ptr [ebp-65]`

`:0045E721 3C06                     cmp al, 06            <==== 注册名长度超过 6 字节则不需要再处理`

`:0045E723 732F                     jnb 0045E754`

`:0045E725 33C9                     xor ecx, ecx`

`:0045E727 8AC8                     mov cl, al`

`:0045E729 41                       inc ecx`

`:0045E72A 83F906                   cmp ecx, 00000006       <==== 多此一举！！`

`:0045E72D 7F25                     jg 0045E754`

`:0045E72F 8D9C0DD0FEFFFF             lea ebx, dword ptr [ebp+ecx-00000130]   <==== 从 注册码 A 的最后一个数开始补足 7 位`

`Referenced by a (U)nconditional or (C)onditional Jump at Address:     <==== 下面这个循环将生成的注册码 A 补足为 7 位`

`:0045E752(C)`

`:0045E736 33C0                     xor eax, eax`

`:0045E738 8A03                     mov al, byte ptr [ebx]      <==== 取得[ebx]处的数字`

`:0045E73A 40                       inc eax            <==== 加一`

`:0045E73B 83E830                   sub eax, 00000030`

`:0045E73E BE0A000000               mov esi, 0000000A`

`:0045E743 33D2                     xor edx, edx`

`:0045E745 F7F6                     div esi            <==== 相除，取模到 edx`

`:0045E747 80C230                   add dl, 30              <==== 变为 ASCII 码`

`:0045E74A 885301                   mov byte ptr [ebx+01], dl      <==== 存放到[ebx+1]处`

`:0045E74D 41                       inc ecx`

`:0045E74E 43                       inc ebx`

`:0045E74F 83F907                   cmp ecx, 00000007       <==== 不足 7 位则继续`

`:0045E752 75E2                     jne 0045E736`

可以分析出，注册码支持所有可打印字符和汉字；将 注册码 A 补足为 7 位的规律是：将最后一位数加一作为下一位数，如果 9 加一则变为 0，依次处理。例如：

注册码 A=285，则补足为 2856789； 注册码 A=3619，则补足为 3619012。

`Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

`:0045E723(C), :0045E72D(C)`

`:0045E754 C685D8FEFFFF00             mov byte ptr [ebp+FFFFFED8], 00   <==== 很粗暴的在第八个字节处置 0，之后的数字被舍弃`

`:0045E75B C605609A460001             mov byte ptr [00469A60], 01      <==== 预置注册标志为 1（成功）`

`:0045E762 B906000000               mov ecx, 00000006      <==== 比较的字节数`

`:0045E767 8D85D2FEFFFF             lea eax, dword ptr [ebp+FFFFFED2]   <==== 算出的注册码，注意第一位被舍弃`

`:0045E76D 8D9537FFFFFF             lea edx, dword ptr [ebp+FFFFFF37]   <==== 你输入的注册码`

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E785(C)`

```
:0045E773 8A18                     mov bl, byte ptr [eax]      
:0045E775 3A1A                     cmp bl, byte ptr [edx]
:0045E777 7409                     je 0045E782
:0045E779 C605609A460000             mov byte ptr [00469A60], 00      <==== 比较失败则置标志为 0
:0045E780 EB05                     jmp 0045E787
```

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E777(C)`

```
:0045E782 42                       inc edx
:0045E783 40                       inc eax
:0045E784 49                       dec ecx
:0045E785 75EC                     jne 0045E773            <==== 不到 6 位则继续
```

`Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

`:0045E6EC(U), :0045E780(U)`

```
:0045E787 33C0                     xor eax, eax
:0045E789 5A                       pop edx
:0045E78A 59                       pop ecx
:0045E78B 59                       pop ecx
:0045E78C 648910                   mov dword ptr fs:[eax], edx
:0045E78F 68A7E74500               push 0045E7A7
```

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0045E7A5(U)`

```
:0045E794 8D85CCFDFFFF             lea eax, dword ptr [ebp+FFFFFDCC]
:0045E79A E82151FAFF               call 004038C0
:0045E79F C3                       ret
:0045E7A0 E9574BFAFF               jmp 004032FC
:0045E7A5 EBED                     jmp 0045E794
:0045E7A7 5F                       pop edi
:0045E7A8 5E                       pop esi
:0045E7A9 5B                       pop ebx
:0045E7AA 8BE5                     mov esp, ebp
:0045E7AC 5D                       pop ebp
:0045E7AD C3                       ret                               <==== 返回
```

上面的判断过程用 C 表示：

```
int Success;

Registration()
{
    int LenName, LenCode;
    char UserName[100], RegCode[100], InputRegCode[6];
    int i;
    GetInfo( &UserName, LenName);      /*  取得注册信息 */
    GetInfo( &InputRegCode, LenCode);    /*       */
    if ( LenName==0 || LenCode!=6 )
    {
        Success=0;
        return;
    }
    RegCode[0]= LenName%10;
    for( i=0; i<LenName; i++)
        RegCode[i+1]= UserName[i]%10;
    if ( LenName<6 )
        for( i=0; i<6-LenName; i++)
            RegCode[i+LenName+1]= (RegCode[i+lenName]+1)%10;
    RegCode[7]='\0';
    Success=1;
    if (!strcmp( &RegCode, &InputRegCode))
        Success=0;
    return;
}
```
