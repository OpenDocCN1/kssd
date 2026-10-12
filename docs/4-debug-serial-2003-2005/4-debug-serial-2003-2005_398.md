
# 极速火龙 CD 压缩器 1.28 版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50344.htm`](https://www.pediy.com/kssd/pediy05/pediy50344.htm)

## 基本信息

- 作者：`nzinzi`
- 破解时间：2022-11-2
- 软件简介：能压缩的格式实在多
- 限制：试用 30 天
- 注册费用：
- 破解工具：`TRW 1.22`
- 难度：易

## 注意事项

注意此软件用放跟踪的请先开软件再开 `TRW`，还有此软件有加 `petite 2.1` 壳，可以不脱直接跟踪~ 呵呵省事~~ 高手不要笑啊，小弟菜啊

## 操作步骤

用 `TRW` 下一个 `bpx hmemcpy` 的万能断点，然后几下 `f12` 来到底下

```
:00476907 8B45F8          mov eax, dword ptr [ebp-08] 
:0047690A 8A00            mov al, byte ptr [eax] 
:0047690C 8845FC          mov byte ptr [ebp-04], al 

                          mov eax, dword ptr [ebp-08] 
                          mov al, byte ptr [eax+01] 
                          mov byte ptr [ebp-03], al 

                          mov eax, dword ptr [ebp-08] 
                          mov al, byte ptr [eax+02] 
                          mov byte ptr [ebp-02], al 

                          mov eax, dword ptr [ebp-08] 
                          mov al, byte ptr [eax+03] 
                          mov byte ptr [ebp-01], al                这一段是把我们的注册名前 4 位提取出来~

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:00402B88(U), :00402BA6(C)

:00402B8E 80EB30          sub bl, 30 
:00402B91 80FB09          cmp bl, 09 
:00402B94 772C            ja 00402BC2  |-ASCLL 码把转成数字 判断是不是 0-9 数字 
:00402B96 39F8            cmp eax, edi 
:00402B98 7728            ja 00402BC2 
:00402B9A 8D0480          lea eax, dword ptr [eax+4*eax]  
:00402B9D 01C0            add eax, eax               10*X 利用十六进制加法把十进制转成十六进制~
:00402B9F 01D8            add eax, ebx               
:00402BA1 8A1E            mov bl, byte ptr [esi] 
:00402BA3 46              inc esi 
:00402BA4 84DB            test bl, bl 
:00402BA6 75E6            jne 00402B8E               把我们输入的注册码转成 16 进制~

:00476958 0FBE45FC        movsx eax, byte ptr [ebp-04]  
:0047695C 0FBE55FD        movsx edx, byte ptr [ebp-03] 
:00476960 03C2            add eax, edx 
:00476962 0FBE55FE        movsx edx, byte ptr [ebp-02] 
:00476966 03C2            add eax, edx 
:00476968 0FBE55FF        movsx edx, byte ptr [ebp-01] 
:0047696C 03C2            add eax, edx             把名字的前四位的 ASCLL 相加      
:0047696E 69C04E61BC00    imul eax, 00BC614E    乘以 BC614E 
:00476974 B9BB000000      mov ecx, 000000BB    
:00476979 99              cdq 
:0047697A F7F9            idiv ecx               除以 BB    
:0047697C 3BD8            cmp ebx, eax           生成真正的注册码在 EAX 和我们输入的注册码 EBX 比较 
                                                     ？ eax ? ebx 就可以看到了~
:0047697E 0F85595A0700    jne 004EC3DD            不相同跳转 
```

把输入的注册名是 `nzinzi`，转成注册码是这样的 `(6e+7a+69+6e)*bc614e/bb=63d6cc` 转成十进制就是 `6543052` `4dd466d0e1e0cc123afbe504da2462ca`
