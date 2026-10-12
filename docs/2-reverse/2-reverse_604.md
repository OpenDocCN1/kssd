
# 汉王手写输入的系统部分

> 原文：[`www.pediy.com/kssd/pediy05/pediy50659.htm`](https://www.pediy.com/kssd/pediy05/pediy50659.htm)

工具: `IDA Pro 4.x`

平台: `Palm OS 3.5`, `Motorola 68000` 指令集

下载: `http://www.hwpen.net/download.htm`

年初, 下了一个汉王试了一下, 觉得它的浮动菜单做的比较好, 因为它在任何 `App` 下都可以冒出来. 要知道 `Palm` 是单任务的, 在一个时候只能有一个 `APP` 在运行. 当时刚刚接触 `Palm`, 如果要让我实现这些功能, 还真的不知道怎么下手. 于是想到了用 `IDA` 试试在非 `Windows` 平台的反汇编. 最后得到的这些代码完全说明了它是怎样实现这些功能的. 对于 `Palm` 程序员是有参考价值的.

另外, 有几点需要特别说明:

1. 对于它的识别引擎, 我没有做逆向, 因为我对它后面的高深理论不了解也没有兴趣.

2. 因为是为了学习, 而不是破解这套软件, 我没有去逆向它的注册码算法部分. 从支持国产软件方面考虑, 我也不会这样做. (言下之意: 汉王你可别找我麻烦 :) )

3. 这里公布的只是任何 `Palm` 书籍上都有的编程的最基础部分, 应该不会对汉王在技术上构成什么危害.

4. 这些可以编译的代码只用于学习目的, 请勿做它用.

5. 用 `SetTrap` 的方法挂接 `HW_SysHandleEvent()` 而没有其它措施是不安全的, 因为系统在特定的时候会移动代码资源, 这肯定会让系统垮掉, 应该使用 `DmDatabaseProtect()` 对这段代码资源加以保护. 希望汉王以后会这样做.

```c
// #include <windows.h>

#include <PalmOS.h>
#include <DLServer.h>
#include <PalmUtils.h>

#define version350 0x3503000

#define HW_MainForm 1000
#define RomIncompatibleAlert 1001
#define HWDefinedCmd 0xddbb
#define HW_COMMONCCHAR 0x0001
#define HW_SUBCOMMONCCHAR 0x0002
#define HW_TRA2SIMPLIFIED 0x0008
#define HW_NUMERIC 0x0010
#define HW_ENGLISH 0x0060
#define HW_SYMBOL 0x0180
#define HW_UPPER2LOWER 0x2000
#define HW_LOWER2UPPER 0x4000
#define HW_HALF2FULL 0x8000

// 菜单模式
#define FOLDED 0 // 收成"V"
#define LOWER 1 // 在屏幕的下半部
#define UPPER 2 // 在屏幕的上半部
#define HIDDEN 3 // 隐藏

// 手写系统阶段
#define READY 0 // 就绪
#define ENTER 1 // 进入窗口或 Form
#define LEAVE 2 // 离开窗口或 Form
#define CHGFLD 3 // 切换到其它字段

typedef struct tagHW_PREF {
    Boolean bInstallHW;     // 是否安装汉王手写输入系统. 0
    Int8 al_b1;            // 未被使用 1
    UInt16 flags;          // 标志位 2
    UInt16 RecogRange;     // 识别范围 4
    Int8 RecogWaitTime;    // 识别等待时间 6
    Int8 PenThickness;     // 笔迹粗细 7
    Int8 PenColor;        // 笔的颜色 8
    Int8 al_b9;           // 未被使用 9
    Int16 ShotcutPenDownX; // 快捷方式的起点屏幕坐标 X a
    Int16 ShotcutPenDownY; // 快捷方式的起点屏幕坐标 Y c
    Int16 ShotcutPenUpX;   // 快捷方式的终点屏幕坐标 X e
    Int16 ShotcutPenUpY;   // 快捷方式的终点屏幕坐标 Y 10
    Boolean bSetting;      // 是否正在设置快捷方式 12
    Int8 bUsingShortcut;   // 是否正在使用快捷方式 13
    Boolean bSplitScreen;  // 全屏分割 14
    Boolean bHalfToFull;   // 半角字符转成全角字符 15
    Boolean bUpperToLower; // 大写字母转成小写字母 16
    Boolean bLowerToUpper; // 小写字母转成大写字母 17
    Boolean bMouse;        // 是否作为鼠标使用 18
    Boolean bReged;        // 是否是注册用户: 0xff 是, 0 否. 19
    Int8 MenuMode;         // 0: FOLDED, 1: LOWER, 2: UPPER, 3: HIDDEN 1a
    Boolean bMenuFolded;   // 菜单是否被收成"V" 1b
    Boolean bOpenHW;       // 是否启动汉王手写输入 1c
    Boolean bHalfScrnInput; // 是否在半屏手写模式 1d
    Boolean bRedrawMenu;   // 菜单是否需要重画 1e
    Int8 Phase;            // 手写系统阶段 0: READY, 1: ENTER, 2: LEAVE, 3: CHGFLD 1f
} HW_PREF;

typedef struct tagHW_PENS {
    UInt16 Cands[20][10];  // 候选字 Buffer 0
    UInt8 index;           // 候选字在候选字 Buffer 中的下标索引 0x190/400
    UInt8 b191;           // not used
    UInt16 InsPtPos;       // 插入点偏移, 192h, 402
    UInt16 CandiDispOfs;   // 候选字显示偏移 194h 404
    char* AssocBuffer;     // 联想字 buf 196h 406
    UInt16 AssocNum;       // 联想字个数 19a 410
    UInt8 AssocOfs;       // 联想字偏移 19c 412
    UInt8 AssocDispOfs;    // 联想字显示偏移 19d 413
} HW_PENS;

typedef struct tagHW_TRACE {
    PointType points[2048]; // 笔迹采样点数组 0
    UInt16 PtCount;         // 笔迹采样存放位置 2000
    UInt32 dw2002;         // 2002
    UInt8 b2006;          // 2006
    UInt8 b2007;          // 2007
    UInt8 b2008;          // 2008
} HW_TRACE;

int HW_Install(void);
void HW_Uninstall(void);

UInt16 GetRecogRange(HW_PREF* pPref)
{
    UInt16 flags;
    if (pPref->RecogRange)
        flags = pPref->RecogRange;
    else
        flags = pPref->flags;

    if (pPref->bHalfToFull)
        flags |= 0x8000;
    if (pPref->bUpperToLower)
        flags |= 0x2000;
    if (pPref->bLowerToUpper)
        flags |= 0x4000;

    return flags;
}

Int8 GetMenuMode(void)
{
    Int16 x, y;
    InsPtGetLocation(&x, &y);
    if (y > 109)
        return UPPER;
    else
        return LOWER;
}

char V[] = "V";

void ClearMenu(Int8 MenuMode)
{
    RectangleType rc;
    if (MenuMode == HIDDEN) return;
    if (MenuMode == FOLDED) {
        WinEraseChars(V, 1, 141, 141);
        return;
    }
    if (MenuMode == LOWER)
        RctSetRectangle(&rc, 0, 118, 160, 30);
    else if (MenuMode == UPPER)
        RctSetRectangle(&rc, 0, 78, 160, 30);
    WinEraseRectangle(&rc, 0 /*cornerDiam, 0 for square corners*/);
}

void RedrawCurForm(void)
{
    FormPtr frm = FrmGetActiveForm();
    WindowType* winHandle = FrmGetWindowHandle(frm);
    if (winGetActiveWindow() == winHandle && FrmValidatePtr(frm))
        FrmDrawForm(frm);
}

void DrawMenu(HW_PREF* pPrefs, HW_PENS* pHWPens) // a3 = pPrefs, a2 = pHWPens
{
    int ofs, x, d5;
    RectangleType rc;
    UInt16* ptr;
    if (pHWPens->index == 0)
        x = 0;
    else
        x = pHWPens->index - 1;

    if (pPrefs->MenuMode == UPPER) {
        RctSetRectangle(&rc, 2, 80, 155, 25);
        WinDrawRectangleFrame(dialogFrame, &rc);
        // This routine does not draw in the gray color; it draws with
        // alternating foreground and background pixels. That is, it uses
        // the grayPattern pattern type.
        WinDrawGrayLine(74, 93, 74, 104);
        WinDrawGragLine(2, 92, 156, 92);
        WinDrawChars("→", 2, 63, 94);
        WinDrawChars("英", 2, 76, 94);
        if (pPrefs->RecogRange == HW_ENGLISH) {
            RctSetRectangle(&rc, 75, 93, 12, 12);
            WinInvertRectangle(&rc, 0);
        }
        WinDrawChars("数", 2, 88, 94);
        if (pPrefs->RecogRange == HW_NUMERIC) {
            RctSetRectangle(&rc, 87, 93, 12, 12);
            WinInvertRectangle(&rc, 0);
        }
        WinDrawChars("符", 2, 100, 94);
        if (pPrefs->RecogRange == HW_SYMBOL) {
            RctSetRectangle(&rc, 99, 93, 12, 12);
            WinInvertRectangle(&rc, 0);
        }
        WinDrawChars("全", 2, 112, 94);
        if (pPrefs->bHalfToFull) {
            RctSetRectangle(&rc, 111, 93, 12, 12);
            WinInvertRectangle(&rc, 0);
        }
        WinDrawChars("窗", 2, 124, 94);
        WinDrawChars("鼠", 2, 136, 94);
        if (pPrefs->bMouse) {
            RctSetRectangle(&rc, 135, 93, 12, 12);
            WinInvertRectangle(&rc, 0);
        }
        WinDrawChars("^", 1, 148, 94);
        WinDrawChars("←", 2, 132, 81);
        WinDrawChars("→", 2, 145, 81);
        d5 = pHWPens->CandiDispOfs * 2;
        ptr = pHWPens->Cands[x];
        // 显示 5 个候选字
        if (ptr[d5] || ptr[d5 + 1]) {
            for (ofs = d5; ofs < d5 + 10; ofs += 2) {
                if (ptr[ofs]) {
                    WinDrawChars(&ptr[ofs], 2, x, 94);
                    x += 12;
                }
                else {
                    if (ptr[d5 + 1] <= ' ')
                        break;
                    WinDrawChars(&ptr[d5 + 1], 1, x, 94);
                    x += 12;
                }
            }
        }
        // 显示 10 个联想字
```



