
# 逆向应该知道的~!

> 原文：[`www.pediy.com/kssd/pediy10/92500.html`](https://www.pediy.com/kssd/pediy10/92500.html)

前段时间，发的贴受到很大的争议，所以，为了避免类似的情况，我想说明一下，此贴只适应对逆向有兴趣爱好的朋友，而且是对逆向有浓厚兴趣的初学者，如果，读贴者高人也，可以 `JMP` 或 `RETURN`，如果读贴者是学习者，请 `CALL`。当觉得此贴对你没有任何价值时，或觉得是忽悠的话，请忽略 `RETURN`，我只写给需要的朋友，请口留吉言，在此，谢过~~！我不想因为我的贴给 `PEDIY` 带来没必要的麻烦，此贴，也没太多技术含量只是一些方法和思路，并不探讨高深技术，所以初学者只要懂点汇编，应该都很快就能明白，如那些地方说的不明白，只管跟贴询问，我看到自然就回答了，^\_^。

正题开始…………

逆向有个误区，需要说明，如果是需要了解流程，完全没必要这样，只需要看懂代码整体实现，类自行写出类似代码，这个不算是逆向，逆向就是要写出几乎编译结果一样的代码。要做到这个是十分不容易的，编译有着太多的不确定，所以要写出编译结果完全一样的代码，以前的我也觉得不可能，可现在似乎我不这么认为，在我的手里已经有编译结果完全一样的代码，编译结果完全一样代表是逆出原来的代码了吗？答案是否，只能说是接近了原来的代码，但是无法完全复原出原来的代码，这又是个误区。逆向比较忌讳这样的做法：

//列如这样，比较复杂点算术函数，由于符点，不太好懂，索性直接嵌入汇编了事，运行结果完全没问题似乎编译结果也是完全一样的，但是，它根本不算什么逆向，只能说是个残废。

```c
void     AngleMatrix(float* unk1, float* unk2)
{
    float   dunk1;

    dunk1 = unk1[2] * _real_004A6CBC;
    float   dunk2=(float)sin((double)dunk1);
    float   dunk3=(float)cos(dunk1);

    dunk1 = unk1[1] * _real_004A6CBC;
    float   dunk4=(float)sin((double)dunk1);
    float   dunk5=(float)cos(dunk1);

    dunk1 = unk1[0] * _real_004A6CBC;
    float   dunk6=(float)sin((double)dunk1);
    float   dunk7=(float)cos(dunk1);

    *unk2=dunk5*dunk3;
    *(unk2+0x04)=dunk5*dunk2;

    _asm{
        fld   dunk4
        fchs
        mov   edx,unk2
        fstp  dword ptr[edx+0x20]
    }

    _asm{
        fld   dunk6
        fmul  dunk4
        fmul  dunk3
        fld   dunk2
        fchs
        fmul  dunk7
        faddp st(1),st
        mov   eax,unk2
        fstp  dword ptr[eax+0x04]
        fld   dunk6
        fmul  dunk4
        fmul  dunk2
        fld   dunk7
        fmul  dunk3
        faddp st(1),st
        mov   ecx,unk2
        fstp  dword ptr[ecx+0x14]
        fld   dunk6
        fmul  dunk5
        mov   edx,unk2
        fstp  dword ptr[edx+0x24]
        fld   dunk7
        fmul  dunk4
        fmul  dunk3
        fld   dunk6
        fchs  
        fld   dunk2
        fchs
        fmulp st(1),st
        faddp st(1),st
        mov   eax,unk2
        fstp  dword ptr[eax+0x08]
        fld   dunk7
        fmul  dunk4
        fmul  dunk2
        fld   dunk6
        fchs
        fmul  dunk3
        faddp st(1),st
        mov   ecx,unk2
        fstp  dword ptr[ecx+0x18]
    }
    *(unk2+0x0A)=dunk7*dunk5;
    unk2[0x03]=0;
    unk2[0x07]=0;
    unk2[0x0B]=0;
}

//我们为什么不吧它写成这样的呢？这段代码和原来的代码编译出来的结果是完全一样的，但是为什么，我们不去花点精神，去把它写好呢，虽然算术符点有点麻烦，但是整体来看，似乎它还是那么的容易，这样的代码是接近原代码了，但是它并不是原来的代码，原来的代码至少有 X 和 Y 之类的变量名，而我却用 A-Z 类代之。但是这样的代码是不是变的十分好懂了许多，我们可以发现，原代码的书写者很有可能是 C 程序原，在我逆的时候发现了，变量定义与 C++的不同。我们可以根据逆出来的代码分析出写代码的人的水平等等~~！

void    AngleMatrix( float * fBuffer1, float fBuffer2[] )
{
    //   
    float  fA, fB, fC, fD, fE, fF, fG;    //04,08,0C,10,14,18,1C
    //
    fA = fBuffer1[2] * 0.01745329f;
    //
    fD = sin( fA );
    fG = cos( fA );
    //
    fA = fBuffer1[1] * 0.01745329f;
    //
    fC = sin( fA );
    fF = cos( fA );
    //
    fA = fBuffer1[0] * 0.01745329f;
    //
    fB = sin( fA );
    fE = cos( fA );
    //
    fBuffer2[0] = fF * fG;
    fBuffer2[4] = fF * fD;
    //
    fBuffer2[8] = -fC;

    fBuffer2[1] = fB * fC * fG + ( -fD * fE );
    fBuffer2[5] = fB * fC * fD + ( fE * fG );

    fBuffer2[9] = fB * fF;

    //
    fBuffer2[2] = fE * fC * fG + ( -fB * -fD );

    //
    fBuffer2[6] = fE * fC * fD + ( -fB * fG );

    //
    fBuffer2[10] = fE * fF;

    //
    fBuffer2[3] = 0;
    fBuffer2[7] = 0;
    fBuffer2[11] = 0;
}
```

并没希望读贴者能够看明白这个函数，是做什么用的，只是想让大家明白一个忌讳，就是需要坚持耐心的做完，这样你就会觉得自己确实再进步。`_asm` 的功用让我们可以偷懒，但是懒惰让我们很难提高，甚至退步。虽然我是异常的懒。^\_^

忌讳第二就是半途而废，做一半太难，就丢下了，需要的是挑战自己，提高自己，不说了，大家自己体会吧。

我们改如何练手逆向呢？建议大家先从简单到难，也就是从 `VC` 程序开始，最好是纯 `API` 的程序，避免 `MFC` 库带来的麻烦，尽量不涉及模板之类的，没有一口气就可以学很多的，需要慢慢来，忌讳三，就是心急。我们可以找个纯 `API` 的程序，最好带 `PDB`，而且是 `DEBUG` 的，建议最后再研究 `release`。但是这样的代码我们几乎无法找到，所以可以自己写点，来研究，刚开始不要涉及太多的流程，这样会很打击信心，研究的目标是，不段的尝试写出编译结果完全一样的代码，当编译结果完全一样后，再对照原代码，或许发现写出来的代码和原来的代码是那么的接近，几乎差那么几步就能完全一样了。

我们遇到一个程序，需要逆向它首先要判断，它的原代码是用什么语言写的，这个只要仔细的区分就可以发现，毕竟 `vc`, `bcb`, `delphi`, `汇编`，写的程序都是那么的不同，我们可以很容易断定出，是什么语言，接下来我们需要的是把编译器设置的和写代码的人完全一样，这个是比较难的，需要尝试。

//例如：一个程序的入口是这样的，那么，我们可以肯定这个是 `WIN32` 程序，我们需要的就是这样的程序，来进行研究，看到这样的入口，第一感觉必须是 `VC6` 版本，至少 `VC6` 可以编译出同样的效果，`VC` 每个版本的入口都略有不同，所以大家仔细对比下会发现其中的不同，我就不一一列举。

代码:
