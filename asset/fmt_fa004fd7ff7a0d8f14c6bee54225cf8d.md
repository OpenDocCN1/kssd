
```
array2        integer 10 dup ( 3 dup (??))  
ptr2Int       dword    ?  
LastName      char     32 dup (?)  
Name          dword    LastName
```

为了同 C/C++一样轻松，UCR 标准库声明看起来很相似。为了这个原因，当编写汇编代码时使用 UCR 标准库会是个好主意。

维护统计：

2006-9-4

修正：将 `endline comments` 翻译为行尾注释，而不是原来的行终止注释。修正部分因打字出现的错误，润饰部分翻译。

`4dd466d0e1e0cc123afbe504da2462ca`
