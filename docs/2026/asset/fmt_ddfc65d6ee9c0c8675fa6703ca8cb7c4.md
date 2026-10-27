
这个类在静态构造里解密字符串数据块，然后由它的各个静态方法如`x33()`返回各个解密后的字符串，`x332B4()`构造了解密类，`x33(byte[])`执行解密。

`x332B4()`中使用了`GetPublicKeyToken()`，因为我们改变了文件的强名，这个解密可能会不正确了（本来似乎应该一直不正确，不过我发现好象有时候IV不同也能正确解密，不知道是什么原因），我们要做的就是将静态构造里的`new byte[]`后面的数据先解出来，然后再用我们新的公钥token给它加密，再用加密后的数据替换原来的数据（`TripleDES`是对称算法，这里`KEY`已经给定，我们只是用公钥token替换`IV`重新加密而已）。

这个也是用`SnippetCompiler`来作了，完成后生成两个十六进字符串（就是可以直接在`hexworkshop`中搜索与替换用的十六进制串的格式），然后我们启动`hexworkshop`,先搜索原来的（没必要指定全部，一般前7到8个字节就唯一了），Select Block，输入数据块的长度，然后，Paste Special，选中“interpret as a hexadecimal string”，点OK，保存文件就可以了。

用同样的方法处理另外几个文件以及由`GetPublicKey`的引用发现的另外几处涉及加密字符串的文件（这是个比较烦人的工作，总共有六处字符串需要处理，也就有六个解密类，特别是`ILMOD.dll`，它的字符串数据有几十K，在拷贝时会倒致程序暂时失去响应）。

## 4、破解功能限制

加密字符串处理完，工作量的一大部分就完了，不过，我们的破解好象还没开始呢，汗。

### 1）破解反编译只有50%方法的限制

运行一下`spices.net`，用`C#`语言反编译任何类时总是只有约50%的方法编译，另外的则注释上了`//evaluation version`，仔细看一下，基本上是隔一个出现一个注释，因为注释使用了字符串，我们就不看看字符串引用的情况，找到我们破解加密字符串时生成的文件，找到与`evaluation version`对应的那个方法`x15.x4.x338E4()`，然后在`reflector.exe`里看一下对它的引用：

```
x15.x4.x338E4() : Object

        Depends On

        Used By

                NineRays.ILOMD.DecompilerManager..cctor()
```

然后我们看一下`DecompilerManager`的静态构造：

```
static DecompilerManager()

{

      DecompilerManager.x15 = (string) x4.x33();

      DecompilerManager.x36 = (string) x4.x338DC();

      DecompilerManager.x11 = (string) x4.x338DD();

      DecompilerManager.x2 = (string) x4.x338DE();

      DecompilerManager.x24 = (string) x4.x338DF();

      DecompilerManager.x28 = (string) x4.x338E0();

      DecompilerManager.x26 = (string) x4.x338E1();

      DecompilerManager.x29 = (string) x4.x338E2();

      DecompilerManager.x19 = (string) x4.x338E3();

      DecompilerManager.x34 = (string) x4.x338E4();<--这个是目标字符串

      DecompilerManager.x73 = (string) x4.x338E5();

}
```

发现这个字符串被赋给`x34`这个成员，我们再查`x34`的引用：

```
NineRays.ILOMD.DecompilerManager.x34 : String

        Used By

                NineRays.ILOMD.DecompilerManager..cctor()

                x15.x439.x15(x349, NamingConventions) : x453
```

看一下`x15`这个方法：

```
private x453 x15(x349 x3, NamingConventions conventions1)

{

      if (x3.x654())

      {

            return null;

      }

      x453 x1 = new x453(null);

      if (this.x24 || ((x3.x21() % 2) < 2))

      {

            x565 x2 = x3.x15();

            try

            {

                  new x441(x1, x3, x2, this, this.x15, conventions1, this.x15, this.x36);

            }

            catch (Exception exception1)

            {

                  x1.x15(new x509(((string) x4.x339CA()) + exception1.Message + ((string) x4.x338F2())));

                  x1.x15(new x509(((string) x4.x339CB()) + exception1.StackTrace + ((string) x4.x338F2())));

            }

            return x1;

      }

      x1.x15(new x509(DecompilerManager.x34));

      return x1;

}
```

