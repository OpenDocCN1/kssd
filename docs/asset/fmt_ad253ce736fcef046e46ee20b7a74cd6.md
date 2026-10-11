
```
0045DEBE    90                 NOP  
0045DEBF    90                 NOP  
0045DEC0    50                 PUSH EAX                            ; patch 点 1 的 Patch 代码  
0045DEC1    A3 B0DE4500        MOV DWORD PTR DS:[45DEB0],EAX     ; 保存动态申请的地址  
0045DEC6    8D4424 04          LEA EAX,DWORD PTR SS:[ESP+4]      
0045DECA    8328 05            SUB DWORD PTR DS:[EAX],5          
0045DECD    8B00               MOV EAX,DWORD PTR DS:[EAX]        
0045DECF    C700 8945E083      MOV DWORD PTR DS:[EAX],83E04589   ; 还原代码防止壳检测  
0045DED5    C640 04 7D         MOV BYTE PTR DS:[EAX+4],7D        
0045DED9    53                 PUSH EBX                            ; 修改 patch 点 2 的代码  
0045DEDA    BB 00DF4500        MOV EBX,0045DF00                    
0045DEDF    B8 34734000        MOV EAX,00407334                    
0045DEE4    2BD8               SUB EBX,EAX                         
0045DEE6    83EB 05            SUB EBX,5                           
0045DEE9    C600 E8            MOV BYTE PTR DS:[EAX],0E8           
0045DEEC    8958 01            MOV DWORD PTR DS:[EAX+1],EBX        
0045DEEF    5B                 POP EBX                             ; 修改完跳回去执行原代码  
0045DEF0    58                 POP EAX                             
0045DEF1    C3                 RETN                                
0045DEF2    90                 NOP  
0045DEF3    90                 NOP  
0045DEF4    90                 NOP  
0045DEF5    90                 NOP  
0045DEF6    90                 NOP  
0045DEF7    90                 NOP  
0045DEF8    90                 NOP  
0045DEF9    90                 NOP  
0045DEFA    90                 NOP  
0045DEFB    90                 NOP  
0045DEFC    90                 NOP  
0045DEFD    90                 NOP  
0045DEFE    90                 NOP  
0045DEFF    90                 NOP  
0045DF00    50                 PUSH EAX                            ; Patch 点 2 的代码  
0045DF01    A1 B0DE4500        MOV EAX,DWORD PTR DS:[45DEB0]      
0045DF06    53                 PUSH EBX                            
0045DF07    BB 40DF4500        MOV EBX,0045DF40                    
0045DF0C    05 A1430200        ADD EAX,243A1                       ; 修改 Patch 点 0 的代码  
0045DF11    2BD8               SUB EBX,EAX                         
0045DF13    83EB 05            SUB EBX,5                           
0045DF16    C600 E8            MOV BYTE PTR DS:[EAX],0E8           
0045DF19    8958 01            MOV DWORD PTR DS:[EAX+1],EBX        
0045DF1C    5B                 POP EBX                             
0045DF1D    8D4424 04          LEA EAX,DWORD PTR SS:[ESP+4]        
0045DF21    8328 05            SUB DWORD PTR DS:[EAX],5            
0045DF24    8B00               MOV EAX,DWORD PTR DS:[EAX]          
0045DF26    C700 833DC042      MOV DWORD PTR DS:[EAX],42C03D83     ; 还原代码防止壳检测  
0045DF2C    C640 04 46         MOV BYTE PTR DS:[EAX+4],46          
0045DF30    58                 POP EAX                             
0045DF31    C3                 RETN                                
0045DF32    90                 NOP  
0045DF33    90                 NOP  
0045DF34    90                 NOP  
0045DF35    90                 NOP  
0045DF36    90                 NOP  
0045DF37    90                 NOP  
0045DF38    90                 NOP  
0045DF39    90                 NOP  
0045DF3A    90                 NOP  
0045DF3B    90                 NOP  
0045DF3C    90                 NOP  
0045DF3D    90                 NOP  
0045DF3E    90                 NOP  
0045DF3F    90                 NOP  
0045DF40    50                 PUSH EAX                            ; Patch 点 0 处的处理代码  
0045DF41    C745 D8 6A08B69    MOV DWORD PTR SS:[EBP-28],94B6086A  
0045DF48    8D4424 04          LEA EAX,DWORD PTR SS:[ESP+4]        
0045DF4C    8328 05            SUB DWORD PTR DS:[EAX],5            
0045DF4F    8B00               MOV EAX,DWORD PTR DS:[EAX]          
0045DF51    C700 33416033      MOV DWORD PTR DS:[EAX],33604133     ; 还原并执行原代码  
0045DF57    C640 04 45         MOV BYTE PTR DS:[EAX+4],45          
0045DF5B    58                 POP EAX                             
0045DF5C    C3                 RETN                                
```

修改完毕，运行一下，YUP，搞定了。`patch` 成功后看了一下 `arm v4.1` 的主程序，发现 `patch` 方法有所不同，可惜我不会破解主程序，所以我看也没看懂:-(，哪位知道的指教一下俺。

`Greetz`:

`Fly.Jingulong`, `yock`, `tDasm.David.hexer`, `hmimys`, `ahao.UFO(brother).alan(sister)`, `all of my friends and you!`

`By loveboom[DFCG][FCG][US]`

`http://blog.csdn.net/bmd2chen`

`Email:loveboom#163.com`

`Date:5/11/2005 3:38:00 PM`

`4dd466d0e1e0cc123afbe504da2462ca`
