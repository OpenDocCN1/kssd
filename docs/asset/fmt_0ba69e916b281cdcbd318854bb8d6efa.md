
在伪 `OEP` 处看看相关寄存器和堆栈数据：`EBP`=0012FFB4 `ESP`=0012FFC0 `EAX`=004E2268

`SVKP` 的 Stolen Code 一般不多，根据以上数据和壳中代码处理部分，补上 `OEP` 处的 Stolen Code

```
004E2660     55                   push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
004E2661     8BEC                 mov ebp,esp
004E2663     83C4 F4              add esp,-0C
004E2666     B8 68224E00          mov eax,004E2268//补上的 Stolen Code   :-)
004E266B     E8 A443F2FF          call RegMedic.00406A14//伪 OEP
004E2670     A1 D4B94F00          mov eax,dword ptr ds:[4FB9D4]
004E2675     8B00                 mov eax,dword ptr ds:[eax]
004E2677     E8 8CC2F6FF          call RegMedic.0044E908
004E267C     A1 D4B94F00          mov eax,dword ptr ds:[4FB9D4]
004E2681     8B00                 mov eax,dword ptr ds:[eax]
004E2683     BA AC264E00          mov edx,RegMedic.004E26AC; ASCII "Registry Medic"
```

运行 `ImportREC`，设置“启用调试权限”的选项，选择这个进程。把 `OEP` 改为 `000E2660`、`RVA`=00114150、`Size`=000007B4，点“Get Import”，呵呵，除了 3 个 `SVK` 反脱壳（注册）函数其他的全部是有效的！`FixDump`。

---

### 三、修正 SVK 反脱壳（注册）函数

程序现在还是无法正常运行的，别忘了还有 3 个 SVK 函数没有修复。

```
0  001148B4  ?  0000  082785D2
0  001148B8  ?  0000  08278098
0  001148BC  ?  0000  08276FC4
```

在这几个地方下 内存 断点，很容易就找到调用的地方：

```
0044FB78     FF25 BC485100       jmp dword ptr ds:[5148BC]
0044FB80     FF25 B8485100       jmp dword ptr ds:[5148B8]
0044FB88     FF25 B4485100       jmp dword ptr ds:[5148B4]
```

把这几个修改为：

```
0044FB78     C3                   retn
0044FB80     C3                   retn
0044FB88     C3                   retn
```

`OK`，程序已经能够运行了。比较幸运的是这个程序没有使用分段解密。

---

### 四、破解

虽然程序能够运行了，但提示 `Trail Expired`，弹出大大的注册框让偶注册。呵呵，看来注册部分没用 SVK 的。

用户名：`@ fly @`

试炼码：`13572-46890-12135－20040－03080－24689－123`

根据提示信息，很容易找到注册核心的地方。

```
004E1294     8B45 F4              mov eax,dword ptr ss:[ebp-C]//[ebp-C]=@ fly @
004E1297     E8 8C2CF2FF          call dumped_.00403F28
004E129C     83F8 06              cmp eax,6//用户名至少 6 位
004E129F     0F8C F0000000        jl dumped_.004E1395
004E12A5     8B45 F8              mov eax,dword ptr ss:[ebp-8]//[ebp-8]=试炼码
004E12A8     E8 7B2CF2FF          call dumped_.00403F28
004E12AD     83F8 21              cmp eax,21//试炼码至少 33 位
004E12B0     7C 1D                jl short dumped_.004E12CF
004E12B2     8B45 FC              mov eax,dword ptr ss:[ebp-4]
004E12B5     E8 CE0A0000          call dumped_.004E1D88//黑名单检测
```

黑名单：`Cindy J. Leonheardt`、`[DJ]neo`  呵呵，希望以后不要加上 `fly`   :-)

