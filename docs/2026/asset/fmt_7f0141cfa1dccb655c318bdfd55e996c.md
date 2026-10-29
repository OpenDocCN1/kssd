
### 静态内部类

静态内部类是在一个类的内部定义的静态类，它和普通的类一样，不依赖于外部类的实例，因此可以直接通过类名来访问。静态内部类不能访问外部类的非静态成员，只能访问外部类的静态成员。静态内部类的定义格式如下：

```
public class Outer {

    private static int x = 10;

    public static class Inner {

        public void print() {

            System.out.println("x = " + x);

        }

    }

}

```

在上面的例子中，`Inner`是`Outer`的静态内部类，它可以访问`Outer`的静态成员`x`。在外部类中创建`Inner`对象的方法如下：

1. `Outer.Inner inner = new Outer.Inner();`
2. `inner.print();`

### 局部内部类

局部内部类是定义在方法内部的类，它只能在该方法内部使用，对外部不可见。局部内部类可以访问外部类的所有成员，包括私有成员。局部内部类的定义格式如下：

```
public class Outer {

    private int x = 10;

    public void test() {

        class Inner {

            public void print() {

                System.out.println("x = " + x);

            }

        }

        Inner inner = new Inner();

        inner.print();

    }

}

```

在上面的例子中，`Inner`是`Outer`方法内部的局部内部类，它可以访问`Outer`的私有成员`x`。在方法内部创建`Inner`对象的方法如下：

```
Outer outer = new Outer();

outer.test();
```

### 匿名内部类

匿名内部类是没有名字的内部类，它通常用于创建一个只需要使用一次的类。匿名内部类可以继承一个类或者实现一个接口，它没有构造方法，但可以使用构造代码块进行初始化。匿名内部类的定义格式如下：

```
new 父类构造器/接口() {

    // 匿名内部类的内容

}
```

在上面的例子中，父类构造器可以是有参数的构造器，也可以是无参数的构造器；接口可以是有多个方法的接口，也可以是只有一个方法的接口。下面是一个使用匿名内部类实现接口的例子：

```
public interface Animal {

    void move();

}



public class Test {

    public static void main(String[] args) {

        Animal animal = new Animal() {

            @Override

            public void move() {

                System.out.println("动物在移动");

            }

        };

        animal.move();

    }

}
```

在上面的例子中，我们创建了一个实现`Animal`接口的匿名内部类，并实现了`move()`方法。在`main`方法中，我们创建了一个`Animal`类型的对象，实际上是一个匿名内部类的对象，然后调用了它的`move()`方法。输出的结果为“动物在移动”。

### 异常

Java中的异常指程序在运行过程中出现的错误或异常情况，例如类型错误、数组下标越界等。Java提供了异常处理机制来处理这些异常情况。

#### 异常的体系：

1. JAVA设计的所有异常类都是有一个祖宗的，这个祖宗类叫`Java.lang.Throwable`，代表这些异常对象都是可以向外抛出去的。

2. 这个祖宗类有两个子类分支，分别是`Error`和`Exception`。

3. `Error`：代表系统级别的错误，属于严重的错误，比如系统出现问题，甲骨文公司就会把这些问题封装成`Error`对象给抛出来。所以说这个不是给开发者用的。

4. `Exception`：这个才是我们熟知的异常，才是代表开发者可能会出现的问题，开发者一般都是使用该类以及其子类来封装程序出现的问题。

5. `Exception`的子类有两类异常，分别是运行时异常和编译时异常。

6. 运行时异常：`RuntimeException`类以及其子类，编译时(写代码的时候)不会出现错误提醒，运行时才会抛出异常。

7. 编译时异常：写代码时会出现错误提醒。

#### 自定义异常并简单处理异常：

Java能提供的异常类是有限的，不可能为全世界出现的问题提供异常类来代表出现的问题。如果某个企业出现自己的问题，想要通过异常类来表示，那就需要自己来定义异常类了。接下来我们自己自定义异常类，假设我们需要收集每个用户合理的年龄，如果检测到某个用户输入的年龄不合理就抛出我们自定义的异常。

