
```asm
        db        'INT 0ch',0

        db        'INT 0dh',0

        db        'INT 0eh',0

        db        'INT 0fh',0

        db        'INT 10h',0

        db        'INT 11h',0

        db        'INT 12h',0

        db        'INT 13h',0

        db        'INT 14h',0

        db        'INT 15h',0

        db            'INT 16h',0

        db        'INT 17h',0

        db        'INT 18h',0

        db        'INT 19h',0
```

;----------------------------------------------------------------

;----------------------------- 主程序 ----------------------------

```asm
begin:

        pushf

        pusha

        ; 显示字符串

        call        wcrlf

        mov        si,msgShow

        mov        bx,810eh

        call        write

        mov        ah,20

        call        delay

        ; 显示IVT

        call        wcrlf

        call        wmem

        call        wcrlf

        mov        ah,50

        call        delay

        popa

        popf

        retf
```

;----------------------------- 主程序结束 --------------------------

```asm
times        1024-($-$$) db 0
```

;---------------------------------------------------------------------

下面我们用 ShowIVT 来测试一下。为了检测ShowIVT的运行情况，我们使用可运行的BIOS Rootkit：IceLord来进行测试。为了实验可行性，我们在Bochs虚拟机上进行实验。

首先进行未加载IceLord的实验，`.bxrc`配置文件核心内容如下：

#-------------------------------------------------------------------

加载IceLord前的Bochs配置信息

#-------------------------------------------------------------------

# filename of ROM images

```ini
romimage: file=../BIOS-bochs-latest

optromimage1: file=showivt.bin, address=0xd0000

vgaromimage: file=../VGABIOS-lgpl-latest
```

#-------------------------------------------------------------------

加载ShowIVT检测程序，查看原始IVT数据。Bochs运行后，效果如下：

![](https://bbs.kanxue.com/upload/attach/200812/46815_1ziu3htf3bq041k.png)

可以看到，将被此时的int 19h的原始中断向量值为：

-------------------------------------------------------------------

加载IceLord前的结果显示

-------------------------------------------------------------------

INT 13H = F000:E3FE

INT 19H = F000:E6F2

-------------------------------------------------------------------

接下来，我们修改`.bxrc`配置文件，这次同时加载IceLord的BIOS核心模块`leaving.bin`，还有检测模块ShowIVT。修改内容如下：

#-------------------------------------------------------------------

加载IceLord后的Bochs配置信息

#-------------------------------------------------------------------

# filename of ROM images

```ini
romimage: file=../BIOS-bochs-latest

optromimage1: file=leaving.bin, address=0xd0000

optromimage2: file=showivt.bin, address=0xd8000

vgaromimage: file=../VGABIOS-lgpl-latest
```

#-------------------------------------------------------------------

这里使用两个`optromimage`选项，加载两个ISA模块。`leaving.bin`为IceLord的ISA模块，`showivt.bin`为我们的检测模块。在3.3.2节，我们分析过`leaving.bin`。它控制CPU流程的核心技术就是Hook IVT。`leaving.bin`通过hook int 19h来取得系统的控制权。程序运行后，BIOS会首先加载`leaving.bin`运行，然后加载检测程序`showivt.bin`。这样，在我们的检测结果中就能体现出被`leaving.bin`修改过的IVT的情况。Bochs运行后的情况如下：

![](https://bbs.kanxue.com/upload/attach/200812/46815_v9gxfb5a3dgfko0.png)

这时的int 13h和int 19h的中断向量值为：

-------------------------------------------------------------------

加载IceLord后的结果显示

-------------------------------------------------------------------

INT 19H = 97C0:00A2

-------------------------------------------------------------------

综合比较加载BIOS Rootkit核心文件`leaving.bin`前后的结果如下：

-------------------------------------------------------------------

结果比较

-------------------------------------------------------------------

加载`leaving.bin`之前        INT 19H = F000:E6F2

加载`leaving.bin`之后        INT 19H = 97C0:00A2

-------------------------------------------------------------------

从比较结果我们可以看出，int 19h的中断向量已被篡改。`leaving.bin`对int 19h中断进行hook，使其中断向量指向自定义程序所在地址97C0:00A2，即物理内存地址0x00097ca2处。这和我们在前两篇文章中调试的结果完全相符。

到这里，大家都会发现：ShowIVT 这个 IVT Hook 检测程序，仅仅是如实的把IVT的数据进行简单显示，甚至都没有加入字符串比较的功能。因此，只是一种检测思路的验证程序。如果你有兴趣在此基础上进行了扩充增强，请把你的代码发一份给我，谢谢。^\_^

针对 ShowIVT 程序的检测思路，如果在Hook了IVT后，进行原始数据的写回，是不是可以欺骗过 ShowIVT呢。答案是：可以也不可以。可以，是指大部分的Hook IVT操作后，都可以进行原始数据的写回，比如这样操作：

;------------------------------------------------------------------
```asm
                pushf

        pusha

        mov        ax,0

        mov        es,ax

        mov        si,19h*4

        mov        di,old_int19h

        mov        cx,[es:si]

        mov        [es:di],cx

        mov        dx,[es:si+2]

        mov        [es:di+2],dx                ; 保存

        mov        [es:di-1],byte 0eah

        mov        [es:si],word OurInt19h

        mov        [es:si+2],cs                ; hook

        mov        cx,[es:di]

        mov        [es:si],cx

        mov        dx,[es:di+2]

        mov        [es:si+2],dx                ; 恢复

        popa

        popf

        retf
```
;------------------------------------------------------------------

看来，我们的 ShowIVT 是名不副实了。:p

刚才说可以也不可以。不可以，指的就是，并不是所有的 IVT 向量被 Hook 后，都能进行原始数据写回操作的。比如：int 19h。

如果这个中断向量也写回的话，那整个BIOS Rootkit会无法正常执行，大家可以想想原因。这样看来，我们的 ShowIVT 还是有点用。^\_^

### 小结

这一小节是IVT Hook专题的最后一篇。在前两篇的基础上，我们设计了一个自己的显示IVT数值的程序，通过和原始IVT数值进行比对，以判断是否存在BIOS Rootkit。当然 ShowIVT 只是一个概念性验证程序，功能还很弱，在这里只是抛砖引玉，希望有兴趣的朋友继续完善。而突破 ShowIVT 的检测并不是不可能，这个问题我们放到下一篇文章中进行讨论。感谢大家对这个系列文章的关注。:)

ppanger 2008.12.18

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [check\_ivt\_1.png](javascript:void(0);)

  （26.96kb，691次下载）

- [check\_ivt\_2.png](javascript:void(0);)

  （30.08kb，690次下载）

- [check\_ivt\_3.png](javascript:void(0);)

  （29.85kb，690次下载）
