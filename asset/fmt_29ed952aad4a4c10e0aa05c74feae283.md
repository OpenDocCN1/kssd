
```asm
    JNZ       localB634     
    
    MOV       EBX,01   
    
    MOV       ESI,0Ah   
    
localB693:   
    
    MOV       ECX,01   
    
localB698:   
    
    lea       eax,  SNstr   
    
    MOVZX      EAX,BYTE ptr [EAX+ECX-01]   
    
    IMUL      EBX         
    
    ADD       EAX,ECX   
    
    ADD       EAX,ESI   
    
    MOV       EBX,0179h   
    
    CDQ                 
    
    IDIV      EBX   
    
    MOV       EBX,EDX   
    
    INC       ECX         
    
    CMP       ECX,BYTE +09h   
    
    JNZ       localB698   
    
    MOV       EAX,EBX   
    
    MOV       ECX,24h         
    
    CDQ   
    
    IDIV      ECX   
    
    lea       eax,  Conststr   
    
    MOV       AL,[EAX+EDX]   
    
    mov       dl,al   
    
    lea       EaX,SNstr   
    
    mov       [eax+esi-01],dl   
    
    INC       ESI           
    
    CMP       ESI,BYTE +12h   
    
    JNZ       localB693   
    
    popad   
    
    ret   
    
Mycal  ENDP   
   
end start   
```
————————————————————————————————

把这三部分分别存为`hd.h`, `snsitman.rc`, `snsitman.asm`，然后在`masmv5.2`下编译，生成注册机`snsitman.exe`。

使用时注意：

1. `name`一定不能为空，否则会出错！！！
2. `name`长度要大于等于 3，小于等于 20。我已在界面上提示要大于等于 3，至于要小于等于 20，我提供的文本框只够输入 20 个字符，所以不怕你多输入，就怕你少输入或不输入。呵呵呵。。。
3. 由于这个软件的注册码生成的特殊性，我在注册机内是事先放了一个假的注册码让程序去计算真的注册码，这个假注册码你可以自己在源程序中修改。但第 4－9 位一定要是数字，其它是数字或字母都可以，没有特别的要求。

这个注册机做好了是好用，可以直接在`window`下运行，输入汉字不再麻烦了，可这个注册机要做出来可不容易，我花了好久才把`masmv5.2`设置好，学会怎么来编译，链接，和生成`exe`文件...倒底谁麻烦？？？

好了，大功告成，打个 Kiss!

`hsf0214` 于 2001.10.31. 下午

`4dd466d0e1e0cc123afbe504da2462ca`
