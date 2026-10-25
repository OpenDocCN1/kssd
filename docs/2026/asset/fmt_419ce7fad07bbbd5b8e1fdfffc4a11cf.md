
```
NTSTATUS

PsSetCreateProcessNotifyRoutine(

    IN PCREATE_PROCESS_NOTIFY_ROUTINE NotifyRoutine,

    IN BOOLEAN Remove

    )

{

    ULONG i;

    for (i=0; i < PSP_MAX_CREATE_PROCESS_NOTIFY; i++) {
        if (Remove) {
            if (PspCreateProcessNotifyRoutine[i] == NotifyRoutine) {
                PspCreateProcessNotifyRoutine[i] = NULL;
                PspCreateProcessNotifyRoutineCount -= 1;
                return STATUS_SUCCESS;
            }

        } else {
            if (PspCreateProcessNotifyRoutine[i] == NULL) {
                PspCreateProcessNotifyRoutine[i] = NotifyRoutine;
                PspCreateProcessNotifyRoutineCount += 1;
                return STATUS_SUCCESS;
            }
        }
    }

    return Remove ? STATUS_PROCEDURE_NOT_FOUND : STATUS_INVALID_PARAMETER;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
