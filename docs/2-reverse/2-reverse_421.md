

# IDA 插件编写中如何添加注释呢

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-311.htm`](https://www.pediy.com/kssd/pediy09/pediy09-311.htm)

> IDA 插件编写中如何给代码行或者数据行添加注释呢  
>   
> 另外哪里有所有 IDA SDK 函数的介绍文档呢, 中英文都行 

> set\_cmt  
>   
> 有篇 E 文的教程,名字记不住了自己找找  
> 也可以翻翻 SDK 的头文件 

> 以下头函数定义摘自 IDA SDK 5.1  
>    
> //--------------------------------------------------------------------------  
> //      I N D E N T E D  C O M M E N T S  
> //--------------------------------------------------------------------------  
> // Set an indented comment  
> //      ea     - linear address         有效地址，比如在 IDA 的反汇编代码中，  
> //                                      有这么一行：.text:00404108         push    ebp  
> //                                      则有效地址即为 404108。  
> //  
> //      comm   - comment string         要添加的注释内容。  
> //  
> //      rptble - is repeatable?         是否设置成可重复性注释，设置为 true，则是可重复性注释  
> // returns: 1-ok, 0-failure             返回 1 表示成功，0 表示错误。  
>   
> idaman bool ida\_export set\_cmt(ea\_t ea, const char \*comm, bool rptble);  
>   
> 楼上的朋友，可能你已经知道怎么用这个 set\_cmt 函数，下面的代码十分简单地在 IDA 光标所在的一行，添加了可重复性注释，  
>   
>     ea\_t current = get\_screen\_ea();         //返回当前光标所在的有效地址，  
>     bool ret = set\_cmt(current, "添加成功", true);  
>     if (ret)  
>     {  
>         info("添加注释成功");  
>     }  
>     else  
>     {  
>         warning("添加注释错误");  
>     } 

> repeatable==可重复 

> softworm，谢谢你指出我的错误，  
> 是我翻译错误了，repeatable 是你解释的这个意思，”重复的“。  
>   
> 我有一点不明白的是，  
> 好像在 IDA 里按下分号键，就弹出 repeatable comment 对话框，  
> 比如输入“逆向”二字，注释显示 “逆向”。  
>   
> 再在同一行，按下 Shift+分号键，则弹出一个 Enter comment 对话框，  
> 输入“工程”二字，注释就被覆盖成 “工程"，刚才输入的”逆向“，就看不到了，  
> 而且，按下分号键，还能看到，”逆向“还在那里。  
>   
> 这样感觉，就好象有点被替换的感觉，为什么会这样的 设计？ 

> 按分号输入的注释,所有交叉参考处都会出现,按冒号(你的是 Shift+分号? 我不知道![](img/9e827a3de2ad55eed0f95082425861b2.jpg "微笑") )  
> 的注释只在该处出现,如果一个地址有 2 种注释,在该地址只会显示非重复注释.  
>   
> 我只是随手写的,没有查帮助,希望没错 

 4dd466d0e1e0cc123afbe504da2462ca