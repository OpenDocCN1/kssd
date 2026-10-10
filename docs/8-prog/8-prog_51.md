

# 编程工具之 Grep 使用介绍

> 原文：[`www.pediy.com/kssd/pediy10/62538.html`](https://www.pediy.com/kssd/pediy10/62538.html)

###########################################################  
编程工具之 Grep 使用介绍  
大家使用过 Borland 编程语言系列产品嘛。  
现在来介绍使用这个 Grep (General Regular Expression Parse)工具。  
  
它在 Borland 编程语言产品中都能见到它的踪迹。  
Grep 是个文字比对程序，在 Windows GUI 尚未流行之前它就存在了。  
如同 Unix 所提供的 grep 文字比对程式。  
  
早在 Dos 3.1 时代吧，那时我的使用方法最大的用处，  
就是在众多吓死人 include \*.h 内容中搜寻某个函数字串。  
当然也可以用来搜寻一般文数字字串的。  
  
  
使用方法::  
01.  
直接在命列行下打入 grep.exe ?  
就会列出相关使用参数

代码:

```
\[D:\\OO\\Bcb6\\Bin\]D:\\OO\\Bcb6\\Bin\\grep.exe ?
Turbo GREP 5.5 Copyright (c) 1992, 2000 Inprise Corporation
Syntax:  GREP \[-rlcnvidzuwo\] searchstring file\[s\] or @filelist

Options are one or more option characters preceded by "-", and optionally
followed by "+" (turn option on), or "-" (turn it off).  The default is "+".
   -r+  Regular expression search       -l-  File names only
   -c-  match Count only                -n-  Line numbers
   -v-  Non-matching lines only         -i-  Ignore case
   -d-  Search subdirectories           -z-  Verbose
   -e   Next argument is searchstring   -w-  Word search
   -o-  UNIX output format                   Default set: \[0-9A-Z\_\]
   -q-  Quiet: supress normal output
   -u xxx Create a copy of grep named 'xxx' with current options set as default

A regular expression is one or more occurrences of:  One or more characters
optionally enclosed in quotes.  The following symbols are treated specially:
      ^  start of line             $  end of line
      .  any character             \\ quote next character
      \*  match zero or more        +  match one or more
      \[aeiou0-9\]   match a, e, i, o, u, and 0 thru 9 ;
      \[^aeiou0-9\]  match anything but a, e, i, o, u, and 0 thru 9

```

02.  
// grep,awk 和 sed 这三个工具功能类似，是不是只要掌握一种就行了？  
// 示列搜寻\\Include\\\*.h 所有 .h 凡有此函数"CreateWindow" 文件 并显示出所在行号  
// D:\\OO\\Bcb6\\Bin\\grep.exe -n -r "CreateWindow" D:\\OO\\Bcb6\\Include\\\*.h 代码:

```
\[D:\\OO\\Bcb6\\Bin\]D:\\OO\\Bcb6\\Bin\\grep.exe -n -r "CreateWindow" D:\\OO\\Bcb6\\Include\\\*.h
File D:\\OO\\Bcb6\\Include\\commctrl.h:
6033                CreateWindow(ANIMATE\_CLASS, NULL,           \\
File D:\\OO\\Bcb6\\Include\\dispdib.h:
208         return CreateWindow(DISPLAYDIB\_WINDOW\_CLASS,"",dwStyle,0, 0,
File D:\\OO\\Bcb6\\Include\\mapinls.h:
118     #define CreateWindowA                                   CreateWindow
File D:\\OO\\Bcb6\\Include\\winuser.h:
1284    CreateWindowStationA(
1292    CreateWindowStationW(
1298    #define CreateWindowStation  CreateWindowStationW
1300    #define CreateWindowStation  CreateWindowStationA
3568     \* Special value for CreateWindow, et al.
3579    CreateWindowExA(
3595    CreateWindowExW(
3609    #define CreateWindowEx  CreateWindowExW
3611    #define CreateWindowEx  CreateWindowExA
3614    #define CreateWindowA(lpClassName, lpWindowName, dwStyle, x, y,\\
3616    CreateWindowExA(0L, lpClassName, lpWindowName, dwStyle, x, y,\\
3618    #define CreateWindowW(lpClassName, lpWindowName, dwStyle, x, y,\\
3620    CreateWindowExW(0L, lpClassName, lpWindowName, dwStyle, x, y,\\
3623    #define CreateWindow  CreateWindowW
3625    #define CreateWindow  CreateWindowA

```

#############################  
以下为几年前搜集整理相关参数解释内容(Grep 老古董工具应该不会过期的)  
Grep 命令参数解释::  
  
01. -A NUM，--after-context=NUM   
  除了列出符合行之外，并且列出后 NUM 行。  
  
  ex: $ grep -A 1 panda file   
  (从 file 中搜寻有 panda 样式的行，并显示该行的后 1 行)  
  
02. -a 或--text    
  grep 原本是搜寻文字档案，若拿二进制的档案作为搜寻的目标，  
  则会显示如下的讯息: Binary file 二进制档案名称 matches 然后结束。  
  
  若加上-a 参数则可将二进制档案视为 Text 搜寻，  
  相当于--binary-files=text 这个参数。  
  
  ex: (从二进制档案 mv 中去搜寻 panda 样式)  
  (错误!!!)  
  $ grep panda mv   
  Binary file mv matches    
  (这表示此档案有 match 之处，详见--binary-files=TYPE )  
  $  
  (正确!!!)  
  $ grep -a panda mv   
  
03. -B NUM，--before-context=NUM  
  与 -A NUM 相对，但这此参数是显示除符合行之外  
  并显示在它之前的 NUM 行。          
  
  ex: (从 file 中搜寻有 panda 样式的行，并显示该行的前 1 行)  
  $ grep -B 1 panda file   
  
04. -C \[NUM\], -NUM, --context\[=NUM\]    
  列出符合行之外并列出上下各 NUM 行，预设值是 2。  
  
  ex: (列出 file 中除包含 panda 样式的行外并列出其上下 2 行)  
  (若要改变预设值，直接改变 NUM 即可)  
  $ grep -C\[NUM\] panda file   
  
05. -b, --byte-offset  
  列出样式之前的内文总共有多少 byte ..  
  
  ex: $ grep -b panda file    
  显示结果类似于:  
  0:panda  
  66:pandahuang  
  123:panda03  
  
06. --binary-files=TYPE  
  此参数 TYPE 预设为 binary(二进制)，若以普通方式搜寻，只有 2 种结果:  
  1.若有符合的地方：显示 Binary file 二进制档案名称 matches  
  2.若没有符合的地方：什么都没有显示。  
  
  若 TYPE 为 without-match，遇到此参数，  
  grep 会认为此二进制档案没有包含任何搜寻样式，与-I 参数相同。  
  
  若 TPYE 为 text, grep 会将此二进制档案视为 text 档案，与-a 参数相同。  
  
  Warning: --binary-files=text 若输出为终端机，可能会产生一些不必要的输出。  
  
07. -c, --count  
  不显示符合样式行，只显示符合的总行数。  
  若再加上-v,--invert-match，参数显示不符合的总行数。  
  
08. -d ACTION, --directories=ACTION  
  若输入的档案是一个资料夹，使用 ACTION 去处理这个资料夹。  
  预设 ACTION 是 read(读取)，也就是说此资料夹会被视为一般的档案；  
  若 ACTION 是 skip(略过)，资料夹会被 grep 略过：  
  若 ACTION 是 recurse(递回)，grep 会去读取资料夹下所有的档案，  
  此相当于-r 参数。  
  
09.  -E, --extended-regexp  
  采用规则表示式去解释样式。  
  
10.  -e PATTERN, --regexp=PATTERN  
  把样式做为一个 partern，通常用在避免 partern 用-开始。    
  
11.  -f FILE, --file=FILE  
  事先将要搜寻的样式写入到一个档案，一行一个样式。  
  然后采用档案搜寻。  
  空的档案表示没有要搜寻的样式，因此也就不会有任何符合。  
  
ex: (newfile 为搜寻样式档案)  
  $grep -f newfile file      
  
12.  -G, --basic-regexp  
  将样式视为基本的规则表示式解释。(此为预设)  
  
13.  -H, --with-filename  
  在每个符合样式行前加上符合的档案名称，若有路径会显示路径。  
  
ex: (在 file 与 testfile 中搜寻 panda 样式)     
  $grep -H panda file ./testfile  
  file:panda  
  ./testfile:panda  
  $  
  
14.  -h, --no-filename    
  与-H 参数相类似，但在输出时不显示路径。  
  
15.  --help   
  产生简短的 help 讯息。  
  
16.  -I  
  grep 会强制认为此二进制档案没有包含任何搜寻样式，  
  与--binary-files=without-match 参数相同。  
  
  ex:  $ grep -I panda mv  
  
17.  -i, --ignore-case        
  忽略大小写，包含要搜寻的样式及被搜寻的档案。  
  
  ex:  $ grep -i panda mv  
  
18.  -L, --files-without-match   
  不显示平常一般的输出结果，反而显示出没有符合的档案名称。  
  
19.  -l, --files-with-matches       
  不显示平常一般的输出结果，只显示符合的档案名称。  
  
20.  --mmap       
  如果可能，使用 mmap 系统呼叫去读取输入，而不是预设的 read 系统呼叫。   
  在某些状况，--mmap 能产生较好的效能。 然而，--mmap   
  如果运作中档案缩短，或 I/O 错误发生时，  
  可能造成未定义的行为(包含 core dump)，。  
  
21.  -n, --line-number  
  在显示行前，标上行号。  
  
   ex:  $ grep -n panda file    
   显示结果相似于下:  
   行号:符合行的内容  
  
22.  -q, --quiet, --silent   
  不显示任何的一般输出。请参阅-s 或--no-messages  
  
23.  -r, --recursive  
  递回地，读取每个资料夹下的所有档案，此相当于 -d recsuse 参数。  
  
24.  -s, --no-messages  
  不显示关于不存在或无法读取的错误讯息。  
  
小注: 不像 GNU grep，传统的 grep 不符合 POSIX.2 协议，  
  因为缺乏-q 参数，且他的-s 参数表现像 GNU grep 的 -q 参数。  
  Shell Script 倾向将传统的 grep 移植，避开-q 及-s 参数，  
  且将输出限制到/dev/null。  
  
// POSIX: 定义 UNIX 及 UNIX-like 系统需要提供的功能。     
  
25.  -V, --version   
  显示出 grep 的版本号到标准错误。  
  当您在回报有关 grep 的 bugs 时，grep 版本号是必须要包含在内的。  
  
26.  -v, --invert-match  
  显示除搜寻样式行之外的全部。  
  
27.  -w, --word-regexp  
  将搜寻样式视为一个字去搜寻，完全符合该"字"的行才会被列出。  
  
28.  -x, --line-regexp  
  将搜寻样式视为一行去搜寻，完全符合该"行"的行才会被列出。  
  
###########################################################