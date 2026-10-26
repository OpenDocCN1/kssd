
```assembly
                .if eax

                        invoke OutputDebugString,SADD("KILL X3")

                        mov eax,offset mainthread

                        invoke CreateThread,NULL,NULL,eax,\
                                        NULL,0,\
                                        ADDR ThreadID

                        invoke CloseHandle,eax

                        invoke OutputDebugString,SADD("KILL X3  \_\_end")

                .endif

                popad

        .elseif reason == DLL\_THREAD\_ATTACH

        .endif

        mov     eax,TRUE

        ret

DllEntry Endp

End DllEntry
```

注在游戏启动时注入dll

本代码只是方法.其它游戏不通用...

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
