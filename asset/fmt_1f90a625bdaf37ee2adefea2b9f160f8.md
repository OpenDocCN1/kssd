
```asm
:00429372 8BEC                   mov ebp, esp
:00429374 837D1C00               cmp dword ptr [ebp+1C], 00000000 这里会有两次循环
:00429378 753F                   jne 004293B9
:0042937A 837D0C05               cmp dword ptr [ebp+0C], 00000005 只要改动下面的跳转
:0042937E 7425                   jne 004293A5 －－>> 广告条就可以消失
```

在我看来这是最简单的去除办法，当然我们还可以修改`cd_clint.dll`这个文件。通过修改它，可以达到修改`regetjr.exe`相同的效果

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042E081(C)
|

```asm
:0042E0D0 8B4650                 mov eax, dword ptr [esi+50]
:0042E0D3 8B3D38034600           mov edi, dword ptr [00460338]
:0042E0D9 33DB                   xor ebx, ebx
:0042E0DB 3BC3                   cmp eax, ebx
:0042E0DD 0F8485000000           je 0042E168
:0042E0E3 399EC0000000           cmp dword ptr [esi+000000C0], ebx
:0042E0E9 747D                   je 0042E168
:0042E0EB 8B08                   mov ecx, dword ptr [eax]
:0042E0ED 8D55F0                 lea edx, dword ptr [ebp-10]
:0042E0F0 52                     push edx
:0042E0F1 FF7660                 push [esi+60]
:0042E0F4 53                     push ebx
:0042E0F5 50                     push eax
:0042E0F6 FF91DC000000           call dword ptr [ecx+000000DC]
:0042E0FC 8B4650                 mov eax, dword ptr [esi+50]
:0042E0FF 8D55F4                 lea edx, dword ptr [ebp-0C]
:0042E102 52                     push edx
:0042E103 FF7660                 push [esi+60]
:0042E106 8B08                   mov ecx, dword ptr [eax]
:0042E108 53                     push ebx
:0042E109 50                     push eax
:0042E10A FF91E0000000           call dword ptr [ecx+000000E0]
:0042E110 8B4650                 mov eax, dword ptr [esi+50]
:0042E113 8D55F8                 lea edx, dword ptr [ebp-08]
:0042E116 52                     push edx
:0042E117 50                     push eax
:0042E118 8B08                   mov ecx, dword ptr [eax]
:0042E11A FF91C0000000           call dword ptr [ecx+000000C0]
:0042E120 8B4650                 mov eax, dword ptr [esi+50]
:0042E123 8D55FC                 lea edx, dword ptr [ebp-04]
:0042E126 52                     push edx
:0042E127 50                     push eax
:0042E128 8B08                   mov ecx, dword ptr [eax]
:0042E12A FF91C8000000           call dword ptr [ecx+000000C8]
:0042E130 6A01                   push 00000001
:0042E132 FF75FC                 push [ebp-04]
:0042E135 FF75F8                 push [ebp-08]
:0042E138 FF75F4                 push [ebp-0C]
:0042E13B FF75F0                 push [ebp-10]
:0042E13E FFB6C0000000           push dword ptr [esi+000000C0]===>高度起始位置设置值(“C0”修改为“00”后任务条框将上移到广告框的地方)
:0042E144 FFD7                   call edi ======>下面任务条设置的 CALL MOVEWINDOW
:0042E146 395D0C                 cmp dword ptr [ebp+0C], ebx 这里还要修改成 0C－》00
:0042E149 751D                   jne 0042E168 或者改动跳转，如果只修改一处，就会很不好看
                                 可以试着最大化一下：）
```

——————————————————————————————————————

```asm
:0042E1C7 FF75F0                 push [ebp-10]
:0042E1CA FFB6B8110000           push dword ptr [esi+000011B8]===>>重点
:0042E1D0 FFD7                   call edi ======>加入广告的调用 CALL MOVEWINDOW
```

==================================================

```asm
:00429E63 FF91C8000000           call dword ptr [ecx+000000C8]
:00429E69 6A01                   push 00000001
:00429E6B FF7510                 push [ebp+10]
:00429E6E FF75FC                 push [ebp-04]
:00429E71 FF75F8                 push [ebp-08]
:00429E74 FF75F4                 push [ebp-0C]
:00429E77 FFB6B8110000           push dword ptr [esi+000011B8]===>>重点
:00429E7D FFD7                   call edi     ===>换 SKIN 时的广告加入 CALL MOVEWINDOW
```

启动时没有广告了，但是换皮肤的时候又会出现广告框。谁解决阿？ `4dd466d0e1e0cc123afbe504da2462ca`