```
004E12BA     84C0                 test al,al
004E12BC     75 11                jnz short dumped_.004E12CF//是黑名单上的就直接 OVER  :oops:
004E12BE     8B55 F4              mov edx,dword ptr ss:[ebp-C]
004E12C1     B8 F4134E00          mov eax,dumped_.004E13F4
004E12C6     E8 492FF2FF          call dumped_.00404214//检测用户名中有无 @ 这个字符
004E12CB     85C0                 test eax,eax
004E12CD     75 45                jnz short dumped_.004E1314//没有@就 OVER  :oops:
004E12CF     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E12D2     BA 00144E00          mov edx,dumped_.004E1400
               //ASCII "Registration failed. Please try again!"       BAD BOY！  :-(
004E130A     E8 8960F2FF          call <jmp.&user32.MessageBoxA>//呵呵，Please try again!
004E130F     E9 B0000000          jmp dumped_.004E13C4
004E1314     8B55 F8              mov edx,dword ptr ss:[ebp-8]
004E1317     8B45 F4              mov eax,dword ptr ss:[ebp-C]
004E131A     E8 91A9FAFF          call dumped_.0048BCB0
004E131F     E8 C4B4FAFF          call dumped_.0048C7E8//运算、比较注册码
004E1324     A1 D4B74F00          mov eax,dword ptr ds:[4FB7D4]
004E1329     8038 00              cmp byte ptr ds:[eax],0//判断[4FB7D4]处标志
004E132C     74 25                je short dumped_.004E1353
004E132E     A1 58B84F00          mov eax,dword ptr ds:[4FB858]
004E1333     8338 00              cmp dword ptr ds:[eax],0//判断[4FB858]处标志
004E1336     74 0C                je short dumped_.004E1344
004E1338     A1 58B84F00          mov eax,dword ptr ds:[4FB858]
004E133D     8B00                 mov eax,dword ptr ds:[eax]
004E133F     E8 68D1FFFF          call dumped_.004DE4AC
004E1344     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E1347     BA 30144E00          mov edx,dumped_.004E1430
               //ASCII "Thank you for registering Registry Medic."  呵呵，胜利女神！:-)
004E134C     E8 EF29F2FF          call dumped_.00403D40
004E1351     EB 0D                jmp short dumped_.004E1360
004E1353     8D45 EC              lea eax,dword ptr ss:[ebp-14]
004E1356     BA 00144E00          mov edx,dumped_.004E1400
               //ASCII "Registration failed. Please try again!"       BAD BOY！  :-(
```

直接看注册码的比较部分吧： :-)

```
0048B300     8B45 D8              mov eax,dword ptr ss:[ebp-28]//EAX=50e8c2d4bc92
0048B303     8B55 CC              mov edx,dword ptr ss:[ebp-34]//EDX=135724689012
0048B306     E8 2D8DF7FF          call dumped_.00404038//比较注册码前 12 位
0048B30B     75 11                jnz short dumped_.0048B31E
0048B30D     8B45 D4              mov eax,dword ptr ss:[ebp-2C]//[ebp-2C]=123
0048B310     8B55 D0              mov edx,dword ptr ss:[ebp-30]//[ebp-30]=2db
0048B313     E8 208DF7FF          call dumped_.00404038//比较注册码最后 3 位，根据前 30 位得出
0048B318     75 04                jnz short dumped_.0048B31E
0048B31A     C645 F7 01           mov byte ptr ss:[ebp-9],1//置成功标志位
0048B31E     33C0                 xor eax,eax
0048B320     5A                   pop edx
0048B321     59                   pop ecx
0048B322     59                   pop ecx
0048B323     64:8910              mov dword ptr fs:[eax],edx
0048B326     68 38B34800          push dumped_.0048B338
0048B32B     E8 5048FCFF          call dumped_.0044FB80
0048B330     C3                   retn
```

---

**【注册信息保存】：**

注册信息和时间信息保存在注册表中的 2 个地方：

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{B087CE89-A8EE-4890-91E7-696B3F749500}]`

`"AppIdK"="CA000DFE903977C2EB7C-5CD3A51B5BED33B772DB-316841B94D9A0C-50E8C2D4BC921352004003080246892DB"`

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{0C985599-3A7C-4D1D-9D73-B6BBC812D950}]`

`"AppIdK"="CA000DFE903977C2EB7C-5CD3A51B5BED33B772DB-316841B94D9A0C-50E8C2D4BC921352004003080246892DB"`

---

**【整理】：**

用户名：`@ fly @`

注册码：`50E8C－2D4BC－92135－20040－03080－24689－2DB`

---

```
        ,      _/      
      /| _.-~/           _     ,       
     ( /~   /            ~-.\ |
     \`\\  _/                  ~ )          忍把浮名  
  _-~~~-.)  )\_/;;,.           _  //'    
 /'\_,   --~    ~~~-  ,;;\_\_(  (.-~~~-.        换了破解轻狂 
 \`~ _( ,_..-- (     ,;'' /    ~--   /.\_\`  
  /~~//'   /`~          ) /--..\_, )\_  `~    
  "  `~"  "      `"      /~'`    \`\\~~   
                       "     "   "~'  ""
```

Cracked By 巢水工作坊——`fly` `[OCN]` `[FCG]` `[NUKE]` `[DCM]`

2004-03-12 14:00

`4dd466d0e1e0cc123afbe504da2462ca`
