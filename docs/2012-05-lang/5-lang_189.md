
# 一款 Java 软件的爆破

> 原文：[`www.pediy.com/kssd/pediy12/131419.html`](https://www.pediy.com/kssd/pediy12/131419.html)

软件：[Shape Collage](http://www.shapecollage.com/)

Shape Collage 是一款免费的图片拼贴制作工具，可以轻松制作任意形状的图片拼图，如矩形、心形、圆形、字母形状，甚至绘制自定义形状。

但是免费版会有水印。而 pro 版本则没有，而且可能保存工程为 PSD 格式文件，并且开放了许多功能。

首先使用 `jd-gui` 进行反编译，通过观察可以发现这个 jar 被 ProGuard 保护过了，典型的就是超过了 26 个 class，然后文件通过改变文件名大小写的方式存在。所以在 Windows 上是不能直接解压出来的，出动虚拟机，开上 ubuntu 进行打包。

因为 pro 版本会在标题显示一个 pro，所以直接搜索即可。

在 `ca\vincentcheung\shapecollage\ShapeCollageGUI.class` 文件中将能找到这么一段。

代码:

```java
  private void H()
  {
    if (this.jdField_f_of_type_Boolean)
    {
      setTitle("Shape Collage Pro 2.5.3");
      this.jdField_p_of_type_JavaxSwingJMenuItem.setText(jdField_a_of_type_JavaUtilResourceBundle.getString("Open_Project"));
      this.jdField_x_of_type_JavaxSwingJMenuItem.setText(jdField_a_of_type_JavaUtilResourceBundle.getString("Save_Project"));
      this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_p_of_type_JavaxSwingJMenuItem, jdField_a_of_type_JavaUtilResourceBundle.getString("Status_menu_open_project"));
      this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_x_of_type_JavaxSwingJMenuItem, jdField_a_of_type_JavaUtilResourceBundle.getString("Status_menu_save_project"));
      this.jdField_p_of_type_JavaxSwingJMenuItem.setEnabled(true);
      this.jdField_x_of_type_JavaxSwingJMenuItem.setEnabled(true);
      this.jdField_z_of_type_JavaxSwingJMenuItem.setText(jdField_a_of_type_JavaUtilResourceBundle.getString("Registration_Info"));
      this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_z_of_type_JavaxSwingJMenuItem, this.jdField_z_of_type_JavaxSwingJMenuItem.getText());
      this.jdField_b_of_type_JavaxSwingJDialog.setTitle(jdField_a_of_type_JavaUtilResourceBundle.getString("Shape_Collage_Pro_Registration_Info"));
      return;
    }
    setTitle("Shape Collage 2.5.3");
    this.jdField_p_of_type_JavaxSwingJMenuItem.setText("(Pro) " + jdField_a_of_type_JavaUtilResourceBundle.getString("Open_Project"));
    this.jdField_x_of_type_JavaxSwingJMenuItem.setText("(Pro) " + jdField_a_of_type_JavaUtilResourceBundle.getString("Save_Project"));
    this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_p_of_type_JavaxSwingJMenuItem, jdField_a_of_type_JavaUtilResourceBundle.getString("Status_menu_open_project") + " (" + jdField_a_of_type_JavaUtilResourceBundle.getString("Upgrade_To_Pro") + ")");
    this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_x_of_type_JavaxSwingJMenuItem, jdField_a_of_type_JavaUtilResourceBundle.getString("Status_menu_save_project") + " (" + jdField_a_of_type_JavaUtilResourceBundle.getString("Upgrade_To_Pro") + ")");
    this.jdField_p_of_type_JavaxSwingJMenuItem.setEnabled(false);
    this.jdField_x_of_type_JavaxSwingJMenuItem.setEnabled(false);
    this.jdField_z_of_type_JavaxSwingJMenuItem.setText(jdField_a_of_type_JavaUtilResourceBundle.getString("Upgrade_To_Pro"));
    this.jdField_a_of_type_JavaUtilHashtable.put(this.jdField_z_of_type_JavaxSwingJMenuItem, this.jdField_z_of_type_JavaxSwingJMenuItem.getText());
    this.jdField_b_of_type_JavaxSwingJDialog.setTitle(jdField_a_of_type_JavaUtilResourceBundle.getString("Upgrade_To_Pro"));
    this.jdField_h_of_type_JavaxSwingJTextField.setText(jdField_a_of_type_JavaUtilResourceBundle.getString("Unregistered"));
  }
```

显然 `this.jdField_f_of_type_Boolean` 就是那个关键变量了，我们寻找一下这个变量是在哪里初始化的。可以找到：

代码:

```java
    localObject5 = ch.a("SC25", (String)localObject5);//这行贴出来是为了方便搜索
    ((ShapeCollageGUI)localObject2).jdField_f_of_type_Boolean = (localObject5 != null);
    ((ShapeCollageGUI)localObject2).H();//这行代码无视
    this.jdField_f_of_type_Boolean = ((ShapeCollageGUI)localObject2).jdField_f_of_type_Boolean;
```

于是你懂的，其实就是 `this.jdField_f_of_type_Boolean = (localObject5 != null);`

那么把!=的结果始终修改为 true 就行了。使用 `JBE` 打开这个 class 文件，选择 init 函数。搜索 `SC25`，于是发现：

代码:

```assembly
2581 ldc_w "SC25"
2582 aload_3
2583 invokestatic ch/a(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
2584 astore_3
2585 aload_2
2586 aload_3
2587 ifnull 2590
2588 iconst_1
2589 goto 2591
2590 iconst_0
2591 putfield ca/vincentcheung/shapecollage/ShapeCollageGUI/f Z
```

如果长期看 Java 字节码的话，是能够明白一些代码意义的。比如这里的 `ifnull 2590` 就是说，如果是 null，则跳转到 2590 行，不是 null 则执行下一行，最后 `goto`，其实和汇编的 JZ 差不多![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")。我们把这两个压入的常量都改成 1 就好了，也就是把 `iconst_0` 改成 `iconst_1`。

保存 class，切换至虚拟机，把修改好的所有文件打包成 `zip`，再改名为 `jar`，双击运行，pro 版本啦![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

本文需要的一些工具

`jd-gui`：`http://u.115.com/file/f0fea0394c`

`jbe`：`http://u.115.com/file/f06aac67f0`
