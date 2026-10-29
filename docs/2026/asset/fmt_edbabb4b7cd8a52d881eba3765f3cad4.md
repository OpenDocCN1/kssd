
```java
public class Cpuinfo {

    public Cpuinfo(LoadPackageParam sharePkgParam) {
        FakeCPUFile(sharePkgParam);
    }
    
    public static boolean CreatDataCpu(Context context) {
        String str = "/data/data/" + context.getPackageName() + "/cpuinfo";
        try {
            AssetManager assets = context.getAssets();
            InputStream open = assets.open("cpuinfo");
            OutputStream fileOutputStream = new FileOutputStream(str);
            writeValue(open, fileOutputStream);
            open.close();
            fileOutputStream.flush();
            fileOutputStream.close();
            Sendfile(str);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    
    private static void writeValue(InputStream inputStream, OutputStream outputStream) {
        try {
            byte[] bArr = new byte[AccessibilityNodeInfoCompat.ACTION_NEXT_HTML_ELEMENT];
            while (true) {
                int read = inputStream.read(bArr);
                if (read != -1) {
                    outputStream.write(bArr, 0, read);
                } else {
                    return;
                }
            }
        } catch (Exception e) {}
    }
    
    private static void Sendfile(String str) {
        IOException e;
        try {
            DataOutputStream dataOutputStream = new DataOutputStream(Runtime.getRuntime().exec("su").getOutputStream());
            try {
                dataOutputStream.writeBytes("mkdir /sdcard/Test/\n");
                dataOutputStream.flush();
                dataOutputStream.writeBytes("chmod 777 /sdcard/Test/\n");
                dataOutputStream.flush();
                dataOutputStream.writeBytes("cp " + str + " /sdcard/Test/\n");
                dataOutputStream.flush();
                dataOutputStream.writeBytes("chmod 444 /sdcard/Test/cpuinfo\n");
                dataOutputStream.flush();
                dataOutputStream.writeBytes("rm " + str + "\n");
                dataOutputStream.flush();
                dataOutputStream.close();
                if (new File("/sdcard/Test/cpuinfo").exists()) {
                    return;
                }
                throw new IOException();
            } catch (IOException e2) {
                e = e2;
                DataOutputStream dataOutputStream2 = dataOutputStream;
                e.printStackTrace();
            }
        } catch (IOException e3) {
            e = e3;
            e.printStackTrace();
        }
    }
    
    public void FakeCPUFile(LoadPackageParam loadPkgParam) { 
        try {         
            XposedBridge.hookAllConstructors(File.class, new XC_MethodHook() {      
                @Override             
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {    
                    super.beforeHookedMethod(param);   
                    if (param.args.length == 1) {       
                        if (param.args[0].equals("/proc/cpuinfo")) {
                            param.args[0] = "/sdcard/Test/cpuinfo";      
                        }  
                    } else if (param.args.length == 2 && !File.class.isInstance(param.args[0])) {  
                        String str = "";    
                        while (i < 2) {  
                            String stringBuilder;    
                            if (param.args[i] != null) {  
                                if (param.args[i].equals("/proc/cpuinfo")) {  
                                    param.args[i] = "/sdcard/Test/cpuinfo";   
                                }              
                                stringBuilder = new StringBuilder(String.valueOf(str)).append(param.args[i]).append(":").toString();   
                            } else {   
                                stringBuilder = str;          
                            }      
                            i++;                    
                            str = stringBuilder;        
                        }          
                    }     
                }      
            });
            XposedHelpers.findAndHookMethod("java.lang.Runtime", loadPkgParam.classLoader, "exec", String[].class, String[].class, File.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                    if (param.args.length == 1) {
                        if (param.args[0].equals("/proc/cpuinfo")) {
                            param.args[0] = "/sdcard/Test/cpuinfo";
                        }
                    } else if (param.args.length == 2 && !File.class.isInstance(param.args[0])) {
                        int i = 0;
                        String str = "";
                        while (i < 2) {
                            String stringBuilder;
                            if (param.args[i] != null) {
                                if (param.args[i].equals("/proc/cpuinfo")) {
                                    param.args[i] = "/sdcard/Test/cpuinfo";
                                }
                                stringBuilder = new StringBuilder(String.valueOf(str)).append(param.args[i]).append(":").toString();
                            } else {
                                stringBuilder = str;
                            }
                            i++;
                            str = stringBuilder;
                        }
                    }
                }
            });
        } catch (Exception e) {
            XposedBridge.log("Fake CPUFile - 1 ERROR: " + e.getMessage());
        }
        try {
            XposedBridge.hookMethod(XposedHelpers.findConstructorExact(ProcessBuilder.class, new Class[] { String[].class }), new XC_MethodHook() {
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                    if (param.args[0] != null) {
                        String[] strArr = (String[]) param.args[0];
                        String str = "";
                        for (String str2 : strArr) {
                            str = new StringBuilder(String.valueOf(str)).append(str2).append(":").toString();
                            if (str2 == "/proc/cpuinfo") {
                                strArr[1] = "/sdcard/Test/cpuinfo";
                            }
                        }
                        param.args[0] = strArr;
                    }
                }
            });
        } catch (Exception e) {
            XposedBridge.log("Fake CPUFile - 2 ERROR: " + e.getMessage());
        }
        try {
            XposedHelpers.findAndHookMethod("java.util.regex.Pattern", loadPkgParam.classLoader, "matcher", CharSequence.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                    if (param.args.length == 1) {
                        if (param.args[0].equals("/proc/cpuinfo")) {
                            param.args[0] = "/sdcard/Test/cpuinfo";
                        }
                    }
                }
            });
        } catch (Exception e) {}
    }
}
```
接下来就是自定义文件了

