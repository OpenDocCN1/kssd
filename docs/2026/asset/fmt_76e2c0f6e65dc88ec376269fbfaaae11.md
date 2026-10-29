
```
std::__ndk1 *__fastcall MultiMediaComponent::CoreThread::StartThread(MultiMediaComponent::CoreThread *this)

{

  int v1; // r8

  MultiMediaComponent::CoreThread *v2; // r4

  std::__ndk1::__thread_struct *v3; // r5

  std::__ndk1::__thread_struct **v4; // r0

  std::__ndk1 *result; // r0

  const char *v6; // r2

  int v7; // [sp+0h] [bp-18h]

  int v8; // [sp+4h] [bp-14h]

  int v9; // [sp+8h] [bp-10h]



  v9 = v1;

  v2 = this;

  *((_BYTE *)this + 136) = 0;

  v3 = (std::__ndk1::__thread_struct *)operator new(4u);

  std::__ndk1::__thread_struct::__thread_struct(v3);

  v4 = (std::__ndk1::__thread_struct **)operator new(0x10u);

  *v4 = v3;

  v4[1] = (std::__ndk1::__thread_struct *)MultiMediaComponent::CoreThread::WorkingThreadFunc;

  v4[2] = 0;

  v4[3] = v2;

  result = (std::__ndk1 *)pthread_create(

                            (pthread_t *)&v7,

                            0,

                            (void *(*)(void *))std::__ndk1::__thread_proxy<std::__ndk1::tuple<std::__ndk1::unique_ptr<std::__ndk1::__thread_struct,std::__ndk1::default_delete<std::__ndk1::__thread_struct>>,void (MultiMediaComponent::CoreThread::*)(void),MultiMediaComponent::CoreThread*>>,

                            v4);

  ...
```

hook 该函数，再次打印调用堆栈

```
    var base_libvoipChannel = Module.findBaseAddress("libvoipChannel.so");

    //73FFC MultiMediaComponent::Connector::StartCoreThread

    if (base_libvoipChannel) {

        var sub_00073FFC  = base_libvoipChannel.add(0x00073FFC  + 1);

        Interceptor.attach(sub_00073FFC , {

            onEnter: function (args) {

                //this.arg2 = args[2];    //result

                console.log("sub_00073FFC  onEnter:");

                console.log("sub_00073FFC" + ' called from:\n' +

                Thread.backtrace(this.context, Backtracer.ACCURATE)

                    .map(DebugSymbol.fromAddress).join('\n') + '\n');

            }, onLeave: function (ret) {

                console.log("sub_00073FFC  onLeave:");

            }

        });

    }
```

```
sub_00073FFC  onEnter:

sub_00073FFC called from:

0xb7066e3f libvoipChannel.so!_ZN19MultiMediaComponent17CTransportChannel10AddNewConnERKNS_18DirectConnAttrInfoE+0x9a

0xb6fc5ec9 libvoipMain.so!AddNewDirectConns+0x65c

0xb6fb8ae7 libvoipMain.so!Java_com_xxxxx_mm_plugin_voip_model_v2protocal_AddNewDirectConns+0x22

0xc4f9a8bb base.odex!oatexec+0x9018bb



sub_00073FFC  onLeave:
```

找到 Java 层调用的函数 `libvoipMain.so` 的 `Java_com_xxxxx_mm_plugin_voip_model_v2protocal_AddNewDirectConns`。

所以语音过程是这样的：

`Java_com_xxxxx_mm_plugin_voip_model_v2protocal_AddNewDirectConns` ---> 创建线程，建立连接 ---> 语音的获取 ---> 通过 `libvoipComm.so` 的 `MMTinyLib::MMTSockSendto`，UDP 发送数据 ---> 通过 `libvoipComm.so` 的 `MMTinyLib::MMTSockRecvfrom`，UDP 接收数据

# 追溯 `sendto` 的数据产生来源

```
//_ZN9MMTinyLib10MMTIoqueue21DispatchWritableEventEPNS_14AsyncUDPSocketE

int __fastcall MMTinyLib::MMTIoqueue::DispatchWritableEvent(MMTinyLib::CPoller *a1, int *a2) {

    ...

    a3 = a2;

    v6 = (_DWORD *)a3[40];

    v7 = (void *)v6[36];

    buff = v7;

    MMTinyLib::MMTSockSendto(*a3, buff, v44, 0, v4, v47, v14);

    ...

}

//可还原出函数定义

int MMTIoqueue::DispatchWritableEvent( AsyncUDPSocket* pAsyncUDPSocket)
```

在该函数中找到赋值点

