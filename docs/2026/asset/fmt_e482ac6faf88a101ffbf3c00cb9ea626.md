
```python
# -*- coding: utf-8 -*-
from com.pnfsoftware.jeb.client.api import IClientContext
from com.pnfsoftware.jeb.core import IRuntimeProject
from com.pnfsoftware.jeb.core.units import IUnit, INativeCodeUnit
from com.pnfsoftware.jeb.core.units.code import EntryPointDescription
from com.pnfsoftware.jeb.core.units.code.asm.analyzer import INativeCodeAnalyzer, INativeCodeModel, IReferenceManager, ICallGraphManager, ICallGraph, CallGraphVertex
from com.pnfsoftware.jeb.core.units.code.asm.items import INativeMethodItem

def Test(ctx):
    input_path = r"D:\tmp\2\project\about_dex_diff\code\xmly\libFace3D.so"
    unit = ctx.open(input_path)
    prj = ctx.getMainProject()
    # 获取INativeCodeUnit并执行解析
    nativeCodeUnit = prj.findUnit(INativeCodeUnit)
    bool = nativeCodeUnit.process()
    # 获取INativeCodeAnalyzer,获取INativeCodeModel
    nativeCodeAnalyzer = nativeCodeUnit.getCodeAnalyzer()
    nativeCodeAnalyzer.analyze()
    nativeCodeModel = nativeCodeAnalyzer.getModel()
    # 返回该地址所在函数的首地址
    print "-------------------"
    r = nativeCodeModel.getContainedRoutineAddresses(0x19A60)
    print ">>> ", hex(r[0])
    # 返回该地址所在基本块
    print "-------------------"
    r = nativeCodeModel.getBasicBlockHeader(0x19A60)
    for insn in r.getInstructions():
        print ">>> ", insn.getMnemonic()
```

![image.png](https://bbs.kanxue.com/upload/attach/202010/760871_HE247W9EEGC3UK8.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-10-28 14:32

被爱吃菠菜编辑，原因：
