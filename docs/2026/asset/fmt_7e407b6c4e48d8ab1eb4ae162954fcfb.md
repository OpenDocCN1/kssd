
# [分享] Python 版本的 Dex 解析器

2 年前，为了了解 Dex 文件格式，顺手写了一个 Dex 解析器，支持 Dex 的反汇编功能。

同时也支持将 Dex 中的 Java 类导出为类似 C/C++ 语言的头文件。

类似下面的头文件：

```cpp
class com_android_internal_os_ProcessStats
{
private:
    const boolean DEBUG = false;
    const int[] LOAD_AVERAGE_FORMAT = NULL;
    const int[] PROCESS_FULL_STATS_FORMAT = NULL;

public:
    const int PROCESS_FULL_STAT_MAJOR_FAULTS = 02;
    const int PROCESS_FULL_STAT_MINOR_FAULTS = 01;
    const int PROCESS_FULL_STAT_STIME = 04;
    const int PROCESS_FULL_STAT_UTIME = 03;
    const int PROCESS_FULL_STAT_VSIZE = 05;

private:
    const int[] PROCESS_STATS_FORMAT = NULL;

public:
    const int PROCESS_STAT_MAJOR_FAULTS = 01;
    const int PROCESS_STAT_MINOR_FAULTS = 00;
    const int PROCESS_STAT_STIME = 03;
    const int PROCESS_STAT_UTIME = 02;

private:
    const int[] SYSTEM_CPU_FORMAT = NULL;
    const String TAG = "ProcessStats";
    const boolean localLOGV = 0;
    const Comparator sLoadComparator = 0;

////////////////////////////////////////////////////////

private:
    long mBaseIdleTime;
    long mBaseIoWaitTime;
    long mBaseIrqTime;
    long mBaseSoftIrqTime;
    long mBaseSystemTime;
    long mBaseUserTime;
    byte[] mBuffer;
    long[] mCpuSpeedTimes;
    long[] mCpuSpeeds;
    int[] mCurPids;
    int[] mCurThreadPids;
    long mCurrentSampleRealTime;
    long mCurrentSampleTime;
    boolean mFirst;

private:
    boolean mIncludeThreads;

private:
    long mLastSampleRealTime;
    long mLastSampleTime;
    float mLoad1;
    float mLoad15;
    float mLoad5;

private:
    float[] mLoadAverageData;
    ArrayList mProcStats;
    long[] mProcessFullStatsData;
    String[] mProcessFullStatsStringData;
    long[] mProcessStatsData;

private:
    long[] mRelCpuSpeedTimes;
    int mRelIdleTime;
    int mRelIoWaitTime;
    int mRelIrqTime;
    int mRelSoftIrqTime;
    int mRelSystemTime;
    int mRelUserTime;

private:
    long[] mSinglePidStatsData;
    long[] mSystemCpuData;
    ArrayList mWorkingProcs;

private:
    boolean mWorkingProcsSorted;

public:
    void <clinit>();

public:
    void <init>(boolean includeThreads);

private:
    int[] collectStats(String statsFile, int parentPid, boolean first, int[] curPids, ArrayList);
    long[] getCpuSpeedTimes(long[] out);
    void getName(ProcessStats$Stats st, String cmdlineFile);
    void printProcessCPU(PrintWriter pw, String prefix, int pid, String label, int totalTime, int user, int system, int iowait, int irq, int softIrq, int minFaults, int majFaults);
    void printRatio(PrintWriter pw, long numerator, long denominator);
    String readFile(String file, char endChar);

//////////////////////virtual method//////////////////////////////////

public:
    virtual void buildWorkingProcs();

public:
    virtual int countStats();
    virtual int countWorkingStats();

public:
    virtual long getCpuTimeForPid(int pid);
    virtual long[] getLastCpuSpeedTimes();

public:
    virtual int getLastIdleTime();
    virtual int getLastIoWaitTime();
    virtual int getLastIrqTime();
    virtual int getLastSoftIrqTime();
    virtual int getLastSystemTime();
    virtual int getLastUserTime();
    virtual ProcessStats$Stats getStats(int index);
    virtual float getTotalCpuPercent();
    virtual ProcessStats$Stats getWorkingStats(int index);

public:
    virtual void init();
    virtual void onLoadChanged(float load1, float load5, float load15);
    virtual int onMeasureProcessName(String name);

public:
    virtual String printCurrentLoad();
    virtual String printCurrentState(long now);

public:
    virtual void update();
}
```

