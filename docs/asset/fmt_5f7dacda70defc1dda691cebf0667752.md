
## 导出函数遍历与校验逻辑

```
0073422E      48               dec eax                        ; 个数-1
0073422F      E9 F6570000      jmp zhengtu.00739A2A
00739A2A      85C0             test eax,eax                   ; 如果输出函数个数小于=0,跳
00739A2C    ^ 0F8C DBA2FDFF    jl zhengtu.00713D0D
00739A32      40               inc eax                        ; 项++
00739A33      8945 DC          mov dword ptr ss:[ebp-24],eax    ; ebp.24=导出函数个数
00739A36      C745 E8 00000000 mov dword ptr ss:[ebp-18],0
00739A3D      8B45 EC          mov eax,dword ptr ss:[ebp-14]    ; eax=ebp.14=IET
00739A40      8B40 20          mov eax,dword ptr ds:[eax+20]    ; eax=IET.Base=基址
00739A43    ^ E9 805EFDFF      jmp zhengtu.0070F8C8
0070F8C8      0345 FC          add eax,dword ptr ss:[ebp-4]     ; eax+hModule=导出函数数组
0070F8CB      33D2             xor edx,edx
0070F8CD      52               push edx                       ; push 0
0070F8CE      50               push eax                       ; push 导出函数数组
0070F8CF      E9 D6C50000      jmp zhengtu.0071BEAA
0071BEAA      8B45 E8          mov eax,dword ptr ss:[ebp-18]    ; eax 已处理函数计数
00712610      C1E0 02          shl eax,2                      ; 序号*4
00712613      99               cdq
00752682      8B00             mov eax,dword ptr ds:[eax]       ; 取导出函数项的 RVA
0071B87F      0345 FC          add eax,dword ptr ss:[ebp-4]     ; RVA+hModule
00730F50      FF45 F4          inc dword ptr ss:[ebp-C]        ; 函数名数组++
007311AD     /0F85 1E7F0100    jnz zhengtu.007490D1             ; != NULL 跳
007311B3      |8B45 FC          mov eax,dword ptr ss:[ebp-4]     ; 得出函数名长度
007311B6      |8945 F4          mov dword ptr ss:[ebp-C],eax
007311B9      |C165 F8 08       shl dword ptr ss:[ebp-8],8       ; 函数名长度左移 8 位???
00708209      0FB600           movzx eax,byte ptr ds:[eax]      ; eax=取函数名的字节值
0070820C      0145 F8          add dword ptr ss:[ebp-8],eax     ; 长度左移 8 位后+函数名字节
0070820F      8B45 F8          mov eax,dword ptr ss:[ebp-8]
00708212      C1E0 0A          shl eax,0A                       ; 再左移 A 位

即出循环

00712CE2      8B45 F8          mov eax,dword ptr ss:[ebp-8]     ; 取函数名运算后的值
0074505D      C1E0 03          shl eax,3                        ; 运算后的值左移 3 位
007236AA      C1E8 0B          shr eax,0B                       ; 左移 B 位
007236AD      3145 F8          xor dword ptr ss:[ebp-8],eax     ; 异域
007236B0      8B45 F8          mov eax,dword ptr ss:[ebp-8]
007236B3      C1E0 0F          shl eax,0F                       ; 左移 F 位
007236B6    ^ E9 2520FFFF      jmp zhengtu.007156E0
007156E0      0145 F8          add dword ptr ss:[ebp-8],eax
007156E3      68 B6947100      push zhengtu.007194B6
007194B6      8B45 F8          mov eax,dword ptr ss:[ebp-8]     ; eax=函数名的运算值 ----函数返回
007194B9      8BE5             mov esp,ebp
007194BB      5D               pop ebp
007194BC      C3               retn
0071B882      E8 D4E80000      call <zhengtu.Get 函数校验值>//以上为该函数的运算
0071B887      3B45 F8          cmp eax,dword ptr ss:[ebp-8]     ; 比较还回的值校验值,是否是需要寻找的 API
0071B88A    ^ 0F85 9499FFFF    jnz zhengtu.00715224             ; 不相等继续
0071B890      E9 9AE30100      jmp zhengtu.00739C2F             ; 相等,跳-----------------------------------在此外下断,就能得到 API 的真实地址
```

## 修复 IAT 代码

随便找一块空白区域写上以下代码：

```
//修复输入表写上去的代码

00753F26      55               push ebp
00753F27      BE 68D06100      mov esi,zhengtu.0061D000        ; //未处理 IAT 首地址
00753F2C      AD               lods dword ptr ds:[esi]
00753F2D      3D 00000010      cmp eax,10000000
00753F32     ^ 77 F8           ja short zhengtu.00753F2C
00753F34      85C0             test eax,eax
00753F36     ^ 74 F4           je short zhengtu.00753F2C
00753F38      8935 40387500    mov dword ptr ds:[753840],esi   ; //保存 IAT 地址
00753F3E      BB 4B3F7500      mov ebx,zhengtu.00753F4B        ; //保存还回的地址
00753F43      891D 44387500    mov dword ptr ds:[753844],ebx
00753F49      FFD0             call eax
00753F4B      8B35 40387500    mov esi,dword ptr ds:[753840]   ; //还原 IAT 地址
00753F51      8946 FC          mov dword ptr ds:[esi-4],eax    ; //保存值
00753F54      81FE 20D36100    cmp esi,zhengtu.0061D320        ; //比较输入表是否被处理完毕
00753F5A     - 77 FE           ja short zhengtu.00753F5A       ; //大于进入死循环
00753F5C     ^ EB CE          jmp short zhengtu.00753F2C      ; //跳回接着处理
```

然后在壳解密出 API 真实地址处写上以下代码：

```
0070A27E      8B1D 44387500    mov ebx,dword ptr ds:[753844]   ; 返回自已写的代码处,继续处理下一个 API
0070A284      53               push ebx
0070A285      C3               retn
```

到此，代码已解开，IAT 也修复了，但 OEP 被抽取的代码，盼望大侠指点

`4dd466d0e1e0cc123afbe504da2462ca`
