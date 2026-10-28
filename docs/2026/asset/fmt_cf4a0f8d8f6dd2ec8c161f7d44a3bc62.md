
- `ini`配置文件的操作：获取当前路径，并将“`Desktop\_.ini`”拼接到当前路径中，同时检查此文件是否存在，若文件存在，则删除

  ![image-20200113083258560](https://bbs.kanxue.com/upload/attach/202002/868592_8D5AWQQC66EDS2W.jpg)

- `ini`配置文件操作完成后，进入新的流程：为了复制病毒信息以感染其他文件，故将病毒文件信息读取到内存中；同时也设置了标记信息

  ![image-20191128091045656](https://bbs.kanxue.com/upload/attach/202002/868592_5CNGQTGK3639Q9Z.jpg)

- 随后跳往`4082DB`处：进行了标记检查工作，来判断病毒程序是否被多次运行

  ![image-20191128090924847](https://bbs.kanxue.com/upload/attach/202002/868592_RYYM7TMZ8Y5KZZN.jpg)

- 随后代码中出现“`spo0lcv.exe`”字符串：找到运行的进程“`spo0lsv.exe`”并终止；病毒第一次执行时，需要将自身复制一份到系统目录下，并伪装成“`spo0lsv.exe`”启动；

  ![image-20200113083320628](https://bbs.kanxue.com/upload/attach/202002/868592_WK4JNK93CETBUS8.jpg)

- 字符串比较后，若当前病毒文件已经复制到了系统目录下且被伪造成了“`spo0lsv.exe`”，则会跳转到`40845E`处：释放之前申请存放病毒文件信息的内存（OD调试时，可手动修改`JZ`条件，使之完成跳转）

  ![image-20191128101017016](https://bbs.kanxue.com/upload/attach/202002/868592_W7PYJKB3RFGGNBU.jpg)

- 随之跳往 `408741`：获取标记并进行检查，当`标记==1`时，会进入 `408477`来进行进一步检查（此代码块只有被感染的程序才会执行，OD打开被感染文件进行调试

  ![image-20191128101935542](https://bbs.kanxue.com/upload/attach/202002/868592_J9GKXAYBGJXG2DK.jpg)

## 3.7-感染过程分析

- 内部三个子模块：前二本地感染，第三通过网络

  ![image-20191127145219156](https://bbs.kanxue.com/upload/attach/202002/868592_4VAQNXQA7WJFEKS.jpg)

- 创建感染线程：找到线程回调函数

  ![image-20191127145152729](https://bbs.kanxue.com/upload/attach/202002/868592_DHJNAQCCGRNXZJS.jpg)

- 感染线程的线程回调函数：遍历驱动器盘符的数目，进入到驱动器中，遍历所有可感染文件以进行病毒复制

  ![image-20191127151719814](https://bbs.kanxue.com/upload/attach/202002/868592_5GBJV92RP7SW4Y6.jpg)

- `_FindAndInfectFile`：遍历驱动器下的文件信息，将文件与文件夹进行了分类，排除了特殊文件夹；在判断是否为特殊文件夹的过程中，都是以大写形式进行对比的，以防止因大小写不同而造成错误；在这可跳到处理文件的地方

  ![image-20191127170811439](https://bbs.kanxue.com/upload/attach/202002/868592_KK339PGY732T9TV.jpg)

- `_FindAndInfectFile`：感染前的检查工作，`ini`中保存的时间与刚刚获取的时间进行比较，来判断是否已经被感染

  ![image-20200113083345505](https://bbs.kanxue.com/upload/attach/202002/868592_73HF4TV9HXP7PMX.jpg)

- 如4中，若当前获取到的是文件时，开始处理文件：对文件进行检查，只要后缀为“GHO”，就判定为ghost备份文件，直接删除，防止用户还原系统（真tm的坏）

  ![image-20191127172133510](https://bbs.kanxue.com/upload/attach/202002/868592_7DWHV52JC8PJJ7D.jpg)

- 一旦删除了系统的备份文件，病毒就开始肆意感染：大致分为两种，一种是本地可执行文件（如`exe`为后缀的），另一种是Web网页文件（如`html`为后缀的）

  ![image-20191127193508763](https://bbs.kanxue.com/upload/attach/202002/868592_72Y89UN9YS4A5KZ.jpg)

  ![image-20191127193912691](https://bbs.kanxue.com/upload/attach/202002/868592_RJVXMB6KECM5KR7.jpg)

- 感染`exe`等本地执行文件，`_InfectFile`：首先将目标文件读取到内存中并获取文件名及其大小、然后将自身文件复制到目标文件前并追加目标程序的原始文件、最后添加感染标记，结束感染过程

  ![image-20191127214435254](https://bbs.kanxue.com/upload/attach/202002/868592_HQ8R5YN5NE9VUZV.jpg)

  ![image-20191127214817563](https://bbs.kanxue.com/upload/attach/202002/868592_YNQ9R3P9UGH75XA.jpg)

- 被感染文件 ： 病毒文件 + 原文件 + 感染标识，标识如下

  ![image-20200113083407839](https://bbs.kanxue.com/upload/attach/202002/868592_Z4EUW2Z947JUZFV.jpg)

## 3.8-自我保护行为

- 若找到标记字符，则会跳转到`408477`，此代码块只有被感染的程序才会执行，现IDA中打开原病毒静态分析（也可OD中打开被感染程序动态分析

- 代码段`408477`：从被感染文件中分离提取出原始文件（由于感染文件时写入了标记信息，根据标记可得到原始文件），执行后，在被感染文件所在目录下释放出其原始文件

  ![image-20200113083433875](https://bbs.kanxue.com/upload/attach/202002/868592_9D3GZ9AYXYP7FJ2.jpg)

- 代码段 `408584`：被感染文件的主要工作是维护进程中伪造的病毒程序`spo0lsv.exe`，若`spo`被删除，则会由被感染的程序再次生成（春风吹又生，除非删除所有被感染程序）

  ![image-20191128113200558](https://bbs.kanxue.com/upload/attach/202002/868592_H88ER6EZAYU725T.jpg)

# 04-解决方案

## 4.1-病毒特征

- 病毒文件的哈希值如下：

  - MD5: 512301C535C88255C9A252FDF70B7A03

  - SHA1: CA3A1070CFF311C0BA40AB60A8FE3266CFEFE870

  - CRC32: E334747C

- 被感染文件中，文件最后有感染表示标识：WhBoy + 原文件名 + exe + + ascii码为1的字符 + 随机数+ ascii码为2的字符

  ![image-20200113083415753](https://bbs.kanxue.com/upload/attach/202002/868592_R4GCUS3NQY6R92A.jpg)

## 4.2-手工查杀

1. `cmd`命令行下`taskkill`杀掉进程，或者直接利用PCHunter等工具结束进程

2. 在系统目录下删除病毒文件：`C:\WINDOWS\system32\drivers\spo0lsv.exe`

3. 删除启动项：Windows+R，键入`msconfig`打开服务配置窗口，切换启动窗口，取消启动项，并且记录下注册表位置

   ![img](https://bbs.kanxue.com/upload/attach/202002/868592_7C9HDNHQQ3MAMKG.jpg)

   ![img](https://bbs.kanxue.com/upload/attach/202002/868592_RTUWDXBPGMEX36Q.jpg)

4. 修改注册表：Windows+R，键入`regedit`打开注册表窗口，找到相应位置，并删除记录

   ![image-20200113083516026](https://bbs.kanxue.com/upload/attach/202002/868592_N6GNCRAJEUP7S6F.jpg)

## 4.3-专杀工具

- 效果图

  ![image-20200113083529845](https://bbs.kanxue.com/upload/attach/202002/868592_897WTWNUU243Z2G.jpg)

  ![image-20200113083541900](https://bbs.kanxue.com/upload/attach/202002/868592_K9J3TCTSM5UYFA2.jpg)

- GitHub地址：fddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9P5e0q4&6L8X3&6Q4x3V1k6m8L8Y4c8A6f1r3q4F1k6r3p5`.

# 05-参考文献

- 《C++反汇编与逆向分析技术解密》

- 15PB视频课程-恶意代码分析实例

- 118K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4u0A6j5$3S2S2M7X3b7I4x3U0x3H3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5e0f1H3y4e0t1H3x3l9`.`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-2-16 14:41  
被21Gun5编辑  
，原因：  

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