自定义运行时异常类：

```
package com.java.day5_exception;



// 1、自定义运行时异常类必须让这个类继承RuntimeException，这样才可以成为一个运行时异常类

// 2、需要重写构造器，当我们创建异常对象时可能需要将参数传给有参构造器进行初始化

public class AgeUnreasonableRuntimeException extends RuntimeException{

    public AgeUnreasonableRuntimeException() {

    }



    public AgeUnreasonableRuntimeException(String message) {  // message参数是用来封装我们自定义异常出现的原因

        // 将封装我们自定义异常出现的原因的参数传给父类的有参构造器，最后由父类来封装我们自定义异常出现的原因

        super(message);

    }

}
```

自定义编译时异常类：

```
package com.java.day5_exception;



// 1、自定义编译时异常类必须让这个类继承Exception，这样才可以成为一个编译时异常类

// 2、需要重写构造器，当我们创建异常对象时可能需要将参数传给有参构造器进行初始化

public class AgeUnreasonableException extends Exception{

    public AgeUnreasonableException() {

    }



    public AgeUnreasonableException(String message) {  // message参数是用来封装我们自定义异常出现的原因

        // 将封装我们自定义异常出现的原因的参数传给父类的有参构造器，最后由父类来封装我们自定义异常出现的原因

        super(message);

    }

}
```

创建一个Java类，用于实现收集每个用户合理的年龄，如果检测到某个用户输入的年龄不合理就抛出我们自定义的异常：

```
package com.java.day5_exception;



import java.util.Scanner; //导入 java.util 包下的 Scanner 类。



public class ExceptionOne {

    public static void main(String[] args) {

        // try……catch用于捕获和处理可能出现的异常。

        try {

            System.out.println("请输入合理的年龄：");

            // 创建了一个Scanner对象，并将System.in作为其参数。System.in取得用户输入的内容后,交给Scanner来作一些处理。

            Scanner scan = new Scanner(System.in);

            // nextInt()方法会等待用户输入一个整数，并将其返回。如果用户输入的内容不是一个有效的整数，则会抛出InputMismatchException异常。

            int num = scan.nextInt();

            // 调用detection方法，如果年龄符合我们定义的合理年龄就正常执行，如果不合理detection方法就会向这里抛出AgeUnreasonableRuntimeException异常

            detection(num);

        } catch (RuntimeException e) { // 如果执行try中代码捕获到RuntimeException异常类以及其子类的对象，那就会执行catch中的代码

            // 打印异常信息

            e.printStackTrace();

        }



        try {

            System.out.println("请输入合理的年龄：");

            Scanner scan2 = new Scanner(System.in);

            int num2 = scan2.nextInt();

            // 调用detection2方法，如果年龄符合我们定义的合理年龄就正常执行，如果不合理detection2方法就会向这里抛出AgeUnreasonableException异常

            detection2(num2);

        } catch (Exception e) {  // 如果执行try中代码捕获到Exception异常类以及其子类的对象，那就会执行catch中的代码

            // 打印异常信息

            e.printStackTrace();

        }

    }



    // 编译时异常

    public static void detection2(int age) throws AgeUnreasonableException{

        if (age>0 && age<150) {

            System.out.println(age + "岁年龄是合理的");

        }else {

            // 用一个编译时异常对象封装这个问题。那我们就需要自己创建一个编译时异常类，所以我创建了一个编译时异常类AgeUnreasonableRuntimeException

            // throw：用于抛出这个异常对象，它会将异常对象先抛到方法的入口这里，再抛给方法的调用者。

            // 所以我们自定义异常会将编译时异常对象先抛到detection方法入口处，再抛给detection方法的调用者。

            // 我们将需要自定义编译时异常类抛出的异常信息作为参数传给了我们自定义的编译时异常类的有参构造器并创建一个编译时异常对象，并将创建的编译时异常对象抛出去。

            // 编译时异常写代码的时候就会强烈提醒你，所以我们自定义编译时异常类被实例化的时候报错是正常的。想要解决这个报错需要用到throws

            // throws：用在方法上，它会将方法内部的异常抛给调用该方法的上层。如果用在main方法上，那就会将main方法内部的异常抛给JVM虚拟机。

            throw new AgeUnreasonableException("The age you entered is" + age +  "years old is not reasonable");

        }

    }



    // 运行时异常

    public static void detection(int age) {

        if (age>0 && age<150) {

            System.out.println(age + "岁年龄是合理的");

        }else {

            // 用一个运行时异常对象封装这个问题。那我们就需要自己创建一个运行时异常类，所以我创建了一个运行时异常类AgeUnreasonableRuntimeException

            // throw：用于抛出这个异常对象，它会将异常对象先抛到方法的入口这里，再抛给方法的调用者。

            // 所以我们自定义异常会将运行时异常对象先抛到detection方法入口处，再抛给detection方法的调用者。

            // 我们将需要自定义运行时异常类抛出的异常信息作为参数传给了我们自定义的运行时异常类的有参构造器并创建一个运行时异常对象，并将创建的运行时异常对象抛出去。

            throw new AgeUnreasonableRuntimeException("The age you entered is" + age +  "years old is not reasonable");

        }

    }

}
```

