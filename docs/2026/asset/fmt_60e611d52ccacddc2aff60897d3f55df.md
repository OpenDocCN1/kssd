
# 一段将长10进制数转化为2进制数的代码

昨天跟个程序，跟了半天都没什么发现，结果迷迷糊糊跟进这个`CALL`里面来了。头都晕了才发现是在作数制转换。输入的是个很长的十进制字符串,大概有7,80位，然后返回对应2进制字符串。

## 字符串分割与初始化

废话的代码就不贴了,从这里开始处理:

```
004CF8B7     |.  >call dumped.00404A60                   ;  取得字符串长度
004CF8BC     |.  >mov ecx,9
004CF8C1     |.  >cdq
004CF8C2     |.  >idiv ecx                               ;  字符串长度/9,因为目前计算机字长一般32位,0xFFFFFFFF对10位的十进制数可能会溢出
004CF8C4     |.  >mov dword ptr ss:[ebp-8],eax           ;  可以分割eax个9位长的10进制子字符串
004CF8C7     |.  >mov eax,dword ptr ss:[ebp-4]
004CF8CA     |.  >call dumped.00404A60                   ;  取得字符串长度
004CF8CF     |.  >mov ecx,9
004CF8D4     |.  >cdq
004CF8D5     |.  >idiv ecx
004CF8D7     |.  >test edx,edx                           ;比较余数,余数为几就还剩下一个几位的字符串
004CF8D9     |.  >je short dumped.004CF8DE
004CF8DB     |.  >inc dword ptr ss:[ebp-8]               ;有余数,子字符串个数+1
004CF8DE     |>  >mov eax,dword ptr ss:[ebp-8]
004CF8E1     |.  >inc eax
004CF8E2     |.  >push eax
004CF8E3     |.  >lea eax,dword ptr ds:[ebx+4]
004CF8E6     |.  >mov ecx,1
004CF8EB     |.  >mov edx,dword ptr ds:[4CF018]          ;  dumped.004CF01C
004CF8F1     |.  >call dumped.00405D74
004CF8F6     |.  >add esp,4
004CF8F9     |.  >mov eax,dword ptr ss:[ebp-8]
004CF8FC     |.  >cdq
004CF8FD     |.  >mov ecx,dword ptr ds:[ebx+4]
004CF900     |.  >mov dword ptr ds:[ecx],eax
004CF902     |.  >mov dword ptr ds:[ecx+4],edx
004CF905     |.  >mov esi,dword ptr ss:[ebp-8]
004CF908     |.  >dec esi
004CF909     |.  >test esi,esi
004CF90B     |.  >jle short dumped.004CF963
004CF90D     |.  >mov edi,1
004CF912     |>  >/lea eax,dword ptr ss:[ebp-24]         ;  这个循环开始把十进制字符串分成子字符串数组
004CF915     |.  >|push eax
004CF916     |.  >|mov eax,dword ptr ss:[ebp-4]
004CF919     |.  >|call dumped.00404A60                  ;  取得十进制字符长度
004CF91E     |.  >|mov edx,eax
004CF920     |.  >|sub edx,8
004CF923     |.  >|mov ecx,9
004CF928     |.  >|mov eax,dword ptr ss:[ebp-4]
004CF92B     |.  >|call dumped.00404CB8                  ;  得到字符串末尾9个字符
004CF930     |.  >|mov eax,dword ptr ss:[ebp-24]         ;  字符串末尾9个字符
004CF933     |.  >|call dumped.004090C4                  ;  eax返回9位字符表示的DEC数的HEX值
004CF938     |.  >|cdq                                   ;  edx=0
004CF939     |.  >|push edx
004CF93A     |.  >|push eax
004CF93B     |.  >|mov eax,dword ptr ds:[ebx+4]          ;  存储数据的基地址
004CF93E     |.  >|pop dword ptr ds:[eax+edi\*8]          ;  a
004CF941     |.  >|pop dword ptr ds:[eax+edi\*8+4]        ;  0
004CF945     |.  >|mov eax,dword ptr ss:[ebp-4]
004CF948     |.  >|call dumped.00404A60
004CF94D     |.  >|mov edx,eax                           ;  字符长度
004CF94F     |.  >|sub edx,8
004CF952     |.  >|lea eax,dword ptr ss:[ebp-4]          ;  字符地址
004CF955     |.  >|mov ecx,9
004CF95A     |.  >|call dumped.00404CF8                  ;  eax字符串砍去末尾9个字符
004CF95F     |.  >|inc edi
004CF960     |.  >|dec esi
004CF961     |.^ >\jnz short dumped.004CF912
004CF963     |>  >mov eax,dword ptr ss:[ebp-4]           ;  最后一个子字符
004CF966     |.  >call dumped.004090C4                   ;  转化为HEX
004CF96B     |.  >cdq
004CF96C     |.  >mov ecx,dword ptr ds:[ebx+4]
004CF96F     |.  >mov esi,dword ptr ss:[ebp-8]
004CF972     |.  >mov dword ptr ds:[ecx+esi\*8],eax
004CF975     |.  >mov dword ptr ds:[ecx+esi\*8+4],edx
004CF979     |.  >lea eax,dword ptr ss:[ebp-14]
004CF97C     |.  >call dumped.004047A8
004CF981     |.  >jmp short dumped.004CF9AD
```

