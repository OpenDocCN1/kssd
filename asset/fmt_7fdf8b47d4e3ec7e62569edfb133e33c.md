
## 六、keymake 汇编注册机代码

大侠刘健英写`keymake`，主要用来做汇编注册机模板的，而我们大多拿来做内存注册机和补丁了，现在试用它来做个注册机，其实很方便的。

具体使用大家可以看下帮助部分，很详细。关键注意作者设计时为了统一，把编辑框输入和输出地址都放在`EAX`。算法部分汇编代码一般直接从程序提取，可以使用`OD`插件`Codedata`或`Asm2clipboard`。本人开始学的时候很笨，直接从`OD`复制下来，自己手动去除虚拟地址和机器码。

代码多的话会累死。感谢插件的作者，给我们节省了时间。如果子程序太多，就需要用`IDA`带脚本提取了。当然如果你对这个模板不满意或想另外学习熟悉些汇编自己写模板也不难。

### 数据区粘贴

代码：

```
szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:qifeon@yahoo.com",0
szErrMess  db "用户名不能为空！",0
szBuffer db 50 dup (0)
szFormat db "%d",0
a dd 0
```

### 代码区

代码：

```
mov a,eax
invoke lstrlen,a
mov ecx,eax
xor esi,esi
mov edx,1h
n1:
  mov eaX,a   
  movzx   eax, byte ptr \[eax+edx-1\]
  add     esi, eax
  inc     edx
  dec     ecx
  jnz n1
 mov edx,esi
 mov eax,esi
  shl     eax, 7
  add     eax, edx
invoke wsprintf,addr szBuffer,addr szFormat,eax
lea eax,szBuffer
```

然后编译就可以了，`keymake`目录下就会出现你需要的可爱注册机了。

如果有一个和我一样菜的菜鸟看了本文能有所收获，就是本人最大的动力。
