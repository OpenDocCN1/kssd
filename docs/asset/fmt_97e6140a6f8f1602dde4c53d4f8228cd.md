
# ODbgScript 入门系列(四) ODbgScript 的 注意事项和使用技巧

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-610.htm`](https://www.pediy.com/kssd/pediy08/pediy8-610.htm)

ODbgScript 入门系列（四） ODbgScript 的 注意事项和新手使用技巧

ODbgScript 还在不断的完善中,它是一个脚本解释器,那么它需要把用户的脚本命令一一解释出来并执行,你必须严格复合它的命令规范它才能解释你的命令.

ODbgScript 程序是这样定义的:

命令 目的操作数,源操作数 \[,附加操作数\]

命令:必须是 ODbgScript 能识别的命令,我前 3 章已经讲解了,不过今后命令不断的扩展中

操作数:

- HEX : 既没有前缀也没有后缀
- DEC : 在后缀中加点.
- VAR : 变量，这个变量必须在使用前用 `Var` 进行定义
- 32REG: 32 位寄存器 (`EAX`, `EBX`, `ECX`, `EDX`, `ESI`, `EDI`, `EBP`, `ESP`, `EIP`)
- 16REG: 16 位寄存器 (`AX`, `BX`, `CX`, `DX`, `SI`, `DI`, `BP`, `SP`)
- 8REG: 8 位的寄存器(`AL`, `AH`, `BH`,`BL`,`CH`,`CL`, `DL`, `DH`)
- \[ \] : 被中括号括起来的内存地址 而\[\]中可以是数值,寄存器,变量
- FLG :一个标志位，带有感叹号前缀(`!CF`, `!PF`, `!AF`, `!ZF`, `!SF`, `!DF`, `!OF`)
- ## :HEX 的数据系列
- "" :字符串系列
- ?? :包含??的 HEX 系列或者字符串系列,代表的是可以代表任意数
- {} :过程计算符,中间可以放的是变量
- : :标号,跟在你命令的名字后面,代表对程序分段

本来 1,47 版可以支持 `+-*/&|^><` 的操作数,这个纯粹是 C 爱好者的一相情愿,也让脚本变得可读性和通用性大大下降,我准备在后续版本中拿掉这个功能.看大家的意愿吧.

这里以看雪的<脱壳基础知识入门（2006 年版）>为蓝本来讲解,这样便于新手对照来学习:

<第一课 PE 格式章节,第三课 认识壳章节>

我们知道一般能玩 PE 文件的大部分是用正常的编程语言才能取 PE 的东西,我们用脚本来完整的实现一次怎么知道 PE 信息.

(GMI 将在下一个版本发布中增加大量的可获取 PE 信息的操作数,基本可以囊括整个 PE 结构的基本信息.这个命令不光查 `exe` 的信息,同样可以查 `DLL` 的详细见发布版 `readme`)

### 脚本一:取得有用的 PE 信息

```
//===============================================
var modulebase
var DOSstub
var PEheard
var codebase
var codesize
var ImportAddressTable
var IATsize
var temp
var tmp
var OrignalFirstThunk
var TimeDateStamp
var ForwardChain
var Allname
var FirstName
var FirstThunk
var Namesize

//获得基本地址
getbaseinfo: 
gmi eip,MODULEBASE
mov modulebase,$RESULT
gmi eip,CODEBASE
mov codebase,$RESULT
gmi eip,CODESIZE
mov codesize,$RESULT

//寻找 DOS 头
mov DOSstub,\[modulebase\],2    //只取 2 个字节,这个小技巧很多大侠都不知道 :)
cmp DOSstub,5a4d                //"MZ"
jne end

//寻找 PE 头地址
find modulebase,#50450000#      // "PE"
mov PEheard,$RESULT

//寻找 IAT 相关数据
mov temp,PEheard
add temp,80                     //PE 文件头偏移 80 处放的就是 IAT
mov temp,\[temp\]
add temp,modulebase
mov ImportAddressTable,temp     //取得 IAT 的地址
mov temp,PEheard
add temp,84                     //PE 文件头偏移 84 处放的就是 IAT 大小
mov temp,\[temp\]
mov IATsize,temp                //取得 IAT 大小

//你将看见第一个 DLL 的 API
mov temp,\[ImportAddressTable+C\]
add temp,modulebase
find temp,#0000#
sub $RESULT,temp
mov Namesize,$RESULT            //求得 ALlName 的大小
readstr \[temp\],Namesize       //读出 ALLName
mov AllName,$RESULT

find temp,#00#
sub $RESULT,temp
mov Namesize,$RESULT            //求得第一个 Name 的大小
readstr \[temp\],Namesize       //读出 Name
mov Name,$RESULT                
mov temp,\[ImportAddressTable+10\]
mov FirstThunk,\[temp+modulebase\]  //求出 FirstThunk
gn FirstThunk                   //查询 FirstThunk 的符号名
mov tmp,$RESULT

//在调试文件所在目录里向 PE.txt 文件写信息
eval "modulebase={modulebase}"
wrt "PE.txt",$RESULT
eval "codebase={codebase}"
wrta "PE.txt",$RESULT
eval "codesize={codesize}"
wrta "PE.txt",$RESULT
eval "ImportAddressTable={ImportAddressTable}"
wrta "PE.txt",$RESULT
eval "IATsize={IATsize}"
wrta "PE.txt",$RESULT
eval "AllName={AllName}"
wrta "PE.txt",$RESULT
eval "Name={Name}"
wrta "PE.txt",$RESULT
eval "FirstThunk={FirstThunk}={tmp}"
wrta "PE.txt",$RESULT

