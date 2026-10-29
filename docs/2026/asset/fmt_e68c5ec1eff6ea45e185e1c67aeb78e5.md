
```
   private String dumpJarFile(int dexFileCount, String dexFilePath, String jarOutputPath, String applicationName) {

        ArrayList<String> dexFileList = createClassesDotDexFileList(dexFileCount);

        for (String dexFileName : dexFileList) {

            String filePath = dexFilePath + dexFileName;

            // 执行dex2jar命令，修改源代码

            boolean isApplicationClassFound = dex2JarCmd(filePath, jarOutputPath, applicationName);

            // 找到了目标应用主application的包名，说明代码注入成功，则返回当前dex文件

            if (isApplicationClassFound) {

                return dexFileName;

            }

        }

        return "";

    }



    private boolean dex2JarCmd(String dexPath, String jarOutputPath, String applicationName) {

        Dex2jarCmd cmd = new Dex2jarCmd();

        String[] args = new String[]{

                dexPath,

                "-o",

                jarOutputPath,

                "-app",

                applicationName,

                "--force"

        };

        cmd.doMain(args);



        // 执行完命令后，会返回查找Application Class的结果

        boolean isApplicationClassFounded = cmd.isApplicationClassFounded();

        if (showAllLogs) {

            System.out.println("isApplicationClassFounded ->  " + isApplicationClassFounded + "the dexPath is  " +

                    dexPath);

        }

        return isApplicationClassFounded;

    }
```

使用`dex2jar`修改完Apk的`Application`类之后，得到的是一个jar文件，再通过`jar2dex`工具转为dex文件：

```
private void jar2DexCmd(String jarFilePath, String dexOutPath) {

        Jar2Dex cmd = new Jar2Dex();

        String[] args = new String[]{

                jarFilePath,

                "-o",

                dexOutPath

        };

        cmd.doMain(args);

    }
```

最后删除生成的jar文件，新的dex文件就是完成代码注入后的dex。

最后，将这些dex文件和so文件压缩为Apk文件，并签名。

至此，完成Apk的篡改，并实现App启动时，加载设备上已安装的所有Xposed插件模块。

# 总结

最后，归纳一下Xpatch破解App的整体流程：

> 1. 利用Android Art Java Hook框架（whale或者SandHook），开发能够加载Xposed模块的Apk，并导出其中的dex和so文件；

> 2. 修改dex2jar工具，以实现在dex转换为jar的过程中，查找App的主Application类，并在此类中插入一段静态代码块，实现加载Xposed模块；

> 3. 将修改后的dex和加载Xposed模块的dex和so文件一起打包签名，从而完成代码注入，实现Xposed模块的加载。

欢迎扫二维码，关注我的技术公众号**Android葵花宝典** ，获取高质量的Android干货分享：

![image](https://bbs.kanxue.com/upload/attach/202001/849838_EAETABGZJQVWMHE.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
