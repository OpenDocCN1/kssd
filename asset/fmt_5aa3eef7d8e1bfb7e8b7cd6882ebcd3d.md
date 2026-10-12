
```
1107599F    8B45 14       MOV EAX,DWORD PTR SS:\[EBP+14\]
110759A2    66:C700 FFFF MOV WORD PTR DS:\[EAX\],0FFFF                   ; 读取标志
110759A7    90           NOP
110759A8    90           NOP
110759A9    33C0         XOR EAX,EAX
```

这是按照 `VB` 自身的要求进行修改的。至于按照 `zhuliang` 先生的文章：

`http://bbs.pediy.com/showthread.php?t=79278`

进行修改后也确能正常工作，那应该是比尔给我们制造的糊涂！
