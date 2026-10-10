

# Crypkey

> 原文：[`www.pediy.com/kssd/pediy05/pediy50369.htm`](https://www.pediy.com/kssd/pediy05/pediy50369.htm)

如果您发现您的软件目录中有 CRP9516x.DLL、CRYP95x.DLL、CRP32DLL.DLL 等文件，那么它很可能采用了 crypkey 加密系统。这种软件一般在它的 File 菜单下有一个 license 子菜单，其中有类似与 authorize 的菜单项，打开它会出现一个对话框，里面显示了一个称为 site code 的字串，它实际上包含了你这台电脑的特定信息，把这个字串发给软件开发商，他会给你一个称为 site key 的字串（当然要先交钱），这个过程叫做 authorization（授权），把这个 site key 填在对话框指定的区域里，你就可以使用该软件了。   
  由 site code 计算 sitekey 的算法中用到了以下几个参数：master key、user key、 product num、password、filename 等，前三个是 crypkey 公司提供给软件开发商的，口令和文件名是开发商自己指定的，这些信息可以通过跟踪调试目标文件得到，有了这些参数，使用 Crypkey SDK 或 Crypkey Instant 提供的 site key generator，就可以生成需要的 site key,从而实现软件的授权。   
   4dd466d0e1e0cc123afbe504da2462ca