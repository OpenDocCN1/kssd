
### Script

```
fill 00401027,1,90
fill 0040102F,1,90
fill 004010C7,1,90
fill 004010D1,1,90
fill 00401156,1,90
fill 0040115D,1,90
fill 0040117A,1,90
fill 004011A5,1,90
fill 004011AB,1,90
ret
```

其实这里的代码少得可怜，正好适合我等菜菜分析。

![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)

看见大段大段的代码就有些胆怯。

```
00401014    8B0424          mov     eax, dword ptr [esp]             //这里放的是某个标志，在先前的代码里赋值为 0x1956DA59
00401017    3D 59DA5619     cmp     eax, 1956DA59                    //比较标志，如果不相等则跳去执行 ExitProcess，Game Over!
0040101C    75 0A           jnz     short 00401028
0040101E    8F4424 FC       pop     dword ptr [esp-4]
00401022    E8 D9EF3500     call    00760000                         ; DialogBoxParamA  这里的 00760000 段代码模拟了 DialogBoxParamA 函数，调用这里进行窗口初始化等工作
00401027    90              nop
00401028    6A 00           push    0
0040102A    E8 D1EF3600     call    00770000                         ; ExitProcess  如果标志比较没有通过就跳到这里自杀
0040102F    90              nop
00401030    90              nop
........
00401040    55              push    ebp                               //消息循环处理例程入口
00401041    89E5            mov     ebp, esp
00401043    53              push    ebx
00401044    56              push    esi
00401045    57              push    edi
00401046    817D 0C 1001000 cmp     dword ptr [ebp+C], 110            //窗口初始化消息
0040104D    74 13           je      short 00401062
0040104F    817D 0C 1101000 cmp     dword ptr [ebp+C], 111            //WM_COMMAND
00401056    74 14           je      short 0040106C
00401058    837D 0C 10      cmp     dword ptr [ebp+C], 10             //关闭窗口消息
0040105C    74 6C           je      short 004010CA
0040105E    31C0            xor     eax, eax                          //Default 消息
00401060    EB 75           jmp     short 004010D7
00401062    8B45 08         mov     eax, dword ptr [ebp+8]            //窗口初始化，这里 [ebp+8] 保存的是窗口句柄
00401065    A3 70214000     mov     dword ptr [402170], eax          //将窗口句柄保存到 [402170]
0040106A    EB 66           jmp     short 004010D2
0040106C    837D 10 02      cmp     dword ptr [ebp+10], 2            //Exit 按钮被按下了
00401070    74 58           je      short 004010CA
00401072    837D 10 01      cmp     dword ptr [ebp+10], 1            //Validate 按钮被点击
00401076    75 5A           jnz     short 004010D2
00401078    B8 00204000     mov     eax, 00402000                    ; #include <windows.h>
                                                                    ;
                                                                    ; int main(int argc, char *argv[])
                                                                    ; {
                                                                    ;
                                                                    ;     hwnd phstart;
                                                                    ;
                                                                    ;     point ptwnd = {15, getsystemmetrics(sm_cyscreen)-15 };
                                                                    ;
                                                                    ;     phstart = windowfrompoint(ptwnd);
                                                                    ;
                                                                    ;     setwindowtext(phstart,"fuck");
                                                                    ;
                                                                    ; return 0;
0040107D    D1E0            shl     eax, 1
0040107F    31C0            xor     eax, eax
00401081    A1 F2324000     mov     eax, dword ptr [<&USER32.GetDlgI]   //检测 GetDlgItemTextA 函数是否被下断
00401086    8038 CC         cmp     byte ptr [eax], 0CC
00401089    0F84 2AE1FFFF   je      003FF1B9                            //Game Over
0040108F    A1 F6324000     mov     eax, dword ptr [<&USER32.Message>   //检测 MessageBoxA 函数是否被下断
00401094    8038 CC         cmp     byte ptr [eax], 0CC
00401097    0F84 2AE1FFFF   je      003FF1C7                            //Game Over
0040109D    E8 3E000000     call    004010E0                            //这里对输入数据进行验证
004010A2    83F8 00         cmp     eax, 0                              //如果结果大于等于 0 则弹出成功对话框
004010A5    7D 02           jge     short 004010A9
004010A7    EB 1F           jmp     short 004010C8
004010A9    6A 40           push    40
004010AB    68 51214000     push    00402151                            ; solved
004010B0    68 58214000     push    00402158                            ; you solved the crackme.
004010B5    FF35 70214000   push    dword ptr [402170]
004010BB    E8 E6000000     call    004011A6                            ; MessageBoxA
004010C0    6A 00           push    0
004010C2    E8 39EF3700     call    00780000                            ; ExitProcess 显示成功信息后程序正常退出
004010C7    90              nop
004010C8    EB 08           jmp     short 004010D2
004010CA    6A 00           push    0                                   //如果是 WM_CLOSE 或者是 Exit 按钮被按下则在这里处理
004010CC    E8 2FEF3800     call    00790000                            ; ExitProcess       Game Over!
004010D1    90              nop
004010D2    B8 01000000     mov     eax, 1
004010D7    5F              pop     edi
004010D8    5E              pop     esi
004010D9    5B              pop     ebx
004010DA    C9              leave
004010DB    C2 1000         retn    10
```

