
# [原创]Magic DVD Ripper 5.2.1 build 2的算法分析（高手略过）

## 文章信息
【文章标题】: Magic DVD Ripper 5.2.1 build 2的算法分析
【文章作者】: iawen
【作者邮箱】: 913221@163.com
【作者QQ号】: 160193626
【软件名称】: Magic DVD Ripper
【软件大小】: 2557KB
【下载地址】: 自己搜索下载
【加壳方式】: 无
【编写语言】: Borland C++ 1999
【使用工具】: OD
【软件介绍】:
Magic DVD Ripper 是一款DVD 电影抓取工具，支持 DVD to VCD，DVD to SVCD 转换,抓取后的DVD已经去除了区位码保护和MacroVision 保护,支持抓取或者转换完成后自动关机的功能,完整的支持市面上各种流行的DVD和VCD刻录机和盘片! 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---
## 详细过程
第一次写算分析，请各位大大不要见笑！常在看雪逛，也分享一点自己的学习！
不说了，请看正文：

OD载入后，运用F12堆栈调用法找到注册码判断的段：

```
0041EF18   .  55            push    ebp
0041EF19   .  8BEC          mov     ebp, esp
0041EF1B   .  83C4 AC       add     esp, -54
            >>>>>>由于侧重算法，这里就不细说如何找到段首了！
```

F8单步：

```
0041EF56   .  E8 99950B00   call    004D84F4                         ;  取出用户名
0041EF5B   .  8D45 F4       lea     eax, dword ptr [ebp-C]
0041EF5E   .  8B00          mov     eax, dword ptr [eax]
            >>>>>>EAX指向的即为用户名了
0041EFA2   > /33C0          xor     eax, eax
0041EFA4   . |8A03          mov     al, byte ptr [ebx]
            >>>>>>按位取用户名的每位的ASCII值
0041EFA6   . |03F0          add     esi, eax
0041EFA8   . |47            inc     edi
0041EFA9   . |43            inc     ebx
0041EFAA   > |8B55 B4       mov     edx, dword ptr [ebp-4C]
0041EFAD   . |52            push    edx
0041EFAE   . |E8 79BC0E00   call    0050AC2C
0041EFB3   . |59            pop     ecx
0041EFB4   . |3BF8          cmp     edi, eax
0041EFB6   .^\72 EA         jb      short 0041EFA2            ;  将用户名的每位ASCII码值累加
            >>>>>>计算完毕，ESI里的值即为累加的结果了
0041EFC8   > \56            push    esi
0041EFC9   .  68 A4D85300   push    0053D8A4                  ;  ASCII "%04X"
0041EFCE   .  8D4D AC       lea     ecx, dword ptr [ebp-54]
0041EFD1   .  51            push    ecx
0041EFD2   .  E8 11F70E00   call    0050E6E8                  ;  格式化为4位长的字符串
0041F05D   > /0FBE03        movsx   eax, byte ptr [ebx]
0041F060   . |83F8 6F       cmp     eax, 6F
0041F063   . |74 05         je      short 0041F06A
0041F065   . |83F8 4F       cmp     eax, 4F
0041F068   . |75 03         jnz     short 0041F06D
0041F06A   > |C603 30       mov     byte ptr [ebx], 30
0041F06D   > |47            inc     edi
0041F06E   . |43            inc     ebx
0041F06F   > |56            push    esi
0041F070   . |E8 B7BB0E00   call    0050AC2C
0041F075   . |59            pop     ecx
0041F076   . |3BF8          cmp     edi, eax                  ;  检查注册码，当出现ASCII值为6F或
0041F078   .^\72 E3         jb      short 0041F05D            ;  4F时，替换为0X30
0041F0A3   .  BA 02000000   mov     edx, 2
0041F0A8   .  E8 B3AB0F00   call    00519C60
            >>>>>>此时，ESI指向了我们输入的假注册码了
            >>>>>>      EBP-54指向了用户生成的4位长字符串
            >>>>>>假设，注册码为：strCode[32]
            >>>>>>      字符串为：strTmp[5]
0041F0AD   .  8A4E 05       mov     cl, byte ptr [esi+5]      ;  strCode[5]
0041F0B0   .  3A4D AC       cmp     cl, byte ptr [ebp-54]     ;  strTmp[0]
0041F0B3   .  75 26         jnz     short 0041F0DB            ;  不等则出错
            >>>>>>所以，strCode[5]应该等于strTmp[0]
0041F0B5   .  8A46 04       mov     al, byte ptr [esi+4]      ;  strCode[4]
0041F0B8   .  3A45 AD       cmp     al, byte ptr [ebp-53]     ;  strTmp[1]
0041F0BB   .  75 1E         jnz     short 0041F0DB
            >>>>>>所以，strCode[4]应该等于strTmp[1]
0041F0BD   .  8A56 12       mov     dl, byte ptr [esi+12]     ;  strCode[18],注12为16进制格式
0041F0C0   .  3A55 AE       cmp     dl, byte ptr [ebp-52]     ;  strTmp[2]
0041F0C3   .  75 16         jnz     short 0041F0DB
            >>>>>>所以，strCode[18]应该等于strTmp[2]
0041F0C5   .  8A0E          mov     cl, byte ptr [esi]        ;  strCode[0]
0041F0C7   .  3A4D AF       cmp     cl, byte ptr [ebp-51]     ;  strTmp[3]
0041F0CA   .  75 0F         jnz     short 0041F0DB
            >>>>>>所以，strCode[0]应该等于strTmp[3]
```

如果上面都通过，则显示正确！

故，只要随意的生成一个长超过19位的字符串，将其中的6，5，19，0替换成按用户名生成的4个字符，即可：

代码如下：

```
char strName[20];//用户名
char strCode[32];//注册码
char strS[]="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

void CreateRegCode(HWND hDlg){
          //利用随机数，生成随机的20位长注册码
          for(int i=0;i<20;i++){
                  int index=(((double) rand() / (double) RAND_MAX) * 62 + 0);
                  strCode[i]=strS[index];
          }
          //计算用户各位ASCII值的和，并格式化为字符串
          char strTmp[5];
          int len=strlen(strName);
          int sum=0;
          for(int i=0;i<len;i++)
                  sum+=strName[i];
          sum&=0x8000FFFF;
          sprintf(strTmp,"%04X",sum);
          //替换注册码中的值
          strCode[5]=strTmp[0];
          strCode[4]=strTmp[1];
          strCode[18]=strTmp[2];
          strCode[0]=strTmp[3];
          //完成，显示到编辑框
          SetDlgItemText(hDlg,IDC_PWD,strCode);
          return;
}
```

---
## 经验总结
没有什么感觉，只要花点时间，细过一边就一目了然了！也算捡了一个软柿子吧！

---
## 版权声明
本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!
2008年02月01日 22:07:53

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：
- [MagicDVDRipper.rar](javascript:void(0);)
  （257.25kb，29次下载）
