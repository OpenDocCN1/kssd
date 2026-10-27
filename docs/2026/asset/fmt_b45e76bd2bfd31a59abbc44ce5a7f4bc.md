
## 十四、清扫战场

```
00463DFD    F785 48A04200 0>test dword ptr ss:[ebp+42A048],8
00463E07    74 08           je short 00463E11
00463E09    8D85 039C4200   lea eax,dword ptr ss:[ebp+429C03]
00463E0F    50              push eax; yP.00463E1D
00463E10    C3              retn
00463E1D    F785 A4A34200 8>test dword ptr ss:[ebp+42A3A4],80
00463E27    75 53           jnz short 00463E7C
00463E29    BB 5F724200     mov ebx,42725F
00463E2E    FF541D 00       call dword ptr ss:[ebp+ebx]; kernel32.IsDebuggerPresent
00463E32    0BC0            or eax,eax
00463E34    74 23           je short 00463E59
00463E59    8B85 3CA04200   mov eax,dword ptr ss:[ebp+42A03C]
00463E5F    50              push eax
00463E60    6A 01           push 1
00463E62    68 FF0F1F00     push 1F0FFF
00463E67    BB 8B724200     mov ebx,42728B
00463E6C    FF541D 00       call dword ptr ss:[ebp+ebx]; kernel32.OpenProcess
00463E70    6A FF           push -1
00463E72    50              push eax
00463E73    BB DB724200     mov ebx,4272DB
00463E78    FF541D 00       call dword ptr ss:[ebp+ebx]; User32.WaitForInputIdle
00463E7C    F785 A4A34200 8>test dword ptr ss:[ebp+42A3A4],80
00463E86    75 17           jnz short 00463E9F
00463E88    BB 5F724200     mov ebx,42725F
00463E8D    FF541D 00       call dword ptr ss:[ebp+ebx]; kernel32.IsDebuggerPresent
00463E93    74 17           je short 00463EAC
00463EAC    F785 A4A34200 0>test dword ptr ss:[ebp+42A3A4],1
00463EB6    74 7C           je short 00463F34
00463F34    F785 A4A34200 8>test dword ptr ss:[ebp+42A3A4],80
00463F3E    74 08           je short 00463F48
00463F48    8D85 349F4200   lea eax,dword ptr ss:[ebp+429F34]
00463F4E    50              push eax; yP.0046414E
00463F4F    C3              retn
0046414E    F785 48A04200 0>test dword ptr ss:[ebp+42A048],8
00464158    74 08           je short 00464162
0046415A    8D85 549F4200   lea eax,dword ptr ss:[ebp+429F54]
00464160    50              push eax; yP.0046416E
00464161    C3              retn
0046416E    32C0            xor al,al
00464170    C685 8E734200 C>mov byte ptr ss:[ebp+42738E],0C3
00464177    8DBD 2B724200   lea edi,dword ptr ss:[ebp+42722B]
0046417D    B9 2F734200     mov ecx,42732F
00464182    81E9 2B724200   sub ecx,42722B
00464188    F3:AA           rep stos byte ptr es:[edi]
//清扫战场  00461445  Size=00000104
0046418A    8DBD E3734200   lea edi,dword ptr ss:[ebp+4273E3]
00464190    B9 369D4200     mov ecx,429D36
00464195    81E9 E3734200   sub ecx,4273E3
0046419B    F3:AA           rep stos byte ptr es:[edi]
//清扫战场  004615FD  Size=00002953
0046419D    8DBD A39F4200   lea edi,dword ptr ss:[ebp+429FA3]
004641A3    B9 BFA44200     mov ecx,42A4BF
004641A8    81E9 A39F4200   sub ecx,429FA3
004641AE    F3:AA           rep stos byte ptr es:[edi]
//清扫战场  004641BD  Size=0000051C
004641B0    61              popad
```

## 十五、飞向光明之巅

```
004641B1    50              push eax
004641B2    33C0            xor eax,eax
004641B4    64:FF30         push dword ptr fs:[eax]
004641B7    64:8920         mov dword ptr fs:[eax],esp
004641BA    EB 01           jmp short 004641BD
004641BD    0000            add byte ptr ds:[eax],al
//yC的典型异常，看堆栈
0012EAF0   0012EC70  指针到下一个 SEH 记录
0012EAF4   00463F50  SE 句柄
BP 00463F50   Shift+F9 中断后取消断点
00463F50    55              push ebp
00463F51    8BEC            mov ebp,esp
00463F53    57              push edi
00463F54    36:8B45 10      mov eax,dword ptr ss:[ebp+10]
00463F58    3E:8BB8 C400000>mov edi,dword ptr ds:[eax+C4]
00463F5F    3E:FF37         push dword ptr ds:[edi]
00463F62    33FF            xor edi,edi
00463F64    64:8F07         pop dword ptr fs:[edi]
00463F67    3E:8380 C400000>add dword ptr ds:[eax+C4],8
00463F6F    3E:8BB8 A400000>mov edi,dword ptr ds:[eax+A4]
00463F76    C1C7 07         rol edi,7
00463F79    3E:89B8 B800000>mov dword ptr ds:[eax+B8],edi
//[eax+B8]=00436885     OEP值
00463F80    B8 00000000     mov eax,0
00463F85    5F              pop edi
00463F86    C9              leave
00463F87    C3              retn
BP 00436885   Shift+F9 中断后取消断点
00436885    6A 60           push 60
//OEP
00436887    68 90564100     push 415690
0043688C    E8 670D0000     call 004375F8
00436891    BF 94000000     mov edi,94
00436896    8BC7            mov eax,edi
00436898    E8 23430000     call 0043ABC0
```

## 十六、终场：如果你是我的传说

用`LordPE`修正`dump.eXe`的`OEP RVA=00036885`和`ImportTable RVA=0005AC50`就行了，再清除掉壳数据，算是完美脱壳吧。

若要快速脱壳的话，避开文中重点提示的反跟踪、自检验和时间检验，避开输入表加密就搞定了。

幕落，该散场了。

```
         ,     \_/
        /| \_.-~/            \\_     ,        青春都一晌
       ( /~   /              \~-.\_ |\ 
       `\\  \_/                \   ~\ )         忍把浮名
   \_-~~~-.)  )\_\_/;;,.          \\_  //'
  /'\_,\   --~   \ ~~~-  ,;;\\_\_\_(  (.-~~~-.       换了脱壳轻狂
`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_`\ 
  /~~//'   /' `~\         ) /--..\_, )\_  `~
  "  `~"  "      `"      /~'`\    `\\~~\
                         "     "   "~'  ""
            UnPacked By :  fly
             2005-05-22 24:00
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