```
if (pHWPens->AssocNum) {   
    x = 3;   
    for (ofs = pHWPens->AssocDispOfs; ofs < pHWPens->AssocOfs + 20; ofs += 2) {   
        if (ofs - pHWPens->AssocOfs < pHWPens->AssocNum)   
            break;   
        WinDrawChars(&pHWPens->AssocBuffer[ofs], 2, x, 81);   
        x += 13;   
    }   
}   
else if (pPrefs->MenuMode == LOWER) {   
    RctSetRectangle(&rc, 2, 120, 155, 25);   
    WinDrawRectangleFrame(dialogFrame, &rc);   
    // This routine does not draw in the gray color; it draws with   
    // alternating foreground and background pixels. That is, it uses   
    // the grayPattern pattern type.   
    WinDrawGrayLine(74, 133, 74, 144);   
    WinDrawGragLine(2, 132, 156, 312);   
    WinDrawChars("→", 2, 63, 134);   
    WinDrawChars("英", 2, 76, 134);   
    if (pPrefs->RecogRange == HW_ENGLISH) {   
        RctSetRectangle(&rc, 75, 93, 12, 12);   
        WinInvertRectangle(&rc, 0);   
    }   
    WinDrawChars("数", 2, 88, 134);   
    if (pPrefs->RecogRange == HW_NUMERIC) {   
        RctSetRectangle(&rc, 87, 93, 12, 12);   
        WinInvertRectangle(&rc, 0);   
    }   
    WinDrawChars("符", 2, 100, 134);   
    if (pPrefs->RecogRange == HW_SYMBOL) {   
        RctSetRectangle(&rc, 99, 133, 12, 12);   
        WinInvertRectangle(&rc, 0);   
    }   
    WinDrawChars("全", 2, 112, 134);   
    if (pPrefs->bHalfToFull) {   
        RctSetRectangle(&rc, 111, 133, 12, 12);   
        WinInvertRectangle(&rc, 0);   
    }   
    WinDrawChars("窗", 2, 124, 134);   
    WinDrawChars("鼠", 2, 136, 134);   
    if (pPrefs->bMouse) {   
        RctSetRectangle(&rc, 135, 133, 12, 12);   
        WinInvertRectangle(&rc, 0);   
    }   
    WinDrawChars("^", 1, 148, 134);   
    WinDrawChars("←", 2, 132, 121);   
    WinDrawChars("→", 2, 145, 121);   
    d5 = pHWPens->CandiDispOfs * 2;   
    ptr = pHWPens->Cands[x];   
    // 显示 5 个候选字   
    if (ptr[d5] || ptr[d5 + 1]) {   
        for (ofs = d5; ofs < d5 + 10; ofs += 2) {   
            if (ptr[ofs]) {   
                WinDrawChars(&ptr[ofs], 2, x, 134);   
                x += 12;   
            }   
            else {   
                if (ptr[d5 + 1] <= ' ')   
                    break;   
                WinDrawChars(&ptr[d5 + 1], 1, x, 134);   
                x += 12;   
            }   
        }   
    }   
    // 显示 10 个联想字   
    if (pHWPens->AssocNum) {   
        x = 3;   
        for (ofs = pHWPens->AssocDispOfs; ofs < pHWPens->AssocOfs + 20; ofs += 2) {   
            if (ofs - pHWPens->AssocOfs > pHWPens->AssocNum)   
                break;   
            WinDrawChars(&pHWPens->AssocBuffer[ofs], 2, x, 121);   
            x += 13;   
        }   
    }   
}   
else if (pPrefs->MenuMode == FOLDED)   
    WinDrawChars("V", 1, 141, 141);   
}   

void RedrawMenu(HW_PREF* pPrefs, char* pHWPens)   
{   
    WinHandle winHandle = WinSetDrawWindow(WinGetDisplayWindow());   
    ClearMenu(pPrefs->MenuMode);   
    DrawMenu(pPrefs, pHWPens);   
    WinSetDrawWindow(winHandle);   
}   

FieldPtr GetCurField(FormPtr frm)   
{   
    // Too bad to access directly FormType data member. Palm strongly disencourage   
    // such behaviors   
    if (frm->focus != noFocus &&   
        frm->objects[frm->focus].object.field != frmFieldObj&&   
        frm->objects[frm->focus].object.field != frmTableObj)   
    {   
        return TblGetCurrentField(frm->objects[frm->focus].object.field);   
    }   
    return NULL;   
}   

void EnqueueChChar(FieldPtr fp, UInt16* pCand, Boolean bBackspace)   
{   
    MemHandle     memHandle;   
    HW_PREF*     pPrefs;   
    UInt16        InsPos;   
    FtrGet('HW99', 10, (UInt32 *)&memHandle);   
    pPrefs = (HW_PREF*)MemHandleLock(memHandle); // a2   
    InsPos = FldGetInsPtPosition(fp);     // d3   
    if (bBackspace)   
        FldDelete(fp, InsPos - 2, InsPos);   
    FldInsert(fp, pCand, 2);   
    if (pPrefs->bReged != 0xff) {   
        pPrefs->bReged++;   
        pPrefs->bReged %= 10;   
        if (pPrefs->bReged == 0)   
            ShowNag();   
    }   
    MemHandleUnlock(memHandle);   
    InsPos = FldGetInsPtPosition(fp);   
    FldSetSelection(fp, InsPos, InsPos);   
}   

void GetAssoc(HW_PENS* pHWPens, UInt16 Cand)   
{   
    pHWPens->AssocNum = 0;   
    pHWPens->AssocOfs = 0;   
    if (GetAssocFromAssocBuf(pHWPens->AssocBuffer, Cand) > 0) {   
        pHWPens->AssocNum = *pHWPens->AssocBuffer;   
        pHWPens->AssocBuffer++;   
        if (pHWPens->AssocNum == -1) {   
            pHWPens->AssocNum += *pHWPens->AssocBuffer;   
            pHWPens->AssocBuffer++;   
        }   
        pHWPens->AssocNum *= 2;   
        pHWPens->AssocOfs = 0;   
    }   
}   

void EnqueueInputKeys(FormType* frm, HW_PENS* pHWPens, UInt16 index, BOOL bAssoc)   
{   
    FieldPtr fp = GetCurField(frm); // a3   
    WChar ascii;   
    UInt16 InsPos;   
    if (fp != NULL) {   
        if ((UInt8)pHWPens->Cands[index][0] == 0) {   
            ascii = *((UInt8*)&pHWPens->Cands[index][0] + 1);   
            if (ascii > ' ') {   
                EvtEnqueueKey(ascii, 0, 0);   
                if (++pHWPens->index == 20)   
                    pHWPens->index = 0;   
                pHWPens->CandiDispOfs = 0;   
                pHWPens->InsPtPos = FldGetInsPtPosition(fp) + 1;   
            }   
            else if (ascii == ' ')   
                EvtEnqueueKey(32, 0, 0);   
            else if (ascii == 8)   
                EvtEnqueueKey(backspaceChr, 0, 0);   
            else if (ascii == '\n')   
                EvtEnqueueKey('\r', 0, 0), EvtEnqueueKey('\n', 0, 0);   
        }   
        else {   
            EnqueueChChar(fp, pHWPens->Cands[index], false);   
            if (++pHWPens->index == 20)   
                pHWPens->index = 0;   
            pHWPens->CandiDispOfs = 0;   
            pHWPens->InsPtPos = FldGetInsPtPosition(fp);   
            if (bAssoc)   
                GetAssoc(pHWPens, pHWPens->Cands[index][0]);   
        }   
    }   
}   

void AdjustCandIndex(FormPtr frm, HW_PENS* pHWPens)   
{   
    UInt16 InsPos = FldGetInsPtPosition(GetCurField(frm)); // d3   
    Int16 InsOfs = InsPos - pHWPens->InsPtPos;   
    Int16 index;   
    pHWPens->InsPtPos = InsPos;   
    if (InsOfs > 40 || InsOfs < -40)   
        return;   
    if (pHWPens->index == 0)   
        index = 19;   
    else   
        index = pHWPens->index - 1; // d3 = index   
    if (InsOfs < 0) {   
        do {   
            if (pHWPens->Cands[index][0] == 0) {   
                if (pHWPens->Cands[index][1] == 0) {   
                    if (index == 19)   
                        pHWPens->index = 0;   
                    else   
                        pHWPens->index = index + 1;   
                    return;   
                }   
                else {   
                    InsOfs++;   
                    if (index == 0)   
                        index = 19;   
                    else   
                        index--;   
                }   
            }   
            else {   
                InsOfs += 2;   
                if (index == 0)   
                    index = 19;   
                else   
                    index--;   
            }   
        } while (InsOfs < 0); // 5F4C here   
    } // goto 5fa2   
    else if (InsOfs > 0) {   
        do {   
            if (pHWPens->Cands[index][0] == 0) {   
```