上面是准备工作,对10进制字符串进行分割.分割是从字符串末位开始的,每9个字符分割1次,剩余的不足9个字符的为一个子字符串具体点,如果是个"123456789987654321000"会被分成"654321000""456789987""123"三个子字符串,并转化为相应的HEX值,即是"27002568","1B3A0FE3","0000007B"。

## 主循环：除以 2

```
004CF983     |>  >/push 0                                ; /Arg2 = 00000000
004CF985     |.  >|push 2                                ; |Arg1 = 00000002
004CF987     |.  >|lea edx,dword ptr ss:[ebp-10]         ; |
004CF98A     |.  >|mov eax,ebx                           ; |
004CF98C     |.  >|call dumped.004CF6CC                  ; \对整个10进制数除以2！
004CF991     |.  >|push dword ptr ss:[ebp-C]             ; /Arg2 = 00000000
004CF994     |.  >|push dword ptr ss:[ebp-10]            ; |余数
004CF997     |.  >|lea eax,dword ptr ss:[ebp-28]         ; |
004CF99A     |.  >|call dumped.00409054                  ; \dumped.00409054
004CF99F     |.  >|mov edx,dword ptr ss:[ebp-28]
004CF9A2     |.  >|lea eax,dword ptr ss:[ebp-14]         ;  这里是保留最后结果2进制字符串的地址
004CF9A5     |.  >|mov ecx,dword ptr ss:[ebp-14]         ;
004CF9A8     |.  >|call dumped.00404AAC                  ;  余数增加到2进制串的最前面(因为是要倒序排列)
004CF9AD     |>  > mov eax,dword ptr ds:[ebx+4]          ;  数据存储基地址
004CF9B0     |.  >|cmp dword ptr ds:[eax+4],0
004CF9B4     |.^ >|jnz short dumped.004CF983
004CF9B6     |.  >|cmp dword ptr ds:[eax],1              ;  比较子字符串数组是否只剩最后一个字符串？
004CF9B9     |.^ >|jnz short dumped.004CF983
004CF9BB     |.  >|mov eax,dword ptr ds:[ebx+4]          ;  数组地址
004CF9BE     |.  >|cmp dword ptr ds:[eax+C],0
004CF9C2     |.^ >|jnz short dumped.004CF983
004CF9C4     |.  >|cmp dword ptr ds:[eax+8],0            ;  比较是否除尽
004CF9C8     |.^ >\jnz short dumped.004CF983
004CF9CA     |.  >lea eax,dword ptr ss:[ebp-14]          ;  压入函数的DEC字符串的2进制
```

都知道,10进制转换2进制只需要不断除以2,一直除到0,将每次除法的余数倒序排列就可以了,这里也是这么做的.不同的是,这里将很长的字符串分成一些子字符串,这样计算机才能计算.要注意的是,123-456789987-654321000和0000007B-1B3A0FE3-27002568的值是不等的,跟进除以2的这个`CALL`:

## 除法子程序分析

