

# 求解反函数(.net 字符串儿加解密)

> 原文：[`www.pediy.com/kssd/pediy10/90723.html`](https://www.pediy.com/kssd/pediy10/90723.html)

下面是一个经流程混淆后的 IL 代码,用于解密字符串儿。常见于 xenocode 混淆后的代码中。这是最近的一个软件中摘出来的。对于破解，求它的反函数多此一举，此处仅作学习探讨。  
希望高手能够给出它的反函数，也即加密函数。（本函数的作者是如何生成那些加密串儿的）  
解密函数用例：  
  string tmpStr=x9919("ckoe",280841806);  
   则：tmpStr=="T"  
  
那么：  
现有一字串儿“The license is not valid for this product.”  intkey=280841806  
如何加密成字串儿  
"ckoedlffnkmffgdgokkgikbhpjihojphekgigknifjejneljdjckkjjkeealpihlniolpifmidmmlidndhknlhbofhiongpogcgpjgnppgeapglakbcblgjbmfackfhcbgoclafdifmdhfdebfkedebfbfifmdpfkeggbang"  
  
求下面函数的反函数！  
  
IL 解密函数：  

代码:

```
.method public hidebysig static string 
          x9919(string x12346\_1,
                int32 x12347\_2) cil managed
  {
    // 代码大小       158 (0x9e)
      .maxstack  5
      .locals init (unsigned int16 V\_0,
               char\[\] V\_1,
               int32 V\_2,
               unsigned int16 V\_3)
      IL\_0000:  ldarg.1           //参数 1 入栈，int32 x12347\_2
      IL\_0001:  conv.u2           //将位于计算堆栈顶部的值转换为 unsigned int16，然后将其扩展为 int32 
      IL\_0002:  br.s       IL\_0060 //跳转到标签处

      IL\_0004:  add         
      IL\_0005:  ldarg.0
      IL\_0006:  br.s       IL\_0024

      IL\_0008:  conv.u2
      IL\_0009:  stloc.3
      IL\_000a:  ldloc.1
      IL\_000b:  ldloc.2
      IL\_000c:  ldloc.3
      IL\_000d:  stelem.i2
      IL\_000e:  br.s       IL\_0016

      IL\_0010:  stloc.0
      IL\_0011:  ldloc.2
      IL\_0012:  ldc.i4.1
      IL\_0013:  add
      IL\_0014:  br.s       IL\_0020

      IL\_0016:  ldloc.0
      IL\_0017:  ldc.i4     0x6fd
      IL\_001c:  add
      IL\_001d:  conv.u2
      IL\_001e:  br.s       IL\_0010

      IL\_0020:  stloc.2
      IL\_0021:  ldloc.2   //V\_2 入栈,int32 V\_2,第一次到此处时为 0
      IL\_0022:  br.s       IL\_003c  ////跳到标签处

      IL\_0024:  ldc.i4.4
      IL\_0025:  ldloc.2
      IL\_0026:  mul
      IL\_0027:  ldc.i4.3
      IL\_0028:  add
      IL\_0029:  callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32)
      IL\_002e:  ldc.i4.s   97
      IL\_0030:  sub
      IL\_0031:  ldc.i4.s   12
      IL\_0033:  shl
      IL\_0034:  add
      IL\_0035:  conv.u2
      IL\_0036:  stloc.3
      IL\_0037:  ldloc.3
      IL\_0038:  ldloc.0
      IL\_0039:  sub
      IL\_003a:  br.s       IL\_0008

      IL\_003c:  ldarg.0   //参数 0 入栈,那个 string
      IL\_003d:  callvirt   instance int32 \[mscorlib\]System.String::get\_Length() //又调用求串儿长度
      IL\_0042:  ldc.i4.4  ////将整数值 4 作为 int32 推送到计算堆栈上 //这段儿和标签 IL\_0061 及其后的作用一样，好象，往下跑着看
      IL\_0043:  div        //除，上面的串儿长度除以 4?
      IL\_0044:  blt.s      IL\_006c //小于跳转

      IL\_0046:  ldloc.1
      IL\_0047:  br.s       IL\_0098

      IL\_0049:  sub   //减 97
      IL\_004a:  ldarg.0 //参数 0 入栈,那个 string
      IL\_004b:  ldc.i4.4 //将整数值 4 作为 int32 推送到计算堆栈上
      IL\_004c:  ldloc.2 ///V\_2 入栈,int32 V\_2
      IL\_004d:  mul     //乘  4\*V\_2 入栈
      IL\_004e:  ldc.i4.1  //将整数值 1 作为 int32 推送到计算堆栈上
      IL\_004f:  add       //加 4\*V\_2+1
      IL\_0050:  callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
      IL\_0055:  ldc.i4.s   97 //将 num 作为 int32 推送到堆栈上（短格式）。
      IL\_0057:  sub          //减
      IL\_0058:  ldc.i4.4     //将整数值 4 作为 int32 推送到计算堆栈上
      IL\_0059:  shl          //左移,移上面指出的 4 位，后面补 0
      IL\_005a:  add          //加
      IL\_005b:  ldarg.0        ////参数 0 入栈,那个 string
      IL\_005c:  ldc.i4.4      //将整数值 4 作为 int32 推送到计算堆栈上
      IL\_005d:  ldloc.2       //V\_2 入栈,int32 V\_2
      IL\_005e:  br.s       IL\_0086

      IL\_0060:  stloc.0     //出栈到 V\_0 int16 型
      IL\_0061:  ldarg.0     //参数 0 入栈,那个入参 string x12346\_1
      IL\_0062:  callvirt   instance int32 \[mscorlib\]System.String::get\_Length() //调用求字符串儿长度函数
      IL\_0067:  ldc.i4.4    //将整数值 4 作为 int32 推送到计算堆栈上
      IL\_0068:  br.s       IL\_0072 //跳到标签处

      IL\_006a:  br.s       IL\_0021 //跳到标签处

      IL\_006c:  ldarg.0   ////参数 0 入栈,那个 string
      IL\_006d:  ldc.i4.4  //将整数值 4 作为 int32 推送到计算堆栈上
      IL\_006e:  ldloc.2   //V\_2 入栈,int32 V\_2
      IL\_006f:  mul       //乘  4\*V\_2 入栈
      IL\_0070:  br.s       IL\_007d //跳转

      IL\_0072:  div        //除，上面的串儿长度除以 4?
      IL\_0073:  newarr     \[mscorlib\]System.Char //Create a zero-base, on-dimensional array
      IL\_0078:  stloc.1    //出栈到 V\_1，那个 char\[\]数组
      IL\_0079:  ldc.i4.0   //将整数值 0 作为 int32 推送到计算堆栈上
      IL\_007a:  stloc.2    //出栈到 V\_2,那个 int32 V\_2 的临时变量
      IL\_007b:  br.s       IL\_006a //跳到标签处

      IL\_007d:  callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
      IL\_0082:  ldc.i4.s   97   //将 num 作为 int32 推送到堆栈上（短格式）。
      IL\_0084:  br.s       IL\_0049 //跳到标签处

      IL\_0086:  mul       //从栈取两值乘，结果入栈
      IL\_0087:  ldc.i4.2  //将整数值 2 作为 int32 推送到计算堆栈上
      IL\_0088:  add       //从栈取两值加，结果入栈
      IL\_0089:  callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
      IL\_008e:  ldc.i4.s   97  //将 num 作为 int32 推送到堆栈上（短格式）。
      IL\_0090:  sub        //减
      IL\_0091:  ldc.i4.8   //将整数值 8 作为 int32 推送到计算堆栈上
      IL\_0092:  shl         //左移 8 位
      IL\_0093:  br         IL\_0004 //跳到标签处

      IL\_0098:  newobj     instance void \[mscorlib\]System.String::.ctor(char\[\])
      IL\_009d:  ret
  } // end of method Form1::x9919

```

虽然我最擅长的领域不是算法，但偶尔客串一下还是可以的。（上次做“有道难题”的在线答题，只得了一个及格分......不爽，所以希望我的回答能满足搂主的需求。）  
我希望大家可以把 DONET 发扬光大，让看雪开一个 DONET 板块是最终目的！  
希望大家能讨论更多的 DONET 问题。  

代码:

```
        private string GetEncyptString(string inputString, int intKey)
        {
            ushort num = (ushort)intKey;
            char\[\] chArray = new char\[4 \* inputString.Length\];
            for (int i = 0; i < inputString.Length; i++)
            {
                ushort num3 = (ushort)(inputString\[i\] + num);
                string str = new string(new char\[\] { this.GetNewChar((byte)(num3 & 15)), this.GetNewChar((byte)((num3 >> 4) & 15)), this.GetNewChar((byte)((num3 >> 8) & 15)), this.GetNewChar((byte)((num3 >> 12) & 15)) });
                chArray\[4 \* i\] = str\[0\];
                chArray\[(4 \* i) + 1\] = str\[1\];
                chArray\[(4 \* i) + 2\] = str\[2\];
                chArray\[(4 \* i) + 3\] = str\[3\];
                num = (ushort)(num + 0x6fd);
            }
            return new string(chArray);
        }

        private char GetNewChar(byte value)
        {
            return (char)(0x61 + value);
        }

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || DEMO.rar  
 |

一看楼上的算法就知道不用验证了。牛人，正解！想不到会这么快，佩服！  
在阅读看雪帖子的过程中，对楼上印象深刻，还望以后多多指教。  
  
下面是手动反流程后的 IL 代码。只要象个精干的妇人收拾屋子般，把它们规整的有序而决不多余就成了。  
![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")  

代码:

```
.method public hidebysig static string 
          x9919(string x12346\_1,
                int32 x12347\_2) cil managed
  {
    // 代码大小       158 (0x9e)
      .maxstack  5
      .locals init (unsigned int16 V\_0,
               char\[\] V\_1,
               int32 V\_2,
               unsigned int16 V\_3)
        ldarg.1           //参数 1 入栈，int32 x12347\_2
        conv.u2           //将位于计算堆栈顶部的值转换为 unsigned int16，然后将其扩展为 int32 
        stloc.0     //出栈到 V\_0 int16 型
        ldarg.0     //参数 0 入栈,那个入参 string x12346\_1
        callvirt   instance int32 \[mscorlib\]System.String::get\_Length() //调用求字符串儿长度函数
        ldc.i4.4    //将整数值 4 作为 int32 推送到计算堆栈上
         div        //除，上面的串儿长度除以 4?
        newarr     \[mscorlib\]System.Char //Create a zero-base, on-dimensional array
        stloc.1    //出栈到 V\_1，那个 char\[\]数组
        ldc.i4.0   //将整数值 0 作为 int32 推送到计算堆栈上
        stloc.2    //出栈到 V\_2,那个 int32 V\_2 的临时变量
       IL\_WHILE: ldloc.2   //V\_2 入栈,int32 V\_2,第一次到此处时为 0
     
        ldarg.0   //参数 0 入栈,那个 string
        callvirt   instance int32 \[mscorlib\]System.String::get\_Length() //又调用求串儿长度
        ldc.i4.4  ////将整数值 4 作为 int32 推送到计算堆栈上 //这段儿和标签 IL\_0061 及其后的作用一样，好象，往下跑着看
        div        //除，上面的串儿长度除以 4?
        blt.s      IL\_IF //小于跳转

        ldloc.1
        br.s       IL\_END
      
      IL\_IF:  ldarg.0   ////参数 0 入栈,那个 string
        ldc.i4.4  //将整数值 4 作为 int32 推送到计算堆栈上
        ldloc.2   //V\_2 入栈,int32 V\_2
        mul       //乘  4\*V\_2 入栈
         callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
        ldc.i4.s   97   //将 num 作为 int32 推送到堆栈上（短格式）。
         sub   //减 97
       ldarg.0 //参数 0 入栈,那个 string
        ldc.i4.4 //将整数值 4 作为 int32 推送到计算堆栈上
        ldloc.2 ///V\_2 入栈,int32 V\_2
        mul     //乘  4\*V\_2 入栈
        ldc.i4.1  //将整数值 1 作为 int32 推送到计算堆栈上
        add       //加 4\*V\_2+1
        callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
        ldc.i4.s   97 //将 num 作为 int32 推送到堆栈上（短格式）。
        sub          //减
        ldc.i4.4     //将整数值 4 作为 int32 推送到计算堆栈上
        shl          //左移,移上面指出的 4 位，后面补 0
        add          //加
        ldarg.0        ////参数 0 入栈,那个 string
        ldc.i4.4      //将整数值 4 作为 int32 推送到计算堆栈上
        ldloc.2       //V\_2 入栈,int32 V\_2
        mul       //从栈取两值乘，结果入栈
       ldc.i4.2  //将整数值 2 作为 int32 推送到计算堆栈上
        add       //从栈取两值加，结果入栈
        callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32) //据一个 int32 值取它对应的字符
        ldc.i4.s   97  //将 num 作为 int32 推送到堆栈上（短格式）。
        sub        //减
        ldc.i4.8   //将整数值 8 作为 int32 推送到计算堆栈上
        shl         //左移 8 位
        add         
        ldarg.0
        ldc.i4.4
        ldloc.2
        mul
        ldc.i4.3
        add
        callvirt   instance char \[mscorlib\]System.String::get\_Chars(int32)
        ldc.i4.s   97
        sub
        ldc.i4.s   12
        shl
        add
        conv.u2
        stloc.3
       ldloc.3
       ldloc.0
       sub
        conv.u2
        stloc.3
        ldloc.1
        ldloc.2
        ldloc.3
        stelem.i2
       ldloc.0
       ldc.i4     0x6fd
        add
        conv.u2
        stloc.0
        ldloc.2
        ldc.i4.1
        add
        stloc.2
       // ldloc.2   //V\_2 入栈,int32 V\_2,第一次到此处时为 0
        br.s       IL\_WHILE  ////跳到标签处
      
      IL\_END:  newobj     instance void \[mscorlib\]System.String::.ctor(char\[\])
      ret
  } // end of method Form1::x9919

```

  
现在可以看到可读性强的代码了，比如 C#：  
代码:

```
private static string x9919(string x12346\_1, int x12347\_2)
{
    ushort num = (ushort) x12347\_2;
    char\[\] chArray = new char\[x12346\_1.Length / 4\];
    for (int i = 0; i < (x12346\_1.Length / 4); i++)
    {
        ushort num3 = (ushort) ((((x12346\_1\[4 \* i\] - 'a') + ((x12346\_1\[(4 \* i) + 1\] - 0x61) << 4)) + ((x12346\_1\[(4 \* i) + 2\] - 0x61) << 8)) + ((x12346\_1\[(4 \* i) + 3\] - 0x61) << 12));
        num3 = (ushort) (num3 - num);
        chArray\[i\] = (char) num3;
        num = (ushort) (num + 0x6fd);
    }
    return new string(chArray);
}

```

那么对比一下楼上的反函数。。。感觉那个---美！