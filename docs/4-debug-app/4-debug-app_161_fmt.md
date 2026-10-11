
# 批处理外挂

> 原文：`www.pediy.com/kssd/pediy11/115329.html` (链接: https://www.pediy.com/kssd/pediy11/115329.html)

今天晚上闲着没事，就玩了一把三子棋，发现赢的线路比较固定，所以就改了一下，就算是外挂能吧~！

**改动的地方：**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

原代码：

```
:begin
:1
echo 请您走棋！（输入数字 1~9）
set /p slt2=
```

改为：

```
:begin
echo 本局您执●！
echo 请您走棋！（输入数字 1~9）
ping -n 4 127.1>nul
goto me2
:1
echo 本局您执○！
echo 请您走棋！（输入数字 1~9）
ping -n 2 127.1>nul
goto me1
:find
```

增加延时，可以看到字幕，嘿嘿。。。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`begin` 结尾处：

原代码：

```
goto begin
```

改为：

```
if %slt1% equ c goto 1
if %slt1% equ h goto begin
```

这里主要是根据情况决定路线。

两个函数：

```
rem 改动处！
:me1
::请看明白了之后再运行。
@echo off
if not exist key.dat echo 0 > key.dat
::判断是否已经存在 key.dat。这个文件用来存储运行次数。
set max=5
::设置最大运行次数
goto begin1
:begin1
for /f %%i in (key.dat) do set flag=%%i
::读取运行之前的次数
echo hello world!
::这一句是程序的主要内容。
set/a flag+=1
::将运行的次数加一。
echo %flag% >key.dat
::将运行的次数保存起来。
if %flag% equ 1  goto ok1
if %flag% equ 2  goto ok2
if %flag% equ 3  goto ok3
if %flag% equ 4  del key.dat&goto ok4
::如果运行次数等于最大运行次数则删除自己和 key.dat。
pause
:ok1
set slt2=1
goto find
:ok2
set slt2=6
goto find
:ok3
set slt2=3
goto find
:ok4
set slt2=2
goto find
:me2
::请看明白了之后再运行。
@echo off
if not exist key.dat echo 0 > key.dat
::判断是否已经存在 key.dat。这个文件用来存储运行次数。
set max=5
::设置最大运行次数
goto begin2
:begin2
for /f %%i in (key.dat) do set flag=%%i
::读取运行之前的次数
echo hello world!
::这一句是程序的主要内容。
set/a flag+=1
::将运行的次数加一。
echo %flag% >key.dat
::将运行的次数保存起来。
if %flag% equ 1  goto ok1
if %flag% equ 2  goto ok2
if %flag% equ 3  goto ok3
if %flag% equ 4  del key.dat&goto ok4
::如果运行次数等于最大运行次数则删除自己和 key.dat。
pause
:ok1
set slt2=5
goto find
:ok2
set slt2=9
goto find
:ok3
set slt2=7
goto find
:ok4
set slt2=8
goto find
```

**赢棋秘诀：**

* 电脑先走：`1--6--3---2`
* 人先走：`5--9--3----8`

游戏及我修改的见附件！！！

**上传的附件**

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `三子棋 惹不起的宝贝.rar` |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `自动对战.rar` |
