
# [原创]Smart PC Suite算法分析+dll类型破解通用方法+源码

【破文标题】Smart PC Suite算法分析+dll类型破解通用方法+源码

【破文作者】Playboysen

【作者邮箱】playboysen@126.com

【破解工具】`PEiD`,`OD`

【破解平台】Windows XP

【软件名称】Smart PC Suite

【软件大小】10 MB

【软件授权】共享版($59.95)

【软件语言】英文

【原版下载】647K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4E0j5i4u0@1M7r3y4@1L8$3!0D9M7#2)9J5k6h3y4G2L8g2)9J5c8Y4m8J5L8$3c8#2j5%4c8K6i4K6u0r3

【保护方式】用户名、注册码

【软件简介】使您的电脑运行速度更快，保护您的安全和恢复丢失的数据的唯一软件套件您最需要的！修复您的计算机自己并使其加载和运行速度更快。清理注册表错误和照顾病毒和间谍软件，确保您的个人隐私，并恢复已删除的档案

【破解声明】一点心得，愿与大家分享o(∩\_∩)o 版权所有，转载注明作者！

## 破解内容

自从上次写完“MP3 WAV OGG WMA AC3 to CD Burner算法分析+简单逆向汇编”至今已经快两个月了，临近毕业，每天为生计奔波，找份工作好难，前途未卜，也没心情搞分析，不过偶尔练习一下，不致手生而已，一点心得，与君分享。
今天我们来研究一个国外公司的几款商业是软件，入手点是`Smart PC Suite`(我专门挑选的售价较贵的o(∩\_∩)o)，总的感觉，这个公司的程序员很傻很天真，看完此文章你就明白我的话了~~~
主程序查壳`Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks`，用`Arma Find Protected`检测

```
<------- 05-11-2008 06:56:49 ------->

C:\Documents and Settings\Administrator\桌面\{app}\SmartPCSuite.exe

!- Protected Armadillo

Protection system (Basic)

!- <Protection Options>

Standard protection or Minimum protection

!- <Backup Key Options>

Fixed Backup Keys

!- <Compression Options>

Better/Slower Compression

!- <Other Options>

!- Version 4.48 14August2006
```

脱壳So easy，不多说，脚本搞定（`Armadillo.V4.0-V5.X.eXe.Standard.Protection.By.fly[CUG].oSc`），不过要提醒大家，虽然是标准壳但是不要用脱壳机，因为此程序加了双层壳，脱壳机太傻，搞不定的

```
013FE001    60              pushad                      ; This is the OEP!  Found By: fly[CUG]

013FE002    E8 03000000     call    013FE00A            ；看到没？脚本停在这里，Aspack，第二层             

013FE007  - E9 EB045D45     jmp     469CE4F7

013FE00C    55              push    ebp

013FE00D    C3              retn

013FE00E    E8 01000000     call    013FE014

013FE013    EB 5D           jmp     short 013FE072

013FE015    BB EDFFFFFF     mov     ebx, -13
```

ESP定律后dump，修复搞定，掠过~~~
试运行注册有错误提示，`OD`加载脱壳程序搜索相关字符到这里