end:
ret
//==================================================================
```

呵呵,我写 `VC` 同样的程序还没有它清晰好读,到处是指针.你看这个多清爽!!!

(注意:这个版本里面加了命令 `readstr`,请下载最新 `ODbgScript`)

### <第二课 SEH 技术>SEH 脚本规避技术

其实,SEH 挺死的,它的结构和原理在看雪已经被各位大侠研究透了,核心关键是抓住 SEH 的就是异常处理的句柄,它让异常的程序能回归进入正常的流程.我这里要强调一点,请小心使用你的硬件断点.至于为什么你去看 SEH 原理.所以请大家都检查一下 `OD`,看选项-调试一栏中,不要钩上硬件断点,否则你每次 `F7` 和 `F8` 都是下的硬件断点.硬件断点最好在需要的时候再下.

### <第六课 寻找 OEP>

(1)根据跨段指令寻找 OEP 和根据堆栈平衡原理找 OEP

`ESP` 定律的发现和不断的完善是脱壳界一个重大进步,我们知道,壳无论怎么玩都是要交回原运行环境给程序,这样就暴露出它的软肋,我们定位好堆栈的数据,只要某段时刻一旦两个环境重合,就说明壳转了一圈回到了初始的环境,即便它不马上跳去 `OEP`.但是,从堆栈平衡的原理来说它完成了它的一个轮回.原理上来说,只要我们能熟练的把握好<堆栈平衡原理>,那么你就能够在各种壳的风口浪尖上不走迷失了方向.我个人也认为<堆栈平衡原理>是对付壳的一个重大武器,我们可以不断的在这个基础上进行深化和突破.

我们常常遇见 `ESP` 定理"失效"的时候,其实,它并没有失效,而是你不要把 `ESP` 定理机械的理解为就是 `OEP` 处,,就像我们知道的两点决定一线一样简单的定律,你只是简单的想成一根线,那么,你还能对付考大学路上的题海么? :)

真正掌握 `ESP` 定律,真正吸取 `ESP` 的精髓,壳等待你的就是你无比尖锐的利剑...

我曾经用 `ESP` 原理破掉了 90%的壳,一个很简单的方法,我写过一个程序,就死死盯住原始环境,你走多少次我记录多少次,然后把纪录一看再作些工作统计一下,`OEP` 绝大部分原形毕露...方法笨是笨点,但找 `OEP` 准确率太高了.

这里是一个简单的 `ESP` 定律脚本,主要是对付一些简单的压缩壳和保护壳,毕竟是教学,就不把它写复杂了.

这里用了几个小技巧就是靠单步来寻找 `pushad`,并且避开当前就是 `pushad`.并且还原堆栈以前的状态.

由于是脚本直接用指令改了 `EIP`,所以,屏幕并没有刷屏,大家要双击一下 `EIP` 就是可以看见堆栈还原后的状态了.

#### 脚本三:ESP 定律

```
//=========================================================================
Start:
msgyn "你是选择 pushad/popad 方式吗(是)?还是狭义 ESP 方式(否)"
cmp $RESULT,1
jne esp
mov tmp,\[eip\],1
cmp tmp,60                  //比较当前指令是否是 pushad(60)如果是转 ESP 处理方式
je esp
findpushad:                 //如果不是就一直找到 pushad(60)的地址
find eip,#60#
cmp $RESULT,0
je findtmp
mov tmp,\[eip\],1
cmp tmp,60
jne findok                 

sti  
jmp esp

findtmp:
sti
jmp findpushad

findok:                       
bp $RESULT
esto
bc $RESULT

esp:
mov tmp,esp
sub tmp,4
sti
bphws tmp,"w"
esto
bphwcall
preop eip                   //向上还原堆栈状态.
mov oep,$RESULT              
add esp,4
mov eip,oep
msg "你已经到达 ESP 平衡处"
end:
ret         // 结束脚本  
//==============================================================
```

(3)根据编译语言特点找 OEP

这个没有什么可教学的,就是需要你平时在计算机里面搜集一下各种没有加过壳的小程序,建立常规程序的搜集库,这些小文件可以很好的教你如何识别 `OEP` 的规律,各类语言编译的文件入口点都有一些规律的,在于你是否熟悉它,千变万化不离其宗.重要的是现在壳开始大规模偷 `OEP` 的头了,这样就很难用一般方法来简单定位 `OEP` 了,偷归偷,可环境它不能乱来,它只能来仿真以前的语句来实现被偷 `OEP` 的运行模式,壳现在的确还没有达到 `IA` 高度智能化的境界.比如我们可以知道 `VC5-6` 有一些版本的首个 `API` 就是调用 `GetVersion` 这个函数,可现在的壳大部分是把这个 `call` 给偷的没有影了,怎么办???

呵呵,小猫腻,我们用脚本来一句就可以定位它.

```
tocnd "eax == 0A280105"
```

只要是 `VC5-6` 某些带 `GetVision` 的版本的程序,它一定要返回固定的版本号(只要你机器装好了 `XP` 操作系统这个 `0A280105` 就一定是它,是 `2000` 的话你自己查一下),机器是傻的人是活的,包裹的再好你也要漏一丝缝,`Themida` 就是被我用这个简单的方法在 `GetVersion` 这个 `call` 中锁的死死的,不管你走多少步,我对照寄存器环境的变换,就可以锁死你的 `VM` 在干什么!实在懒得在 `IDA` 中分析了,就这个方法破花指令和 `VM` 是特别有效的.

(小提示:这些 `VC5-VC6` 的程序,你只要 `OD` 能完整的跑一次,就可以用我这个方法跑到 `GetVersion` 这个 `CALL` 上不管它怎么偷,当然你的 `CPU` 要够强劲,剩下的事情你可以去看连续剧了,慢是慢点,剩心剩事,顺便还可以把 `oep` 被偷的头记录的完完整整)

`4dd466d0e1e0cc123afbe504da2462ca`
