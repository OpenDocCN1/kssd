
# 维克仓库管理系统2.7简单分析+注册机

【破解作者】 qfejj

【作者邮箱】 qfejj@163.com

【使用工具】 `OllyDbgv1.10`;`PEIDv0.93`;`MASM32`

【破解平台】 `WindowsXP`

【软件名称】 维克仓库管理系统2.7(build20050530)

【下载地址】 bb1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4J5j5#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3@1p5^5x3W2)9J5c8X3k6A6L8r3g2K6i4K6u0r3N6X3E0U0K9#2)9J5k6h3g2^5k6b7``.``

【软件简介】 帮助用户 管理出入库业务

【软件大小】 3.2

【加壳方式】 无

【破解声明】 本人破解软件无任何商业目的,纯粹兴趣使然

--------------------------------------------------------------------------------

## 【破解内容】

维克仓库管理系统2.7单机板,5月30号的新版本。修改别人的软件总是不好的,再说这个软件好歹也是我们杭州人自己写的，我总不能不给他们面子。所以软件我是绝对不会去改的，但写个注册机却也无妨。

软件马马虎虎，加密形同虚设。

软件是delphi写的(没有加壳,有点失败!)，这也太自信了吧？

还是简单说一下软件的加密算法(明文比较，有点失败!),至于怎么找到算法部分我就不说了。

大致过程如下：

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

换算                                以前面得到的地址，从固定字串中取出一个值，作为注册码

用户名-------->得到结果，参与字串的取值运算----------------------------------------------------->注册码

其中，固定字串 ASCII "`1B2D3F4H5J6L7N8P9R8T7V6X5Z41W3F5Y7R9Q1C2E3G4I5K6M7B8Q7SYU5W8Y9`"，我写注册机时定义在了常量中，这个字串在调试时会明文出现。

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

要获得注册码，关键是获得定位固定字串元素的地址，用户名换算用了很多不同的算法，根据用户输入的序列号分别采用不同的运算，输入的序列号位数不同，算法就不同。不过都是大同小异，调试时跟着慢走，就全都清晰了。

注释中`005A7C84`固定字串附近都是计算注册码的计算过程，视输入的序列号不同来选择不同的计算过程。每次取固定字串前的运算都是为了获得指向字串元素的地址。(我写的注册机里没有使用软件全部的注册算法，为了能使最后的注册码结果正确，我自做主张地把用户名转化得到固定位数的序列号,实际情况并不是一个用户名对应一个序列号，要是换了其他的序列号，只要能和注册码对应起来，一样可以完成注册)

下面是算法的简单分析，算法不难，看我的注册机源码会觉得更简单(函数`GetRegistKey proc`,至于前面的花哨东西大可不去理会)。如果要自己写注册机，`mov edx,005A7C84` 这句前面的代码几乎是不用修改就可以拿来用，当然了，变量还是要重设的，我自己倒是改了点。

;----------------------------------------------------------------------------------------------------------

