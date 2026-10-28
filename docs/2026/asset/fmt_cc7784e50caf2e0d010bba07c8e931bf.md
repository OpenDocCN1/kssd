
# `ProcessListEntry` 进程链表

想要隐藏进程把所在数据的链表进程断链即可

```
//断链关键代码

FORCEINLINE

BOOLEAN

MyRemoveEntryList(

    _In_ PLIST_ENTRY Entry

)



{



    PLIST_ENTRY PrevEntry;

    PLIST_ENTRY NextEntry;



    NextEntry = Entry->Flink;

    PrevEntry = Entry->Blink;

    if ((NextEntry->Blink != Entry) || (PrevEntry->Flink != Entry)) {

        return FALSE;

    }



    PrevEntry->Flink = NextEntry;

    NextEntry->Blink = PrevEntry;

    return (BOOLEAN)(PrevEntry == NextEntry);

}

//





	//断链 ProcessListEntry



	PLIST_ENTRY ProfileListHead = (PLIST_ENTRY)((PUCHAR)mypEProcess + 0x18);

	MyRemoveEntryList(ProfileListHead);





	//断链   struct _LIST_ENTRY ThreadListHead;                                      //0x30

	PLIST_ENTRY ThreadListHead = (PLIST_ENTRY)((PUCHAR)mypEProcess + 0x30);

	MyRemoveEntryList(ProfileListHead);





	//断链struct_LIST_ENTRYSessionProcessLinks

	PLIST_ENTRY essionProcessLinks = (PLIST_ENTRY)((PUCHAR)mypEProcess + get_eprocess_session_offset());

	MyRemoveEntryList(essionProcessLinks);



  	//断链 private ObjectTable

	_HANDLE_TABLE* table= (_HANDLE_TABLE*)((PUCHAR)mypEProcess + get_eprocess_objecttable_offset());

	 MyRemoveEntryList(table->HandleTableList);
```

当然仅仅断链是不够的，因为操作系统还有一张全局句柄表 `PspCidTable`，其中存储了线程和进程，我们还需要把我们的进程从全局句柄表中抹除，可以使用 `ExDestroyHandle`

```
_int64 __fastcall ExDestroyHandle(__int64 a1, __int64 a2, __int64 a3)

{

  unsigned int v6; // ebx



  if ( *(_QWORD *)(a1 + 96) )

    ExpUpdateDebugInfo(a1, KeGetCurrentThread(), a2, 2i64);

  v6 = ExSweepSingleHandle(a1, a3);

  ExpFreeHandleTableEntry(a1, a2, a3);

  return v6;

}
```

```
	if (instance->fn_get_os_build_number() == 7600 || instance->fn_get_os_build_number() == 7601) table_code = ((PHANDLE_TABLE_W7)table)->TableCode;

	else table_code = table->TableCode;



	auto level = table_code & 3;



	if (level == 1) {



		return (PHANDLE_TABLE_ENTRY)(*(uint64_t*)(table_code - 1 + 8 * (u_handle >> 10)) + 4 * (u_handle & 0X3FF));

	}

	else if (level == 2) {



		return (PHANDLE_TABLE_ENTRY)(*(uint64_t*)(*(uint64_t*)(table_code - 2 + 8 * (u_handle >> 19)) + 8 * (u_handle >> 10 & 0X1FF)) + 4 * (u_handle & 0x3ff));



	}

	else {



		return (PHANDLE_TABLE_ENTRY)(table_code + 4 * (u_handle & 0x3ff));



	}
```

**注:** 高版本对全局句柄表有加密

## 实验

隐藏前DebugView 8404

![](https://bbs.kanxue.com/upload/attach/202507/1001878_B9GFGXVMK8Y63C6.jpg)

隐藏后

![](https://bbs.kanxue.com/upload/attach/202507/1001878_P5RWRYZA5ACMYKH.jpg)

需要注意，关闭进程前需要还原全局句柄表，不然会蓝屏；其次在高版本系统上PG增加了对隐藏进程的检测，有概率触发蓝屏，不过没那么快，运气好能挺好几个小时；具体的隐藏进程完整代码可以参考一份开源的:[b9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6P5s2W2Y4k6h3^5I4j5e0q4Q4x3V1k6t1K9h3c8W2f1s2u0G2j5$3g2K6M7H3`.`.](https://bbs.kanxue.com/elink@59cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6P5s2W2Y4k6h3^5I4j5e0q4Q4x3V1k6t1K9h3c8W2f1s2u0G2j5$3g2K6M7H3`.`.)

那么问题来了，如果恶意软件隐藏自己的进程我们如何查询呢？

## 思考

进程是给用户看的，CPU不认识进程，代码也是跑在线程上面的 ，最重要的是cpu线程的调度是基于某些链表进行调度的，如果把cpu调度链表也断了，进程也就跑不起来了，可谓是真正实现了无痕进程隐藏；其次内核中有很多链表都能回溯到进程信息。

本人采用的方法是通过遍历枚举线程去回溯到进程，在 `_ETHREAD` 中 [`_CLIENT_ID`](https://bbs.kanxue.com/elink@730K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2M7r3q4A6i4K6u0W2K9h3#2Q4x3V1k6C8k6i4u0F1k6h3I4K6i4K6u0r3P5o6j5@1i4K6u0r3g2$3W2F1k6r3!0%4M7#2)9J5y4e0t1H3x3e0q4Q4x3V1j5J5x3f1R3J5i4K6t1#2x3U0m8Q4x3U0f1J5z5q4u0f1e0g2)9J5y4e0t1&6i4K6u0r3i4K6g2X3b7@1I4u0c8f1&6f1i4K6g2X3d9f1c8Q4x3X3g2Z5N6r3#2D9)储存了当前线程所属的进程id

```
//0x10 bytes (sizeof)

struct _CLIENT_ID

{

    VOID* UniqueProcess;                                                    //0x0

    VOID* UniqueThread;                                                     //0x8

};
```

查询恶意隐藏进程思路:正常调用API遍历进程并存储表A，暴力枚举线程回溯进程并存储表B，如果B中的进程在A中找不到基本可以判断进程进行了隐藏。

这里为了省事和便于观看就简写代码:

```
	PETHREAD Thread = NULL;

	NTSTATUS status;;

  //直接假设句柄表是三级，问就是图省事

	for (size_t i = 0; i < 1024*1024*512; i++)

	{

		status=PsLookupThreadByThreadId(i, &Thread);

		if (NT_SUCCESS(status))

		{



			ULONG64 UniqueProcess = *(ULONG64*)((PUCHAR)Thread + 0x4c8);

		//	DbgPrintEx(77, 0, "%d\r\n", UniqueProcess);

			if (UniqueProcess == 8404)

			{

				DbgPrintEx(77, 0, "找到隐藏的进程 %d\r\n", UniqueProcess);

				break;



			}

		}



	  }
```

![](https://bbs.kanxue.com/upload/attach/202507/1001878_MZ8GUNN8DJZM8R7.jpg)

总的来说如果对抗同在内核层隐藏进程什么用都没有，有机会下期更新 进程伪装对抗

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-11-26 10:19 被TkBinary编辑，原因：公开技术,造福大家.无需点赞回复.

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
