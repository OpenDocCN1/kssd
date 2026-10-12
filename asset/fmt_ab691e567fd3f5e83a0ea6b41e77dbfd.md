
```
|:00402948(C)   

|   

:0040294F 83F831               cmp eax, 00000031<---如果已经是数学了，则返回   

:00402952 7C05                 jl 00402959   

:00402954 83F839               cmp eax, 00000039   

:00402957 7E07                 jle 00402960   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00402952(C)   

|   

:00402959 83F00E               xor eax, 0000000E<----继续变化   

:0040295C 0C31                 or al, 31   

:0040295E EBB4                 jmp 00402914   

------------------------------------------------------------
```

注册机的代码与 `XXXX3000` 英雄版基本相同，只需要修改以下几处即可：

`reg2 = (reg1+8)*reg1`

`reg3 = reg2 +((reg1 ^ reg2)+1)*reg1`

`reg4 = reg1 +((reg2 * reg1)+1)*reg3` `4dd466d0e1e0cc123afbe504da2462ca`