```
if (pHWPens->Cands[index][1] == 0) {
    if (index == 19)
        pHWPens->index = 0;
    else
        pHWPens->index = index + 1;
    return;
}
else {
    InsOfs--;
    if (index == 19)
        index = 0;
    else
        index++;
}
} else {
    InsOfs  -= 2;
    if (index == 19)
        index = 0;
    else
        index++;
}
} while (InsOfs > 0);
 }
 if (index == 19)
     pHWPens->index = 0;
 else
     pHWPens->index = index + 1;
}

BOOL OnPenDown(EventPtr eventP, HW_PREF* pPrefs, HW_PENS* pHWPens, HW_TRACE* pHWTrace)

{
    Coord x, y;
    FormPtr frm;
    Int8 MenuMode;
    UInt16 screenX, screenY;

    // d3 = eventP(a6)
    // a2 = pPrefs
    // a3 = pHWPens
    // a4 = pHWTrace

    x = eventP->screenX;
    y = eventP->screenY;
    WinWindowToDisplayPt(&x, &y);
    if (x > 160 || y > 160)
        return 0;
    MenuMode = pPrefs->MenuMode;
    switch (MenuMode)
    {
    case FOLDED:
        if (screenX > 141 && screenX < 151 && screenY > 141 && screenY < 151)
            break;
    case LOWER:
        if (screenY > 120 && screenY < 145)
            break;
    case UPPER:
        if (screenY > 80 && screenY < 105)
            break;
    default:
        if (pPrefs->bMouse)
            return false;
        pHWTrace->b2006 = 0;
        pHWTrace->points[pHWTrace->PtCount].x = screenX;
        pHWTrace->points[pHWTrace->PtCount].y = screenY;
        if (pHWTrace->PtCount == 0)
            pHWTrace->b2007 = 1;
        pHWTrace->PtCount++;
        pHWTrace->dw2002 = 0;
        return true;
    }

    // 如果已有笔迹采样点, 将新点加入笔迹数组.
    if (pHWTrace->PtCount) {
        pHWTrace->points[pHWTrace->PtCount].x = screenX;
        pHWTrace->points[pHWTrace->PtCount].y = screenY;
        pHWTrace->PtCount++;
        return true;
    }

    pHWTrace->b2006 = true;
    if (MenuMode == LOWER && x > 3 && x < 63 && y > 134 && y < 146 ||
        MenuMode == UPPER && x > 3 && x < 63 && y > 94 && y < 106)
    {
        UInt16 ofs = (pHWPens->CandiDispOfs + (x - 1)) / 12;
        char* p = (char*)&pHWPens->Cands[pHWPens->index - 1][ofs];
        if (*p == 0) {
            if (*(p + 1) == 0)
                return true;
            EvtEnqueueKey(backspaceChr, 0, 0);
            EvtEnqueueKey(*(p + 1), 0, 0);
        }
        else {
            EnqueueChChar(GetCurField(FrmGetActiveForm()), p, true);
            GetAssoc(pHWPens, *(UInt16*)p);
        }
        pPrefs->bRedrawMenu = true;
        return true;
    }
    if (MenuMode == LOWER && x > 3 && x < 133 && y > 121 && y < 133 ||
        MenuMode == UPPER && x > 3 && x < 133 && y > 81 && y < 93) // else goto 6b4a
    {
        UInt16 pos = (x - 1) / 13 * 2;
        if (pHWPens->AssocOfs + pos < pHWPens->AssocNum) {
            EnqueueChChar(GetCurField(FrmGetActiveForm()), pHWPens->AssocBuffer[pos], false);
            pHWPens->AssocNum = 0;
            pHWPens->AssocOfs = 0;
        }
        return true;
    }
    if (MenuMode == LOWER && x > 63 && x < 75 && y > 134 && y < 146 ||
        MenuMode == UPPER && x > 63 && x < 75 && y > 94 && y < 106) // else goto 6be0
    {
        UInt16 ofs = pHWPens->CandiDispOfs + 5;
        if (ofs == 10) {
            pHWPens->CandiDispOfs = 0;
            pPrefs->bRedrawMenu = true;
        }
        else if (pHWPens->Cands[pHWPens->index - 1][ofs] != 0) {
            pHWPens->CandiDispOfs + 5;
            pPrefs->bRedrawMenu = true;
        }
        return true;
    }
    if (MenuMode == LOWER && x > 132 && x < 144 && y > 121 && y < 133 ||
        MenuMode == UPPER && x > 132 && x < 144 && y > 81 && y < 93) // else goto 6c4a
    {
        if (pHWPens->AssocOfs) {
            pHWPens->AssocOfs -= 20;
            pPrefs->bRedrawMenu = true;
        }
        return true;
    }
    if (MenuMode == LOWER && x > 145 && x < 157 && y > 121 && y < 133 ||
        MenuMode == UPPER && x > 145 && x < 157 && y > 81 && y < 93) // else goto 6cba
    {
        if (pHWPens->AssocOfs + 20 < pHWPens->AssocNum) {
            pHWPens->AssocOfs += 20;
            pPrefs->bRedrawMenu = true;
        }
        re

BOOL OnPenDown(EventPtr eventP, HW_PREF* pPrefs, HW_PENS* pHWPens, HW_TRACE* pHWTrace)

{
    Coord x, y;
    FormPtr frm;
    Int8 MenuMode;
    UInt16 screenX, screenY;

    // d3 = eventP(a6)
    // a2 = pPrefs
    // a3 = pHWPens
    // a4 = pHWTrace

    x = eventP->screenX;
    y = eventP->screenY;
    WinWindowToDisplayPt(&x, &y);
    if (x > 160 || y > 160)
        return 0;
    MenuMode = pPrefs->MenuMode;
    switch (MenuMode)
    {
    case FOLDED:
        if (screenX > 141 && screenX < 151 && screenY > 141 && screenY < 151)
            break;
    case LOWER:
        if (screenY > 120 && screenY < 145)
            break;
    case UPPER:
        if (screenY > 80 && screenY < 105)
            break;
    default:
        if (pPrefs->bMouse)
            return false;
        pHWTrace->b2006 = 0;
        pHWTrace->points[pHWTrace->PtCount].x = screenX;
        pHWTrace->points[pHWTrace->PtCount].y = screenY;
        if (pHWTrace->PtCount == 0)
            pHWTrace->b2007 = 1;
        pHWTrace->PtCount++;
        pHWTrace->dw2002 = 0;
        return true;
    }

    // 如果已有笔迹采样点, 将新点加入笔迹数组.
    if (pHWTrace->PtCount) {
        pHWTrace->points[pHWTrace->PtCount].x = screenX;
        pHWTrace->points[pHWTrace->PtCount].y = screenY;
        pHWTrace->PtCount++;
        return true;
    }

    pHWTrace->b2006 = true;
    if (MenuMode == LOWER && x > 3 && x < 63 && y > 134 && y < 146 ||
        MenuMode == UPPER && x > 3 && x < 63 && y > 94 && y < 106)
    {
        UInt16 ofs = (pHWPens->CandiDispOfs + (x - 1)) / 12;
        char* p = (char*)&pHWPens->Cands[pHWPens->index - 1][ofs];
        if (*p == 0) {
            if (*(p + 1) == 0)
                return true;
            EvtEnqueueKey(backspaceChr, 0, 0);
            EvtEnqueueKey(*(p + 1), 0, 0);
        }
        else {
            EnqueueChChar(GetCurField(FrmGetActiveForm()), p, true);
            GetAssoc(pHWPens, *(UInt16*)p);
        }
        pPrefs->bRedrawMenu = true;
        return true;
    }
    if (MenuMode == LOWER && x > 3 && x < 133 && y > 121 && y < 133 ||
        MenuMode == UPPER && x > 3 && x < 133 && y > 81 && y < 93) // else goto 6b4a
    {
        UInt16 pos = (x - 1) / 13 * 2;
        if (pHWPens->AssocOfs + pos < pHWPens->AssocNum) {
            EnqueueChChar(GetCurField(FrmGetActiveForm()), pHWPens->AssocBuffer[pos], false);
            pHWPens->AssocNum = 0;
            pHWPens->AssocOfs = 0;
        }
        return true;
    }
    if (MenuMode == LOWER && x > 63 && x < 75 && y > 134 && y < 146 ||
        MenuMode == UPPER && x > 63 && x < 75 && y > 94 && y < 106) // else goto 6be0
    {
        UInt16 ofs = pHWPens->CandiDispOfs + 5;
        if (ofs == 10) {
```



