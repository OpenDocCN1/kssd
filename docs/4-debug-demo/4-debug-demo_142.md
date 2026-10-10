

# 谈【注册机】的失效及解决方案

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-204.htm`](https://www.pediy.com/kssd/pediy09/pediy09-204.htm)

> 【文章标题】: 谈【注册机】的失效及解决方案  
> 【文章作者】: 啊 CR/FTSTT  
> 【作者主页】: Viper.68ab.com  
> 【软件名称】: 某 keygen  
> 【下载地址】: 自己搜索下载  
> 【作者声明】: 偶是小鸟一只，希望大家照顾^\_^  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   大家都知道注册机会失效，本文就此引出。  
>     
>   算法大牛、分析高手们就不要看了/lianh  
>     
>     
>   注册机为什么会失效，原因有三点：  
>   1，算法改变  
>   2，常数改变  
>   3，针对性的 ban  
>     
>     
>   我们不是 注册机 的作者，该怎么办呢？  
>     
>   如果是 算法改变 的话，失效的注册机就没什么用处了，如果有能力去修改大量的汇编代码不如去重写一个新的。  
>     
>   如果是 常数改变\[通常是版本号\] ，这个我们就有一定的发挥了，我们可以去修改常数，使其生成正确的注册码【例一】。  
>     
>   如果是 针对性的 ban ，又分两种情况，  
>                     1 注册机作者在注册码中保存了签名  
>                     2 注册机作者使用了多组有效密匙中的一个，作者把这组密匙列入了黑名单  
>     
>   其中，对于保存签名被 ban 的情况，我们可以尝试修改这个签名，使目标程序无法识别注册机就可以注册成功【例二】。  
>     
>   另一种情况，如果我们有能力找到另一组密匙，我们就可以简单的替换密匙，是注册机有效。难度不小【例三】。  
>     
>     
>   上面所说的都有例外，无法一一列举。  
>     
>     
>   下面是三个例子。  
>     
>     
>   【例一】  
>     
>   目标：  
>     
>   HWiNFO32 v1.72  Incl KeyfileMaker      
>     
>   失效原因：版本改变  
>   解决方法：替换常数  
>     
>   下载了最新的 1.75，发现没有注册的地方，仔细一看，原来是 Keyfile 汗~！  
>     
>   复制 KeyfileMaker 但程序目录注册后发现程序提示版本不正确，使用 OD 跟踪一下，看看是怎么判断版本的。  
>     
>   程序加未知壳，不管它，直接载入。  
>     
>     
>   注册机生成  
>     
>   HWiNFO32.KEI 和 HWiNFO32.KEY 两个文件  
>     
>   在 OD 中下断点 Bp CreateFileA  
>     
>     
>   第一次断下就是检查 HWiNFO32.KEY，Ctrl + F9 返回，一路 F8，读取 HWiNFO32.KEY   
>     
>   然后读取 HWiNFO32.KEI 然后继续 F8  
>     
>   直到  
>   00423BA4   E8 80A10200      CALL HWiNFO32.0044DD29    黑名单  
>   00423BA9   85C0             TEST EAX,EAX  
>   00423BAB   74 11            JE SHORT HWiNFO32.00423BBE  
>   00423BAD   6A 10            PUSH 10  
>     
>   这里改为 JMP，不知为什么不跳。  
>     
>   继续  
>     
>     
>     
>   00423BE2   56               PUSH ESI  
>   00423BE3   E8 F63F0400      CALL HWiNFO32.00467BDE                   ; JMP 到 VERSION.GetFileVersionInfoA  
>   00423BE8   85C0             TEST EAX,EAX  
>   00423BEA   74 44            JE SHORT HWiNFO32.00423C30  
>   00423BEC   8D45 E0          LEA EAX,DWORD PTR SS:\[EBP-20\]  
>   00423BEF   50               PUSH EAX  
>   00423BF0   8D45 D0          LEA EAX,DWORD PTR SS:\[EBP-30\]  
>   00423BF3   50               PUSH EAX  
>   00423BF4   68 BCD54C00      PUSH HWiNFO32.004CD5BC                   ; ASCII "\\StringFileInfo\\040904B0\\FileVersion"  
>   00423BF9   57               PUSH EDI  
>   00423BFA   E8 D93F0400      CALL HWiNFO32.00467BD8                   ; JMP 到 VERSION.VerQueryValueA  
>   00423BFF   85C0             TEST EAX,EAX  
>   00423C01   74 2D            JE SHORT HWiNFO32.00423C30  
>   00423C03   8D45 E4          LEA EAX,DWORD PTR SS:\[EBP-1C\]  
>     
>   获取版本号  
>     
>   00423C0B   8D45 D4          LEA EAX,DWORD PTR SS:\[EBP-2C\]  
>   00423C0E   50               PUSH EAX  
>   00423C0F   68 B0D54C00      PUSH HWiNFO32.004CD5B0                   ; ASCII "%d,%d,%d"  
>   00423C14   FF75 D0          PUSH DWORD PTR SS:\[EBP-30\]  
>     
>   版本号的 去 格式化。  
>     
>     
>   00423C4F   8B0D 749A4F00    MOV ECX,DWORD PTR DS:\[4F9A74\]  
>   00423C55   8BF8             MOV EDI,EAX  
>   00423C57   E8 13A10200      CALL HWiNFO32.0044DD6F  
>   00423C5C   3BFB             CMP EDI,EBX  
>   00423C5E   74 04            JE SHORT HWiNFO32.00423C64  
>   00423C60   3BFE             CMP EDI,ESI  
>   00423C62   7F 08            JG SHORT HWiNFO32.00423C6C  
>   00423C64   3BC3             CMP EAX,EBX  
>   00423C66   74 16            JE SHORT HWiNFO32.00423C7E  
>   00423C68   3BC6             CMP EAX,ESI  
>   00423C6A   7D 12            JGE SHORT HWiNFO32.00423C7E  
>     
>   这里比较当前的版本号和注册文件中的版本号，要求注册文件的版本号范围包含当前版本。  
>     
>   重复前面的跟踪，发现注册文件版本从 HWiNFO32.KEI 中读出，是一个有效范围 1.7.2-1.7.2，这就是为什么注册文件在 1.7.5 中失效了。  
>     
>   关闭 OD，修改 HWiNFO32.KEI 中 1.7.2-1.7.2 为 0.0.0-9.9.9 试一下，运行程序，显示为注册成功。  
>     
>   修改注册机。  
>     
>   FSG2.0 找个工具轻松脱之，106 进制工具尝试搜索 1.7.2-1.7.2，找到了替换为 0.0.0-9.9.9，这样生成的注册文件就是有效的了。假如搜索不到，就要跟踪判断了。  
>     
>     
>     
>   ///注册机中范围设为 1.7.2-1.7.2，大概是注册机作者的策略，使软件作者在测试新版，注册机无效，免得作者更换算法，而升级注册机时更新常数即可。我是这样想的。  
>   ///此方法修改 winhex 注册机时失败了，winhex 还要另外修改其他部分对抗程序的检测。  
>     
>     
>     
>   【例二】  
>     
>   目标：  
>     
>   WinZip.Professional.v10.0.6685.Kg（注册机在 11.0 中失效，本文不讨论此问题）  
>     
>   失效原因：没有失效  
>   解决方法：仅修改签名  
>     
>     
>   注册机有一个选项 Signed by ZWT，很有意思，选中后生成的注册码   
>     
>   就是这样的  
>   TT8K2-0AU98-R8824-2811A-G56XJ-ZWTKG  
>     
>   已-ZWTKG 结尾，就是-ZWT 的注册机生成的意思，很有趣，尝试直接修改注册码中的标志，结果注册失败。  
>     
>   注册机加了壳  
>   ASProtect 2.1x SKE -> Alexey Solodovnikov  
>     
>   你可以选择脱掉，但我没脱，直解 OD 跟踪，  
>     
>   载入 OD 后 F9 运行， Alt + M  
>     
>   二进制搜索 “ZWTKG” 不含引号。  
>     
>   找到  
>     
>   0045E21D  43 68 69 6E 61 00 00 01 26 DD AB 91 5F D1 CC 01  ZWTKG..&莴 aq 烟  
>     
>   替换为  
>     
>   0045E21D  43 68 69 6E 61 00 00 01 26 DD AB 91 5F D1 CC 01  China..&莴 aq 烟  
>     
>     
>     
>   在注册机中选中 Signed by ZWT ，算个号码  
>     
>   China  
>   FK2M8-X3JNM-2DNUU-1F7Y2-RP7P6-China  
>   F52E0-U3NGP-N1W46-PV9L9-TZUMX-China  
>     
>   注册一下，不成功，如果要注册成功还要修改一些其他内容，本文只为说明说这种方法。  
>     
>   ///不成功的案例，但很多时候是可以成功的  
>   ///其他内容是什么呢？不要问我，我也不知道:-)  
>     
>     
>   【例三】  
>     
>   目标：  
>     
>   Flash2X Flash Hunter v1.2.0.100503's keygen   ORiON  
>     
>     
>   失效原因：密匙改变  
>   解决方法：替换密匙  
>     
>     
>   注册机和软件的版本号完全一致，却注册失败，这就是软件的所谓再版。  
>     
>   通常，我在遇到这情况是都是先跟一遍注册机的流程，这样再跟软件会节省不少时间。  
>     
>   注册机加壳  
>   UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo  
>   UPX -d 轻松脱去  
>   载入 OD，F9 运行。  
>     
>   然后 右键 查找——所有参考文本字串，然后在每个命令上设置断点。（这样最方便）  
>     
>     
>   点击任务栏的软件，把遇到的断点全部取消。  
>     
>   输入注册名，程序会自动算出注册码。  
>     
>   00401D0E  |. BF C8CC4000    MOV EDI,Keygen-F.0040CCC8  
>     
>   这里停下，然后你可以 F8，大致的看一下。你会发现注册机在调用 40C0FC 位置的一个字串表计算注册码，这就足够了，退出程序。  
>     
>   .0040C0FC ASCII "2jKc7FqMBy43IXstGJDPlaQVT6LWrAzS9Yd1vkUN0ORowxnfm5EZi8hCbHuepg"  
>     
>     
>   用 OD 载入目标程序，不必跟踪，右键 查找——所有参考文本字串，慢慢上翻，  
>     
>   文本字串参考位于 Flash\_Hu:CODE, 条目 4198  
>    地址=00492EF4  
>    反汇编=MOV EDX,Flash\_Hu.004931A0  
>    文本字串=ASCII "T1GECyYnpA6SPjxdmWVs2czoIUD8tOwJlKNXeBufrZH7Lgi0h3q4M59kbRvFQa"  
>     
>   文本字串参考位于 Flash\_Hu:CODE, 条目 4202  
>    地址=00492F66  
>    反汇编=MOV EDX,Flash\_Hu.004931F4  
>    文本字串=ASCII "RegName"  
>     
>     
>   00492EF4 位置的字串和注册机中很像啊，下面还有字串 RegName，替换一下试试。  
>     
>   用十六进制编辑器搜索注册机中的  
>     
>   2jKc7FqMBy43IXstGJDPlaQVT6LWrAzS9Yd1vkUN0ORowxnfm5EZi8hCbHuepg  
>     
>   替换为  
>   程序中的  
>     
>   T1GECyYnpA6SPjxdmWVs2czoIUD8tOwJlKNXeBufrZH7Lgi0h3q4M59kbRvFQa  
>     
>   保存在运行注册机，算个注册码，打开目标程序，注册成功！  
>     
>     
>     
>   ////这个例子比较特殊，也可以放到常数那类，由于程序是主观再版，索性发到密匙这类。  
>     
>   ////此例说明某些情况下替换被 ban 的密匙是可以完成注册的。  
>     
>     
>     
> \--------------------------------------------------------------------------------  
>   
>                                                        2007 年 07 月 15 日 2:42:57 

 4dd466d0e1e0cc123afbe504da2462ca