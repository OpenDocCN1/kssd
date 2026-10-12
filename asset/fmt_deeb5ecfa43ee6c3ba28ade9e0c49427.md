
```
:00401949  3C                   SetLastSystemError   ;Kernel GetLastError //针对调用 MessageBox 函数,取得扩展错误信息  

:0040194A  3208006CFF68FF64    FFreeStr             ;Do SysFreeString [arg_n]; [arg_n]=0 0008/2 times ~ arg  

:00401955  13                   ExitProcHresult      ;  

:00401956  0000                LargeBos             ;IDE beginning of line with 00 byte codes  

                                                     Moodsky[DFCG]  

                                                        2005.02.01
```

`![img/b16bec246e75299c87cf57e1fb808ac9.jpg](img/b16bec246e75299c87cf57e1fb808ac9.jpg)`

`4dd466d0e1e0cc123afbe504da2462ca`
