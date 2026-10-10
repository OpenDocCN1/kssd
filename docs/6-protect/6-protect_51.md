

# 常用 ShellCode Hash 算法-Delphi 内镶 BASM

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-679.htm`](https://www.pediy.com/kssd/pediy08/pediy8-679.htm)

看到北极星 GG~发布了这么多代码/..小弟自感惭愧..  
最近精神好一点..也捣鼓了一点点东西...  
看了一下 ShellCode  
发现有许多不同的字符串 Hash..感觉蛮有意思的  
在这里发出来一下  
  
ShellCode 中应用最多的 hash 算法...  
  
1.ror 13 算法--刚开始用 Pascal 写.发现里面居然没有循环移位.指令唉~~  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> //与$D 作循环移位计算
> function GetRORHashEx(StrAPI:Pchar):DWORD;
> var
>   I:Integer;
> begin
>   Result := 0;
>   for I:= 0 to Length(StrAPI)-1 do
>   begin
>     asm
>       ror Result,$D
>     end;
>     Result := Result + ord(StrAPI\[I\]);
>   end;
> end;
> 
> * * *
> 
> 
> ```

  
  
PEID 不知道是什么原理居然可以扫描出来算法..Poly Crc32?  
  
2.Crc32b 算法---小弟只在 Coban2k 的 Hash Unit 中看到过~Crc32a 算法  
但是感觉差不多~估计是因为有重复 Hash 出现所以开发了 b 算法  
一下代码是..从 zhengxin 的 病毒代码中提取出来的  
  
这里抱怨一下...BASM 真难用..字符串赋值老是出错...  
估计可能是..我用的 Delphi 6 的问题..郁闷的说  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> //Crc32b 算法
> function GetCrc32bHash(StrAPI:PChar):DWORD;
> asm
>   mov     esi, eax//算法太经典了~
>   xor     eax, eax
>   dec     eax
>   mov     edx, esi
> @@1:
>   cmp     byte ptr \[edx\], 0//判断字符串是否是 00..是的话就结束
>   je      @@4
>   xor     al, \[edx\]
>   mov     bl, 8
> @@2:
>   shr     eax, 1
>   jnc     @@3
>   xor     eax, 0EDB88320h//太熟悉了~~
> @@3:
>   dec     bl
>   jnz     @@2
>   inc     edx
>   jmp     @@1
> @@4:
>   not     eax
>   mov     Result,eax
> end;
> 
> * * *
> 
> 
> ```

  
  
3.xCrc32 算法--一种 Crc32b 的变型算法...这个算法是从  
z0mbie 的 Crc 算法库中提取出来的..自己加工了一下..  
..呵呵~~不知道为什么在 Delphi 10 下就可以象我想的这么跑了  
但是不知道为什么 Delphi 6 就不正常了~~居然会把  
第二个参数放在 edx 中??哪位高手能告诉我为什么会这样??  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> //xCrc32 算法
> function GetxCrc32Hash(StrAPI:PChar;StrSize:Integer):DWORD;
> asm
>   mov     ecx, edx            //将数据长度保存在 ECX 中
>   add     ecx, 1              //由于 char \*是以 00 结束的所以这里多循环一次
>   mov     edx, StrAPI         //将 StrAPI 指针保存在 edx 中...不知道为什么汇编参数不好用~
> 
>   jecxz   @@4                 //是否循环完毕~ecx 为 0 就.跳!
>   xor     eax, eax            //EAX 清 0
>   not     eax                 //翻转 EAX---EAX=$FFFFFFFF
> @@1:
>   xor     al, \[edx\]
>   inc     edx
>   mov     bl, 8
> @@2:
>   shr     eax, 1
>   jnc     @@3
>   xor     eax, 0EDB88320h
> @@3:
>   dec     bl
>   jnz     @@2
>   loop    @@1
>   not     eax
> @@4:
>   mov     Result,eax
> end;
> 
> * * *
> 
> 
> ```

  
  
附件附原代码+工程：Hash.rar  
希望大家批评.. 

 4dd466d0e1e0cc123afbe504da2462ca