```
004CF6CC     /$  >push ebp
004CF6CD     |.  >mov ebp,esp
004CF6CF     |.  >add esp,-10
004CF6D2     |.  >push ebx
004CF6D3     |.  >push esi
004CF6D4     |.  >push edi
004CF6D5     |.  >mov esi,edx
004CF6D7     |.  >mov dword ptr ss:[ebp-4],eax
004CF6DA     |.  >mov eax,dword ptr ss:[ebp-4]
004CF6DD     |.  >mov eax,dword ptr ds:[eax+4]
004CF6E0     |.  >mov edi,dword ptr ds:[eax]             ;  子字符串数组基地址
004CF6E2     |.  >mov dword ptr ds:[esi],0               ;  [esi]和[esi+4]是2个循环变量，初始化为0
004CF6E8     |.  >mov dword ptr ds:[esi+4],0
004CF6EF     |.  >mov ebx,edi                            ;  字符串数组长度
004CF6F1     |.  >cmp ebx,1                              ;  是否小于1,小于1就直接出CALL
004CF6F4     |.  >jl short dumped.004CF76C
```

为方便表述,把子数值串记为`N1,N2,N3...Nm`.原来的长十进制数=`Dec(Nm)|Dec(Nm-1)...|Dec(N1)`,这里`Dec()`表示转化为10进制,"|",表示连接字符串的作用

下面开始做除法,取子数值串的顺序为`Nm,Nm-1...N2,N1`,既是从最高位除起.

```
004CF6F6     |>  >/push 0
004CF6F8     |.  >|push 3B9ACA00                         ;  0x3B9ACA00=1000000000(DEC),9个"0"
004CF6FD     |.  >|mov eax,dword ptr ds:[esi]
004CF6FF     |.  >|mov edx,dword ptr ds:[esi+4]
004CF702     |.  >|call dumped.0040579C                  ;  [esi]和[esi+4]\*=3B9ACA00,因为前面分子字符串是按9位分的,所以Nm中的1=Nm-1中的1\*1000000000
004CF707     |.  >|mov dword ptr ds:[esi],eax            ;  [esi]=(Nm+1 mod 2)\*1000000000(DEC)
004CF709     |.  >|mov dword ptr ds:[esi+4],edx          ;  [esi+4]指示[esi]和上一次数据相加是否产生进位
004CF70C     |.  >|mov eax,dword ptr ds:[esi]
004CF70E     |.  >|mov edx,dword ptr ds:[esi+4]
004CF711     |.  >|push edx
004CF712     |.  >|push eax
004CF713     |.  >|mov eax,dword ptr ss:[ebp-4]
004CF716     |.  >|mov eax,dword ptr ds:[eax+4]          ;  数组基地址
004CF719     |.  >|mov edx,dword ptr ds:[eax+ebx\*8+4]
004CF71D     |.  >|mov eax,dword ptr ds:[eax+ebx\*8]      ;  从数组末开始取一个数,最高位的子数值串先作除法
004CF720     |.  >|add eax,dword ptr ss:[esp]            ;  Nm+=[esi],即加上上一次除法的余数
004CF723     |.  >|adc edx,dword ptr ss:[esp+4]          ;  这里看出[esi+4]的作用,即表示Nm+(Nm+1 mod 2)是否>0xFFFFFFFF,产生进位
004CF727     |.  >|add esp,8                             ;  [esi].[esi+4]出栈，恢复指针
004CF72A     |.  >|mov dword ptr ss:[ebp-10],eax
004CF72D     |.  >|mov dword ptr ss:[ebp-C],edx
004CF730     |.  >|push dword ptr ss:[ebp+C]             ;  0
004CF733     |.  >|push dword ptr ss:[ebp+8]             ;  2
004CF736     |.  >|mov eax,dword ptr ss:[ebp-10]         ;  临时变量保存数组数据
004CF739     |.  >|mov edx,dword ptr ss:[ebp-C]          ;  [esi+4]
004CF73C     |.  >|call dumped.004057C0                  ;  eax是Nm/2的商,如果[esi+4]!=0,还要计算上[esi+4]
004CF741     |.  >mov ecx,dword ptr ss:[ebp-4]
```
