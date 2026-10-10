

# 一个比较另类的 ECC stuff

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-28.htm`](https://www.pediy.com/kssd/pediy08/pediy8-28.htm)

这个**crackme** 是 tkm 2004 年的 strainer,用 ECC 保护的。虽然 ECC 大家玩得比较熟了，不过这个东东的保护方法还是有点意思。  
用 PEiD 查壳并脱壳，unpack 不是目的，算法才是关键。unpack 后载入 ida,用 miracl 的 sig，或  
  
http://www.secretashell.com/cryptocrk/tutorials/miracl.calls.zip 这里的方法识别 miracl 库函数，检查序列号的流程一目了然。  
1.获取 name,sn1,sn2  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:0040194C loc\_40194C:                   
> Un\_FSG\_\_:0040194C                 push    50h
> Un\_FSG\_\_:0040194E                 push    offset name
> Un\_FSG\_\_:00401953                 push    2
> Un\_FSG\_\_:00401955                 push    edi
> Un\_FSG\_\_:00401956                 call    GetDlgItemText ; name>=4
> Un\_FSG\_\_:0040195B                 mov     esi, eax
> Un\_FSG\_\_:0040195D                 push    50h
> Un\_FSG\_\_:0040195F                 push    offset sn1
> Un\_FSG\_\_:00401964                 push    3
> Un\_FSG\_\_:00401966                 push    edi
> Un\_FSG\_\_:00401967                 call    GetDlgItemText ; sn1>=4
> Un\_FSG\_\_:0040196C                 mov     ebx, eax
> Un\_FSG\_\_:0040196E                 push    50h
> Un\_FSG\_\_:00401970                 push    offset sn2
> Un\_FSG\_\_:00401975                 push    4
> Un\_FSG\_\_:00401977                 push    edi
> Un\_FSG\_\_:00401978                 call    GetDlgItemText ; sn2>=1
> 
> * * *
> 
> 
> ```

  
2.检查序列号  
1)椭圆曲线 E(a,b,p)、点(pt1,pt2,pt3,pt4,pt5)初始化  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:0040149C                 call    epoint\_init
> Un\_FSG\_\_:004014A1                 mov     edi, eax
> Un\_FSG\_\_:004014A3                 mov     \[esp+1F8h+var\_1F8\], 0
> Un\_FSG\_\_:004014AA                 call    \_mirvar
> Un\_FSG\_\_:004014AF                 mov     \[ebp+pt2\_x\], eax
> Un\_FSG\_\_:004014B5                 call    epoint\_init
> Un\_FSG\_\_:004014BA                 mov     \[esp+1F8h+var\_1F8\], 0
> Un\_FSG\_\_:004014C1                 mov     \[ebp+pt2\], eax
> Un\_FSG\_\_:004014C7                 call    \_mirvar
> Un\_FSG\_\_:004014CC                 mov     \[ebp+big\_t\], eax
> Un\_FSG\_\_:004014D2                 call    epoint\_init
> Un\_FSG\_\_:004014D7                 mov     \[ebp+pt3\], eax
> Un\_FSG\_\_:004014DD                 call    epoint\_init
> Un\_FSG\_\_:004014E2                 mov     \[ebp+pt4\], eax
> Un\_FSG\_\_:004014E8                 call    epoint\_init
> Un\_FSG\_\_:004014ED                 mov     dword ptr \[esi+220h\], 10h
> Un\_FSG\_\_:004014F7                 mov     \[ebp+pt5\], eax
> ...
> Un\_FSG\_\_:00401512                 mov     ebx, \[ebp+a\]
> Un\_FSG\_\_:00401518                 push    offset a416d656e657369 ; "416D656E65736961"
> Un\_FSG\_\_:0040151D                 push    ebx
> Un\_FSG\_\_:0040151E                 call    \_cinstr
> Un\_FSG\_\_:00401523                 pop     eax
> Un\_FSG\_\_:00401524                 pop     edx
> Un\_FSG\_\_:00401525                 mov     eax, \[ebp+b\]
> Un\_FSG\_\_:0040152B                 push    offset a1b35b7093fee5a ; "1B35B7093FEE5AE601A"
> Un\_FSG\_\_:00401530                 push    eax
> Un\_FSG\_\_:00401531                 call    \_cinstr
> Un\_FSG\_\_:00401536                 pop     ebx
> Un\_FSG\_\_:00401537                 pop     esi
> Un\_FSG\_\_:00401538                 mov     esi, \[ebp+p\]
> Un\_FSG\_\_:0040153E                 push    offset aFffffffdffffff ; "FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF"
> Un\_FSG\_\_:00401543                 push    esi
> Un\_FSG\_\_:00401544                 call    \_cinstr
> Un\_FSG\_\_:00401549                 pop     eax
> Un\_FSG\_\_:0040154A                 pop     edx
> Un\_FSG\_\_:0040154B                 mov     edx, \[ebp+pt1\_x\]
> Un\_FSG\_\_:00401551                 push    offset a71263a72c2fdfb ; "71263A72C2FDFB8FE851182B408210A4"
> Un\_FSG\_\_:00401556                 push    edx
> Un\_FSG\_\_:00401557                 call    \_cinstr
> Un\_FSG\_\_:0040155C                 mov     eax, \[ebp+pt2\_x\]
> Un\_FSG\_\_:00401562                 pop     ecx
> Un\_FSG\_\_:00401563                 pop     ebx
> Un\_FSG\_\_:00401564                 push    offset a2c052124bdd94e ; "2C052124BDD94E5645E99B01DCECA28D"
> Un\_FSG\_\_:00401569                 push    eax
> Un\_FSG\_\_:0040156A                 call    \_cinstr
> Un\_FSG\_\_:0040156F                 mov     eax, \[ebp+p\]
> Un\_FSG\_\_:00401575                 push    1
> Un\_FSG\_\_:00401577                 push    eax
> Un\_FSG\_\_:00401578                 mov     eax, \[ebp+b\]
> Un\_FSG\_\_:0040157E                 push    eax
> Un\_FSG\_\_:0040157F                 mov     eax, \[ebp+a\]
> Un\_FSG\_\_:00401585                 push    eax
> Un\_FSG\_\_:00401586                 call    \_ecurve\_init
> Un\_FSG\_\_:0040158B                 mov     eax, \[ebp+pt1\_x\]
> Un\_FSG\_\_:00401591                 add     esp, 20h
> Un\_FSG\_\_:00401594                 push    edi             ; pt1
> Un\_FSG\_\_:00401595                 push    0
> Un\_FSG\_\_:00401597                 push    eax
> Un\_FSG\_\_:00401598                 mov     eax, \[ebp+pt1\_x\]
> Un\_FSG\_\_:0040159E                 push    eax
> Un\_FSG\_\_:0040159F                 call    \_epoint\_set
> Un\_FSG\_\_:004015A4                 mov     ecx, \[ebp+pt2\_x\]
> Un\_FSG\_\_:004015AA                 mov     eax, \[ebp+pt2\]
> Un\_FSG\_\_:004015B0                 mov     ebx, \[ebp+pt2\_x\]
> Un\_FSG\_\_:004015B6                 push    eax
> Un\_FSG\_\_:004015B7                 push    1
> Un\_FSG\_\_:004015B9                 push    ebx
> Un\_FSG\_\_:004015BA                 push    ecx
> Un\_FSG\_\_:004015BB                 call    \_epoint\_set
> 
> * * *
> 
> 
> ```

  
其中 E 的参数为:  
a:416D656E65736961  
b:1B35B7093FEE5AE601A  
p:FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF  
pt1 的 x 坐标为:71263A72C2FDFB8FE851182B408210A4  
pt2 的 x 坐标为:2C052124BDD94E5645E99B01DCECA28D  
2)sn1,sn2 转变为大数,并变形  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:004015C0                 mov     esi, \[ebp+big\_sn1\]
> Un\_FSG\_\_:004015C6                 add     esp, 18h
> Un\_FSG\_\_:004015C9                 movsx   ebx, ds:Serial\_sign
> Un\_FSG\_\_:004015D0                 push    offset sn1
> Un\_FSG\_\_:004015D5                 sub     ebx, 30h
> Un\_FSG\_\_:004015D8                 push    esi
> Un\_FSG\_\_:004015D9                 lea     esi, \[ebp+var\_1A8\]
> Un\_FSG\_\_:004015DF                 call    \_cinstr
> Un\_FSG\_\_:004015E4                 pop     eax
> Un\_FSG\_\_:004015E5                 pop     edx
> Un\_FSG\_\_:004015E6                 mov     eax, \[ebp+big\_sn2\]
> Un\_FSG\_\_:004015EC                 push    offset sn2
> Un\_FSG\_\_:004015F1                 push    eax
> Un\_FSG\_\_:004015F2                 call    \_cinstr
> 
> * * *
> 
> 
> ```

  
其中 sn1 的第一个 byte 是正负标志位，剩下的部分转化为大数  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:004015F7                 mov     eax, \[ebp+big\_t\]
> Un\_FSG\_\_:004015FD                 push    eax
> Un\_FSG\_\_:004015FE                 mov     eax, \[ebp+p\]
> Un\_FSG\_\_:00401604                 push    eax
> Un\_FSG\_\_:00401605                 mov     eax, \[ebp+big\_sn1\]
> Un\_FSG\_\_:0040160B                 push    1
> Un\_FSG\_\_:0040160D                 push    eax
> Un\_FSG\_\_:0040160E                 call    \_powmod
> Un\_FSG\_\_:00401613                 mov     eax, \[ebp+big\_t\]
> Un\_FSG\_\_:00401619                 add     esp, 20h
> Un\_FSG\_\_:0040161C                 mov     edx, \[ebp+pt3\]
> Un\_FSG\_\_:00401622                 push    edx
> Un\_FSG\_\_:00401623                 push    ebx
> Un\_FSG\_\_:00401624                 xor     ebx, ebx
> Un\_FSG\_\_:00401626                 push    eax
> Un\_FSG\_\_:00401627                 mov     eax, \[ebp+big\_t\]
> Un\_FSG\_\_:0040162D                 push    eax
> Un\_FSG\_\_:0040162E                 call    \_epoint\_set     
> 
> * * *
> 
> 
> ```

  
pt3 的 x 坐标为 big\_sn1%p  
3)计算 name 的 sha 值,并变形  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:00401636                 call    \_shs\_init
> Un\_FSG\_\_:0040163B                 mov     al, ds:name
> Un\_FSG\_\_:00401640                 add     esp, 10h
> Un\_FSG\_\_:00401643                 test    al, al
> Un\_FSG\_\_:00401645                 jz      short loc\_401661
> Un\_FSG\_\_:00401647 
> Un\_FSG\_\_:00401647 loc\_401647:                             ; CODE XREF: \_check+27Fj
> Un\_FSG\_\_:00401647                 movsx   ecx, al
> Un\_FSG\_\_:0040164A                 push    edx
> Un\_FSG\_\_:0040164B                 inc     ebx
> Un\_FSG\_\_:0040164C                 push    edx
> Un\_FSG\_\_:0040164D                 push    ecx
> Un\_FSG\_\_:0040164E                 push    esi
> Un\_FSG\_\_:0040164F                 call    \_shs\_process
> Un\_FSG\_\_:00401654                 mov     al, ds:name\[ebx\]
> Un\_FSG\_\_:0040165A                 add     esp, 10h
> Un\_FSG\_\_:0040165D                 test    al, al
> Un\_FSG\_\_:0040165F                 jnz     short loc\_401647
> Un\_FSG\_\_:00401661 
> Un\_FSG\_\_:00401661 loc\_401661:                             ; CODE XREF: \_check+265j
> Un\_FSG\_\_:00401661                 push    ebx
> Un\_FSG\_\_:00401662                 push    ebx
> Un\_FSG\_\_:00401663                 lea     ebx, \[ebp+var\_38\]
> Un\_FSG\_\_:00401666                 push    ebx
> Un\_FSG\_\_:00401667                 push    esi
> Un\_FSG\_\_:00401668                 call    \_shs\_hash
> Un\_FSG\_\_:0040166D                 mov     edx, \[ebp+big\_hName\]
> Un\_FSG\_\_:00401673                 add     esp, 0Ch
> Un\_FSG\_\_:00401676                 push    edx
> Un\_FSG\_\_:00401677                 push    ebx
> Un\_FSG\_\_:00401678                 push    14h
> Un\_FSG\_\_:0040167A                 call    \_bytes\_to\_big
> Un\_FSG\_\_:0040167F                 mov     eax, \[ebp+big\_hName\]
> Un\_FSG\_\_:00401685                 push    eax
> Un\_FSG\_\_:00401686                 mov     eax, \[ebp+p2\]    ;FFFFFFFE000000006FAFED9C3353F08D
> Un\_FSG\_\_:0040168C                 push    eax
> Un\_FSG\_\_:0040168D                 mov     eax, \[ebp+big\_hName\]
> Un\_FSG\_\_:00401693                 push    3
> Un\_FSG\_\_:00401695                 push    eax
> Un\_FSG\_\_:00401696                 call    \_powmod
> 
> * * *
> 
> 
> ```

  
big\_hName=sha(name)³%p2  
p2 被初始化为 FFFFFFFE000000006FAFED9C3353F08D  
4)计算中间值  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:0040169B                 mov     eax, \[ebp+pt4\]
> Un\_FSG\_\_:004016A1                 add     esp, 1Ch
> Un\_FSG\_\_:004016A4                 push    eax
> Un\_FSG\_\_:004016A5                 mov     eax, \[ebp+pt3\]
> Un\_FSG\_\_:004016AB                 push    eax
> Un\_FSG\_\_:004016AC                 mov     eax, \[ebp+big\_sn2\]
> Un\_FSG\_\_:004016B2                 push    eax
> Un\_FSG\_\_:004016B3                 call    \_ecurve\_mult     ; pt4=pt3\*big\_sn2
> Un\_FSG\_\_:004016B8                 mov     eax, \[ebp+big\_hName\]
> Un\_FSG\_\_:004016BE                 add     esp, 0Ch
> Un\_FSG\_\_:004016C1                 push    edi
> Un\_FSG\_\_:004016C2                 push    edi
> Un\_FSG\_\_:004016C3                 push    eax
> Un\_FSG\_\_:004016C4                 call    \_ecurve\_mult     ; pt1=pt1\*big\_hName
> Un\_FSG\_\_:004016C9                 mov     ebx, \[ebp+big\_sn1\]
> Un\_FSG\_\_:004016CF                 add     esp, 0Ch
> Un\_FSG\_\_:004016D2                 mov     ecx, \[ebp+pt5\]
> Un\_FSG\_\_:004016D8                 mov     esi, \[ebp+pt2\]
> Un\_FSG\_\_:004016DE                 push    ecx
> Un\_FSG\_\_:004016DF                 push    esi
> Un\_FSG\_\_:004016E0                 push    ebx
> Un\_FSG\_\_:004016E1                 call    \_ecurve\_mult     ; pt5=pt2\*big\_sn1
> Un\_FSG\_\_:004016E6                 pop     eax
> Un\_FSG\_\_:004016E7                 pop     edx
> Un\_FSG\_\_:004016E8                 mov     edx, \[ebp+pt5\]
> Un\_FSG\_\_:004016EE                 push    edx
> Un\_FSG\_\_:004016EF                 push    edi
> Un\_FSG\_\_:004016F0                 call    \_ecurve\_add      ; pt5=pt5+pt1
> Un\_FSG\_\_:004016F0                                         ; pt5=pt1\*big\_hName+
> Un\_FSG\_\_:004016F0                                         ;        pt2\*big\_sn1
> 
> * * *
> 
> 
> ```

  
pt4=pt3\*big\_sn2  
pt5=pt5+pt1  
   =pt1\*big\_hName+pt2\*big\_sn1  
5)比较  
通过两处进行比较  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Un\_FSG\_\_:004016F5                 mov     eax, \[ebp+pt4\]
> Un\_FSG\_\_:004016FB                 pop     ecx
> Un\_FSG\_\_:004016FC                 pop     ebx
> Un\_FSG\_\_:004016FD                 push    eax
> Un\_FSG\_\_:004016FE                 mov     eax, \[ebp+pt5\]
> Un\_FSG\_\_:00401704                 push    eax
> Un\_FSG\_\_:00401705                 call    \_epoint\_comp     ; pt4=t5
> ...
> Un\_FSG\_\_:004017F5                 mov     eax, \[ebp+pt5\]
> Un\_FSG\_\_:004017FB                 push    edi
> Un\_FSG\_\_:004017FC                 push    eax
> Un\_FSG\_\_:004017FD                 call    \_epoint\_comp     ;pt5!=pt1
> 
> * * *
> 
> 
> ```

  
如果 pt4=pt5 且 pt5!=pt1 则通过  
3.Keygen  
最终只要 pt3\*big\_sn2=pt1\*big\_hName+pt2\*big\_sn1 即可通过检查  
其中 pt3 的 x 坐标为 big\_sn1%p  
big\_hName 是 name 的 hash 值变形得来，是不可逆的,同样 pt1,pt2 是给定的也做不了文章,能下手的就是 pt3,sn1,sn2 这几部分  
不妨试试,令 pt3=pt1+pt2,big\_sn2=big\_hName  
那么方程被简化为 pt2\*big\_hName=pt2\*big\_sn1  
椭圆曲线作为一个阶数为 n 的有限群，它上面任意一点满足 pt\*n=indefinite(因为元素的阶是群的阶的因子)。如果椭圆曲线不好理解，可以拿模质数的乘法群来类比。indefinite 相当于 1。在{1,2,3,4,5,6}这几个数构成的 mod 7 群里，群的阶(元素个数)为 6，任意一个数的 6 次方除以 7 均余 1 也就是那个单位元。  
所以只要 big\_sn1%n=big\_hName 即可  
那么最终我们得到了一组同余方程  
big\_sn1=pt3\_x   mod p  
big\_sn1=big\_hName  mod n  
得到 sn1  
利用 big\_sn2=big\_hName  
得到 sn2  
当时在 keygen 的时候我对 miracl 还不熟，我自己手工解这个同余方程，大家可以用 miracl 实现一个更好的 keygen 下面是 keygen 的链接  
**keygen** 

 4dd466d0e1e0cc123afbe504da2462ca