```c
    pHWPens->CandiDispOfs = 0;   

    pPrefs->bRedrawMenu = true;   

    }   

    else if (pHWPens->Cands[pHWPens->index - 1][ofs] != 0) {   

    pHWPens->CandiDispOfs + 5;   

    pPrefs->bRedrawMenu = true;   

    }   

    return true;   

}   

    if (MenuMode == LOWER && x > 132 && x < 144 && y > 121 && y < 133 ||   

    MenuMode == UPPER && x > 132 && x < 144 && y > 81 && y < 93) // else goto 6c4a   

    {   

    if (pHWPens->AssocOfs) {   

    pHWPens->AssocOfs -= 20;   

    pPrefs->bRedrawMenu = true;   

    }   

    return true;   

    }   

    if (MenuMode == LOWER && x > 145 && x < 157 && y > 121 && y < 133 ||   

    MenuMode == UPPER && x > 145 && x < 157 && y > 81 && y < 93) // else goto 6cba   

    {   

    if (pHWPens->AssocOfs + 20 < pHWPens->AssocNum) {   

    pHWPens->AssocOfs += 20;   

    pPrefs->bRedrawMenu = true;   

    }   

    return true;   

    }   

    if (MenuMode == LOWER && x > 148 && x < 158 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 148 && x < 158 && y > 94 && y < 106) // else goto 6d46   

    {   

    frm = FrmGetActiveForm();   

    if (FrmValidatePtr(frm))   

        FrmUpdateForm(FrmGetFormId(frm), frmRedrawUpdateCode);   

    ClearMenu(MenuMode);   

    MenuMode = FOLDED;   

    pPrefs->bMenuFolded = true;   

    return true;   

    }   

    if (x > 141 && x < 151 && y > 141 && y < 151 && MenuMode == LOWER)   

    {   

    ClearMenu(MenuMode);   

    pPrefs->MenuMode = GetMenuMode();   

    pPrefs->bMenuFolded = false;   

    pPrefs->bRedrawMenu = true;   

    return true;   

    }   

    if (MenuMode == LOWER && x > 76 && x < 88 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 76 && x < 88 && y > 94 && y < 106) // else goto 6dfa   

    {   

    if (pPrefs->RecogRange == HW_ENGLISH)   

        pPrefs->RecogRange = 0;   

    else   

        pPrefs->RecogRange = HW_ENGLISH;   

    RedrawMenu(pPrefs, pHWPens);   

    return true;   

    }   

    if (MenuMode == LOWER && x > 88 && x < 100 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 88 && x < 100 && y > 94 && y < 106) // else goto 6e6a   

    {   

    if (pPrefs->RecogRange == HW_NUMERIC)   

        pPrefs->RecogRange = 0;   

    else   

        pPrefs->RecogRange = HW_NUMERIC;   

    RedrawMenu(pPrefs, pHWPens);   

    return true;   

    }   

    if (MenuMode == LOWER && x > 100 && x < 112 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 100 && x < 112 && y > 94 && y < 106) // else goto 6e6a   

    {   

    if (pPrefs->RecogRange == HW_SYMBOL)   

        pPrefs->RecogRange = 0;   

    else   

        pPrefs->RecogRange = HW_SYMBOL;   

    RedrawMenu(pPrefs, pHWPens);   

    return true;   

    }   

    if (MenuMode == LOWER && x > 112 && x < 124 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 112 && x < 124 && y > 94 && y < 106) // else goto 6f42   

    {   

    pPrefs->bHalfToFull = !pPrefs->bHalfToFull;   

    RedrawMenu(pPrefs, pHWPens);   

    return true;   

    }   

    if (MenuMode == LOWER && x > 136 && x < 148 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 136 && x < 148 && y > 94 && y < 106) // else goto 6fa8   

    {   

    pPrefs->bMouse = !pPrefs->bMouse;   

    RedrawMenu(pPrefs, pHWPens);   

    return true;   

    }   

    if (MenuMode == LOWER && x > 124 && x < 136 && y > 134 && y < 146 ||   

    MenuMode == UPPER && x > 124 && x < 136 && y > 94 && y < 106) // else goto 7022   

    {   

    LocalID dbID = DmFindDatabase(0, "HWPenS-1");   

    UInt32 Results;   

    if (dbID)   

        SysAppLaunch(0, dbID, 0, HWDefinedCmd, NULL, &Results);   

    return true;   

    }   

    return false;   

}   

BOOL OnPenUp(EventPtr eventP, HW_PREF* pPrefs, HW_PENS* pHWPens, HW_TRACE* pHWTrace)   

{   

    Boolean handled;   

    EventType evt1, evt2;   

    UInt16 PtCount = pHWTrace->PtCount;   

    if (PtCount == 0)   

        handled = false;   

    else if (pHWTrace->b2006)   

        handled = true;   

    else if (pHWTrace->b2007) {   

        pHWTrace->b2007 = 0;   

        pHWTrace->b2008 = 1;   

        evt1 = *eventP;   

        evt1.eType = penDownEvent;   

        EvtAddEventToQueue(&evt1);   

        EvtAddEventToQueue(eventP);   

        pHWTrace->PtCount = 0;   

        handled = true;   

    }   

    else {   

        pHWTrace->points[PtCount].x = 0xff; //(FF,00)表示笔划结束   

        pHWTrace->points[PtCount].y = 0;   

        pHWTrace->PtCount++;   

        evt2.eType = firstUserEvent + 1;   

        EvtAddEventToQueue(&evt2);   

        pHWTrace->dw2002 = TimGetTicks();   

        handled = true;   

    }   

    return handled;   

}   

BOOL OnPenMove(EventPtr eventP, HW_PREF* pPrefs, HW_PENS* pHWPens, HW_TRACE* pHWTrace)   

{   

    Coord x, y;   

    UInt16 var_e, err, i;   

    RectangleType rc;   

    WindowType* winHandle, *winHandleSaved;   

    x = eventP->screenX;   

    y = eventP->screenY;   

    WinWindowToDisplayPt(&x, &y);   

    if (x > 160 || y > 160)   

        return false;   

    if ((var_e = pHWTrace->PtCount) == 0)   

        return false;   

    if (pHWTrace->b2006 != 0)   

        return true;   

    pHWTrace->points[var_e].x = x;   

    pHWTrace->points[var_e].y = y;   

    pHWTrace->PtCount++;   

    if (pHWTrace->b2007) { // or jump to 719e   

        Int16 deltaX, deltaY, d;   

        deltaX = x - pHWTrace->points[0].x;   

        if (deltaX >= 0)   

            d = deltaX;   

        else   

            d = -deltaX;   

        if (d <= 8) { // or 70f0   

            deltaY = y - pHWTrace->points[0].y;   

            if (deltaY >= 0)   

                d = deltaY;   

            else   

                d = -deltaY;   

            if (d <= 8)   

                return true;   

        }   

        RctSetRectangle(&rc, 0, 0, 160, 160);   

        winHandle = WinSetDrawWindow(WinGetDisplayWindow()); // d4   

        InsPtEnable(false);   

        winHandleSaved = WinSaveBits(&rc, &err); // d3   

        MemPtrSetOwner(winHandleSaved, 0);   

        InsPtEnable(true);   

        WinSetDrawWindow(winHandle);   

        FtrSet('HW99', 14, winHandleSaved);   

        pHWTrace->b2007 = false;   

        for (i = 0; i < pHWTrace->PtCount - 1; i++) // d3 = i   

            DrawStroke(pPrefs, pHWTrace->points[i], pHWTrace->points[i + 1]);   

        return true;   

    }   

    // only draw last stroke   

    i = pHWTrace->PtCount - 2;   

    DrawStroke(pPrefs, pHWTrace->points[i], pHWTrace->points[i + 1]);   

    return true;   

}   

BOOL HW_SysHandleEvent(EventPtr eventP)   

{   

    MemHandle     memPref, mem414, mem8202;   

    HW_PREF*      pPrefs;   

    HW_PENS* pHWPens;   

    char* pHWTrace;   

    BOOL (*OrgSysHandleEvent)(EventPtr eventP);   

    FormPtr frm;   

    WinHandle winHandle;   

    EventType event;   

    UInt32 var_2e, i, j;   

    Int8 WaitTime;   

    UInt16 DeltaX, DeltaY;   

    if (!FtrGet('HW99', 10, (UInt32 *)&memPref))   

        pPrefs = (HW_PREF*)memPrefLock(memPref); // a2 = pPrefs   

    if (!FtrGet('HW99', 12, (UInt32 *)&mem414))   

        pHWPens = (HW_PENS*)memPrefLock(mem414); // a3   

    if (!FtrGet('HW99', 11, (UInt32 *)&mem8202))   

        pHWTrace = (char*)memPrefLock(mem8202); // d3   

    // 如果是半屏模式, 则退出.   

    if (!pPrefs->bHalfScrnInput) {// d4 = eventP   

        if (pHWTrace[8194] && eventP->eType == firstUserEvent + 1) {   

            WaitTime = pPrefs->RecogWaitTime * 10;   

            // 如果识别等待时间已过, 就开始识别
```



