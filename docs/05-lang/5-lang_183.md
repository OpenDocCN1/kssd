
# JAVA 程序的简单破解说明

> 原文：[`www.pediy.com/kssd/pediy10/57887.html`](https://www.pediy.com/kssd/pediy10/57887.html)

`Crinity` 是 `3RSOFT` 公司所开发的一套基于 `Postfix` `MTA` 的大型企业邮件系统, 核心程序基于 `Postfix` + `sasl` `pwcheck_method` + `james` 构建, 采用 `JAVA` 开发 `Webmail` client. 该系统标准版本 500 用户售价达到 3 万元. 具体的介绍大家可以去 `http://www.3rsoft.com.cn` 查看.

`3RSOFT` 网站提供了 `Crinity` 7.0 for `Linux` 试用版本的下载, 试用版和正式版本的区别是试用版只支持 5 个用户, 其他无任何限制.

## 破解说明

以下过程你需要有一定的 `Linux`, `Java`, 和汇编基础并熟练掌握 `DJ`, `IDA` 和 `Uedit`

1. 按照说明文档在 `Redhat` `Linux` 9.0 系统下安装了这套邮件系统
2. 进入系统后台管理界面, 利用管理员帐户成功的注册了 5 个用户
3. 当添加第 6 个用户时系统提示 `current user number is biger than license number`
4. 利用 `DJ` `Java Decompiler` 对所有 `JAVA` `CLASS` 文件进行反编译得到 `JAVA` 程序代码
5. 搜索文本字串 `current user number is biger than license number`
6. 在 `AdminUser.class` 中发现关键代码 (注册限制在多个文件中存在这里只以此文件为例)

```java
        String s8;
        if(i < usermodel._quotaTotal)
            usermodel._quotaTotal = i;
        if(usermodel._quotaTotal > domainmodel._maxUserQuota)
            usermodel._quotaTotal = domainmodel._maxUserQuota;
        obj = new UserHandler(usermodel);
        super.sys = SystemConfiguration.getConfiguration();
        String s5 = sys.getString("License");
        l = u_DAO.getUserTotalNum("ALL", 0);
        if(s5 == null) // <-- 在 crinity.conf 中把 License 设置为空即可绕过下面的程序
            break MISSING_BLOCK_LABEL_684;
        LKChecker lkchecker = new LKChecker();
        lkchecker.setVerbose(true);
        lkchecker.parse(s5);
        Boolean boolean1 = lkchecker.checkUserLimit(l);
        if(boolean1 != null && boolean1.booleanValue()) // <-- 非关键判断我们只需在 crinity.conf 中把 License 设置为空即可绕过
            break MISSING_BLOCK_LABEL_756;
        context.put("msg", "this license donot allow you add user");
        s8 = "t_error_msg.html";
        if(u_DAO != null)
            u_DAO.freeResource();
        if(d_DAO != null)
            d_DAO.freeResource();
        if(t_DAO != null)
            t_DAO.freeResource();
        return s8;
        String s6;
        if(l <= 5)  // <-- 关键判断
            break MISSING_BLOCK_LABEL_756;
        context.put("msg", "current user number is biger than license number");
        s6 = "t_error_msg.html";
        if(u_DAO != null)
            u_DAO.freeResource();
        if(d_DAO != null)
            d_DAO.freeResource();
        if(t_DAO != null)
            t_DAO.freeResource();
        return s6;
```

7. 分析上述代码不难发现关键判断语句为 `if(l <= 5)` 这里 5 表示系统中现有注册用户数如条件成立则跳过下面的错误提示及返回
8. 查看此文件 `import` 发现试用版缺少以下 class

```java
        import com.r3s.sls.checker.LKChecker;
        import com.r3s.xdk.configuration.Configuration;
```

9. 以上结果告诉我们除非自行恢复这两个缺少的类文件重写所有函数否则我们不可能正常的修改并重新编译 `AdminUser.java`
10. 得到结论必须设法利用对源文件 `AdminUser.class` 进行反汇编, 并通过直接修改 class 文件完成破解
11. 利用 `IDA pro` 对 `AdminUser.class` 进行反汇编, 在汇编代码中查找 `CreateUser` 函数发现以下指令

```assembly
        iconst_5
        if_icmple met005_756
```

12. 按照以上结果我们知道 `iconst_5` 表示将整形常量 5 压入栈, `if_icmple` 表示做小于等于判断
13. 查找 `JVM` 手册 `http://java.sun.com/docs/books/jvms/` 发现此处语句的 `HEX` 表示方式应该为 `08A4`
14. 我们需要将这里的代码修改成类似 `if(l > -1)` 的形式让判断语句永远成立 (因为已注册用户数永远大于 -1) 对应汇编代码为

```assembly
        iconst_ml
        if_icmpgt met005_756
```

15. 查询 `JVM` 手册后 `HEX` 表示方式应该为 `02A3`
16. 利用 `Ultraedit` 的 `HEX` 编辑功能查找 `08A4` 修改为 `02A3` 并保存
17. 将保存后的 `AdminUser.class` 覆盖原文件
18. 按此方法依次修改 `AdminConf.class` `AdminUser.class` `DomainUser.class` `UserHandler.class` 完成破解

我是新手一直看各位 DX 的文章学了不少，发现论坛里讨论 `JAVA` 程序破解的帖子比较少于是发了一篇欢迎大家拍砖!
