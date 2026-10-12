
```
00C10063    890E            MOV DWORD PTR DS:[ESI],ECX  
00C10065    8305 1020C100 0>ADD DWORD PTR DS:[C12010],4  
00C1006C    5A              POP EDX  
00C1006D    59              POP ECX  
00C1006E    C3              RETN  
00C1006F    90              NOP  
00C10070    90              NOP  
00C10071    90              NOP  
00C10072    90              NOP  
00C10073    90              NOP  
00C10074    90              NOP  
00C10075    90              NOP  
00C10076    90              NOP  
00C10077    90              NOP  
00C10078    90              NOP  
00C10079    90              NOP  
00C1007A    51              PUSH ECX                         ; AL==4 的第一个分支修复代码，第二个分支调用 AL==1 的代码  
00C1007B    52              PUSH EDX  
00C1007C    8B5424 2C       MOV EDX,DWORD PTR SS:[ESP+2C]  
00C10080    3B15 0020C100   CMP EDX,DWORD PTR DS:[C12000]          ; wsock32.#1139  
00C10086    74 0D           JE SHORT 00C10095  
00C10088    8915 0020C100   MOV DWORD PTR DS:[C12000],EDX  
00C1008E    8305 1020C100 0>ADD DWORD PTR DS:[C12010],4  
00C10095    8B0D 1020C100   MOV ECX,DWORD PTR DS:[C12010]          ; UnKillEr.0047E0A4  
00C1009B    890E            MOV DWORD PTR DS:[ESI],ECX  
00C1009D    8305 1020C100 0>ADD DWORD PTR DS:[C12010],4  
00C100A4    5A              POP EDX  
00C100A5    59              POP ECX  
00C100A6    8B43 40         MOV EAX,DWORD PTR DS:[EBX+40]  
00C100A9    C3              RETN  
00C100AA    90              NOP  
00C100AB    90              NOP  
00C100AC    90              NOP  
00C100AD    90              NOP  
00C100AE    90              NOP  
00C100AF    90              NOP  
00C100B0    51              PUSH ECX                         ; AL == 0 时的修复代码  
00C100B1    52              PUSH EDX  
00C100B2    8B5424 1C       MOV EDX,DWORD PTR SS:[ESP+1C]  
00C100B6    3B15 0020C100   CMP EDX,DWORD PTR DS:[C12000]          ; wsock32.#1139  
00C100BC    74 0D           JE SHORT 00C100CB  
00C100BE    8915 0020C100   MOV DWORD PTR DS:[C12000],EDX  
00C100C4    8305 1020C100 0>ADD DWORD PTR DS:[C12010],4  
00C100CB    8B4D 10         MOV ECX,DWORD PTR SS:[EBP+10]  
00C100CE    8B15 1020C100   MOV EDX,DWORD PTR DS:[C12010]          ; UnKillEr.0047E0A4  
00C100D4    8911            MOV DWORD PTR DS:[ECX],EDX            
00C100D6    8902            MOV DWORD PTR DS:[EDX],EAX  
00C100D8    8305 1020C100 0>ADD DWORD PTR DS:[C12010],4  
00C100DF    5A              POP EDX  
00C100E0    59              POP ECX  
00C100E1    C3              RETN  
00C100E2    90              NOP  
```

代码写完后，记得先在`C12010`处写上一个起始地址，否则会异常的^\_^。

现在取消`009ECCCE`处的断点，忽略全部异常。运行中断在我们第一次设断处中断。断下后，取消内存异常，`F9`运行发生异常：

```
009F6807    C700 7F0677B9   MOV DWORD PTR DS:[EAX],B977067F        ; 异常发生在这里  
009F680D    FB              STI  
```

异常后`CTRL+B`查找二进制`FF35`，可以找到几地方，正确的地方应该是:

```
push [addr]
ret
```

找到下面的地址后下断，`shift+f9`，中断后`ret`返回的地址就是`Stolen code`的开始处。

```
009F6AF0    FF35 4024A000   PUSH DWORD PTR DS:[A02440]     
009F6AF6    C3              RETN  
```

后面的`Stolen code`只能用手工慢慢找，当然你也可以用`fly`的方法来做的:-P.

修复之类的和本文无关，所以不写了:- )。这个方法对`2.0.6.23`的以上版本无效的说。

最后我想对所以的新们说的：

很多初学者喜欢崇洋眉外，说什么国内的文章写没有国外的好之类的，国内的文章多数写`How to do it`，而不是`why?`多数写怎么做，而不是说为什么要这么做。对此我说一下自己的意见，如果你连尖字都不会写，连汉语都不会写，你会知道尖字为什么是这样写吗?你会理解汉字中的深澳吗?如果连`mov eax,edx`之类的都不知道，你会知道`why?`吗，于其去问点解倒不如去做。

Greetz:

Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!

By loveboom[DFCG][FCG][US]

Email:loveboom#163.com

http://bulo.163.com/-BPEl.html

Date:4/9/2005 12:56:00 PM

4dd466d0e1e0cc123afbe504da2462ca
