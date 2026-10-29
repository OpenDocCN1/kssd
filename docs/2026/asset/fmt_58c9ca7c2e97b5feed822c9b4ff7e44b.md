
```
com.example.goal I/hooking: HookGoal hookGoalNumber:666

com.example.goal I/hooking: eat is hooking

com.example.goal I/hooking: HookGoal hookGoalNumber:666

com.example.goal I/hooking: welcome

com.example.goal I/hooking: 调用DiyClass中getData() 返回值：666

com.example.goal I/hooking: func2 is hooking

com.example.goal I/func3: [Lcom.example.goal.DiyClass;@4a89752c

com.example.goal I/hooking: func3修改参数

com.example.goal I/hooking: DiyClass[0].getData:666

com.example.goal I/hooking: DiyClass[1].getData:666

com.example.goal I/hooking: DiyClass[2].getData:666

com.example.goal I/inner Constructor: is hooking

com.example.goal I/hooking: InnerClass 构造函数 is hooking

com.example.goal I/hooking: InnerClass innerNumber:0

com.example.goal I/inner Constructor: 修改前的innerNumber：0

com.example.goal I/inner Constructor: 修改后的innerNumber：6

com.example.goal I/innerFunc: 修改前的innerNumber：6

com.example.goal I/innerFunc: 修改后的innerNumber：666

com.example.goal I/innerFunc: is hooking

com.example.goal I/hooking: InnerClass innerFunc is hooking
```

错误之处，还请不吝赐教，十分感谢！

同时也希望有疑惑的同学留下你的问题，大家多多交流。

实践是检验真理的唯一标准，今后还要勤加练习，多多实战。

**计划：**

- 掌握Java层xposed hook及原理
- 掌握native层frida hook及原理
- 自己实现hook框架

学如逆水行舟，不进则退。与君共勉！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-6-25 20:44 被堂前燕编辑，原因：修正一些错误
