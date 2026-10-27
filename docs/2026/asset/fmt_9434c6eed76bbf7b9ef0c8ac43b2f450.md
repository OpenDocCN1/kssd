
# [原创]Python逆向——Pyinstaller逆向

## Pyinstaller

最常见的打包库，打包后可以通过`Extractor`工具或者`pyinstaller`内置的`achieve_viewer.py`直接解包，使用`—key`参数可以使其具有一定程度的反逆向能力，但由于其本身是个开源的库，也能找到其加密的原理，所以不是很难逆向

## 无-key参数的逆向

得到可执行文件，使用`Extractor`工具进行解包

```powershell
python pyinstxtractor.py xx.exe
```

将会得到一个文件夹，里面包含主程序所引用的所有库以及代码的`pyc`文件

![](upload/attach/202201/927366_U4MV6DK9CAMRMEJ.png)

`pyinstaller`有一个奇怪的地方，它会把主函数`pyc`文件的文件头进行更改，这里我们就可以用到上面那个文件夹里面一定会有的`struct.pyc`文件，这个文件的文件头一般是不会更改的，将这个文件的文件头进行复制然后更改主函数`pyc`文件头就可以了。

## 有-key参数的逆向

### 逆向前的源码分析

由于`Pyinstaller`是个开源的包，这也给我们逆向提供了便利

在官方给出的用法中，有给出`-key`这个参数，说是可以将文件`pyc`进行一定的压缩加密，以防止被逆向

`Pyinstaller`这个库本身的打包原理大概就是先将`py`编译成`pyc`，然后部分压缩成`pyz`，程序再通过对`pyc`和`pyz`的调用

这个是`pyinstaller`的打包过程语句

```powershell
➜ pyinstaller.exe -F test.py
150 INFO: PyInstaller: 4.3
150 INFO: Python: 3.8.9
150 INFO: Platform: Windows-10-10.0.22000-SP0
152 INFO: wrote H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\test.spec
158 INFO: UPX is not available.
205 INFO: Extending PYTHONPATH with paths
['H:\\My\\_CTF\\Tools\\Python\\_Reverse\\pyinstaller',
'H:\\My\\_CTF\\Tools\\Python\\_Reverse\\pyinstaller']
300 INFO: checking Analysis
300 INFO: Building Analysis because Analysis-00.toc is non existent
300 INFO: Initializing module dependency graph...
304 INFO: Caching module graph hooks...
314 WARNING: Several hooks defined for module 'win32ctypes.core'. Please take care they do not conflict.
335 INFO: Analyzing base_library.zip ...
2867 INFO: Processing pre-find module path hook distutils from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks\\pre_find_module_path\\hook-distutils.py'.
2869 INFO: distutils: retargeting to non-venv dir 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib'
6284 INFO: Caching module dependency graph...
6460 INFO: running Analysis Analysis-00.toc
6464 INFO: Adding Microsoft.Windows.Common-Controls to dependent assemblies of final executable
required by c:\users\lenovo\appdata\local\programs\python\python38\python.exe
7015 INFO: Analyzing H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\test.py
7022 INFO: Processing module hooks...
7022 INFO: Loading module hook 'hook-difflib.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7025 INFO: Loading module hook 'hook-distutils.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7027 INFO: Loading module hook 'hook-distutils.util.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7028 INFO: Loading module hook 'hook-encodings.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7113 INFO: Loading module hook 'hook-heapq.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7115 INFO: Loading module hook 'hook-lib2to3.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7189 INFO: Loading module hook 'hook-multiprocessing.util.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7191 INFO: Loading module hook 'hook-pickle.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7193 INFO: Loading module hook 'hook-sysconfig.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7195 INFO: Loading module hook 'hook-xml.etree.cElementTree.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7197 INFO: Loading module hook 'hook-xml.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7275 INFO: Loading module hook 'hook-\_tkinter.py' from 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks'...
7564 INFO: checking Tree
7574 INFO: Building Tree because Tree-00.toc is non existent
7576 INFO: Building Tree Tree-00.toc
7747 INFO: checking Tree
7747 INFO: Building Tree because Tree-01.toc is non existent
7747 INFO: Building Tree Tree-01.toc
7857 INFO: checking Tree
7857 INFO: Building Tree because Tree-02.toc is non existent
7857 INFO: Building Tree Tree-02.toc
7875 INFO: Looking for ctypes DLLs
7895 INFO: Analyzing run-time hooks ...
7896 INFO: Including run-time hook 'c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_multiprocessing.py'
7902 INFO: Looking for dynamic libraries
8061 INFO: Looking for eggs
8062 INFO: Using Python library c:\users\lenovo\appdata\local\programs\python\python38\python38.dll
8062 INFO: Found binding redirects:
[]
8066 INFO: Warnings written to H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\build\test\warn-test.txt
8099 INFO: Graph cross-reference written to H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\build\test\xref-test.html
8114 INFO: checking PYZ
8115 INFO: Building PYZ because PYZ-00.toc is non existent
8115 INFO: Building PYZ (ZlibArchive) H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\build\test\PYZ-00.pyz
8658 INFO: Building PYZ (ZlibArchive) H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\build\test\PYZ-00.pyz completed successfully.
8673 INFO: checking PKG
8673 INFO: Building PKG because PKG-00.toc is non existent
8673 INFO: Building PKG (CArchive) PKG-00.pkg
10402 INFO: Building PKG (CArchive) PKG-00.pkg completed successfully.
10406 INFO: Bootloader c:\users\lenovo\appdata\local\programs\python\python38\lib\site-packages\PyInstaller\bootloader\Windows-64bit\run.exe
10407 INFO: checking EXE
10407 INFO: Building EXE because EXE-00.toc is non existent
10407 INFO: Building EXE from EXE-00.toc
10473 INFO: Copying icons from ['c:\\users\\lenovo\\appdata\\local\\programs\\python\\python38\\lib\\site-packages\\PyInstaller\\bootloader\\images\\icon-console.ico']
10522 INFO: Writing RT_GROUP_ICON 0 resource with 104 bytes
10522 INFO: Writing RT_ICON 1 resource with 3752 bytes
10522 INFO: Writing RT_ICON 2 resource with 2216 bytes
10523 INFO: Writing RT_ICON 3 resource with 1384 bytes
10523 INFO: Writing RT_ICON 4 resource with 37019 bytes
10524 INFO: Writing RT_ICON 5 resource with 9640 bytes
10524 INFO: Writing RT_ICON 6 resource with 4264 bytes
10524 INFO: Writing RT_ICON 7 resource with 1128 bytes
10531 INFO: Updating manifest in H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\build\test\run.exe.7_01cl0t
10592 INFO: Updating resource type 24 name 1 language 0
10599 INFO: Appending archive to EXE H:\My\_CTF\_Tools\Python\_Reverse\pyinstaller\dist\test.exe
12711 INFO: Building EXE from EXE-00.toc completed successfully.
```

可以看到对`ctypes`的调用，`pyz`的生成等等

生成之后的`build`文件夹如下

![](upload/attach/202201/927366_V63V8M2QRFQ23FV.png)

那个`Base_Library.zip`里面都是库文件的`pyc`

然后我们在`pyinstaller`库中找到`archieve`这么一个文件夹，里面有一个`pyz_crypto.py`文件，是对`pyz`文件的加密代码
