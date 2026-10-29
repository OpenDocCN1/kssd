
# 用Javassis分析jeb.jar

关于`jeb.jar`，不知上下文而又对本文感兴趣的兄弟，参看:

`http://bbs.pediy.com/showthread.php?t=189980`

`jeb.jar`的逆向工作其实已经阶段性中止了。有人提出能不能补上"Save As"功能，我是业余时间看看，如果它的功能代码本来就存在的话，等我有时间就来补上，如果不存在，以我的功力就不要指望太多了。

我最初是`Eclipse`、`BTrace`、`JD-GUI`、`Source Insight`相结合着把`jeb.jar`剁了，向诸君推荐`BTrace`时抱怨过一句:

`BTrace`还是不够狠，不能修改参数和返回值，只能查看，可以调用`public method`，可以occan直接修改`public member`，但仍受`private`的限制。如果这些都能解决该多好。

针对我这个抱怨，有人向我推荐了`Javassist`。怀着将信将疑的态度研究了一下`Javassis`。

`Javassist`和`BTrace`的侧重不同，仔细测试前者之后感觉前者与后者可以互补。前者在修改`class`方面足够强大，但在动态注入方面颇为鸡肋，它的`HotSwapper`实现远不如`BTrace`实用，几乎是个演示性玩具。假设目标进程已经以`jdwp`方式启动，而`Javassist`进程事后通过`HotSwapper`类`Attach`上去，`reload()`时会在`reload2()`中抛出"`failed to reload`"异常。如果二者是合兵一处的，就可以`reload()`。

如果不考虑动态注入，`Javassis`确实是个`NB`工具，不是一般`NB`，是相当`NB`。不过要想爽它，建议还是把它自带教程完整读一遍，然后挨个实测。碰上各个类、方法时，更应该看在线帮助:

`e6cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4K6k6#2)9J5k6h3y4A6i4K6u0W2K9g2)9J5k6i4g2Q4x3X3c8@1L8$3E0&6L8#2)9J5k6h3q4U0i4K6u0W2K9Y4m8Q4x3V1k6Q4y4@1g2U0K9r3W2T1j5g2)9J5c8X3A6S2N6X3q4K6M7$3W2K6N6q4)9J5c8X3S2@1L8h3I4Q4x3V1k6B7j5i4k6S2M7%4y4A6M7%4c8Q4x3V1j5`.

阅读其源码倒不是特别必要。不知为何大量介绍文章在演示最无聊的`writeFile`，个人经验是，只要同时搜到`Javassis`和`writeFile`，这篇文章就不值得看，跳过。

扯远了，回到正题，用`Javassis`分析`jeb.jar`。我这次演示用`Javassist`如何修改`jebglobal.rP.dN(byte[] paramArrayOfByte,int paramInt1,int paramInt2)`。这个函数在解码被编码过的敏感字符串，比如"`License key error.`"，你直接在反编译结果中找不到这个明文字符串，它被编码后存放了。显然那些被编码过的敏感字符串就是最好的路标，拦截`rP.dN()`，将它们找出来，再结合行号信息，就比较容易理清整个流程。推荐作法是写`BTrace`脚本，自动跟踪`rP.dN()`，将执行路径上的敏感字符串都解码还原出来，事实上我就是这样破解`jeb.jar`的。但我这不是被人忽悠到`Javassist`上去`了么`，于是我就用`Javassist`重现了`BTrace`脚本的工作。

用`Javassis`重新实现了`rP.dN()`，根据需要，先输出调用栈回溯，再输出解码结果:

