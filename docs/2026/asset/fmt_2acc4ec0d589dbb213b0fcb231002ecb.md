
```c
pVectordListEntr->Head->pPrevious=vl;
pVectordListEntr->Head=vl;
```

`RtlReleaseSRWLockExclusive(pVectordListEntr->Head);`

`return vl;`
}

从上面的代码可以看出VEH的地址是存储在`LdrpActiveThreadCount`这个全局变量里的，而VEH有两个表链入口，表链入口0，和链表入口1，

表链入口0所挂载的异常处理函数，是函数`AddVectoredExceptionHandler`挂载上去的，换句话说，表链入口0所挂载的函数是要在SEH异常之前执行的。

而链表入口1所挂载的异常处理函数，是通过`AddVectoredContinueHandler`挂上去的，也就是说，表链入口1所挂载的函数是要在SEH异常之后执行的。

`AddVectoredContinueHandler`和`AddVectoredExceptionHandler`的代码上的差别，上面有提到，这里不在说了。

## 进阶篇

如果要替换修改VEH异常处理函数可以如下操作

![](https://bbs.kanxue.com/upload/attach/201303/292006_9d8xs2wngr4ku6w.png)

卸载VEH异常处理函数可以这么做

![](https://bbs.kanxue.com/upload/attach/201303/292006_un8a78be8ukz12q.png)

让VEH失效的话可以这么做

![](https://bbs.kanxue.com/upload/attach/201303/292006_mr7kx76sdyvsau9.png)

下面的附件里包含了文中的所有代码资源

[VEH分析.zip](https://bbs.kanxue.com/attach-download-77528.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [image001.png](javascript:void(0);)
  （9.51kb，37次下载）
- [image003.png](javascript:void(0);)
  （19.07kb，55次下载）
- [image005.png](javascript:void(0);)
  （13.74kb，16次下载）
- [image007.png](javascript:void(0);)
  （10.26kb，15次下载）
- [image009.png](javascript:void(0);)
  （13.54kb，10次下载）
- [image011.png](javascript:void(0);)
  （10.32kb，16次下载）
- [VEH分析.zip](javascript:void(0);)
  （86.25kb，631次下载）