```
if (TimGetTicks() - pHWTrace[8194] > WaitTime) {

    UInt16 range = GetRecogRange(pPrefs);

    frm = FrmGetActiveForm(); // a4

    pHWTrace[pHWTrace[8192] << 2] = 0xff; // (FF,FF)表示字的结束

    pHWTrace[pHWTrace[8192] << 2 + 2] = 0xff;

    pHWTrace[8192]++;

    FtrGet('HW99', 14, &var_2e);

    winHandle = WinSetDrawWindow(WinGetDisplayWindow()); // a5

    WinRestoreBits(var_2e, 0, 0);

    WinSetDrawWindow(winHandle);

    if (pPrefs->bSplitScreen) {

        pHWPens->index = 0;

        if ((i = HWHalfScreenRecognize(pHWTrace, &pHWPens->Cands[pHWPens->index], 10, range)) > 0) { // d5

            pHWPens->AssocNum = 0;

            pHWPens->AssocOfs = 0;

            j = 0; // d4

            do EnqueueInputKeys(frm, pHWPens, j++, 0);

            while (j < i - 1);

            EnqueueInputKeys(frm, pHWPens, i - 1, 1);

        }

    }

    else if (HWFullScreenRecognize(pHWTrace, pHWPens->Cands[pHWPens->index], 10, range) > 0) {

        pHWPens->AssocNum = 0;

        pHWPens->AssocOfs = 0;

        EnqueueInputKeys(frm, pHWPens, pHWPens->index, 1);

        pPrefs->bRedrawMenu = true;

    }

    MemSet(pHWTrace, 0, 8202, 0);

    goto _exit;

}

else {

    if (eventP->eType == firstUserEvent + 1) {

        EvtGetEvent(&event, evtWaitForever);

        if (event.eType == nilEvent) {

            event.eType = firstUserEvent + 1;

            EvtAddEventToQueue(&event);

        }

        goto _exit;

    }

}

if (eventP->eType == penDownEvent) {

    if (pPrefs->bSetting) {

        pPrefs->ShotcutPenDownX = eventP->screenX;

        pPrefs->ShotcutPenDownY = eventP->screenY;

    }

    else {

        if (eventP->screenX - pPrefs->ShotcutPenDownX >= 0)
            DeltaX = eventP->screenX - pPrefs->ShotcutPenDownX;
        else
            DeltaX = -(eventP->screenX - pPrefs->ShotcutPenDownX);

        if (DeltaX <= 15) {

            if (eventP->screenY - pPrefs->ShotcutPenDownY >= 0)
                DeltaY = eventP->screenY - pPrefs->ShotcutPenDownY;
            else
                DeltaY = -(eventP->screenY - pPrefs->ShotcutPenDownY);

            if (DeltaY <= 15) {

                pPrefs->bUsingShortcut = 1;

                goto _exit;

            }

        }

    }

}

else if (eventP->eType == penUpEvent) {

    if (pPrefs->bSetting) {

        if (pPrefs->ShotcutPenDownX != -1) {

            ControlType *controlP;

            frm = GetActiveForm();

            controlP = FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1058));

            pPrefs->ShotcutPenUpX = eventP->screenX;

            pPrefs->ShotcutPenUpY = eventP->screenY;

            pPrefs->bSetting = false;

            CtlSetLabel(controlP, "设置完成!");

        }

        goto _exit;

    }

    else if (pPrefs->bUsingShortcut) {

        frm = GetActiveForm();

        if (eventP->screenX - pPrefs->ShotcutPenUpX > 0)
            DeltaX = eventP->screenX - pPrefs->ShotcutPenUpX;
        else
            DeltaX = -(eventP->screenX - pPrefs->ShotcutPenUpX);

        if (DeltaX <= 15) {

            if (eventP->screenY - pPrefs->ShotcutPenUpY > 0)
                DeltaY = eventP->screenY - pPrefs->ShotcutPenUpY;
            else
                DeltaY = -(eventP->screenY - pPrefs->ShotcutPenUpY);

            if (DeltaY <= 15) {

                UInt16 focus;

                if ((focus = FrmGetFocus(frm)) != -1) {

                    pPrefs->bOpenHW = !pPrefs->bOpenHW;

                    if (pPrefs->bOpenHW)
                        pPrefs->bRedrawMenu = true;
                    else {

                        ClearMenu(pPrefs->MenuMode);

                        RedrawCurForm();

                    }

                    pPrefs->bUsingShortcut = 0;

                    goto _exit;

                }

            }

        }

    }

}

if (pPrefs->bOpenHW) {

    if (pPrefs->bRedrawMenu) {

        RedrawMenu(pPrefs, pHWPens);

        pPrefs->bRedrawMenu = false;

    }

    if (pPrefs->Phase == ENTER) {

        MemSet(pHWPens, 414, 0);

        frm = FrmGetActiveForm();

        if (frm && GetCurField(frm)) {

            pHWPens->InsPtPos = FldGetInsPtPosition(GetCurField(frm));

            if (pPrefs->bMenuFolded)
                pPrefs->MenuMode = FOLDED;
            else
                pPrefs->MenuMode = GetMenuMode();

            pPrefs->bRedrawMenu = true;

        }

        else
            pPrefs->MenuMode = HIDDEN;

        pPrefs->Phase = READY;

    }

    else if (pPrefs->Phase == LEAVE) {

        if (pPrefs->MenuMode != HIDDEN) {

            frm = FrmGetActiveForm(); // a4

            if (frm) {

                winHandle = WinGetActiveWindow();

                if (winHandle == FrmGetWindowHandle(frm) && frm->focus != -1)
                    FrmDrawForm(frm);

            }

        }

        pPrefs->Phase = READY;

    }

    else if (pPrefs->Phase == CHGFLD) {

        frm = FrmGetActiveForm();

        if (InsPtEnabled()) {

            AdjustCandIndex(frm, pHWPens);

            if (pPrefs->bMenuFolded) {

                pPrefs->MenuMode = FOLDED;

                pPrefs->bRedrawMenu = false; // then goto 6684

            }

            else {

                Int8 MenuMode = GetMenuMode();

                if (pPrefs->MenuMode == HIDDEN) {

                    pPrefs->MenuMode = MenuMode;

                    pPrefs->bRedrawMenu = true;

                    // goto 6684

                }

                else if (pPrefs->MenuMode != MenuMode) {

                    ClearMenu(pPrefs->MenuMode);

                    frm = FrmGetActiveForm();

                    if (FrmGetFocus(frm) != -1) {

                        RedrawCurForm();

                        pPrefs->MenuMode = MenuMode;

                    }

                } // goto 6684

            }

        }

        else if (pPrefs->MenuMode != HIDDEN) {

            frm = GetActiveForm();

            ClearMenu(pPrefs->MenuMode);

            if (FrmGetFocus(frm) != -1) {
```