```java
String filePath = "/sdcard/Test/";
String fileName = "cpuinfo";
Mnt.makeFilePath(filePath, fileName);

String strFilePath = filePath+fileName;
String strContent = "Processor    : ARMv7 Processor rev 0 (v7l)" + "\r\n";
String strContent2 = "processor    : 0" + "\r\n";
String strContent3 = "BogoMIPS    : 38.40";
String strContent4 = "" + "\r\n";
String strContent5 = "" + "\r\n";
String strContent6 = "processor    : 1"+ "\r\n";
String strContent7 = "BogoMIPS    : 38.40"+ "\r\n";
String strContent8 = ""+ "\r\n";
String strContent9 = "Features    : swp half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt"+ "\r\n";
String strContent10 = "CPU implementer    : 0x51"+ "\r\n";
String strContent11 = "CPU architecture: 7"+ "\r\n";
String strContent12 = "CPU variant    : 0x2"+ "\r\n";
String strContent13 = "CPU part    : 0x06f"+ "\r\n";
String strContent14 = "CPU revision    : 0"+ "\r\n";
String strContent15 = ""+ "\r\n";
String strContent16 = "Hardware   :MT123     \n";
String strContent17 = "Revision    : 000d"+ "\r\n";
String strContent18 = "Serial        : 0000088900004e4b"+ "\r\n";
try {
    File file = new File(strFilePath);
    if (!file.exists()) {
        file.getParentFile().mkdirs();
        file.createNewFile();
    }
    if(file.exists()){
        file.delete();
    }
    RandomAccessFile raf = new RandomAccessFile(file, "rwd");
    raf.seek(file.length());
    raf.write(strContent.getBytes());
    raf.write(strContent2.getBytes());
    raf.write(strContent3.getBytes());
    raf.write(strContent4.getBytes());
    raf.write(strContent5.getBytes());
    raf.write(strContent6.getBytes());
    raf.write(strContent7.getBytes());
    raf.write(strContent8.getBytes());
    raf.write(strContent9.getBytes());
    raf.write(strContent10.getBytes());
    raf.write(strContent11.getBytes());
    raf.write(strContent12.getBytes());
    raf.write(strContent13.getBytes());
    raf.write(strContent14.getBytes());
    raf.write(strContent15.getBytes());
    raf.write(strContent16.getBytes());
    raf.write(strContent17.getBytes());
    raf.write(strContent18.getBytes());
    raf.close();
} catch (Exception e) {
    Log.e("TestFile", "Error on write File:" + e);
}
```
```java
public static File makeFilePath(String filePath, String fileName) {
    File file = null;
    makeRootDirectory(filePath);
    try {
        file = new File(filePath + fileName);
        if (!file.exists()) {
            file.createNewFile();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return file;
}

public static void makeRootDirectory(String filePath) {
    File file = null;
    try {
        file = new File(filePath);
        if (!file.exists()) {
            file.mkdir();
        }
    } catch (Exception e) {
        Log.i("error:", e+"");
    }
}
```
