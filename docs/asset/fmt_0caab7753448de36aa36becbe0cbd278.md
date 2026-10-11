
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
