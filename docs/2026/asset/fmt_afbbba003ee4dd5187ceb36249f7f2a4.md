
```python
# 学习pe的最好方法，就是自己写一个PE文件。这个例子展示了用python生成一个pe文件
import struct
import time
MZ\_MAGIC = 0x5A4D
PE\_MAGIC = 0x4550
IMAGE\_FILE\_MACHINE\_I386 = 0x014c
IMAGE\_SCN\_MEM\_EXECUTE = 0x20000000 # Section is executable.
IMAGE\_SCN\_MEM\_READ = 0x40000000 # Section is readable.
IMAGE\_SCN\_MEM\_WRITE = 0x80000000 # Section is writeable.
IMAGE\_SCN\_CNT\_CODE = 0x00000020
IMAGE\_SCN\_CNT\_INITIALIZED\_DATA = 0x00000040
class DOS\_HEADER\_32(object):
'''
DOS头只关心 magic 和 e\_lfanew 位置就行
'''
e\_magic = MZ\_MAGIC
e\_cblp, e\_cp, e\_crlc, e\_cparhdr, e\_minalloc, e\_maxalloc, e\_ss, e\_sp, \
e\_csum, e\_ip, e\_cs, e\_lfarlc, e\_ovno, e\_res, e\_oemid, \
e\_oeminfo, e\_res2, e\_lfanew = [0] \* 18
def \_\_init\_\_(self):
self.fmt = "<30HL"
# 小端模式 30个H(unsigned short 占2byte) 后一个是L(unsigned long 占 4byte)
self.e\_res = [0, 0, 0, 0]
self.e\_res2 = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
def raw(self):
return struct.pack(self.fmt, self.e\_magic, self.e\_cblp, self.e\_cp,
self.e\_crlc, self.e\_cparhdr, self.e\_minalloc,
self.e\_maxalloc, self.e\_ss, self.e\_sp, self.e\_csum,
self.e\_ip, self.e\_cs, self.e\_lfarlc, self.e\_ovno,
self.e\_res[0], self.e\_res[1], self.e\_res[2], self.e\_res[3],
self.e\_oemid, self.e\_oeminfo,
self.e\_res2[0], self.e\_res2[1], self.e\_res2[2], self.e\_res2[3],
self.e\_res2[4], self.e\_res2[5], self.e\_res2[6], self.e\_res2[7],
self.e\_res2[8], self.e\_res2[9], self.e\_lfanew)
# e\_lfanew是文件偏移
def getPEOffset(self):
return self.e\_lfanew
def getSize(self):
'''
DOS头,SIZE:30\*2+1\*4=64
:return:
'''
return struct.calcsize(self.fmt)
class IMAGE\_NT\_HEADER\_32(object):
def \_\_init\_\_(self):
self.Signature = PE\_MAGIC
self.file\_header = self.IMAGE\_FILE\_HEADER()
self.optional\_header = self.IMAGE\_OPTIONAL\_HEADER32()
def getSize(self):
'''
PE文件头,SIZE:4+20+224=248
:return:
'''
return 4 + self.file\_header.getSize() + self.optional\_header.getSize()
def raw(self):
return struct.pack("<L", self.Signature) + self.file\_header.raw() + self.optional\_header.raw()
class IMAGE\_FILE\_HEADER:
Machine, \
NumberOfSections, \
TimeDateStamp, \
PointerToSymbolTable, \
NumberOfSymbols, \
SizeOfOptionalHeader, \
Characteristics = IMAGE\_FILE\_MACHINE\_I386, 0, 0, 0, 0, 0, 0
def \_\_init\_\_(self):
self.fmt = "<2H3L2H"
def getSize(self):
'''
PE文件逻辑分布的信息,SIZE:2\*2+3\*4+2\*2=20
:return:
'''
return struct.calcsize(self.fmt)
def raw(self):
return struct.pack(self.fmt, self.Machine,
self.NumberOfSections,
self.TimeDateStamp,
self.PointerToSymbolTable,
self.NumberOfSymbols,
self.SizeOfOptionalHeader,
self.Characteristics)
class IMAGE\_OPTIONAL\_HEADER32:
Magic = 0x10b # 32位为0x10B，64位为0x20B，ROM镜像为0x107
MajorLinkerVersion = 0
MinorLinkerVersion = 0
SizeOfCode = 0 # 一般放在“.text”节里。如果有多个代码节的话，它是所有代码节的和。必须是FileAlignment的整数倍，是在文件里的大小。
SizeOfInitializedData = 0
SizeOfUninitializedData = 0
AddressOfEntryPoint = 0 # 代码入口点的偏移量,RVA
BaseOfCode = 0 # 代码基址,可执行代码的偏移值,RVA
BaseOfData = 0 # 数据基址,已初始化数据的偏移值,RVA
ImageBase = 0 # 程序默认装入基地址,提供整个二进制文件包括所有头的优先（线性）载入地址,RVA
SectionAlignment = 0
FileAlignment = 0
MajorOperatingSystemVersion = 0
MinorOperatingSystemVersion = 0
MajorImageVersion = 0
MinorImageVersion = 0
MajorSubsystemVersion = 4
MinorSubsystemVersion = 0
Win32VersionValue = 0
SizeOfImage = 0 # 内存中整个PE映像体的尺寸。它是所有头和节经过节对齐处理后的大小。
SizeOfHeaders = 0 # DOS头、PE头、区块表的总大小，也就等于文件尺寸减去文件中所有节的尺寸。可以以此值作为PE文件第一节的文件偏移量。
CheckSum = 0 # 映像效验和
Subsystem = 2 # 文件子系统,NT用来识别PE文件属于哪个子系统。 对于大多数Win32程序，只有两类值: Windows GUI 和Windows CUI (控制台)。
DllCharacteristics = 0
SizeOfStackReserve = 0
SizeOfStackCommit = 0
SizeOfHeapReserve = 0
SizeOfHeapCommit = 0
LoaderFlags = 0
NumberOfRvaAndSizes = 0x10 # 指定DataDirectory的数组个数，由于以前发行的Windows NT的原因，它只能为16。 -> 00 00 00 10
DATA\_DIRECTORY = []
def \_\_init\_\_(self):
self.fmt = "<HBB9L6H4L2H6L"
def getSize(self):
'''
SIZE:19\*4+9\*2+2\*1+16\*8=224
:return:
'''
selfsize = struct.calcsize(self.fmt)
for image\_data in self.DATA\_DIRECTORY:
selfsize += image\_data.getSize()
return selfsize
def raw(self):
selfdata = struct.pack(self.fmt, self.Magic,
self.MajorLinkerVersion,
self.MinorLinkerVersion,
self.SizeOfCode,
self.SizeOfInitializedData,
self.SizeOfUninitializedData,
self.AddressOfEntryPoint,
self.BaseOfCode,
self.BaseOfData,
self.ImageBase,
self.SectionAlignment,
self.FileAlignment,
self.MajorOperatingSystemVersion,
self.MinorOperatingSystemVersion,
self.MajorImageVersion,
self.MinorImageVersion,
self.MajorSubsystemVersion,
self.MinorSubsystemVersion,
self.Win32VersionValue,
self.SizeOfImage,
self.SizeOfHeaders,
self.CheckSum,
self.Subsystem,
self.DllCharacteristics,
self.SizeOfStackReserve,
self.SizeOfStackCommit,
self.SizeOfHeapReserve,
self.SizeOfHeapCommit,
self.LoaderFlags,
self.NumberOfRvaAndSizes)
for image\_data in self.DATA\_DIRECTORY:
selfdata += image\_data.raw()
return selfdata
class IMAGE\_DATA\_DIRECTORY:
VirtualAddress = 0
Size = 0
def \_\_init\_\_(self):
pass
def raw(self):
return struct.pack("<2L", self.VirtualAddress, self.Size)
def getSize(self):
return 0x4 \* 2
class Section:
def \_\_init\_\_(self):
self.fmt = "<LLLLLLHHL"
self.Name = ""
self.VirtualSize = self.VirtualAddress = self.SizeOfRawData = self.PointerToRawData = \
self.PointerToRelocations = self.PointerToLinenumbers = \
self.NumberOfRelocations = self.NumberOfLinenumbers = \
self.Characteristics = 0
# VirtualSize 被实际使用的区块大小,也可是PhysicalAddress,在可执行文件中，它是内容的大小.在目标文件中，它是内容重定位到的地址；
# VirtualAddress 区块的RAV地址（相对虚拟地址）。,节中数据的RVA。
# SizeOfRawData 该块在磁盘中所占的大小,原始数据大小,经过文件对齐处理后节尺寸，PE装载器提取本域值了解需映射入内存的节字节数
# PointerToRawData 该块在磁盘文件中的偏移,文件偏移,这是节基于文件的偏移量，PE装载器通过本域值找到节数据在文件中的位置。
def getSize(self):
return struct.calcsize(self.fmt) + 8
def has(self, rva, imagebase=0):
return (self.VirtualAddress + imagebase) <= rva < (self.VirtualAddress + self.VirtualSize + imagebase)
def hasOffset(self, offset):
return self.PointerToRawData <= offset < (self.PointerToRawData + self.VirtualSize)
def raw(self):
self.Name = (self.Name + "\x00" \* (8 - len(self.Name)))[:8]
return self.Name.encode() + struct.pack(self.fmt, self.VirtualSize,
self.VirtualAddress, self.SizeOfRawData, self.PointerToRawData,
self.PointerToRelocations, self.PointerToLinenumbers,
self.NumberOfRelocations, self.NumberOfLinenumbers,
self.Characteristics)
class ImportDescriptor:
def \_\_init\_\_(self):
self.fmt = "<LLLLL"
self.OriginalFirstThunk = self.TimeDateStamp = self.ForwarderChain = self.Name = \
self.FirstThunk = 0
def raw(self):
return struct.pack(self.fmt, self.OriginalFirstThunk, self.TimeDateStamp, self.ForwarderChain, self.Name, \
self.FirstThunk)
def getSize(self):
return struct.calcsize(self.fmt)
# typedef struct \_IMAGE\_THUNK\_DATA32 {
# union {
# DWORD ForwarderString; // PBYTE
# DWORD Function; // PDWORD
# DWORD Ordinal;
# DWORD AddressOfData; // PIMAGE\_IMPORT\_BY\_NAME
# } u1;
# } IMAGE\_THUNK\_DATA32;
class ImageThunkData32:
Function = 0
def getSize(self):
return 4
def raw(self):
return struct.pack("<L", self.Function)
class ImageImportByName:
def \_\_init\_\_(self):
self.fmt = "<H"
self.Hint = 0
self.Name = ""
def getSize(self):
size = len(self.Name) + 3 # 1 for \0 + 2 for Hint
if size % 2:
size += 1 # Padding
return size
def raw(self):
raw = struct.pack(self.fmt, self.Hint) + self.Name.encode() + b"\x00"
if len(raw) % 2:
raw += "\0" # padding
return raw
def align(idx, aligment):
return (idx + aligment) & ~(aligment - 1)
def dword(v):
return struct.pack("<L", v)
if \_\_name\_\_ == '\_\_main\_\_':
length = 0
mz = DOS\_HEADER\_32()
mz.e\_lfanew = mz.getSize()
length += mz.getSize()
# 设置pe头入口
pe = IMAGE\_NT\_HEADER\_32()
pe.file\_header.NumberOfSections = 1 # section数量
pe.file\_header.TimeDateStamp = int(time.time())
pe.file\_header.Characteristics = 1 + 2 + 4 + 256
# refer https://blog.csdn.net/qiming\_zhang/article/details/7309909#3.2.2
pe.optional\_header.AddressOfEntryPoint = 0x1000
pe.optional\_header.ImageBase = 0x400000
pe.optional\_header.SectionAlignment = 0x1000
pe.optional\_header.FileAlignment = 0x200
for i in range(pe.optional\_header.NumberOfRvaAndSizes):
pe.optional\_header.DATA\_DIRECTORY.append(pe.IMAGE\_DATA\_DIRECTORY())
pe.file\_header.SizeOfOptionalHeader = pe.optional\_header.getSize()
length += pe.getSize()
# .text section
text = Section()
text.Name = ".text"
text.Characteristics = IMAGE\_SCN\_CNT\_CODE | IMAGE\_SCN\_MEM\_READ | IMAGE\_SCN\_MEM\_EXECUTE
text.VirtualAddress = 0x1000
# .rdataracteristics = IMAGE\_SCN\_CNT\_INITIALIZED\_DATA | IMAGE\_SCN\_MEM\_READ
length += text.getSize()
# pading
pe.optional\_header.SizeOfHeaders = align(length, pe.optional\_header.FileAlignment)
padding = (pe.optional\_header.SizeOfHeaders - length) \* b'\x00'
length = pe.optional\_header.SizeOfHeaders
# 写入text代码
buf = b""
buf += b"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b"
buf += b"\x50\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7"
buf += b"\x4a\x26\x31\xff\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf"
buf += b"\x0d\x01\xc7\xe2\xf2\x52\x57\x8b\x52\x10\x8b\x4a\x3c"
buf += b"\x8b\x4c\x11\x78\xe3\x48\x01\xd1\x51\x8b\x59\x20\x01"
buf += b"\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b\x01\xd6\x31"
buf += b"\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03\x7d"
buf += b"\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66"
buf += b"\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0"
buf += b"\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f"
buf += b"\x5f\x5a\x8b\x12\xeb\x8d\x5d\x6a\x01\x8d\x85\xb2\x00"
buf += b"\x00\x00\x50\x68\x31\x8b\x6f\x87\xff\xd5\xbb\xf0\xb5"
buf += b"\xa2\x56\x68\xa6\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a"
buf += b"\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x53"
buf += b"\xff\xd5\x63\x61\x6c\x63\x00"
section\_text = buf
text.VirtualSize = len(section\_text)
text.SizeOfRawData = align(text.VirtualSize, pe.optional\_header.SectionAlignment)
text.PointerToRawData = length
section\_text += b"\x00" \* (text.SizeOfRawData - len(section\_text))
length += len(section\_text)
# 最后数据的完善
pe.optional\_header.SizeOfImage = align(length,
pe.optional\_header.SectionAlignment) # // Image大小,内存中整个PE文件的映射的尺寸，可比实际的值大，必须是SectionAlignment的整数倍
# 生成二进制
code = b""
code += mz.raw()
code += pe.raw()
# 生成section代码
code += text.raw()
code += padding
# 生成每个section具体代码
code += section\_text
print(code)
with open("test.exe", "wb") as f:
f.write(code)
```
