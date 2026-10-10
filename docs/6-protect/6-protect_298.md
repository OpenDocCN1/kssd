

# 清除彩虹狗花指令的脚本

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-413.htm`](https://www.pediy.com/kssd/pediy09/pediy09-413.htm)

> var x  
> VAR y  
> var Z  
> var TMP  
> var J  
>   
> mov x,401792  
> start:   
> find x,#70037101#  
> cmp $RESULT ,0  
> je end  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp start   
>   
> end:  
>   
> mov x,401792  
> start:   
> find x,#72037301#  
> cmp $RESULT ,0  
> je end1  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end  
>   
>   
> end1:  
>   
> mov x,401792  
> start:   
> find x,#76037701#  
> cmp $RESULT ,0  
> je end2  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end1  
>   
> end2:  
>   
> mov x,401792  
> start:   
> find x,#79037801#  
> cmp $RESULT ,0  
> je end3  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end2  
>   
> end3:  
>   
> mov x,401792  
> start:   
> find x,#eb01#  
> cmp $RESULT ,0  
> je end4  
>   
> mov \[$RESULT\],#909090#,3  
>   
>   
>   
> add x,9  
> log x  
> jmp end3  
>   
>   
> end4:  
>   
> mov x,401792  
> start:   
> find x,#7a037b01#  
> cmp $RESULT ,0  
> je end5  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end4  
>   
>   
> end5:  
>   
>   
> mov x,401792  
> start:   
> find x,#75037401#  
> cmp $RESULT ,0  
> je end6  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end5  
>   
>   
> end6:  
>   
>   
> mov x,401792  
> start:   
> find x,#7c037d01#  
> cmp $RESULT ,0  
> je end7  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end6  
>   
> end7:  
>   
>   
> mov x,401792  
> start:   
> find x,#7e037f01#  
> cmp $RESULT ,0  
> je end8  
>   
> mov \[$RESULT\],#9090909090#,5  
>   
> add x,9  
> log x  
> jmp end7  
>   
> end8:  
>   
>   
> ret  
>   
>   
>   
>   
> 代码简单没什么技术含量  
> 当时　为了快　就是反复　copy，没好好规划，最笨的方法  
> 有兴趣的　可以优化优化 

 4dd466d0e1e0cc123afbe504da2462ca