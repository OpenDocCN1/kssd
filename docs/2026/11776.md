
# [原创]查询系统是否静音和设置静音

有时忘记将系统静音，关机时声音很大，想办法咯！结果，网上找了半天也没找到一个怎么查询和设置系统静音的函数！![](https://bbs.kanxue.com/view/img/face/1.gif)

都是你抄我的我抄你的，没有几个解决实际问题的，只是想查询一下系统是否静音和设置设置与否，结果都是讲一大通废话，哈哈，好在自己能去其糟粕取其精华，去除一大段废话与无用的代码加上自己理解得到两精简版函数，在看雪上存盘了：

```cpp
#include <mmsystem.h> //我一直记成是妹妹系统，哈哈，妹妹的系统都是与多媒体有关的

#pragma comment(lib, "winmm.lib")  //windows里有妹妹哦！

BOOL IsSystemMute(LONG &lvalue){ //lvalue为0时系统非静音，否则是静音,函数返回成功与否

        HMIXER hMixer;

        MIXERLINE mixerline;

        MIXERLINECONTROLS mlcs;

        MIXERCONTROL mc;

        MIXERCONTROLDETAILS_BOOLEAN mcdMute;

        MIXERCONTROLDETAILS mcd;

        try{

                if (MMSYSERR_NOERROR == mixerOpen(&hMixer, 0, 0, 0, 0)){

                        RtlZeroMemory(&mixerline, sizeof(MIXERLINE));

                        mixerline.cbStruct = sizeof(MIXERLINE);

                        mixerline.dwComponentType = MIXERLINE_COMPONENTTYPE_DST_SPEAKERS;

                        if (MMSYSERR_NOERROR == mixerGetLineInfo((HMIXEROBJ)hMixer, &mixerline, MIXER_GETLINEINFOF_COMPONENTTYPE)){

                                RtlZeroMemory(&mlcs, sizeof(MIXERLINECONTROLS));

                                mlcs.cbStruct = sizeof(MIXERLINECONTROLS);

                                mlcs.dwLineID = mixerline.dwLineID;

                                mlcs.dwControlType =  MIXERCONTROL_CONTROLTYPE_MUTE;

                                mlcs.cControls = 1;

                                mlcs.cbmxctrl = sizeof(MIXERCONTROL);

                                mlcs.pamxctrl = & mc;

                                if (MMSYSERR_NOERROR == mixerGetLineControls((HMIXEROBJ)hMixer, &mlcs, MIXER_GETLINECONTROLSF_ONEBYTYPE)){

                                        RtlZeroMemory(&mcd, sizeof(MIXERCONTROLDETAILS));

                                        mcd.cbStruct = sizeof(MIXERCONTROLDETAILS);

                                        mcd.dwControlID = mc.dwControlID;

                                        mcd.cChannels = 1;

                                        mcd.cbDetails = sizeof(MIXERCONTROLDETAILS_BOOLEAN);

                                        mcd.paDetails = &mcdMute;

                                        if (MMSYSERR_NOERROR == mixerGetControlDetails((HMIXEROBJ)hMixer, &mcd, MIXER_GETCONTROLDETAILSF_VALUE)){

                                                lvalue = mcdMute.fValue;

                                                return TRUE;

                                        }

                                }

                        }

                }

                mixerClose(hMixer);

                return FALSE;

        }catch(...){

                mixerClose(hMixer);

                return FALSE;

        }

}

BOOL SetSystemMute(LONG lMute){ //0为非静音,1为静音

        HMIXER hMixer;

        MIXERLINE mixerline;

        MIXERLINECONTROLS mlcs;

        MIXERCONTROL mc;

        MIXERCONTROLDETAILS_BOOLEAN mcdMute;

        MIXERCONTROLDETAILS mcd;

        try{

                if (MMSYSERR_NOERROR == mixerOpen(&hMixer, 0, 0, 0, 0)){

                        RtlZeroMemory(&mixerline, sizeof(MIXERLINE));

                        mixerline.cbStruct = sizeof(MIXERLINE);

                        mixerline.dwComponentType = MIXERLINE_COMPONENTTYPE_DST_SPEAKERS;

                        if (MMSYSERR_NOERROR == mixerGetLineInfo((HMIXEROBJ)hMixer, &mixerline, MIXER_GETLINEINFOF_COMPONENTTYPE)){

                                RtlZeroMemory(&mlcs, sizeof(MIXERLINECONTROLS));

                                mlcs.cbStruct = sizeof(MIXERLINECONTROLS);

                                mlcs.dwLineID = mixerline.dwLineID;

                                mlcs.dwControlType =  MIXERCONTROL_CONTROLTYPE_MUTE;

                                mlcs.cControls = 1;

                                mlcs.cbmxctrl = sizeof(MIXERCONTROL);

                                mlcs.pamxctrl = & mc;

                                if (MMSYSERR_NOERROR == mixerGetLineControls((HMIXEROBJ)hMixer, &mlcs, MIXER_GETLINECONTROLSF_ONEBYTYPE)){

                                        RtlZeroMemory(&mcd, sizeof(MIXERCONTROLDETAILS));

                                        mcdMute.fValue = lMute; //静音

                                        mcd.cbStruct = sizeof(MIXERCONTROLDETAILS);

                                        mcd.dwControlID = mc.dwControlID;

                                        mcd.cChannels = 1;

                                        mcd.cbDetails = sizeof(MIXERCONTROLDETAILS_BOOLEAN);

                                        mcd.paDetails = &mcdMute;

                                        if (MMSYSERR_NOERROR == mixerSetControlDetails((HMIXEROBJ)hMixer, &mcd, MIXER_SETCONTROLDETAILSF_VALUE)){

                                                return TRUE;

                                        }

                                }

                        }

                }

                mixerClose(hMixer);

                return FALSE;

        }catch(...){

                mixerClose(hMixer);

                return FALSE;

        }

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