```
* 
* javac -g -cp "javassist.jar;jeb.xxx.jar" crack_jeb_o.java
* java -cp "javassist.jar;jeb.xxx.jar;." crack_jeb_o > crack_jeb_o.txt
* 
* jeb.xxx.jar就是将jeb.jar中META-INF/下的JEBKEY.*删掉，再编辑一下
* MANIFEST.MF得到。否则java.lang.ClassLoader会在安全检查时抛出异常。
* javassist.Loader的安全检查要弱很多，可是它有别的缺陷导致不能用于此处。
* 最后用jeb.xxx.jar而不是jeb.orig.jar，纯属无奈之举。
* 
* 仅为Java逆向之Javassist演示代码，非最终Loader。
*/

import javassist.*;

import jeb.Launcher;

class crack_jeb_o
{
    public static void main ( String[] argv ) throws Throwable
    {
        /*
         * 调试过程中可以在该目录下获取被修改过的class。这条代码本身也是调
         * 试目的，可以注释掉。
         */
        CtClass.debugDump           = "./debugDump/";
        ClassPool   pool            = ClassPool.getDefault();
        pool.importPackage( "jebglobal.rP" );
        CtClass     old_class       = pool.get( "jebglobal.rP" );
        old_class.detach();
        /*
         * 由于jeb.jar混淆得厉害，只能用这种精确方式获取method，否则只需要
         * 用名字即可定位method。
         */
        CtMethod    old_method      = old_class.getDeclaredMethod
        (
            "dN",
            new CtClass[]
            {
                pool.get( byte[].class.getName() ),
                pool.get( int.class.getName() ),
                pool.get( int.class.getName() )
            }
        );
        old_class.removeMethod( old_method );
        /*
         * 跟BTrace比，Javassist在干这类事时简直就是渣。被迫实现Helper.class，
         * Javasssist内置编译器不支持String.format()这种变长形参函数。
         *
         * 没有用Thread.currentThread().dumpStack()，因为它向stderr输出，不
         * 利于分离输出，此外它的显示破坏了我的洁癖。最后用的是
         * Helper.GetStackTrace()。
         */
        CtMethod    new_method      = CtNewMethod.make
        (
            "public static String dN ( byte[] paramArrayOfByte, int paramInt1, int paramInt2 )" +
            "{" +
            "    String  result;" +
            "" +
            "    System.out.println( Helper.GetStackTrace() );" +
            "    result  = new rP(paramArrayOfByte, paramInt1, paramInt2).dN();" +
            "    System.out.println( Helper.hexdump( result.getBytes(), 0, result.getBytes().length, 16 ) );" +
            "    return( result );" +
            "}",
            old_class
        );
        old_class.addMethod( new_method );
        old_class.toClass();
        /*
         * ( new Loader( pool ) ).run( "jeb.Launcher", argv );
         *
         * 这里不能用javassist.Loader，它的实现有缺陷，会导致在
         * jebglobal.eI.<clinit>(SourceFile:158)
         * 抛出异常
         * throw new RuntimeException("Could not determine the program directory")
         *
         * 被迫使用java.lang.ClassLoader
         */
        Launcher.main( argv );
    }
}
```

```
/*
* javac -g Helper.java
*/

public class Helper
{
    public static String hexdump
    (
        byte[]  buf,
        int     off,
        int     len,
        int     count
    )
    {
        String  ret = "";
        int     offset, k, j, i, m;
        if ( len <= 0 || count <= 0 || null == buf )
        {
            return( ret );
        }
        ret    += String.format( "[%d bytes] -> %d bytes per line\n", len, count );
        i       = 0;
        offset  = 0;
        m       = ( count + 1 ) / 2;
        for ( k = len / count; k > 0; k--, offset += count )
        {
            ret    += String.format( "%08X ", offset );
            for ( j = 0; j < count; j++, i++ )
            {
                if ( m == j )
                {
                    ret    += String.format( "-%02X", buf[off+i] );
                }
                else
                {
                    ret    += String.format( " %02X", buf[off+i] );
                }
            }
            ret    += String.format( "    " );
            i  -= count;
            for ( j = 0; j < count; j++, i++ )
            {
                if ( ( buf[off+i] >= ' ' ) && ( buf[off+i] < 0x7F ) )
                {
                    ret    += String.format( "%c", buf[off+i] );
                }
                else
                {
                    ret    += String.format( "." );
                }
            }
            ret    += String.format( "\n" );
        }
        k       = len - i;
        if ( k <= 0 )
        {
            return( ret );
        }
        ret    += String.format( "%08X ", offset );
        for ( j = 0 ; j < k; j++, i++ )
        {
            if ( m == j )
            {
                ret    += String.format( "-%02X", buf[off+i] );
            }
            else
            {
                ret    += String.format( " %02X", buf[off+i] );
            }
        }
        i      -= k;
        for ( j = count - k; j > 0; j-- )
        {
            ret    += String.format( "   " );
        }
        ret    += String.format( "    " );
        for ( j = 0; j < k; j++, i++ )
        {
            if ( ( buf[off+i] >= ' ' ) && ( buf[off+i] < 0x7F ) )
            {
                ret    += String.format( "%c", buf[off+i] );
            }
            else
            {
                ret    += String.format( "." );
            }
        }
        return( ret );
    }  /* end of hexdump */
    public static String GetStackTrace ()
    {
        int                 i, j;
        String              ret     = "[";
        StackTraceElement[] frame   = new Throwable().getStackTrace();
        if ( null != frame )
        {
            j   = frame.length - 1;
            for( i = 1; i < j; i++ )
            {
                ret    += String.format( "\n%s", frame[i] );
            }
        }
        return( ret + "\n]" );
    }  /* end of GetStackTrace */
}
```

不考虑`CtClass.debugDump`，`crack_jeb_o.java`实际就是个`Loader`，不会静态修改`jeb.xxx.jar`。

`$ java -cp "javassist.jar;jeb.xxx.jar;." crack_jeb_o > crack_jeb_o.txt`

弹出对话框提示输入`key`时随便输个`123456`啥的，我们的目的是找出执行过程中会有哪
些敏感字符串出现。展示一下`crack_jeb_o.txt`:

```
[
jebglobal.rP.dN(SourceFile)
jebglobal.eI.<clinit>(SourceFile:125)
jeb.Launcher.main(SourceFile:159)
]

[8 bytes] -> 16 bytes per line
```
