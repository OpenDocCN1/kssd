
# 在 VC 中播放 mid 音乐的最简单方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-782.htm`](https://www.pediy.com/kssd/pediy07/pediy7-782.htm)

前一段时间看到有人发布 `XM` 音乐的播放方法，主要用于在注册机加一个小巧的背景音乐。

但由于 `XM` 音乐比较少，比较小巧的就更少了，况且还依赖 `minifmod` 库。

所以考虑到以下因素，同样推荐在注册机加入 `MIDI` 音乐：

1. `MIDI` 音乐文件使用更广泛，更容易获得、制作和编辑。
2. `DirectMusic` 附带 4MB 音色库的 `Roland` 软波表，音质也不比 `XM` 差。
3. `MIDI` 数据比 `XM` 数据更容易被压缩（`MIDI` 文件压缩比在 1:8 左右）。
4. 不需要外部编程库，系统自带播放内核，添加的代码量极少。

这里给出一个最简单的 `MIDI` 音乐的播放方法，即使用 `DirectMusic` 高层库。

虽然利用 `MCI` 的高层也可以播放，但它似乎不支持内存载入，而且无法避免与其他 `MIDI` 音乐的同时播放的冲突。

估计都有装了 `DirectX` 吧，播放的通用性应该没有问题。但编译这个程序还需要装上 `DirectX SDK`。

```c
// Tiny MIDI player [By Dwing]
#pragma comment(lib,"ole32.lib")
#pragma comment(lib,"dxguid.lib")
#pragma comment(linker,"/ENTRY:Entry")
#include<dmusici.h>

unsigned char mididata[]={0x4D,0x54,0x68,0x64,......}; // 这里加入 mid 文件的数据.

IDirectMusicPerformance*    performance=0;
IDirectMusicSegment*        segment    =0;
IDirectMusicLoader*         loader     =0;

bool PlayMIDI(unsigned char* data,unsigned int size)
{
    DMUS_OBJECTDESC desc={0};
    desc.dwSize     =sizeof(desc);
    desc.guidClass  =CLSID_DirectMusicSegment;
    desc.pbMemData  =data;
    desc.llMemLength=size;
    desc.dwValidData=DMUS_OBJ_CLASS|DMUS_OBJ_MEMORY;
    if(FAILED(CoInitialize(0))) return false;
    if(FAILED(CoCreateInstance(CLSID_DirectMusicPerformance,0,CLSCTX_INPROC,
        IID_IDirectMusicPerformance,(void**)&performance))) return false;
    if(FAILED(CoCreateInstance(CLSID_DirectMusicLoader,0,CLSCTX_INPROC,
        IID_IDirectMusicLoader,(void**)&loader))) return false;
    if(FAILED(loader->GetObject(&desc,IID_IDirectMusicSegment,(void**)&segment))) return false;
    if(FAILED(performance->Init(0,0,0))) return false;
    if(FAILED(performance->AddPort(0))) return false;
    if(FAILED(segment->SetParam(GUID_StandardMIDIFile,-1,0,0,performance))) return false;
    if(FAILED(segment->SetParam(GUID_Download,-1,0,0,performance))) return false;
    if(FAILED(segment->SetRepeats(DMUS_SEG_REPEAT_INFINITE))) return false;
    if(FAILED(performance->PlaySegment(segment,0,0,0))) return false;
    return true;
}

void StopMIDI()
{
    if(segment)
    {
        if(performance)
        {
            performance->Stop(segment,0,0,0);
            segment->SetParam(GUID_Unload,-1,0,0,(void**)performance);
        }
        segment->Release();
        segment=0;
    }
    if(loader)
    {
        loader->Release();
        loader=0;
    }
    if(performance)
    {
        performance->CloseDown();
        performance->Release();
        performance=0;
    }
    CoUninitialize();
}

void Entry()
{
    if(PlayMIDI(mididata,sizeof(mididata)))
    {
        MessageBox(0,"Playing...","MIDI",0);
        StopMIDI();
    }
    else
        MessageBox(0,"Error!","MIDI",0);
    ExitProcess(0);
}
```

`mid` 文件的数据可用一个小工具自己导出成数组格式。

执行 `PlayMIDI()` 后音乐的播放会有约 1~2 秒的延迟，估计不会有太大影响的。

如果在程序退出前不需要停止播放音乐，可以不调用 `StopMIDI()`，程序会更小。

`PlayMIDI()` 函数用 `VC` 的最小代码编译，只有 200 多字节！

恩，是个好的方法，我以前写的注册机是把 `MIDI` 数据写到磁盘上再播放，这个方法省了读写磁盘文件的步骤，方便了一些。但是手写那个 `MIDI` 的二进制数据也是个非常麻烦的事情。推荐流行时代 `Sen` 写的文章，将二进制数据写入资源，然后在内存中释放，这样就可以把手写 `MIDI` 数据的步骤也省略掉了，强烈推荐：