## Dex 解析输出

输出类似以下信息：

```text
void android.media.videoeditor.AudioTrack::extractAudioWaveform(android.media.videoeditor.ExtractAudioWaveformProgressListener)

registers_size      :0000000d:        13
insns_size          :0000009e:       158
debug_info_off      :0089ba78:   9026168
ins_size            :00000002:         2
outs_size           :00000009:         9
tries_size          :00000000:         0
insns               :002c9da0:   2923936
tries               :00000000:         0
handlers            :00000000:         0

002c9da0: 1208                                 |0000: const/4 v8, 0
002c9da2: 54b0 9035                            |0001: iget-object v0, v11, field@mAudioWaveformFilename  //java.lang.String mAudioWaveformFilename
002c9da6: 3900 6300                            |0003: if-nez v0, 0066
002c9daa: 54b0 9935                            |0005: iget-object v0, v11, field@mMANativeHelper  //android.media.videoeditor.MediaArtistNativeHelper mMANativeHelper
002c9dae: 6e10 5342 0000                       |0007: invoke-virtual v0, meth@getProjectPath  //java.lang.String android.media.videoeditor.MediaArtistNativeHelper::getProjectPath()
002c9db4: 0c0a                                 |000a: move-result-object v10
002c9db6: 2200 0e17                            |000b: new-instance v0, type@Ljava/lang/StringBuilder;
002c9dba: 7010 e0eb 0000                       |000d: invoke-direct v0, meth@<init>  //void java.lang.StringBuilder::<init>()
002c9dc0: 6e20 eceb a000                       |0010: invoke-virtual v0, v10, meth@append  //java.lang.StringBuilder java.lang.StringBuilder::append(java.lang.String)
002c9dc6: 0c00                                 |0013: move-result-object v0
002c9dc8: 1a01 6a0e                            |0014: const-string v1, "/audioWaveformFile-"
002c9dcc: 6e20 eceb 1000                       |0016: invoke-virtual v0, v1, meth@append  //java.lang.StringBuilder java.lang.StringBuilder::append(java.lang.String)
002c9dd2: 0c00                                 |0019: move-result-object v0
002c9dd4: 6e10 e741 0b00                       |001a: invoke-virtual v11, meth@getId  //java.lang.String android.media.videoeditor.AudioTrack::getId()
002c9dda: 0c01                                 |001d: move-result-object v1
002c9ddc: 6e20 eceb 1000                       |001e: invoke-virtual v0, v1, meth@append  //java.lang.StringBuilder java.lang.StringBuilder::append(java.lang.String)
002c9de2: 0c00                                 |0021: move-result-object v0
002c9de4: 1a01 2a0e                            |0022: const-string v1, ".dat"
002c9de8: 6e20 eceb 1000                       |0024: invoke-virtual v0, v1, meth@append  //java.lang.StringBuilder java.lang.StringBuilder::append(java.lang.String)
002c9dee: 0c00                                 |0027: move-result-object v0
002c9df0: 6e10 feeb 0000                       |0028: invoke-virtual v0, meth@toString  //java.lang.String java.lang.StringBuilder::toString()
002c9df6: 0c00                                 |002b: move-result-object v0
002c9df8: 2381 471b                            |002c: new-array v1, v8, type@[Ljava/lang/Object;
002c9dfc: 7120 a1eb 1000                       |002e: invoke-static v0, v1, meth@format  //java.lang.String java.lang.String::format(java.lang.String,java.lang.Object[])
002c9e02: 0c03                                 |0031: move-result-object v3
002c9e04: 54b0 9935                            |0032: iget-object v0, v11, field@mMANativeHelper  //android.media.videoeditor.MediaArtistNativeHelper mMANativeHelper
002c9e08: 52b1 8f35                            |0034: iget v1, v11, field@mAudioType  //int mAudioType
```