```
    RedrawCurForm();   

    pPrefs->MenuMode = HIDDEN;   

    }   

    }   

    pPrefs->Phase = READY; // 6684 here   

    }   

    switch (eventP->eType)   

    {   

    case winEnterEvent:   

    case frmOpenEvent:   

        pPrefs->Phase = ENTER;   

        break;   

    case winExitEvent:   

        ClearMenu(pPrefs->MenuMode);   

        pPrefs->Phase = LEAVE;   

        break;   

    case fldEnterEvent:   

    case tblSelectEvent:   

        pPrefs->Phase = CHGFLD;   

        break;   

    case fldChangedEvent:   

    case frmUpdateEvent:   

        pPrefs->bRedrawMenu = true;   

        break;   

    default:   

        if (pPrefs->MenuMode != HIDDEN) {   

            switch (eventP->eType)   

            {   

            case penDownEvent:   

                if (pHWTrace[8200] == 0)   

                    if (OnPenDown(eventP, pPrefs, pHWPens, pHWTrace))   

                        goto _exit;   

                break;   

            case penUpEvent:   

                if (pHWTrace[8200] != 0)   

                    pHWTrace[8200] = 0;   

                else if (pPrefs->bMouse == 0 && OnPenUp(eventP, pPrefs, pHWPens, pHWTrace))   

                    goto _exit;   

                break;   

            case penMoveEvent:   

                if (pPrefs->bMouse == 0 && OnPenMove(eventP, pPrefs, pHWPens, pHWTrace))   

                    goto _exit;   

            }   

        }   

    }   

    }   

    }   

    MemHandleUnlock(memPref);   

    MemHandleUnlock(mem414);   

    MemHandleUnlock(mem8202);   

    FtrGet('HW99', 13, (UInt32)&OrgSysHandleEvent);   

    return OrgSysHandleEvent(eventP);   

_exit:   

    MemHandleUnlock(memPref);   

    MemHandleUnlock(mem414);   

    MemHandleUnlock(mem8202);   

    return 1;   

}   

static Err RomVersionCompatible (UInt32 requiredVersion, UInt16 launchFlags)   

{   

    UInt32 romVersion;   

    DmSearchStateType stateInfoP;   

    UInt16 cardNo;   

    LocalID dbID;   


    // See if we have at least the minimum required version of the ROM or later.   

    FtrGet(sysFtrCreator, sysFtrNumROMVersion, &romVersion);   

    if (romVersion < requiredVersion)   

        {   

        if ((launchFlags & (sysAppLaunchFlagNewGlobals | sysAppLaunchFlagUIApp)) ==   

            (sysAppLaunchFlagNewGlobals | sysAppLaunchFlagUIApp))   

            {   

            FrmAlert (RomIncompatibleAlert);   

            // Pilot 1.0 will continuously relaunch this app unless we switch to   

            // another safe one.   

            if (romVersion < version350)   

                {   

                DmGetNextDatabaseByTypeCreator(true, &stateInfoP, 'appl', 'pref', true, &cardNo, &dbID);   

                if (!dbID)   

                    ErrDisplayFileLineMsg("HWGBIMEMain.c", 551, "Could not find app");   

                if (dbID && SysUIAppSwitch(cardNo, dbID, 0, 0))   

                    ErrDisplayFileLineMsg("HWGBIMEMain.c", 551, "Could not launch app");   

                //AppLaunchWithCommand(sysFileCDefaultApp, sysAppLaunchCmdNormalLaunch, NULL);   

                }   

            }   

        return (sysErrRomIncompatib   

static Err RomVersionCompatible (UInt32 requiredVersion, UInt16 launchFlags)   

{   

    UInt32 romVersion;   

    DmSearchStateType stateInfoP;   

    UInt16 cardNo;   

    LocalID dbID;   


    // See if we have at least the minimum required version of the ROM or later.   

    FtrGet(sysFtrCreator, sysFtrNumROMVersion, &romVersion);   

    if (romVersion < requiredVersion)   

        {   

        if ((launchFlags & (sysAppLaunchFlagNewGlobals | sysAppLaunchFlagUIApp)) ==   

            (sysAppLaunchFlagNewGlobals | sysAppLaunchFlagUIApp))   

            {   

            FrmAlert (RomIncompatibleAlert);   

            // Pilot 1.0 will continuously relaunch this app unless we switch to   

            // another safe one.   

            if (romVersion < version350)   

                {   

                DmGetNextDatabaseByTypeCreator(true, &stateInfoP, 'appl', 'pref', true, &cardNo, &dbID);   

                if (!dbID)   

                    ErrDisplayFileLineMsg("HWGBIMEMain.c", 551, "Could not find app");   

                if (dbID && SysUIAppSwitch(cardNo, dbID, 0, 0))   

                    ErrDisplayFileLineMsg("HWGBIMEMain.c", 551, "Could not launch app");   

                //AppLaunchWithCommand(sysFileCDefaultApp, sysAppLaunchCmdNormalLaunch, NULL);   

                }   

            }   

        return (sysErrRomIncompatible);   

        }   

    return (0);   

}   

static void SetDefaultValues(void)   

{   

    MemHandle    memHandle;   

    HW_PREF*    pPrefs;   

    FormPtr frm;   

    frm = FrmGetActiveForm(); // a2 = frm   

    if (FtrGet('HW99', 10, (UInt32 *)&memHandle)) {   

        FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1001), 0);   

        return;   

    }   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1001), 1);   

    pPrefs = (HW_PREF*)MemHandleLock(memHandle); // a3 = ptr   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1008), (Int16)(pPrefs->flags & HW_NUMERIC));    //数字   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1010), (Int16)(pPrefs->flags & HW_ENGLISH));    //英文   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1012), (Int16)(pPrefs->flags & HW_SYMBOL));    //符号   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1011), (Int16)(pPrefs->flags & HW_SUBCOMMONCCHAR));    //次常用汉字   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1009), (Int16)(pPrefs->flags & HW_COMMONCCHAR));       //常用汉字   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1013), (Int16)(pPrefs->flags & HW_TRA2SIMPLIFIED));    //写繁得简   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1003), pPrefs->bSplitScreen);       //启动全屏分割   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1060), (Int16)(pPrefs->flags & HW_HALF2FULL));    //半角字符转成全角字符   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1061), (Int16)(pPrefs->flags & HW_UPPER2LOWER));    //大写字母转成小写字母   

    FrmSetControlValue(frm, FrmGetObjectIndex(frm, 1062), (Int16)(pPrefs->flags & HW_LOWER2UPPER));    //小写字母转成大写字母   

    FrmSetControlGroupSelection(frm, 3, (Int16)(pPrefs->PenColor + 1017));       //笔迹颜色选择   

    FrmSetControlGroupSelection(frm, 4, (Int16)(pPrefs->RecogWaitTime + 1043));    //识别等待时间   

    FrmSetControlGroupSelection(frm, 2, (Int16)(pPrefs->PenThickness + 1052));    //笔迹粗细   

    CtlSetLabel(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1058)), "设置快捷方式");   

    MemHandleUnlock(memHandle);   

}   

void DrawTabLines(void)   

{   

    int i;   

    Coord x1 = 3;   

    Coord y1 = 22;   

    Coord x2, y2;   

    WinHandle winHandle = WinGetDisplayWindow(); // a2 = winHandle   

    WinSetDrawWindow(winHandle);   

    for (i = 0; i < 4; i++) {   

        x2 = x1;   

        y2 = y1 - 15;   

        WinDrawLine(x1, y1, x2, y2);   

        y1 = y2;   

        x2 += 4;   

        y2 -= 4;   

        WinDrawLine(x1, y1, x2, y2);   

        x1 = x2;   

        y1 = y2;   

        x2 = x1 + 30;   

        WinDrawLine(x1, y1, x2, y2);   

        x1 = x2;   

        x2 += 4;   

        y2 += 4;   

        WinDrawLine(x1, y1, x2, y2);   

        x1 = x2;   

        y1 = y2;   

        y2 += 15;   

        WinDrawLine(x1, y1, x2, y2);   

    }   

    y2 += 135;   

    WinDrawLine(x1, y1, x1, y2);   

    WinDrawLine(x1, y2, 3, y2);   

    WinDrawLine(3, y2, 3, (Coord)(y2 - 135));
```



`WinDrawLine(41, y2, 3 + 4 * 38, y2);`

`}`

`void DrawActiveTab(int);`

`void DrawGroupFrame(Int16 left, Int16 top, Int16 right, Int16 bottom, char* title, Int16 i, Boolean b);`

// 显示"设置"选项卡中的控件

`void ShowSettingControls(void)`

`{`

`FormPtr frm;`

`MemHandle memHandle;`

`frm = FrmGetActiveForm(); // a2 = frm`

`DrawActiveTab(0);`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1001)));`

`if (!FtrGet('HW99', 10, (UInt32 *)&memHandle)) {`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1004)), true); //"设置"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1005)), true); //"笔迹"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1006)), true); //"辅助"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1007)), true); //"关于"选项卡`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1008))); //数字`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1009))); //常用汉字`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1010))); //英文`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1011))); //次常用汉字`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1012))); //符号`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1013))); //写繁得简`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1044))); //识别等待时间: 1`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1045))); //识别等待时间: 2`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1046))); //识别等待时间: 3`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1047))); //识别等待时间: 4`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1048))); //识别等待时间: 5`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1049))); //识别等待时间: 6`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1050))); //识别等待时间: 7`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1051))); //识别等待时间: 8`

`CtlShowControl(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1052))); //识别等待时间: 9`

`FrmShowObject(frm, FrmGetObjectIndex(frm, 1015)); //长`

`FrmShowObject(frm, FrmGetObjectIndex(frm, 1016)); //短`

`DrawGroupFrame(50, 8, 65, 140, "识别范围", 10, true); //"识别范围"框`

`DrawGroupFrame(125, 8, 25, 140, "识别等待时间", 10, true); //"识别等待时间"框`

`}`

`} else {`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1004)), false); //"设置"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1005)), false); //"笔迹"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1006)), false); //"辅助"选项卡`

`CtlSetEnabled(FrmGetObjectPtr(frm, FrmGetObjectIndex(frm, 1007)), false); //"关于"选项卡`

`}`

`}`

`static Boolean HW_HandleEvent(EventPtr event)`

`{`

`FormPtr frm;`

`Boolean handled = false;`

`MemHandle memHandle;`

`HW_PREF* pPrefs;`

`MemHandle BitmapHandle;`

`BitmapPtr BitmapPtr;`

`frm = FrmGetActiveForm (); // a3 = frm`

`if (event->eType == frmOpenEvent) {`

`BitmapHandle = DmGet1Resource(bitmapRsc, 0x44c); // a2 = MemHandle`

`BitmapPtr = MemHandleLock(BitmapHandle); // a4`

`WinPaintBitmap(BitmapPtr, 40, 10);`

`MemHandleUnlock(BitmapHandle);`

`DmReleaseResource(BitmapHandle);`

`WinScreenLock(winLockCopy);`

`SetDefaultValues();`

`FrmDrawForm(frm);`

`DrawTabLines();`

`ShowSettingControls();`

`WinScreenUnlock();`

`handled = true;`

`}`

`} else if (event->eType == ctlSelectEvent) {`

