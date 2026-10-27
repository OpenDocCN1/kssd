
```python
#!/usr/bin/env python  # -\*- coding: UTF-8 -\*-
'''  SimpleGoParser.py:  IDA Plugin for Golang Executable file parsing.  '''
import idautils, idc, idaapi
import sys
import string

def getstring(addr):
    out = ""
    while True:
        value = idc.get_wide_byte(addr)
        if value != 0:
            out += chr(value)
        else:
            break
        addr += 1
    return out

STRIP_CHARS = [ '(', ')', '[', ']', '{', '}', ' ', '"' ]
REPLACE_CHARS = ['.', '\*', '-', ',', ';', ':', '/', '\xb7' ]
def cleanfuncname(funcName):
    for c in STRIP_CHARS:
        funcName = funcName.replace(c, '')
    for c in REPLACE_CHARS:
        funcName = funcName.replace(c, '\_')
    return funcName

class Pclntab():
    # symtab.go file
    # type pcHeader struct {
    #     magic          uint32  // 0xFFFFFFFA
    #     pad1, pad2     uint8   // 0,0
    #     minLC          uint8   // min instruction size
    #     ptrSize        uint8   // size of a ptr in bytes
    #     nfunc          int     // number of functions in the module
    #     nfiles         uint    // number of entries in the file tab.
    #     funcnameOffset uintptr // offset to the funcnametab variable from pcHeader
    #     cuOffset       uintptr // offset to the cutab variable from pcHeader
    #     filetabOffset  uintptr // offset to the filetab variable from pcHeader
    #     pctabOffset    uintptr // offset to the pctab varible from pcHeader
    #     pclnOffset     uintptr // offset to the pclntab variable from pcHeader
    # }

    def __init__(self):
        self.MAGIC = 0xFFFFFFFA
        self.offset = 0
        self.baseAddress = 0
        self.minLC = 0
        self.ptrSize = 0
        self.functionCounts = 0
        self.fileCounts = 0
        self.functionNameOffset = 0
        self.cuOffset = 0
        self.filetabOffset = 0
        self.pctabOffset = 0
        self.functabOffset = 0

    def parse(self, baseAddress):
        print("Pclntab Go")
        magic = idc.get_wide_dword(baseAddress)
        if magic != self.MAGIC:
            print("Pclntab address invalid");
            return False

        self.baseAddress = baseAddress
        self.offset = 4 + 2
        self.minLC = idc.get_wide_byte(baseAddress + self.offset)
        self.offset = self.offset + 1

        self.ptrSize = idc.get_wide_byte(baseAddress + self.offset)
        self.offset = self.offset + 1

        self.functionCounts = idc.get_wide_dword(baseAddress + self.offset)
        self.offset = self.offset + 8    #  静态文件中实际占8字节

        self.fileCounts = idc.get_wide_dword(baseAddress + self.offset)
        self.offset = self.offset + 8    #  静态文件中实际占8字节

        self.functionNameOffset = baseAddress + idc.get_qword(baseAddress + self.offset)
        self.offset = self.offset + 8

        self.cuOffset = baseAddress + idc.get_qword(baseAddress + self.offset)
        self.offset = self.offset + 8

        self.filetabOffset = baseAddress + idc.get_qword(baseAddress + self.offset)
        self.offset = self.offset + 8

        self.pctabOffset = baseAddress + idc.get_qword(baseAddress + self.offset)
        self.offset = self.offset + 8

        self.functabOffset = baseAddress + idc.get_qword(baseAddress + self.offset)

        print("Pclntab Base Info")
        print("Min Instuction Size %d Pointer Size %d Function Counts %d File Counts %d" % (self.minLC, self.ptrSize, self.functionCounts, self.fileCounts))
        print("Function Name Offset 0x%x CuOffset 0x%x File Table Offset 0x%x Pctab Offset 0x%x Function Table Offset 0x%x" % (self.functionNameOffset, self.cuOffset, self.filetabOffset, self.pctabOffset, self. functabOffset))

        return True

    def setfuncname(self):
        index = 0
        while index < self.functionCounts:
            funcAddr = idc.get_qword(self.functabOffset + 2 * index * self.ptrSize)
            funcMetaAddr = self.functabOffset + idc.get_qword(self.functabOffset + 2 * index * self.ptrSize + self.ptrSize)
            if funcAddr != idc.get_qword(funcMetaAddr):
                print("Traverse Function Failed")
            else:
                funcNameAddr = self.functionNameOffset + idc.get_wide_dword(funcMetaAddr + self.ptrSize)
                funcName = getstring(funcNameAddr)
                funcName = cleanfuncname(funcName)
                print("Function Address 0x%x Name %s" % (funcAddr, funcName))
                if idc.get_wide_dword(funcAddr) == 0x0c8b4865:
                   idc.create_insn(funcAddr)
                   idc.add_func(funcAddr)
                idc.set_name(funcAddr, funcName, SN_CHECK)

            index = index + 1

def main():
    pc = Pclntab()
    if pc.parse(0xE90E60) == True:
        pc.setfuncname()

if __name__ == '__main__':
    main()
```

## 未完待续

1. 当前脚本需要手动寻找`pclntab`所在地址，可以优化自动寻找；
2. Go二进制中的数据结构解析；
3. 很多Go函数IDA不能很好识别，如下所示以字节码（65 48 8B 0C 25 28 00 00 00    mov     rcx, gs:28h）开始的都是函数，即使调用`idc.add_func()`定义为函数IDA也不能全部转换，不知道道友们否有更好的解决办法

![](https://bbs.kanxue.com/upload/attach/202106/771611_9QEBX8YENYMSHB5.jpg)

## 引用

[1] [1c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9q4)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6s2L8H3`.`.](https://bbs.kanxue.com/elink@34dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9q4)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6s2L8H3`.`.)

[2] [720K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9h3q4F1j5$3q4A6j5h3#2S2L8#2)9J5k6h3N6A6N6r3u0G2L8$3E0K6i4K6u0W2K9h3!0Q4x3V1k6Y4L8#2)9J5k6r3W2F1N6r3g2J5L8X3q4D9M7#2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6*7K9q4)9J5c8U0l9K6i4K6u0W2y4g2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@db0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9h3q4F1j5$3q4A6j5h3#2S2L8#2)9J5k6h3N6A6N6r3u0G2L8$3E0K6i4K6u0W2K9h3!0Q4x3V1k6Y4L8#2)9J5k6r3W2F1N6r3g2J5L8X3q4D9M7#2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6*7K9q4)9J5c8U0l9K6i4K6u0W2y4g2)9J5k6h3S2@1L8h3H3`.)

[3] [e72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1M7W2)9J5k6r3E0F1P5W2)9J5k6h3&6W2N6q4)9J5c8X3N6G2i4K6u0V1j5$3q4D9L8r3W2F1k6#2)9J5k6r3y4G2L8Y4k6W2L8Y4c8A6L8$3&6Q4x3X3c8^5z5o6k6Q4x3X3b7$3y4q4)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@c47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1M7W2)9J5k6r3E0F1P5W2)9J5k6h3&6W2N6q4)9J5c8X3N6G2i4K6u0V1j5$3q4D9L8r3W2F1k6#2)9J5k6r3y4G2L8Y4k6W2L8Y4c8A6L8$3&6Q4x3X3c8^5z5o6k6Q4x3X3b7$3y4q4)9J5k6h3S2@1L8h3H3`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
