
```
712055C8     E8 0FD2F1FF        call 711227DC

//里面有异常

712055CD     A1 18DE2171        mov eax,dword ptr ds:[7121DE18]

//此处需要设置断点后Shift+F9，中断后取消断点，否则将在712055C8里异常导致无法继续调试

712055D2     8038 00            cmp byte ptr ds:[eax],0

//比较校验标志位

712055D5     74 05              je short 712055DC

712055D7     E8 907AF2FF        call 7112D06C

712055DC     A1 C8DF2171        mov eax,dword ptr ds:[7121DFC8]

712055E1     8038 00            cmp byte ptr ds:[eax],0

712055E4     74 17              je short 712055FD

712055FD     E8 01000000        call 71205603

71205603     58                 pop eax

71205604     83C0 0C            add eax,0C

71205607     60                 pushad

71205608     E8 F7EAFEFF        call 711F4104



7120561B     E8 CCD7FFFF        call 71202DEC

//这里设断，Shift+F9中断后取消断点

//处理CodeReplace，进入
```

下面修改代码的编号自上面延续，第五部分自10开始编号
