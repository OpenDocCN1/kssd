

# 我自己的 PE 文件 RVA-VA-Offset 心得……

> 原文：[`www.pediy.com/kssd/pediy10/61737.html`](https://www.pediy.com/kssd/pediy10/61737.html)

VA    |   Memory    |           Offset  | Disk Files  |  
  
         |             |                   |             |  
00400000 +-------------+<---------00000000 +-------------+   
         | DOS Header  |                   | DOS Header  |  
         +-------------+                   +-------------+  
         |  DOS stub   |                   |  DOS stub   |  
         +-------------+                   +-------------+  
         |  PE header  |                   |  PE header  |  
         +-------------+                   +-------------+  
         |Section Table|                   |Section Table|  
         +-------------+  /-------00000600 +-------------+  
         |000 Full 0000|  |                |   .text     |  
         |000 Zero 0000|  |  /----0000D400 +-------------+  
00401000 +-------------+<-/  |             |   .data  -A\-| 0000D450   
         |   .text     |     |  /-00010000 +-------------+  
         +-------------+     |  |          |   .rsrc     |  
         |000 Full 0000|     |  |          +-------------+  
         |000 Zero 0000|     |  |  
0040E000 +-------------+<----/  |  
0040E050 |-A .data     |        |  
         +-------------+        |  
         |000 Full 0000|        |  
         |000 Zero 0000|        |  
00413000 +-------------+<-------/  
         |   .rsrc     |  
         +-------------+                   
  
#   Name     Virt Size   RVA       Phys Size  Phys Off   Flags    
\--  -------- ---------  ---------  ---------  ---------  ---------   
01  .text     0000CCC0   00001000   0000CE00   00000600   60000020 \[\]   
02  .data     00004628   0000E000   00002C00   0000D400   C0000040 \[\]   
03  .rsrc     000003C8   00013000   00000400   00010000   40000040 \[\]   
  
  
对于变量 A 来说：  
  
File\_Offset   就是磁盘文件中 A 的位置。  
              File\_Offset = VA - ImageBase - VRk = RVA - VRk  
  
ImageBase     就是文件加载到内存的起始位置。  
              ImageBase = VA - RVA  
              多为：0x00400000,0x01000000  
  
EntryPoint    就是.text 的 VA 地址。  
  
VA    就是内存映像中 A 的位置，即 A 的地址。  
      VA = ImageBase + FileOffset + VRk = ImageBase + RVA  
  
RVA   就是内存映像中 A 的位置与文件映射基址的差。  
      RVA = VA - ImageBase = File\_Offset + VRk  
  
  PS: VA  就好比是“某一时刻”，例如：九点到校上课。  
      RVA 就好比是“某一时间段”，例如：七点起床后，过两个小时到校上课。  
      VA  是绝对的，一旦确定便不可更改。九点到校迟一秒钟也不行。  
      RVA 是系相对的，虽然说过两个小时到校上课，但是如果我七点起床，便是九点上课。  
         而如果我八点起床，便是十点上课了。  
  
VRk  就是文件映射到内存后，每一节之间填充的 00 的个数。  
         由于要进行对齐，所以文件加载到内存后每一节之间要填充大量 00  
         因此文件中 A 的位置会变化。由于各个文件头与各个节大小不变，所以:  
         VRk = RVA - File\_Offset = <填充的 00 的个数>  
  
最终,我们计算：  
  
由 FileOffset 到 RVA/VA:   
  
     ImageBase    = 00400000  
     A FileOffset = 00000450  
  
     RVA = FileOffset(A) + ( RVA(.data) - FileOffset(.data) )  
         = FileOffset(A) +  VRk  
         = 0000D450 + ( 0000E000 - 0000D400 )  
         = 0000D450 + C00  
         = 0000E050  
  
     VA  = RVA + ImageBase = 0000E050 + 00400000 = 0040E050  
  
  
由 RVA/VA 到 FileOffset:  
  
     RVA = 0000E050   /   VA = 0040E050   /   ImageBase = 00400000  
  
     FileOffset = RVA(A) - RVk  
                = RVA(A) - ( RVA(.data) - FileOffset(.data) )  
                = 0000E050 - ( 0000E000 - 0000D400 )  
                = 0000E050 - C00  
                = 0000D450