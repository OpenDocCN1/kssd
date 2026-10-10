

# Asprotect-小甜饼

> 原文：[`www.pediy.com/kssd/pediy03/forum313.htm`](https://www.pediy.com/kssd/pediy03/forum313.htm)

 小甜饼 --- 有关新版 Asprotect 加壳程序的脱壳的又一种思路   
1、用 PEditor 来 Dump 出整个文件。   
2、用 BWW 得到 Entry Point。   
3、用 Imp\_List 获得 Import Table。   
虽然我们得到的 Import Table 有部分函数不对，但是比完全重建构建一个 Import Table 的工作量还是小多了。下一步就来找到这些被 Asprotect 作了手脚的函数。   
  
方法之一：找出脱壳过的老版本对比引入函数，可以知道。这种方法不用动态跟踪。方便。   
  
方法之二：动态跟踪也可以得到这些函数。其实被 Asprotect 处理过的这些函数的引入地址被定位到 Asprotect 自己的一个过渡函数，在这个过渡函数中会用 XOR 方法还原出真正的函数名，并用 LoadLibraryA 等加载，得到函数真正的入口。然后进入调用该函数。   
  
4、按照通常的脱壳方法中做修补工作。按 Import Table 的结构修改即可。   
  
  
有兴趣的朋友可以想想办法，编个程序得到这些加密处理过的函数名称。可能要挂钩某些函数，要些技巧。   
  
ljttt 写于 2000-09-10 日   4dd466d0e1e0cc123afbe504da2462ca