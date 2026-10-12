
```
CODE:004014BD             push    0               ; lpThreadAttributes  
CODE:004014BF             call    CreateThread  
CODE:004014C4             mov     ds:hHandle, eax  
CODE:004014C9  
CODE:004014C9 locret_4014C9:  
CODE:004014C9             retn  
CODE:004014C9 sub_40148D      endp  
```

创建了一个线程（^_^，到这里才刚刚开始噢！），肯定不会干什么好事，下次我们再详细分析这个线程。

到这里有个 2 问题：

1. 由函数`sub_4033E7`算出的内存数据做何用？
2. 文件`srv32Res`、`sccss`和`srv32tsk`做何用？

`4dd466d0e1e0cc123afbe504da2462ca`
