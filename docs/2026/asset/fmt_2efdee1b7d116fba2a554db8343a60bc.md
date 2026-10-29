
```
//匹配字符串，不区分大小写，参数：源字符串，匹配的字符串
public static Boolean matcherChar(String strName,String matChar){
    Pattern pattern = Pattern.compile(matChar, Pattern.CASE_INSENSITIVE);
    Matcher matcher = pattern.matcher(strName);
    return matcher.find();
}
```

#### (5)服务器版本和服务器名获取

##### 细粒度获取：

```
/*
 * 匹配包头信息和服务器html信息，获取服务器名和服务器版本
 */
public static void strName(WebInfo wi,String strcontent) throws IOException {
    if(!strcontent.isEmpty()){
        BufferedReader br = new BufferedReader(new InputStreamReader(new ByteArrayInputStream(strcontent.getBytes(Charset.forName("utf-8")))));
        String line;
        String[] items;
        StringBuffer strbuf = new StringBuffer();
        while ((line = br.readLine())!=null){
            String reg = "Server:\\s(\\D*)(\\s|\\/|\\*)(.*)";
            String reg1 = "Server:\\s(\\D*)";
            String[] result = RegCheck(line,reg);
            if(result.length>0){
                //System.out.println("result.length:"+result.length);
                wi.http_server = result[1];
                wi.http_server_version = result[3];
                if(wi.http_server==null){
                    wi.http_server = "";
                }
                if(wi.http_server.contains("*")){
                    wi.http_server = "";
                    wi.isServerCrypto = true;
                }
            }
            if(line.contains("Set-Cookie")){
                String[] arr = line.split(":",2);
                for(String str:arr){
                    if(!str.equals("Set-Cookie")){
                        wi.set_Cookie = str;
                    }
                }
            }
            if(line.contains("X-Powered-By")){
                String[] arr = line.split(":",2);
                for(String str:arr){
                    if(!str.equals("X-Powered-By")){
                        wi.X_Powered_By = str;
                    }
                }
            }
            //检测主体中的服务器版本信息
            if (matcherChar(line, "Apache")&&wi.http_server.isEmpty()){
                wi.http_server = "Apache";
            }
            if (matcherChar(line, "Nginx")&&wi.http_server.isEmpty()){
                wi.http_server = "Nginx";
            }
            if (matcherChar(line, "Lighttpd ")&&wi.http_server.isEmpty()){
                wi.http_server = "Lighttpd";
            }
            if (matcherChar(line, "IIS ")&&wi.http_server.isEmpty()){
                wi.http_server = "IIS ";
            }
            if (matcherChar(line, "WebSphere")&&wi.http_server.isEmpty()){
                wi.http_server = "WebSphere";
            }
            if (matcherChar(line, "Weblogic")&&wi.http_server.isEmpty()){
                wi.http_server = "WebSphere";
            }
            if (matcherChar(line, "Boa")&&wi.http_server.isEmpty()){
                wi.http_server = "Boa";
            }
            if (matcherChar(line, "Jigsaw")&&wi.http_server.isEmpty()){
                wi.http_server = "Jigsaw";
            }
        }
        if(wi.http_server.isEmpty()){
            marcherServer(wi,strcontent);
        }
        br.close();
    }
}
```

##### 版本顺序识别：

```
/*
 * 处理加密服务器，无法获得版本
 * 匹配服务器顺序,根据主流服务器的响应头顺序来识别
 * 例如：Apache 顺序：Http->Date->Server
 */
public static void marcherServer(WebInfo wi,String strcontent) throws IOException {
    if(!strcontent.isEmpty()) {
        BufferedReader br = new BufferedReader(new InputStreamReader(new ByteArrayInputStream(strcontent.getBytes(Charset.forName("utf-8")))));
        String line;
        int i=0;
        StringBuffer strbuf = new StringBuffer();
        while ((line = br.readLine()) != null) {
            i=i+1;
            if(line.contains("HTTP")&&i==1){
                // i=2;
            }
            if(line.contains("Date")&&i==2){
                wi.http_server="Apache";
                break;
            }
            if(line.contains("Server")&&i==2){
                //i=3;
            }
            if(line.contains("Expires")&&i==3){
                wi.http_server="IIS";
                break;
            }
            if(line.contains("Data")&&i==3){
                //i=4;
            }
            if(line.contains("Content-Type")&&i==4){
                wi.http_server="Enterprise ";
                break;
            }
            if(line.contains("Content-length")&&i==4){
                wi.http_server="SunONE";
                break;
            }
            if(i==5){
                break;
            }
        }
    }
}
```

#### (6)服务器语言获取

##### 细粒度获取：

```
//用于检测服务器语言的指纹信息,输入参数：webInfo对象
public static void LanguageCheck(WebInfo wi) {
    if(wi.set_Cookie.contains("PHPSSIONID")&&wi.language.isEmpty()){
        wi.language = "PHP";
    }
    if(wi.set_Cookie.contains("JSESSIONID")&&wi.language.isEmpty()){
        wi.language = "JAVA";
    }
    if(wi.X_Powered_By.contains("ASP.NET")||wi.set_Cookie.contains("ASPSESS")||wi.set_Cookie.contains("ASP.NET")&&wi.language.isEmpty()){
        wi.language = "ASP.NET";
        if(wi.http_server.isEmpty()){
            wi.http_server = "IIS";
        }
    }
    if(wi.X_Powered_By.contains("JBoss")&&wi.language.isEmpty()){
        wi.language = "JAVA";
        if(wi.http_server.isEmpty()){
            wi.http_server = "JBOSS";
        }
    }
    if(wi.X_Powered_By.contains("Servlet")&&wi.language.isEmpty()){
        wi.language = "JAVA";
        if(wi.http_server.isEmpty()){
            wi.http_server = "SERVLET";
        }
    }
    if(wi.X_Powered_By.contains("Next.js")&&wi.language.isEmpty()){
        wi.language = "NODEJS";
    }
    if(wi.X_Powered_By.contains("Express")&&wi.language.isEmpty()){
        wi.language = "NODEJS";
    }
    if(wi.X_Powered_By.contains("Dragonfly CMS")&&wi.language.isEmpty()){
        wi.language = "PHP";
    }
    if(wi.X_Powered_By.contains("PHP")&&wi.language.isEmpty()){
        wi.language = "PHP";
    }
    if(wi.X_Powered_By.startsWith("JSF")&&wi.language.isEmpty()){
        wi.language = "JAVA";
        if(wi.http_server.isEmpty()){
            wi.http_server = "SERVLET";
        }
    }
    if(wi.X_Powered_By.startsWith("WP")&&wi.language.isEmpty()){
        wi.language = "PHP";
    }
    if(wi.X_Powered_By.startsWith("enduro")&&wi.language.isEmpty()){
        wi.language = "NODEJS";
    }
}
```

##### 粗粒度获取：