可以看到最后第二句使用了前面的字符串，函数有三个出口，最后一个出口应该就是显示`evaluation version`的了，第二个是正常反编译的，看一下`if`的第二个条件，有一个`% 2 == 1`的操作，正好是隔一个编译一个，与我们前面观察到的特征符合，就是它了，改变的办法很简单，我们将`% 2 == 1`改为`% 2 < 2`这个条件就会永远为真了。接下来我们来看做这个修改的IL指令与字节码，在`reflector.exe`中切换到IL显示方式，同时用`ildasm`反编译原来的`NineRays.Decompiler.dll`,保存一个IL文本文件（要选上“显示字节”与“显示标记值”）。

用`research.net`找开反混淆过的DLL，因为`x439`这个类有许多方法叫`x15`，`research.net`在显示方法表时没有显示完整的方法原型，所以我们没办法直接找到方法的token，我们找这个类的第一个方法的token:`06000399`，用`ultraedit`找开我们在`ildasm`里反出来的文件，在其中查找这个标记值，定位到了`x439`类的第一个方法：

```
.method /\*06000399\*/ public hidebysig specialname rtspecialname

          instance void  .ctor(valuetype NineRays.ILOMD.DecompilerOptions/\*020000A5\*/ A\_1,

                               class NineRays.ILOMD.NamingConventions/\*02000015\*/ A\_2,

                               bool A\_3) cil managed

  // SIG: 20 03 01 11 82 94 12 54 02

  {

    // 方法在 RVA 0x2106c 处开始

    // 代码大小       148 (0x94)
```

然后我们以`.method`为搜索条件，在`ultraedit`里向后浏览各个方法，注意方法参数的第二个参数类型为`NineRays.ILOMD.NamingConventions`的方法：

```
.method /\*060003B7\*/ private hidebysig

          instance class ''.'3'/\*02000033\*/

          ''(class [NineRays.ILOMD/\*23000004\*/]''.'9'/\*01000046\*/ A\_1,

              class NineRays.ILOMD.NamingConventions/\*02000015\*/ A\_2) cil managed

  // SIG: 20 02 12 80 CC 12 81 19 12 54

  {

    // 方法在 RVA 0x236d0 处开始

    // 代码大小       172 (0xac)
```

对照`reflector.exe`里IL，我们找到`% 2 == 1`的地方：

```
    IL_001b:  /\* 6F   | (0A)00018E       \*/ callvirt   instance int32 [NineRays.ILOMD/\*23000004\*/]''.'3'/\*0100004D\*/::''() /\* 0A00018E \*/

    IL_0020:  /\* 18   |                  \*/ ldc.i4.2

    IL_0021:  /\* 5D   |                  \*/ rem

    IL_0022:  /\* 16   |                  \*/ ldc.i4.0

    IL_0023:  /\* FE01 |                  \*/ ceq

    IL_0025:  /\* 2B   | 01               \*/ br.s       IL_0028
```

我们要将它改为`% 2 < 2`则需要将

```
    IL_0022:  /\* 16   |                  \*/ ldc.i4.0

    IL_0023:  /\* FE01 |                  \*/ ceq
```

改为

```
                ldc.i4.2

                clt
```

对应的十六进制IL码为`18 FE 04`， （查找指令的指令码比较方便的方法是用MSDN看`.net`框架库参考里的`System.Reflection.Emit.OpCodes`结构的静态只读成员的注释，每个对应一条指令，说明也比较详细。）

现在用`hexworkshop`打开未反混淆的原程序，搜索`6F8E01000A185D16FE012B01`,如果不唯一就再往后多看几个(或者看一下是否在`RVA 0x236d0`附近[`ildasm`在方法开始给出RVA])，呵呵（注意`ildasm`显示的指令码中操作数部分不是文件顺序而32位值的显示顺序，如上面的（`0A`）`00018E`实际上在文件应反过来`8E01000A`），找到后将其中的`16 FE 01`改为`18 FE 04`，保存即可。

### 2）破解不能反编译spices.net处理过的文件的限制

