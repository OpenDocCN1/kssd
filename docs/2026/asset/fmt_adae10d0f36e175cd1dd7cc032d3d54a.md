
# [原创]说说VA_X的补丁方法

【文章标题】: 说说VA_X的补丁方法
【文章作者】: yangjt
【作者邮箱】: yangjietao123@163.com
【作者QQ号】: 325002492
【软件名称】: VA_X
【加壳方式】: Armadillo V5.00-V5.X Dll -> Silicon Realms Toolworks
【操作平台】: Win XP sp3
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

总体来说补丁方法可以分为：

1. 直接打补丁（即Code Injection，好像是给Arm释放的那个`ArmAccess.DLL` Inline补丁，我还没太明白，就不赘述了^\_^）
2. 脱壳后打补丁（速度啊……）

从打的补丁的内容分可以分为三类，以VAX10.5.1709.0为例

载入后入口如下

```
  1F256C0A >/$  837C24 08 01  [color=#0000FF]cmp[/color]     [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+8], 1
  1F256C0F  |.  75 05         [color=#0000FF]jnz[/color]     [color=#FF0000]short[/color] 1F256C16
  1F256C11  |.  E8 CA4A0000   [color=#0000FF]call[/color]    1F25B6E0
  1F256C16  |>  FF7424 04     [color=#0000FF]push[/color]    [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+4]
  1F256C1A  |.  8B4C24 10     [color=#0000FF]mov[/color]     [color=#808000]ecx[/color], [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+10]
  1F256C1E  |.  8B5424 0C     [color=#0000FF]mov[/color]     [color=#808000]edx[/color], [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+C]
  1F256C22  |.  E8 EDFEFFFF   [color=#0000FF]call[/color]    1F256B14
  1F256C27  |.  59            [color=#0000FF]pop[/color]     [color=#808000]ecx[/color]
  1F256C28  \.  C2 0C00       [color=#0000FF]retn[/color]    0C
```

为了图方便，这里不详述脱壳步骤，用我提供的脚本就好……第一次写脚本……所以只能帮助大家走到`MagicJump`后啦~~

```
  00CB865A   /EB 03           [color=#0000FF]jmp[/color]     [color=#FF0000]short[/color] 00CB865F
  00CB865C   |D6              [color=#0000FF]salc[/color]
  00CB865D   |D6              [color=#0000FF]salc[/color]
  00CB865E   |8F              ???                                      [color=#008000]; ????[/color]
  00CB865F   \8B15 8C4CD100   [color=#0000FF]mov[/color]     [color=#808000]edx[/color], [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [D14C8C]
  00CB8665    8995 B4FDFFFF   [color=#0000FF]mov[/color]     [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]ebp[/color]-24C], [color=#808000]edx[/color]
```

应该会停在这里，然后Alt+M在这里

![](https://bbs.kanxue.com/upload/attach/200812/122145_fptm8hvhvhzm6mg.jpg)

然后就到了OEP，Dump后修复，就该打补丁了……

![](https://bbs.kanxue.com/upload/attach/200812/122145_gf1wpizp2oojpmz.jpg)

### 第一种CCDebuger的方法

好像原理是把监测是否注册那段不让它运行，然后把`eax`改成`TRUE`

先搜索字符串`VAX:IDE Main Thread`

找到以后往下拉，然后把找到的地方如下改好

```
  1EDA8119  |.  68 00000100   [color=#0000FF]push[/color]    10000                            [color=#008000];  UNICODE "=::=::\"[/color]
  1EDA811E  |.  E8 4FB41900   [color=#0000FF]call[/color]    1EF43572
  1EDA8123  |.  83C4 08       [color=#0000FF]add[/color]     [color=#808000]esp[/color], 8
  1EDA8126  |>  E8 A5CAF7FF   [color=#0000FF]call[/color]    1ED24BD0
  1EDA812B  |.  50            [color=#0000FF]push[/color]    [color=#808000]eax[/color]
  1EDA812C  |.  8D4C24 1C     [color=#0000FF]lea[/color]     [color=#808000]ecx[/color], [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+1C]
  1EDA8130  |.  E8 7B68F6FF   [color=#0000FF]call[/color]    1ED0E9B0
  1EDA8135  |.  89BC24 CC0100>[color=#0000FF]mov[/color]     [color=#FF0000]dword[/color] [color=#FF0000]ptr[/color] [[color=#808000]esp[/color]+1CC], [color=#808000]edi[/color]
  1EDA813C  |.  E8 AF29FAFF   [color=#0000FF]call[/color]    1ED4AAF0               //[color=#0000FF]这个地方改成mov[/color] [color=#808000]eax[/color],1
  1EDA8141  |.  85C0          [color=#0000FF]test[/color]    [color=#808000]eax[/color], [color=#808000]eax[/color]
  1EDA8143  |.  0F84 71020000 [color=#0000FF]je[/color]      1EDA83BA
```

然后接着搜索`VAX:ArmThread`

处理成如下这个样

```
  1ED82201   .  68 9C86FD1E   [color=#0000FF]push[/color]    1EFD869C                         [color=#008000];  ASCII "VAX:ArmThread"[/color]
  1ED82206   .  8BF1          [color=#0000FF]mov[/color]     [color=#808000]esi[/color], [color=#808000]ecx[/color]
  1ED82208   .  E8 4304FAFF   [color=#0000FF]call[/color]    1ED22650
  1ED8220D   .  83C4 04       [color=#0000FF]add[/color]     [color=#808000]esp[/color], 4
  1ED82210   .  E8 BB3A1000   [color=#0000FF]call[/color]    1EE85CD0
  1ED82215      85C0          [color=#0000FF]test[/color]    [color=#808000]eax[/color], [color=#808000]eax[/color]         //[color=#0000FF]这里改成inc[/color] [color=#808000]eax[/color]
```

如此便完成了第一种方法……保存以后拍屁股走人

### 第二种是BRD的方法

据说可以跳过暗桩，实际上就是无限延长了试用期限

具体操作见我跟踪BRD破解文件的笔记
