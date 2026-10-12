
# 浅谈 JAVA 程序破解(原创)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-749.htm`](https://www.pediy.com/kssd/pediy07/pediy7-749.htm)

浅谈 JAVA 程序破解  

作者：舵手  

申明：如转载请保证文章的完整性以及出处  

最近对 JAVA 程序的破解比较感兴趣，拿几个行业软件练了一下手，略有心得，拿出来与菜鸟分享！注意只是一点心得，本文并不涉及具体软件的破解。初学破解，失误之处在所难免，敬请高手赐教！  

直接进入正题，对 JAVA 的破可从下面几方面入手：  

## 一、反编译  

工具很多，建意用 GUI 工具，命令行下的 `JAD` 很容易因为不能反编译某一个方法或某一行代码而终止整个文件的反编译，但 GUI 的工具却能搞定，虽然反编译后部分代码较难看懂，但总比看 `jvm` 指命要好得多。而且，GUI 的工具多数有批量反编译功能，且能让反编译的文件直接以 `.java` 为后缀保存，也是方便之处。  

## 二、方法调用  

安全意识强的开发者会把他的程序进行高质量的混淆，下面就是一个例子  

```java
public static Object getRemoteEJBHome(String OOOoOo00oO0O0O0ooOoOO, Class OO0oOO0O0o0oO0o00oOoO)
    throws NamingException
    {
    try
    {
        if(OoO0o0o0O0oo0oO00oOO0 == null)
            OoO0o0o0O0oo0oO00oOO0 = OoOOoOOO0Oo0OO0OooO0o();
        Object OOOOOo00000OoOoO0O000 = PortableRemoteObject.narrow(OoO0o0o0O0oo0oO00oOO0.lookup(OOOoOo00oO0O0O0ooOoOO), OO0oOO0O0o0oO0o00oOoO);
        Object obj = OOOOOo00000OoOoO0O000;
        return obj;
    }
    catch(NamingException OO0Ooo0oOO0OO0OOOoOo0)
    {
        System.out.println(OO0Ooo0oOO0OO0OOOoOo0.getMessage());
        throw OO0Ooo0oOO0OO0OOOoOo0;
    }
}
```

这是我见过的最好的混淆效果，变量都是由大小写的 `O` 和数字零组成，要看懂这样的程序基本上是不可能的，可能会有人想到用有意义的变量进行替换，当然这也是一个方法，但如果应用所包括的 `class` 文件数以千计，那这个工作量是相当大的。B/S 结构的授权方式一般都是文件的形式，当然，肯定是经过加密的。像下面的 `license` 就是经过了 `RSA` 非对称加密算法，要分析 `license` 的构成，有明文的 `license` 就更方便了，而公钥是直接被写在 `class` 文件中的

```
24D568B6A27AEFD683BC7A1CC93D11D074FB6B982A4B6E269712773BE536B40A67F1D345654F659C66D4265F5CE8FE0494B3A
F33A8299A4F6B0E7500275A27EFF3B6D2E4983F14A9EA38A1AE3394B28A9C6D6924C15027F9B689FD9A3A689A301C4D4EB878
D75C207F68BAA352F550D8F19876FFA255864FDE8A7E5939202E9F
```

那么我们可以用 `eclipse` 建一个 `JAVA` 项目，把应用的 `jar` 加入该项目的库搜索路径，写一个自己的类调用解密方法，得到明文 `license` 再分析。当然，也可以调用其它一些方法，从调用参数和最后的返回值我们也可大概猜对该方法的作用，对付象上面经过高质量混淆的代码也比较管用。当然，我这里只是简单的举两个例子，其实“方法调用”的妙用还很多，自己慢慢琢磨吧！  

## 三、为 class 添加代码  

反编译多数情况下也只能让我们看看作者的思路，如果想把反编译出来的代码经过修改后再编译成 `class`，通常是行不通了。而且有时候必须让程序运行在它本身的环境才能行，否则一些类无法得到正确的初始化，“方法调用”也就起不了什么作用。搞过 `java` 的人一定知道 `javassist`，这个库提供了足够多的方法让你直接修改 `class` 文件，而 不需要你了解字节码的相关知识，我们可以利用这个库解决上述的问题。下面是我写的一个修改字节码的类，目前还不完善，真正要用时可能需要根据情况做一些修改。  