`Spices.net`在反编译它自己时会提示`...protected from decompilation by ...`然后就反编译了，这个与上面一样的方法破解，还是查一下这个相关字符串的解密函数`x15.x4.x339C6()`，在`reflector`中查一下引用：

```
x15.x4.x339C6() : Object

        Depends On

        Used By

                x15.x439..cctor()
```

来到`x15.x439`的静态构造：

```
static x439()

{

      x439.x15 = (string) x4.x339C5();

      x439.x36 = ((string) x4.x339C6()) + x439.x15 + ((string) x4.x339C7());<--这个包含目标字符串

      x439.x11 = (string) x4.x339C8();

}
```

再查`x36`成员的引用：

```
x15.x439.x36 : String

        Used By

                x15.x439..cctor()

                x15.x439.x15(CodeCompileUnit, String, x351[]) : Void

                x15.x439.x15(x351) : x493

                x15.x439.x15(x351, x349, Boolean) : CodeTypeMember

                x15.x439.x36(x16) : CodeCompileUnit
```

这个引用的次数比较多，我们打开跳到其中的一个看一下，`x15.x439.x15(x351)`：

```
private x493 x15(x351 x4)

{

      if (!x4.x658())

      {

            x493 x3 = new x493(x4.x9());

            if (this.x15((x68) x4) || this.x15((x68) x4.x36()))

            {

                  x3.Comments.Add(new CodeCommentStatement(string.Format(x439.x36, x3.Name)));

                  return x3;

            }

...
```

可以看到在一个条件判断后使用了目标字符串，然后函数返回，我们看一下这个条件判断使用的函数`x15`：

```
private bool x15(x68 x3)

{

      foreach (x563 x1 in x3.x15())

      {

            x351 x2 = x1.x36();

            if (x2.x26().EndsWith(x439.x15))

            {

                  return true;

            }

      }

      return false;

}
```

可以看到是判断以一个字符串结尾时返回true倒致不编译的，`x439.x15`这个字符串在静态构造里使用`x4.x339c5()`初始化，查一下我们先前解密的字符串，是“NineRays.Decompiler.NotDecompile”，这是一个自定义属性，可见`Spices.net`对这个属性的方法或类型不反编译，解决办法很简单，将`return true`改为`return flase`即可。如何将这些改动应用于原执行文件与1）中相同，这里就不多说了。前面说过这个字符串的引用比较多，我们改了一处就可以了吗？我们看一下我们改过的`x15`方法的引用：

```
x15.x439.x15(x68) : Boolean

        Depends On

        Used By

                x15.x439.x15(CodeCompileUnit, String, x351[]) : Void

                x15.x439.x15(x351) : x493

                x15.x439.x15(x351, x349, Boolean) : CodeTypeMember

                x15.x439.x36(x16) : CodeCompileUnit
```

可以看到正是上面四处引用了这个方法，也就是改这一处就可以了，我们也可以一个一个方法的检查一下，确实如此，呵呵。

### 3）破解PE/metadata功能里显示evaluation的限制

`spices.net`的`PE/metadata`功能是与`research.net`相似的查看`.net` PE文件格式的功能，有些方面比`research.net`做得更出色，不过在评估版里很多元数据项目写成了`evaluation`，我们现在就是要破解这个限制，当然我们也可以用1）2）中的字符串找来寻找关键点，不过这个模块的字符串没有用强名信息加密，所以我们先前没有将这些字符串解密出来，字符串解密毕竟太烦了，我们尝试看有没有别的办法来找到这个关键点。经过观察发现`evaluation`似乎是随机出现的，没有什么规律。不过，这本来就是一个规律了，随机？在`.net`里取随机数的类是`System.Random`，我们就来看看程序里有哪些地方使用了这个类：

```
System.Random

        Depends On

        Used By

                x11.x36..ctor(x87, x370, x26, Options, Hashtable)

                x11.x36.x11 : Random

                x11.x36.x11(x351, String[]&) : Void

                x15.x362.x15(String) : String

                x15.x362+x19..ctor(Random)

                x15.x362+x19.Compare(Object, Object) : Int32

                x15.x362+x19.x15 : Random

                x15.x605.x34() : String
```
