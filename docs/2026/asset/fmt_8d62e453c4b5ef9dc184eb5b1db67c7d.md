
``` 
{安装SEH}

  asm

              push offset DefaultExceptionHandler

              push fs:[0]

              mov fs:[0], esp



              mov sseh.SafeEip, offset SafePlace

              mov sseh.PrevEbp, ebp

              mov sseh.PrevEsp, esp

  end;
```

但如果是`D7`以后的版本，你就要写成：

``` 
{安装SEH}

          asm

            push offset DefaultExceptionHandler

            push fs:[0]

            mov fs:[0], esp



            mov eax, dword ptr sseh ;在`D2010`里,`sseh`是个指针

            mov (_SEH PTR [eax]).SafeEip, offset SafePlace

            mov (_SEH PTR [eax]).PrevEbp, ebp

            mov (_SEH PTR [eax]).PrevEsp, esp

          end;
```

对于嵌入汇编的这个差别大家就一定要注意了，这个`rmcoff`也无能为力，只有大家写程序的时候自己注意了。不过我还是推荐大家用`Delphi2010`写驱动，因为`2010`提供的一些新特性可以大大降低写驱动的复杂性。当然`Kmdkit4D`会尽力兼容`Delphi7`的。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