`switch (event->data.ctlSelect.controlID)`

`{`

`case 1001: //启动汉王手写输入法`

`if (event->data.ctlSelect.on) {`

`if (HW_Install()) return true;`

`SetDefaultValues();`

`ShowSettingControls();`

`if (FtrGet('HW99', 10, (UInt32 *)&memHandle))`

`return true;`

`pPrefs = (HW_PREF*)MemHandleLock(memHandle); // a3 = ptr`

`pPrefs->bOpenHW = true;`

`pPrefs->MenuMode = HIDDEN;`

`MemHandleUnlock(memHandle);`

`}`

`} else {`

`HideSettingControls();`

`HW_Uninstall();`

`}`

`handled = true;`

`break;`

`case 1004: //"设置"选项卡`

`HidePenControls();`

`HideAccessorialControls();`

`HideAboutControls();`

`ShowSettingControls();`

`handled = true;`

`break;`

`case 1005: //"笔迹"选项卡`

`HideSettingControls();`

`HideAccessorialControls();`

`HideAboutControls();`

`ShowPenControls();`

`handled = true;`

`break;`

`case 1006: //"辅助"选项卡`

`HideSettingControls();`

`HidePenControls();`

`HideAboutControls();`

`ShowAccessorialControls();`

`handled = true;`

`break;`

`case 1007: //"关于"选项卡`

`HideSettingControls();`

`HidePenControls();`

`HideAccessorialControls();`

`ShowAboutControls();`

`handled = true;`

`break;`

`case 1017: //笔迹颜色: 彩色`

`case 1018: //笔迹颜色: 红色`

`case 1019: //笔迹颜色: 绿色`

`case 1020: //笔迹颜色: 棕色`

`case 1021: //笔迹颜色: 黑色`

`case 1022: //笔迹颜色: 蓝色`

`case 1023: //笔迹颜色: 黄色`

`case 1024: //笔迹颜色: 紫色`

`handled = true;`

`break;`

`case 1058: //启动/关闭快捷方式设置`

`handled = true;`

`break;`

`case 1061: //大写字母转成小写字母`

`handled = true;`

`break;`

`case 1062: //小写字母转成大写字母`

`handled = true;`

`break;`

`}`

`}`

`return (handled);`

`}`

`static Boolean ApplicationHandleEvent (EventPtr event)`

`{`

`UInt16 formId;`

`FormPtr frm;`

`if (event->eType == frmLoadEvent)`

`{`

`// Load the form resource.`

`formId = event->data.frmLoad.formID;`

`frm = FrmInitForm (formId);`

`FrmSetActiveForm (frm);`

`// Set the event handler for the form. The handler of the currently`

`// active form is called by FrmHandleEvent each time is receives an`

`// event.`

`if (formId == HW_MainForm)`

`FrmSetEventHandler (frm, HW_HandleEvent);`

`return (true);`

`}`

`return (false);`

`}`

`static void EventLoop (void)`

`{`

`EventType event;`

`do {`

`EvtGetEvent (&event, evtWaitForever);`

`if (! SysHandleEvent (&event))`

`if (! ApplicationHandleEvent (&event))`

`FrmDispatchEvent (&event);`

`} while (event.eType != appStopEvent);`

`}`

// 加载汉字识别编码库

`int LoadRecognitionDB()`

`{`

`DmOpenRef dor;`

`MemPtr* mp;`

`MemPtr* ptr;`

`UInt16 i;`

`dor = DmOpenDatabaseByTypeCreator('GB68', 'HW99', dmModeReadOnly); // a3`

`if (!dor) return -1;`

`mp = (MemPtr*)MemPtrNew(86 * sizeof(void*)); // a4`

`if (!mp) {`

`DmCloseDatabase(dor);`

`return -2;`

`}`

`////////////////////////////////////////////////////////////////////////////////`

`// Specify 0 to set the owner to the operating system.`

`// When you allocate a parameter block to pass to SysUIAppSwitch or`

`// SysAppLaunch, you must call MemPtrSetOwner or MemHandleSetOwner to grant`



```c
    // ownership of the parameter block chunk, and any other chunks referenced   
    // in it, to the OS (your application is originally set as the owner).   
    // If you don't change the ownership of the parameter block, it will get freed   
    // before the application you're launching has a chance to use it.   
    ////////////////////////////////////////////////////////////////////////////////// 
    MemPtrSetOwner(mp, 0); 
    for (i = 0; i < 22; i++) // d3 = i, a6 = mp 
        mp[i] = MemHandleLock(DmQueryRecord(dor, i)); 
    DmCloseDatabase(dor); 
    for (i = 6, ptr = &mp[22]; i < 22; i++) { 
        *ptr++ = mp[i]; 
        *ptr++ = (Int8*)mp[i] + 9757; 
        *ptr++ = (Int8*)mp[i] + 19514; 
        *ptr++ = (Int8*)mp[i] + 29271; 
    } 
    if (FtrSet('HW99', 1, (UInt32)mp)) { 
        for (i = 0; i < 22; i++) 
            MemPtrUnlock(mp[i]); 
        MemChunkFree(mp); 
        return -3; 
    } 
    return 0; 
} 

// 加载多元词组联想库 
int LoadPhraseDB() 
{ 
    DmOpenRef dor; 
    MemHandle** mp; 
    UInt16 i; 
    // In Resource Database Header, there are type and creator fields holding 
    // 4-byte signatures of the database type and creator as defined by the 
    // application that created the database. 
    dor = DmOpenDatabaseByTypeCreator('HMPR', 'HW99', dmModeReadOnly); // a3 
    if (!dor) return -1; 
    mp = (MemHandle**)MemPtrNew(4 * sizeof(void*)); // a4 
    if (!mp) { 
        DmCloseDatabase(dor); 
        return -2; 
    } 
    MemPtrSetOwner(mp, 0); 
    for (i = 0; i < 4; i++) // d3 = i, a6 = mp 
        mp[i] = MemHandleLock(DmQueryRecord(dor, i)); 
    DmCloseDatabase(dor); 
    if (FtrSet('HW99', 2, (UInt32)mp)) { 
        for (i = 0; i < 4; i++) 
            MemPtrUnlock(mp[i]); 
        MemChunkFree(mp); 
        return -3; 
    } 
    return 0; 
} 

// 安装汉王手写系统 
int HW_Install(void) 
{ 
    MemHandle memHandlePref, memHandle2, memHandle3; 
    HW_PREF* pPref; 
    void* mp2, *mp3; 
    UInt16 sz = sizeof(HW_PREF); 
    LocalID dbID; 
    LoadRecognitionDB();    // 加载汉字识别编码库 
    LoadPhraseDB();         // 加载多元词组联想库 
    memHandlePref = MemHandleNew(sz); 
    if (!memHandlePref) return 1; 
    MemHandleSetOwner(memHandlePref, 0); 
    memHandle2 = MemHandleNew(8202); 
    if (!memHandle2) { 
        MemHandleFree(memHandlePref); 
        return 1; 
    } 
    MemHandleSetOwner(memHandle2, 0); 
    memHandle3 = MemHandleNew(414); 
    if (!memHandle3) { 
        MemHandleFree(memHandlePref); 
        MemHandleFree(memHandle2); 
        return 1; 
    } 
    MemHandleSetOwner(memHandle3, 0); 
    if (FtrSet('HW99', 10, (UInt32)memHandlePref) || 
        FtrSet('HW99', 11, (UInt32)memHandle2) || 
        FtrSet('HW99', 12, (UInt32)memHandle3)) 
    { 
        MemHandleFree(memHandlePref); 
        MemHandleFree(memHandle2); 
        MemHandleFree(memHandle3); 
        return 1; 
    } 
    pPref = (HW_PREF*)MemHandleLock(memHandlePref); // a2 
    mp2 = MemHandleLock(memHandle3); // d4 
    mp3 = MemHandleLock(memHandle2); // d5 
    if (PrefGetAppPreferences('HW99', 0, pPref, &sz, true) == noPreferenceFound) { 
        pPref->bInstallHW = true; 
        pPref->flags = 3; 
        pPref->RecogRange = HW_COMMONCCHAR; // 默认识别范围为: 常用汉字 
        pPref->RecogWaitTime = 3; 
        pPref->PenThickness = 2; 
        pPref->PenColor = 0; 
        pPref->ShotcutPenDownX = 0; 
        pPref->ShotcutPenDownY = 0; 
        pPref->ShotcutPenUpX = 0; 
        pPref->ShotcutPenUpY = 0; 
        pPref->bSetting = false; 
        pPref->bUsingShortcut = false; 
        pPref->bSplitScreen = false; 
        pPref->bHalfToFull = false; 
        pPref->bLowerToUpper = false; 
        pPref->bUpperToLower = false; 
        pPref->bOpenHW = true; 
        pPref->bHalfScrnInput = false; 
        pPref->bMouse = false; 
        pPref->bRedrawMenu = false; 
        pPref->Phase = READY; 
        if (ChkReg() == 99) 
            pPref->bReged = 0xff; 
        else 
            pPref->bReged = 0; 
        pPref->MenuMode = HIDDEN; 
        pPref->bMenuFolded = 1; 
        PrefSetAppPreferences('HW99', 6, 0, pPref, sz, true); 
    } 
    else { 
        if (ChkReg() == 99) 
            pPref->bReged = 0xff; 
        else 
            pPref->bReged = 0; 
    } 
    MemSet(mp2, 414, 0); 
    MemSet(mp3, 8202, 0); 
    if (FtrSet('HW99', 13, (UInt32)SysGetTrapAddress(sysTrapSysHandleEvent))) 
        return 1; 
    SysSetTrapAddress(sysTrapSysHandleEvent, HW_SysHandleEvent); 
    dbID = DmFindDatabase(0, "HWPenS-1"); // d4 
    if (dbID) { 
        SysNotifyRegister( 
            0,     // cardNo of the storage card on which the app code resource resides. 
            dbID,  // Local ID of the application or code resource 
            'sync', // The notification that the application wants to receive 
            NULL,  // Set to NULL to receive the notification as an app launch code. 
            sysNotifyNormalPriority, 
            0     // userDataP Caller-defined data to pass to the notification handler 
        ); 
    } 
    MemHandleUnlock(memHandlePref); 
    MemHandleUnlock(memHandle2); 
    MemHandleUnlock(memHandle3); 
    FtrSet('HW99', 24, 0); 
    return 0; 
} 

int UnloadRecognitionDB(void) 
{ 
    void** mp; 
    int i; 
    if (FtrGet('HW99', 1, (UInt32*)&mp) || !mp) 
        return -1; 
    for (i = 0; i < 22; i++) 
        MemPtrUnlock(mp[i]); 
    MemChunkFree(mp); 
    FtrUnregister('HW99', 1); 
    return 0; 
} 

int UnloadPhraseDB(void) 
{ 
    void** mp; 
    int i; 
    if (FtrGet('HW99', 2, (UInt32*)&mp) || !mp) 
        return -1; 
    for (i = 0; i < 4; i++) 
        MemPtrUnlock(mp[i]); 
    MemChunkFree(mp); 
    FtrUnregister('HW99', 2); 
    return 0; 
} 

// 卸载汉王手写系统 
void HW_Uninstall(void) 
{ 
    MemHandle memHandlePref, memHandle1, memHandle2; 
    void *procP; 
    HW_PREF* pPrefs; 
    UnloadRecognitionDB();    // 卸载汉字识别编码库 
    UnloadPhraseDB();        // 卸载多元词组联想库 
    if (!FtrGet('HW99', 10, (UInt32*)&memHandlePref)) { 
        pPrefs = (HW_PREF*) MemHandleLock(memHandlePref); 
        pPrefs->bInstallHW = false; 
        PrefSetAppPreferences('HW99', 6, 0, pPrefs, sizeof(HW_PREF), true); 
    } 
    FtrGet('HW99', 11, (UInt32*)&memHandle1); 
    FtrGet('HW99', 12, (UInt32*)&memHandle2); 
    MemHandleUnlock(memHandlePref); 
    MemHandleFree(memHandlePref); 
    MemHandleFree(memHandle1); 
    MemHandleFree(memHandle2); 
    FtrUnregister('HW99', 10); 
    FtrUnregister('HW99', 11); 
    FtrUnregister('HW99', 12); 
    if (!FtrGet('HW99', 13, (UInt32*)&procP)) 
        SysSetTrapAddress(sysTrapSysHandleEvent, procP); 
    FtrUnregister('HW99', 13); 
    FtrUnregister('HW99', 24); 
} 

void GetSettings(void) 
{ 
    FormPtr frm; 
    MemHandle memHandlePref; 
    HW_PREF* pPref; 
    UInt16 select; 
    frm = FrmGetActiveForm(); // a3 = frm 
    if (!FtrGet('HW99', 10, (UInt32 *)&memHandlePref)) { 
        pPref = (HW_PREF*)MemHandleLock(memHandlePref); // a2 
        pPref->flags = 0; 
        if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1008))) 
            pPref->flags |= HW_NUMERIC; 
        if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1010))) 
            pPref->flags |= HW_ENGLISH; 
        if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1012))) 
            pPref->flags |= HW_SYMBOL; 
```



