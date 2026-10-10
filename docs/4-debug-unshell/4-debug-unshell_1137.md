

# Upx 壳

> 原文：[`www.pediy.com/kssd/pediy03/forum385.htm`](https://www.pediy.com/kssd/pediy03/forum385.htm)

 今天下午刚下载了 ASPack+ASProtect 中文注册版，用 fi 查出 aspack.exe   
是被 upx 压缩的，于是掏出 upx 解之，结果 upx 却报错。   
  
用 hedit 打开 aspack.exe，发现此文件头被修改过。   
在 1A0 处可看见 UPX1 字样，它的上面有个 code 字样，把 code 改为 UPX0，   
往下在 27C 处可看到 0C 09 02 0A，把它前面的 4 个字节改为 UPX!，保存。   
本以为这样就可以解开 aspack.exe，可是用 upx 一解压还是报错。   
嘿，有点意思。   
  
再仔细看看 aspack.exe，从 298 到 2AD 处都为 0，不好，莫非它把那重要的   
24 个字节给清空了，要是那样可就无法解开喽。还好，那 24 给字节还在，   
只是往前移了一点。把从 278 到 297 复制下来（共 32 字节），替换掉 2AE 前   
的 32 个字节，再把 278 处的 UPX!改为 0000（随便改，只要不是 UPX!就可   
以了），保存。用 upx 解压 OK。   
  
用兴趣的可以去解 aspr-rp.exe，方法相同。   4dd466d0e1e0cc123afbe504da2462ca