```
005A7AA7      lea edx,dword ptr ds:[ebx+ebx*2]     ;初始值为0,我注册机里设为变量CmpTime
005A7AAA      mov eax,edx
005A7AAC      add eax,3
005A7AAF      cmp eax,dword ptr ss:[ebp-C]
005A7AB2      jg vkckmini.005A7B52
005A7AB8      mov eax,edx
005A7ABA      inc eax
005A7ABB      mov ecx,dword ptr ss:[ebp-4]         ;  堆栈的地址是输入的序列号存放处的地址
005A7ABE      movzx eax,byte ptr ds:[ecx+eax-1]    ;  取出注册码值(ASCII)
005A7AC3      shr eax,2                            ;  逻辑右移,后面取值要用到
005A7AC6      mov ecx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7ACB      mov al,byte ptr ds:[ecx+eax]         ;  取固定字串中的一个值，地址由前面ecx和eax决定
005A7ACE      mov byte ptr ss:[ebp-D],al           ;  保存结果作为注册码的其中一位
005A7AD1      lea eax,dword ptr ds:[ebx+ebx*2]
005A7AD4      inc eax
005A7AD5      mov ecx,dword ptr ss:[ebp-4]         ;  堆栈的地址是输入的序列号存放处的地址
005A7AD8      mov al,byte ptr ds:[ecx+eax-1]       ;  取单个注册码ASCII值
005A7ADC      shl eax,4
005A7ADF      and al,30
005A7AE1      add edx,2
005A7AE4      mov ecx,dword ptr ss:[ebp-4]         ;  堆栈的地址是输入的序列号存放处的地址
005A7AE7      movzx edx,byte ptr ds:[ecx+edx-1]    ;  取单个注册码ASCII值
005A7AEC      shr edx,4                            ;
005A7AEF      or al,dl
005A7AF1      and eax,0FF                          ;  后面取值要用到eax
005A7AF6      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7AFB      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7AFE      mov byte ptr ss:[ebp-E],al           ;  保存结果作为注册码的其中一位
005A7B01      lea eax,dword ptr ds:[ebx+ebx*2]
005A7B04      add eax,2
005A7B07      mov edx,dword ptr ss:[ebp-4]         ;  堆栈的地址是输入的序列号存放处的地址
005A7B0A      mov al,byte ptr ds:[edx+eax-1]       ;  取出单个注册码值(ASCII)
005A7B0E      shl eax,2                            ;
005A7B11      and al,3C
005A7B13      lea edx,dword ptr ds:[ebx+ebx*2]     ;
005A7B16      add edx,3
005A7B19      mov ecx,dword ptr ss:[ebp-4]         ;  堆栈的地址是输入的序列号存放处的地址
005A7B1C      mov dl,byte ptr ds:[ecx+edx-1]
005A7B20      xor ecx,ecx
005A7B22      mov cl,dl
005A7B24      shr ecx,6
005A7B27      or al,cl
005A7B29      and eax,0FF
005A7B2E      mov ecx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7B33      mov al,byte ptr ds:[ecx+eax]         ;  取固定字串中的一个值
005A7B36      mov byte ptr ss:[ebp-F],al           ;  保存结果作为注册码的其中一位
005A7B39      mov eax,edx
005A7B3B      and al,3F
005A7B3D      and eax,0FF
005A7B42      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7B47      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7B4A      mov byte ptr ss:[ebp-10],al          ;  保存结果作为注册码的其中一位
005A7B4D      jmp vkckmini.005A7BF9
005A7B52      mov eax,edx
005A7B54      add eax,2
005A7B57      cmp eax,dword ptr ss:[ebp-C]
005A7B5A      jg short vkckmini.005A7BC2
005A7B5C      inc edx
005A7B5D      mov eax,dword ptr ss:[ebp-4]
005A7B60      movzx eax,byte ptr ds:[eax+edx-1]
005A7B65      shr eax,2
005A7B68      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7B6D      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7B70      mov byte ptr ss:[ebp-D],al           ;  保存结果作为注册码的其中一位
005A7B73      lea eax,dword ptr ds:[ebx+ebx*2]
005A7B76      inc eax
005A7B77      mov edx,dword ptr ss:[ebp-4]
005A7B7A      mov al,byte ptr ds:[edx+eax-1]
005A7B7E      shl eax,4
005A7B81      and al,30
005A7B83      lea edx,dword ptr ds:[ebx+ebx*2]
005A7B86      add edx,2
005A7B89      mov ecx,dword ptr ss:[ebp-4]
005A7B8C      mov dl,byte ptr ds:[ecx+edx-1]
005A7B90      xor ecx,ecx
005A7B92      mov cl,dl
005A7B94      shr ecx,4
005A7B97      or al,cl
005A7B99      and eax,0FF
005A7B9E      mov ecx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7BA3      mov al,byte ptr ds:[ecx+eax]         ;  取固定字串中的一个值
005A7BA6      mov byte ptr ss:[ebp-E],al           ;  保存结果作为注册码的其中一位
005A7BA9      mov eax,edx
005A7BAB      shl eax,2
005A7BAE      and al,3C
005A7BB0      and eax,0FF
005A7BB5      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7BBA      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7BBD      mov byte ptr ss:[ebp-F],al           ;  保存结果作为注册码的其中一位
005A7BC0      jmp short vkckmini.005A7BF9
005A7BC2      inc edx
005A7BC3      mov eax,dword ptr ss:[ebp-4]
005A7BC6      movzx eax,byte ptr ds:[eax+edx-1]
005A7BCB      shr eax,2
005A7BCE      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7BD3      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7BD6      mov byte ptr ss:[ebp-D],al           ;  保存结果作为注册码的其中一位
005A7BD9      lea eax,dword ptr ds:[ebx+ebx*2]
005A7BDC      inc eax
005A7BDD      mov edx,dword ptr ss:[ebp-4]
005A7BE0      mov al,byte ptr ds:[edx+eax-1]
005A7BE4      shl eax,4
005A7BE7      and al,30
005A7BE9      and eax,0FF
005A7BEE      mov edx,vkckmini.005A7C84            ;  005A7C84是固定字串的地址
005A7BF3      mov al,byte ptr ds:[edx+eax]         ;  取固定字串中的一个值
005A7BF6      mov byte ptr ss:[ebp-E],al           ;  保存结果作为注册码的其中一位
005A7BF9      mov eax,dword ptr ss:[ebp-8]         ;  下面的代码不在算法中，不必理会
005A7BFC      push dword ptr ds:[eax]
005A7BFE      lea eax,dword ptr ss:[ebp-14]
005A7C01      mov dl,byte ptr ss:[ebp-D]
005A7C04      call vkckmini.00404E5C
005A7C09      push dword ptr ss:[ebp-14]
005A7C0C      lea eax,dword ptr ss:[ebp-18]
005A7C0F      mov dl,byte ptr ss:[ebp-E]
005A7C12      call vkckmini.00404E5C
005A7C17      push dword ptr ss:[ebp-18]
005A7C1A      lea eax,dword ptr ss:[ebp-1C]
005A7C1D      mov dl,byte ptr ss:[ebp-F]
005A7C20      call vkckmini.00404E5C
005A7C25      push dword ptr ss:[ebp-1C]
005A7C28      lea eax,dword ptr ss:[ebp-20]
005A7C2B      mov dl,byte ptr ss:[ebp-10]
005A7C2E      call vkckmini.00404E5C
005A7C33      push dword ptr ss:[ebp-20]
005A7C36      mov eax,dword ptr ss:[ebp-8]
005A7C39      mov edx,5
005A7C3E      call vkckmini.00404FF4
005A7C43      inc ebx
005A7C44      dec esi
005A7C45      jnz vkckmini.005A7AA7
```

附上注册机源码，以供参考。不当之处，请各位指正！

--------------------------------------------------------------------------------

## 【汇编注册机】

;-------------------------------------------------------------------------------

;编译命令:

;`ml /c /coff keygen,asm`

;`rc keygen.rc`