```
int __fastcall MMTinyLib::MMTIoqueue::Poll(_DWORD *a1){

    ...

    pAsyncUDPSocket = *((_DWORD *)v14 - 2);

    MMTinyLib::MMTSockRecvfrom(*(struct sockaddr **)pAsyncUDPSocket);

    ...

    MMTinyLib::MMTIoqueue::DispatchWritableEvent((MMTinyLib::CPoller *)v1, *((int **)v14 - 2));

}



//对应汇编

.text:0001B38E                 ADD.W           R0, R4, #0x2C ; this

.text:0001B392                 STR             R0, [SP,#0x2C0+var_2AC]

.text:0001B3A6                 ADD.W           R8, SP, #0x2C0+var_2A0

.text:0001B646                 LDR.W           R1, [R8,#-8]

.text:0001B64A                 MOV             R0, R4

.text:0001B64C                 BLX             j__ZN9MMTinyLib10MMTIoqueue21DispatchWritableEventEPNS_14AsyncUDPSocketE ; MMTinyLib::MMTIoqueue::DispatchWritableEvent(MMTinyLib::AsyncUDPSocket *)
```

综上，得到对应结构体偏移

```
class MMTIoqueue {

    char[8];

    AsyncUDPSocket* pAsyncUDPSocket;

    ...

}



struct AsyncUDPSocket {

    char[40*sizeof(int)];

    UserData* pUserData;

    ...

}



struct UserData {

    char[36*sizeof(int)];

    char* buff;

    ...

}
```

找到 `MMTinyLib::MMTIoqueue` 中疑似构造 `AsyncUDPSocket` 结构体的函数

```
.text:0001A930 ; MMTinyLib::MMTIoqueue::RecreateUdpSocket(MMTinyLib::AsyncUDPSocket *)



.text:0001A1BC ; int __fastcall MMTinyLib::MMTIoqueue::CreateUdpSocket
```

hook 试试

```
var base_libvoipComm = Module.findBaseAddress("libvoipComm.so");



    if (base_libvoipComm) {

        ////1A930   MMTinyLib::MMTIoqueue::RecreateUdpSocket(MMTinyLib::AsyncUDPSocket *)

        var sub_0001A930  = base_libvoipComm.add(0x0001A930  + 1);

        Interceptor.attach(sub_0001A930 , {

            onEnter: function (args) {

                //this.arg2 = args[2];    //result

                console.log("RecreateUdpSocket  onEnter:");

                console.log("RecreateUdpSocket" + ' called from:\n' +

                Thread.backtrace(this.context, Backtracer.ACCURATE)

                    .map(DebugSymbol.fromAddress).join('\n') + '\n');

            }, onLeave: function (ret) {

                console.log("RecreateUdpSocket  onLeave:");

            }

        });



        //1A1BC  int __fastcall MMTinyLib::MMTIoqueue::CreateUdpSocket

        var sub_0001A1BC  = base_libvoipComm.add(0x0001A1BC  + 1);

        Interceptor.attach(sub_0001A1BC , {

            onEnter: function (args) {

                //this.arg2 = args[2];    //result

                console.log("CreateUdpSocket  onEnter:");

                console.log("CreateUdpSocket" + ' called from:\n' +

                Thread.backtrace(this.context, Backtracer.ACCURATE)

                    .map(DebugSymbol.fromAddress).join('\n') + '\n');

            }, onLeave: function (ret) {

                console.log("CreateUdpSocket  onLeave:");

            }

        });

    }
```

得到结果

```
[Pixel::com.xxxxx.mm]-> CreateUdpSocket  onEnter:

CreateUdpSocket called from:

0xb70a8cb9 libvoipChannel.so!_ZN19MultiMediaComponent10CoreThread19CreateCoreSocketNewERN9MMTinyLib20mmt_sockaddr_storageEPNS_21CoreUdpSocketCallbackEPvj+0xdc

0xb707a1d9 libvoipChannel.so!_ZN19MultiMediaComponent8RelayMgr16GetNewCoreSocketEPNS_9RelayRoomEiNSt6__ndk112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEE+0x140

0xb707ad8b libvoipChannel.so!_ZN19MultiMediaComponent8RelayMgr12RelayCheckInEPNS_9RelayRoomE+0x10e

0xb707aae9 libvoipChannel.so!_ZN19MultiMediaComponent8RelayMgr13OpenRelayConnEPNS_8ConnInfoEPKNS_19conn_private_data_tE+0x7c

0xb70676db libvoipChannel.so!_ZN19MultiMediaComponent17CTransportChannel10AddNewConnERKNS_17RelayConnAttrInfoE+0x126

0xb6fc5785 libvoipMain.so!AddNewRelayConns+0x7c0

0xb6fb8aa7 libvoipMain.so!Java_com_xxxxx_mm_plugin_voip_model_v2protocal_AddNewRelayConns+0x22

0xc4f9a8bb base.odex!oatexec+0x9018bb



CreateUdpSocket  onLeave:
```

所以构造 `AsyncUDPSocket` 结构体的是 `CreateUdpSocket`。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-7-13 15:03 被咸鱼炒白菜编辑 ，原因：