原文如下：

> 近来看到 `prince` 发贴甚多，想我自论坛创建来就没写什么东西，因为本人才疏学浅。今天突然感觉心中不快，写篇小儿文章让大家见笑。文中尚有很多愚钝之辞，还望各位兄弟指出。

> 很久以前 `love` 和 `小子` 就写了一个文件补丁工具，他们的是 `asm` 版本，我今天的是 `C` 的版本，如果没有兴趣，可以就此打住。

我们写程序的时候经常要生成或者是创建新的文件，这个是怎么做到的呢？老的 `CIH` 病毒是这么写的：

```c
BYTE byExe[] = {
  0x4d, 0x87 ......
  ..... .....
}
```

这个如果要生成的文件很大的时候这个就会很难写，而且写出来的源文件会非常的庞大。这个时候，我们可以用载入资源的方式来解决。

用到的相关 `API`：

* `FindResource`：查找一个资源。我们是把相关要生成的文件载入，就是用这个函数来确定其资源的位置。
* `SizeofResource`：获得资源的尺寸。
* `LoadResource`：装载资源，装入到内存中。
* `LockResource`：锁定资源，在内存中锁定。

好了，现在在 `VC` 的工程中载入这个文件吧。首先，我们把 `***.exe` 或者 `***.mid` 改成 `***.bin` 二进制文件，在资源文件上点击右键，选择 `Import`（导入）。这里我们为自定义资源类型，即 `Custom Resource Type`，`Resource type` 为读者兴趣随便填写，这里用 `MyRes`，资源名称用 `IDR_MyRes`。

好了，现在就可以写一个函数进行文件的生成了：

```c
BOOL Create()
{
    HRSRC hResInfo;
    HGLOBAL hResData;
    DWORD dwSize, dwWritten;
    HANDLE hFile;
    //开始所需的资源
    hResInfo = FindResource(NULL,MAKEINTRESOURCE(IDR_MyRes),"MyRes");
    if(hResInfo == NULL)
        return FALSE;
    //获得资源尺寸
    dwSize = SizeofResource(NULL,hResInfo);
    //装载
    hResData = LoadResource(NULL,hResInfo);
    if(hResData == NULL)
        return FALSE;
    //最重要的地方，写文件
    hFile = CreateFile("c:\\MyResG.exe",GENERIC_WRITE,0,NULL,CREATE_ALWAYS,0,NULL);
    if(hFile == NULL)
        return FALSE;
    WriteFile(hFile,(LPCVOID)LockResource(hResData),dwSize,&dwWritten,NULL);
    CloseHandle(hFile);
    return TRUE;
}
```

用这个函数就可以把资源文件的东西拖出来了，哈哈。完工。

原来那些数据需要手写 `-_-!` 汗~~~

写个小程序来自动帮我们写吧！

```c
#include<stdio.h>
void main()
{
    unsigned char c;
    unsigned int  n=0;
    FILE *fin =fopen("data.bin","rb");
    FILE *fout=fopen("data.h"  ,"wb");
    if(!fin||!fout) {printf("not found data.bin!\n");return;}
    fprintf(fout,"unsigned char *data[]={\r\n");
    while(1)
    {
        c=fgetc(fin);
        if(feof(fin)) break;
        fprintf(fout,"0x%02X,",c);
        if(++n%16==0) fprintf(fout,"\r\n");
    }
    fseek(fout,-1,SEEK_CUR);
    fprintf(fout,"\r\n};\r\n");
    fclose(fout);
    fclose(fin);
}
```

当然使用资源也是一个好办法，但比起直接使用数组缺点如下：

1. 需要调用 `FindResource`, `SizeofResource`, `LoadResource`, `LockResource` 等函数，代码量多一些。
2. 资源目录表和资源名称（如果使用字符串格式）需要一些额外的数据，且不能被加壳工具压缩。
3. 使用资源就非常明显让别人轻易提取出该资源的数据。
4. 便于某些无资源编译器或不方便编辑资源的编译器编译（如 `Dev C++`）。

还有另一个方法可以直接包含文件，而不是一堆十六进制数据。但需要有 `nasm` 汇编器（把 `nasmw.exe` 放入 `VC` 编译程序目录中）。

工程中添加文件 `mididata.asm`，内容：

```asm
section .data
global _mididata
_mididata incbin "midi.mid" ;包含文件的文件名,与此文件同目录
```

设置 `mididata.asm` 的编译属性：

* `Custom Build/Commands`: `nasmw -f win32 -o $(OutDir)\$(InputName).obj $(InputPath)`
* `Custom Build/Outputs`: `$(OutDir)\$(InputName).obj`

然后只需要在需要 `mididata` 的 `C++` 文件中加入下面一行即可：

```c
extern "C" unsigned char mididata[12345]; //大小需要手动输入
```

哪个方法更方便自己决定吧......

`4dd466d0e1e0cc123afbe504da2462ca`