当输入的年龄合理时：

![](https://bbs.kanxue.com/upload/attach/202308/926486_JBVF2S6VPTH87BR.png)

当输入的年龄不合理时：

![](https://bbs.kanxue.com/upload/attach/202308/926486_GH3CP47DV3Z98XS.png)

#### 开发中处理异常的几种方式：

虽说当将异常对象抛给`main`方法或者`main`方法出现异常都可以将异常对象抛给JVM虚拟机，然后JVM虚拟机会进行处理后将异常信息抛给用户，这对于用户来说是极其影响体验的，所以接下来讲讲开发中一般会怎么处理异常。一般情况下处理异常会用以下几种方式：

第一种、捕获异常，然后将合适的信息响应给用户。

第二种、捕获异常，然后尝试修复异常。

在前面我们简单处理异常的时候，如果用户输入非`int`型的数据，那肯定是会抛出异常的。所以我们还是围绕收集每个用户合理的年龄这个主题，使用这两种异常处理的方式将之前的代码完善。

捕获异常，然后将合适的信息响应给用户：

```
package com.java.day5_exception;



import java.util.Scanner;



public class ExceptionTwo {

    public static void main(String[] args) {

        try {

            runs();

        } catch (Exception e) { // 如果调用runs方法因为输入非int型的数据触发了Exception异常就打印对应的中文异常信息

            System.out.println("您需要输入整数型数据，而非其他类型数据！");

        }

    }



    public static void runs() throws Exception{

        System.out.println("请输入合理的年龄：");

        Scanner scan = new Scanner(System.in);

        int num = scan.nextInt();

        try {

            detection(num);

            detection2(num);

        } catch (AgeUnreasonableRuntimeException e) { // 如果出现我们自定义的运行时异常就打印对应的中文异常信息提醒用户

            System.out.println("您输入的年龄不合理！触发了运行时异常。");

        } catch (AgeUnreasonableException e) { // 如果出现我们自定义的编译时异常就打印对应的中文异常信息提醒用户

            System.out.println("您输入的年龄不合理！触发了编译时异常。");

        }

    }



    // 编译时异常

    public static void detection2(int age) throws AgeUnreasonableException{

        if (age>0 && age<150) {

            System.out.println(age + "岁年龄是合理的");

        }else {

            throw new AgeUnreasonableException("The age you entered is" + age +  "years old is not reasonable");

        }

    }



    // 运行时异常

    public static void detection(int age) {

        if (age>0 && age<150) {

            System.out.println(age + "岁年龄是合理的");

        }else {

            throw new AgeUnreasonableRuntimeException("The age you entered is" + age +  "years old is not reasonable");

        }

    }

}
```
