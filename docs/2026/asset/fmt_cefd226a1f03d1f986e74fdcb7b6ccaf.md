
```delphi
   FOnAskForKey : TOnAskForKey;
   FOnCorrupt : TNotifyEvent;
   procedure WriteStr(S: String; Stream: TStream);
   function ReadStr(Stream: TStream): string;
   Procedure LoadFromStreamNor(ms : TStream);
   Procedure SaveToStreamNor(ms : TStream);
   function CheckGood(ms: TStream): boolean;
   procedure FoundCorrupt;
  public
   StreamList : TStringList;
   Constructor Create(AOWner : TComponent); override;
   Destructor Destroy; override;
   Procedure LoadFromFile(const Filename : string);
   Procedure SaveToFile(const Filename : string);
   Procedure AddStream(Const ID : string; Source : TStream);
   Procedure RemoveStream(Const ID : String);
   Procedure LoadFromStream(ms : TStream);
   Procedure SaveToStream(ms : TStream);
   Procedure GetStream(Const ID : string; Dest : TStream);
   Procedure ClearStreams;
  published
   property Compressed : boolean read FCompressed write FCompressed;
   property Encrypted : boolean read FEncrypted write FEncrypted;
   Property Key : string read FKey write FKey;
   Property OnAskForKey : TOnAskForKey read FOnAskForKey write FOnAskForKey;
   property OnCorrupt : TNotifyEvent read FOnCorrupt write FOnCorrupt;
  end;
```

`THKStreams`数据保存在`streamList`结构中，像这样 “BAT”bat data; “Files”FileData …..

完了之后，汇编就非常容易阅读了。可以轻而易举地写出 delphi 解压源码。(解密代码请见`\hks\UnpackQuickBatch.drp`)

`THKStreams`这个类来自第三方，可以从网络上下载到。Quick Batch 的加密解密都是用了它。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`delphi.zip`](javascript:void(0);)

  （510.65kb，128次下载）

- [`project9.part1.rar`](javascript:void(0);)

  （900.00kb，226次下载）

- [`project9.part2.rar`](javascript:void(0);)

  （394.08kb，165次下载）

- [`test idb.zip`](javascript:void(0);)

  （561.06kb，202次下载）