```
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1011)))
        pPref->flags |= HW_SUBCOMMONCCHAR;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1009)))
        pPref->flags |= HW_COMMONCCHAR;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1013)))
        pPref->flags |= HW_TRA2SIMPLIFIED;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1003)))
        pPref->bSplitScreen = true;
    else
        pPref->bSplitScreen = false;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1060)))
        pPref->flags |= HW_HALF2FULL, pPref->bHalfToFull = true;
    else
        pPref->bHalfToFull = false;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1061)))
        pPref->flags |= HW_UPPER2LOWER, pPref->bUpperToLower = true;
    else
        pPref->bUpperToLower = false;
    if (FrmGetControlValue(frm, FrmGetObjectIndex(frm, 1062)))
        pPref->flags |= HW_LOWER2UPPER, pPref->bLowerToUpper = true;
    else
        pPref->bLowerToUpper = false;
    select = FrmGetControlGroupSelection(frm, 3);
    if (select == frmNoSelectedControl)
        pPref->PenColor = 4;
    else
        pPref->PenColor = FrmGetObjectId(frm, select) + 7;
    select = FrmGetControlGroupSelection(frm, 4);
    pPref->RecogWaitTime = FrmGetObjectId(frm, select) - 19;
    select = FrmGetControlGroupSelection(frm, 2);
    pPref->PenThickness = FrmGetObjectId(frm, select) - 28;
    MemHandleUnlock(memHandlePref);
}
}
```

```
UInt32 HWGBIMEMain(UInt16 cmd, void * UNUSED_PARAM(cmdPBP), UInt16 launchFlags)
{
    UInt16 error, RegCode;
    MemHandle memHandle;
    HW_PREF* prefs;
    UInt16 sz;
    LocalID DbId;

    error = RomVersionCompatible (0x3503000, launchFlags);
    if (error) return (error);

    switch (cmd)
    {
    case sysAppLaunchCmdNormalLaunch:
        FrmGotoForm(HW_MainForm);
        EventLoop();
        GetSettings();
        FrmCloseAllForms();
        if (!FtrGet('HW99', 10, (UInt32 *)&memHandle)) {
            prefs = (HW_PREF*)MemHandleLock(memHandle); // a3 = ptr
            PrefSetAppPreferences('HW99', 6, 0, prefs, sizeof(HW_PREF), true/*in 'saved' database*/);
            MemHandleUnlock(memHandle);
        }
        break;
    case sysAppLaunchCmdSystemReset:
        sz = sizeof(HW_PREF);
        memHandle = MemHandleNew(sz);
        prefs = (HW_PREF*)MemHandleLock(memHandle);
        //////////////////////////////////////////////////////////////////////
        // Preferences (including those stored in the "unsaved" database)
        // survive a soft reset because they reside in the storage heap.
        // Features are deleted upon a soft reset. For this reason, preferences
        // are more appropriate for storing application state than feature are.
        //
        // The "saved" preferences database is backed up when a user performs
        // the HotSync operation. The "unsaved" preferences database is not
        // backed up by default. (The user can use a third-party tool to set the
        // backup bit in the "unsaved" preferences database, which would cause it
        // to be backed up.) Both the "saved" and the "unsaved" preferences
        // reside in the storage heap and thus persist across soft resets.
        // The only way that preferences are lost is if a hard reset is performed.
        //////////////////////////////////////////////////////////////////////
        //////////////////////////////////////////////////////////////////////
        // PrefGetAppPreferences() function returns the version number of the
        // preference structure that it retrieved from the database.
        //////////////////////////////////////////////////////////////////////
        // If HW has already been run and its preferences indicates it's installed,
        // then install HW system
        if (PrefGetAppPreferences('HW99', 0, prefs, &sz, true) != noPreferenceFound &&
            prefs->bInstallHW &&
            !HW_Install() &&
            (DbId = DmFindDatabase(0, "HWPenS-1")) != 0)
        {
            SysNotifyRegister(0, DbId, 'sync', 0, 0, 0);
        }
        MemHandleUnlock(memHandle);
        MemHandleFree(memHandle);
        break;
    case sysAppLaunchCmdNotify:
        if (!FtrGet('HW99', 10, (UInt32 *)&memHandle)) {
            prefs = (HW_PREF*)MemHandleLock(memHandle);
            RegCode = ChkReg();
            if (RegCode == 99) {
                prefs->bReged = 0xff; // 是注册用户
                PrefSetAppPreferences('HW99', 6, 0, prefs, 32, true); // 写入"saved" preferences
            }
            MemHandleUnlock(memHandle);
        }
        break;
    case HWDefinedCmd:
        if (!FtrGet('HW99', 10, (UInt32 *)&memHandle)) {
            prefs = (HW_PREF*)MemHandleLock(memHandle);
            prefs->bHalfScrnInput = true;
            _967c();
            prefs->bHalfScrnInput = false;
            MemHandleUnlock(memHandle);
        }
        break;
    }
    return 0;
}
```

```
UInt32 _PilotMain(UInt16 cmd, void * UNUSED_PARAM(cmdPBP), UInt16 launchFlags)
{
    return HWGBIMEMain(cmd, cmdPBP, launchFlags);
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
