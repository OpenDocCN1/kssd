
# Ruby 和 MinGW GCC 的一次融合

> 原文：[`www.pediy.com/kssd/pediy11/112770.html`](https://www.pediy.com/kssd/pediy11/112770.html)

希望给大家提供一些新的思路。

我们在使用 C 编程时会遇到一个问题，比如头文件的一个函数包含在一个 `lib` 中，但在实际链接中我们不知道它在哪个库中。也许可行的一种办法是直接上网查询某个函数的依赖条件，这对于常用函数是没问题的！但是对于复杂而又缺少文档的第三方 `lib` 来说，无异于大海捞针。

自此通过 2 种办法来尝试解决这个问题，我们先看第一种：

在 Windows 下使用批处理语言来查找指定的函数符号：

`crt.bat`:

```batch
@echo off

set gccpath=d:\downloadsvr\mingwstudio\mingw

dir %gccpath%\lib /b>files.txt

if not exist files md files

for /f %%i in (files.txt) DO (
  set fprefix=%%i
  %gccpath%\bin\nm.exe -s -D %gccpath%\lib\%%i>files\%%i.txt
  for %%a in (files\%%i.txt) do (
  if %%~za equ 0 del %%a
  )
)

del files.txt

@echo ######## Create Names Database Successed!!! #########

pause
```

`fnd.bat`:

```batch
@echo off

dir files\ /b >tmp.txt

for /f %%i in (tmp.txt) DO (
  find /i /n "%1" files\%%i
)

del tmp.txt
```

以上有 2 个 `bat` 文件，`crt.bat` 是生成对应 `lib` 文件夹中的所有 `lib` 的符号，分别放在若干个文本文件中。只要 `lib` 不发生变动，那么只要生成一次即可！然后使用 `fnd.bat` 来查询某个符号。

that's all！！！

那么使用 Ruby 能否完成这一功能呢？答案是肯定的：

代码:

```ruby
class LibSyms
  attr_accessor :libs
  
  def save_syms(fname="cache_syms.dat")
    data = Marshal.dump(@syms)
    File.open(fname,"w+b") {|f|f.puts data}
  end
  
  def load_syms(fname="cache_syms.dat")
    data=nil
    File.open(fname,"r+b"){|f|data=f.read}
    @syms=Marshal.load(data)
  end
  
  def enum_syms
    @syms={}
    @libs = []
    `dir \"#{@path}\\lib\" /b`.lines {|line|@libs<<@path+"\\lib\\"+line.chomp}
    
    @libs.each do |fname|
      tmp=`\"#{@path}\\bin\\nm.exe\" -s -D \"#{fname}\"`.split("\n")
      @syms[fname]=tmp
    end
  end
  
  def initialize(name,path,fname="cache_syms.dat")
    @name,@path=name,path
    if File.exist?(fname)
      p "find syms file,try to load!"
      load_syms(fname)
    else
      p "no find syms file,try to enum and save!"
      enum_syms
      save_syms(fname)
    end
  end
  
  def find(sym2find)
    p "into %s"%__callee__
    @syms.each do |fname,syms|
      finded = syms.find_all do |sym|
        sym =~ /#{sym2find}/i
      end
      (p "find at #{fname} :";puts finded;p "\n") unless finded.empty?
    end
  end
end

$ls=LibSyms.new('gcc','C:\Program Files\MinGW5.1.6')
loop do
  print "input a sym to check:"
  sym=gets
  case sym
  when /^quit$/i;break
  else;$ls.find(sym.chomp)
  end
end
```

`class LibSyms` 使用了批处理文件中类似的缓存机制，在无变动的情况下只会执行一次昂贵耗时的动作！

over!欢迎指正。
