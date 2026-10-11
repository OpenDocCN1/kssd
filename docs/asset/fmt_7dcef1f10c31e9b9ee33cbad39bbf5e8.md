
```text
00563DA4  |.  8BD0           |mov    edx, eax                      ;  商值赋给 edx  
00563DA6  |.  80C2 41        |add    dl, 41                        ;  商转换为大写字母  
00563DA9  |.  8D45 F0        |lea    eax, dword ptr [ebp-10]      
00563DAC  |.  E8 DF01EAFF    |call   swdj_Unp.00403F90             
00563DB1  |.  FF75 F0        |push   dword ptr [ebp-10]           
00563DB4  |.  8B45 FC        |mov    eax, dword ptr [ebp-4]       
00563DB7  |.  B9 1A000000    |mov    ecx, 1A                       
00563DBC  |.  99             |cdq                                  ;  
00563DBD  |.  F7F9           |idiv   ecx                            ;  带符号数除法  
00563DBF  |.  80C2 41        |add    dl, 41                        ;  余数转换为大写字母  
00563DC2  |.  8D45 EC        |lea    eax, dword ptr [ebp-14]       
00563DC5  |.  E8 C601EAFF    |call   swdj_Unp.00403F90             
00563DCA  |.  FF75 EC        |push   dword ptr [ebp-14]           
00563DCD  |.  8BC7           |mov    eax, edi                       
00563DCF  |.  BA 03000000    |mov    edx, 3                        
00563DD4  |.  E8 4F03EAFF    |call   swdj_Unp.00404128             
00563DD9  |.  43             |inc    ebx                            
00563DDA  |.  4E             |dec    esi                            
00563DDB  |.^ 75 AF           \jnz  short swdj_Unp.00563D8C　　　　　循环操作  
00563DDD  |>  33C0           |xor    eax, eax                       
00563DDF  |.  5A             |pop    edx                            
00563DE0  |.  59             |pop    ecx                            
00563DE1  |.  59             |pop    ecx                            
00563DE2  |.  64:8910        |mov    dword ptr fs:[eax], edx       
00563DE5  |.  68 FF3D5600    |push   swdj_Unp.00563DFF             
00563DEA  |>  8D45 EC        |lea    eax, dword ptr [ebp-14]       
00563DED  |.  BA 03000000    |mov    edx, 3                        
00563DF2  |.  E8 1500EAFF    |call   swdj_Unp.00403E0C             
00563DF7  |\  C3             |retn                                 
00563DF8   .- E9 67FAE9FF    |jmp    swdj_Unp.00403864              
00563DFD   .^ EB EB          |jmp    short swdj_Unp.00563DEA       
00563DFF   .  5F             |pop    edi                            
00563E00   .  5E             |pop    esi                            
00563E01   .  5B             |pop    ebx                            
00563E02   .  8BE5           |mov    esp, ebp                       
00563E04   .  5D             |pop    ebp                            
00563E05   .  C3             |retn                                 
```

---

**【经验总结】**

软件注册算法大致如下：

1. 将机器码依次与一数据进行异或
2. 将变形后的机器码依次与 `1A` 相除，分别取商和余数，并将两数化为大写字母
3. 将以上所得的字母相连即为注册码

以上是菜鸟的分析，如果不对地方请大家多多指教！

---

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 21 日 13:00:48

`4dd466d0e1e0cc123afbe504da2462ca`
