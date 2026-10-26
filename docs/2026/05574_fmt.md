
# 重建重定位表脚本

☆☆☆☆☆重建重定位表脚本--Reloc Mission☆☆☆☆☆

由于看了 fly 的 `OD` 脱 `upx` `dll` 文章心里还是一直不爽--用不了◇`UPXAngela.exe`◇，没有`天使.exe`。为何重定位表非要用看雪老师的宝贝`UPXAngela.exe`，偶就不能自己处理实现吗？又是重定位认识一知不解，动以何功，于是到处翻资料，终于找到琢磨琢磨，再琢磨……，弄明白了`PE`装载器对重定位表的要求，于是手起键落，可是老毛病还没改掉，写得慢呀，测试了也提烦，终于不负自己写下了这个脚本。

如果重定位表被“干掉”了，那么在它解密实现重定位时，应该可以使用此脚本为你的`PE`文件（多数是`DLL`吧）重建一份重定位表。我相信它有通用性，因为重定位指针是不可能不出现的（除非它不进行重定位），但未经怎么测试，不知效果如何……放出来让大家调戏它，看它是不是“阿茂整饼”^o^！欢迎批评指正，给出意见！

**使用环境：**`OllyDbg1.10`，`OllyScript 0.92`，`OD`运行平台，重定位表没有存在内存

**使用说明：**基本上我都把注释写清楚了，而且使用中文设计，没理由你不明白，除非你根本不知重定位为什么！不按要求输入，造成脚本无法使用不关我事。另外，`ask`命令的部分，你完全可以省去，将它下面的`mov`语句的`$RESULT`改为你对应的输入值即可。如果重定位指针指令不只1处重定位，那么此脚本要修改了，我想这样的可能性不大吧，因为要浪费代码空间！另外，本脚本有比较多的出错考虑，所以你如果熟悉重定位表实现过程，就可省省省……

```
var P+

var VA

var RVA

var mRVA

var pRVA

var RAT

var SOB

var bPoint

var bound

var START

var SIZE

var ImageBase

var endaddr

ask "请输入现在调试进程的基址："

cmp $RESULT,0

je non\_error

mov ImageBase,$RESULT

ask "请输入自定义重建重定位表起始地址："  //可以是00空间地址，也可以是垃圾空间地址

cmp $RESULT,0

je non\_error

mov START,$RESULT

mov P+,START

mov VA,P+

ask "请输入设定检测重定位表边界地址："

cmp $RESULT,0

je non\_error

mov bound,$RESULT //设定重定位表在内存的边界--大于起始地址，检测是否超范围（一般我们不知重建需要多少空间），我设计成结束后才检测而不是实时，不然要死脑细胞-\_-

cmp bound,START

jb enter\_error

cmp bound,80000000

jae enter\_error

ask "请输入重定位指针的中断指令地址："

cmp $RESULT,0

je non\_error

mov bPoint,$RESULT

bp bPoint  //upx壳是这么一条指令：mov dword ds:[ebx],eax

ask "请输入重定位处理结束的指令地址："

cmp $RESULT,0

je non\_error

mov endaddr,$RESULT

bp endaddr  //upx壳是 POPAD 指令

msgyn "你的输入已经完成，现在继续运行点按 'Y'，否则点按 'N'？Y/N"

cmp $RESULT,0

jne pass

msg "脚本已经暂停，点 Resume 继续运行"

pause

pass:

eob break

eoe exception

run

break:

cmp endaddr,eip

je end

cmp bPoint,eip

je hRVA

jmp berror

hRVA:     //处理VA，SOB

mov mRVA,ebx    //☆☆☆映像区块重定位指针地址，需手工设置，一般是寄存器，比如：upx壳是ebx☆☆☆

and mRVA,0FFFF000

sub mRVA,ImageBase  //计算VA块RVA地址

cmp mRVA,[VA]    //是否还是同一VA块

je hPoint

mov VA,P+

mov [VA],mRVA

add P+,4  //修正移动指针

mov SOB,P+

mov [SOB],8

add P+,4  //修正移动指针

jmp hPoint

hPoint:    //处理Point

mov pRVA,ebx //放入Point地址

and pRVA,0FFF //取低12位

add pRVA,3000 //定为：HIGLOW 类型，这里没有考虑设计 ASBSOULT 类型，俺觉得没必要复杂化！

mov [P+],pRVA

add P+,2  //修正移动指针

add [SOB],2 //实时修正SIZEOfBlock

jmp pass

exception:

msg "脚本发现不明的异常，脚本安全至上现在结束任务！"

ret

enter\_error:

msg "边界地址小于起始地址或共享地址错误！"

ret

berror:

msg "脚本发现不明的中断，脚本安全至上现在结束任务！"

ret

non\_error:

msg "你没有输入任何地址，脚本结束！"

ret

merror:

msg "检测到超出指定的边界范围，或者你需要关闭OD重来！"

jmp count

end:

cob

coe

bc bPoint

bc endaddr

mov [P+],0  //RAT最后一个VA修0作为结束！

add P+,4  //修正移动指针

cmp P+,bound  //检测边界是否合格

ja merror

count:

mov SIZE,P+

sub SIZE,START  //计算RAT的大小

mov RVA,START

sub RVA,ImageBase

msg "重定位表写结束！"

cmt eip,"具体信息参看Log窗口！"

log "下面是本脚本修复的重定位表信息："

log START

log ImageBase

log RVA

log SIZE

msgyn "你需要将重定位表另存为 C:\Reloc.bin 文件吗？Y/N"

cmp $RESULT,0

je exit

dm START,SIZE,"C:\Reloc.bin"

exit:

ret
```