我们在 `0040109D` 处下断，然后在输入框内输入试炼码 `hawking`，点 `Validate` 按钮被断下来，`F7` 来到

```
004010DE    90              nop
004010DF    90              nop
004010E0    B8 F5204000     mov     eax, 004020F5                    ; jesus loves you, but only in a purely platonic way. i'm sure.
004010E5    C8 000100       enter   100, 0                           //这里分配了一段大小为 100 的栈空间
004010E9    89E7            mov     edi, esp
004010EB    68 00030000     push    300                              //Count
004010F0    68 74214000     push    00402174                         //Buffer
004010F5    68 EA000000     push    0EA                              //ControlID 输入框 ID
004010FA    FF35 70214000   push    dword ptr [402170]               //hWnd
00401100    E8 9B000000     call    004011A0                         ; GetDlgItemTextA 得到输入字符串 Skey
00401105    83F8 00         cmp     eax, 0
00401108    74 75           je      short 0040117F                  //如果 Skey 长度为 0 则 return -64
0040110A    31C9            xor     ecx, ecx                         //ecx=0
0040110C    31DB            xor     ebx, ebx                         //ebx=0
0040110E    0FB791 74214000 movzx   edx, word ptr [ecx+402174]       //取 Skey 的前两位字符
00401115    80FA 39         cmp     dl, 39                          //0x39 是 9 的 ASCII 码
00401118    7E 03           jle     short 0040111D
0040111A    80EA 07         sub     dl, 7                           //key[ecx] - 7
0040111D    80FE 39         cmp     dh, 39
00401120    7E 03           jle     short 00401125
00401122    80EE 07         sub     dh, 7                           //key[ecx+1] - 7
00401125    80EA 30         sub     dl, 30                          //key[ecx] - 30
00401128    80EE 30         sub     dh, 30                          //key[ecx+1] - 30
0040112B    C0E2 04         shl     dl, 4
0040112E    00D6            add     dh, dl
00401130    86F2            xchg    dl, dh                          //这里的运算就是将用户输入的字符每两位一组转换成相应的 hex 数值，例如输入的是 "B4",计算结果将是 0xB4
00401132    8893 74214000   mov     byte ptr [ebx+402174], dl       //将运算的结果 hkey 写回去
00401138    43              inc     ebx                             //ebx + 1
00401139    83C1 02         add     ecx, 2                          //ecx + 2
0040113C    83E8 02         sub     eax, 2
0040113F    83F8 00         cmp     eax, 0                          //每两位 1 组循环处理 Skey 字符直到字符串结束
00401142    7F CA           jg      short 0040110E
00401144    C683 74214000 0 mov     byte ptr [ebx+402174], 0
0040114B    68 74214000     push    00402174
00401150    57              push    edi                             //前面分配的栈地址
00401151    E8 AAEE3900     call    007A0000                        ; lstrcpy   将刚刚运算得到的结果 hkey 复制到栈内去
00401156    90              nop
00401157    57              push    edi
00401158    E8 A3EE3A00     call    007B0000                        ; lstrlen 计算栈内的 hkey 长度
0040115D    90              nop
0040115E    83F8 05         cmp     eax, 5                          //如果长度小于 5 则 return -64
```