```java
import java.lang.reflect.*;
import javassist.*;
import java.io.*;

/**
 * <p>Title: JAVA 字节码修改类</p>
 * <p>Description: 得到类的相关信息或修改该类</p>
 * <p>Copyright: Copyright () 2005</p>
 * @author 舵手
 * @version 1.0
 */
public class ModifyClass {
  private static int call_method;
  private static String _class;
  private static ClassPool pool;
  private static CtClass cc;
  private static String[] clas;

  /**
   * 修改字节码中的方法
   * @param clas[0] 待修改类的方法名
   * @param clas[1] 修改位置定义
   * @param clas[2] 使用 insertAt 方法插放代码时行号参数
   * @param clas[3] 修改内容
   * @return
   */
  private static void modifyMethod() {
    String _method;
    _method = clas[0];
    try {
      CtClass[] param = new CtClass[4];
      //param[0] = pool.get("");
      //param[1] = pool.get("");
      //param[2] = pool.get("java.lang.String");
      //param[3] = pool.get("java.lang.String");
      CtMethod cm = cc.getDeclaredMethod(_method);
      if (clas[1].toLowerCase().equals("a")) {
        //方法的尾部加入代码
        cm.insertAfter(clas[3]);
      }
      if (clas[1].toLowerCase().equals("b")) {
        //方法的首部加入代码
        cm.insertBefore(clas[3]);
      }
      if (clas[1].toLowerCase().equals("i")) {
        System.out.println(cm.insertAt((Integer.valueOf(clas[2]).intValue()),clas[3]));
      }
      cc.writeFile();
    }
    catch(Exception e) {
      e.printStackTrace();
    }
  }

  /**
   * 在类中增加方法
   * @param clas[0] 源方法名称
   * @param clas[1] 新方法名称
   * @param clas[2] 增加类型
   * @param clas[3] 方法内容
   * @return
   */
  private static void addMethod() {
    String _oldmethod;
    String _newmethod;
    _oldmethod = clas[0];
    _newmethod = clas[1];
    try {
      StringBuffer newMethodBody = new StringBuffer();
      if (clas[2].toLowerCase().equals("c")) {
        //add new Method (copy)
        CtMethod oldMethod = cc.getDeclaredMethod(_oldmethod);
        CtMethod newMethod = CtNewMethod.copy(oldMethod, _newmethod, cc, null);
        newMethodBody.append(clas[3]);
        newMethod.setBody(newMethodBody.toString());
        cc.addMethod(newMethod);
      }
      if (clas[2].toLowerCase().equals("r")) {
        //add new Method (create)
        CtMethod newMethod = CtNewMethod.make(clas[3], cc);
        cc.addMethod(newMethod);
      }
      cc.writeFile();
    }
    catch(Exception e) {
      e.printStackTrace();
    }
  }

  private static void getMethods(){
    CtMethod[] cms = cc.getDeclaredMethods();
    System.out.println();
    System.out.println(cc.getName()+" 类的所有方法：");
    for (int i=0 ; i<cms.length ; i++ ) {
      System.out.println(cms[i].getName());
    }
  }

  private static void getFields(){
    CtField[] cfs = cc.getDeclaredFields();
    System.out.println();
    System.out.println(cc.getName()+" 类的所有属性：");
    for (int i=0 ; i<cfs.length ; i++ ) {
      System.out.println(cfs[i].getName());
    }
  }

  private static void delMethod(){
    try{
      CtMethod cm = cc.getDeclaredMethod(clas[0]);
      cc.removeMethod(cm);
    }catch(Exception e){
      e.printStackTrace();
    }
  }

  public static void main(String[] args) {
    StringBuffer buf = new StringBuffer(500);
    int c;
    System.out.print("请输入操作类名：");
    try{
      while ((c = System.in.read()) != 13) {
        buf.append((char)c);
      }
      _class = buf.toString();
      pool = ClassPool.getDefault();
      cc = pool.get(_class);
      buf.delete(0,buf.length());
      System.out.println("****************************************************************");
      System.out.println("可供调用的方法有：");
      System.out.println("1-modifyMethod，2-addMethod，3-getMethods，4-getFields，5-removeMethod");
      System.out.println("****************************************************************");
      System.out.print("请选择调用方法：");
      while ((c = System.in.read()) != 13) {
        if (c == 10)
          continue;
        buf.append((char)c);
      }
      call_method = Integer.parseInt(buf.toString());
      if (call_method == 1) {
        System.out.println("****************************************************************");
        System.out.println("调用 modifyMethod 方法参数：");
        System.out.println("方法名称,插入位置,行号,内容");
        System.out.println("****************************************************************");
        buf.delete(0,buf.length());
        while ((c = System.in.read()) != 13) {
          if (c == 10)
            continue;
          buf.append((char)c);
        }
        clas = (buf.toString()).split(",");
        modifyMethod();
      }
      buf.delete(0,buf.length());
      if (call_method == 2) {
        System.out.println("****************************************************************");
        System.out.println("调用 addMethod 方法参数：");
        System.out.println("源方法,目标方法,建立方式,内容");
```
