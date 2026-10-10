

# 简单云查杀客户端源代码

> 原文：[`www.pediy.com/kssd/pediy12/132163.html`](https://www.pediy.com/kssd/pediy12/132163.html)

代码:

```

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Net;
using System.IO;

//
//+++++++++++++++++++++++++++++++++++++++++++++
//
// CloudScanner 
//
// Written by Spark.   nkspark\_at\_gmail.com
// 
//          2011.03.26
//+++++++++++++++++++++++++++++++++++++++++++++
//

namespace CloudScanner
{
    class Program
    {
        static string GetScanResult(string strFileName)
        {
            //VirusTotal 的查询请求链接。
            string strQueryURL = "http://www.virustotal.com/search.html"
            string strMD5;

            FileStream fsSource = new FileStream(strFileName, FileMode.Open, FileAccess.ReadWrite);

            byte\[\] bIn = new byte\[fsSource.Length\];
            fsSource.Read(bIn, 0, (int)fsSource.Length);
            fsSource.Close();

            System.Security.Cryptography.MD5 md5 = System.Security.Cryptography.MD5.Create();
            strMD5 = BitConverter.ToString(md5.ComputeHash(bIn)).Replace("-", "");
            strMD5 = "chain=" + strMD5;



            WebClient webClient = new WebClient();
            webClient.Encoding = System.Text.Encoding.UTF8;
            System.Net.ServicePointManager.Expect100Continue = false;


            return webClient.UploadString(strQueryURL, strMD5);
        }

        //从 VirusTotal 的返回结果中提取病毒名称。
        static string ExtractResult(string strPage)
        {
            string strAVProductName;
            string strVersion;
            string strUpdate;
            string strVirusName = "";

            
            int iStartPos = -1;
            int iEndPos = -1;

            iStartPos = strPage.IndexOf("id=\\"tablaMotores\\"");
            if (iStartPos == -1) return "";           
            iEndPos = strPage.IndexOf("</table>", iStartPos);
            strPage = strPage.Substring(iStartPos, iEndPos-iStartPos);

            TagInfo ti = new TagInfo();

            string strTRLine;
            ti.TagName = "tr";
            ti.GetPos(strPage);
            strTRLine = ti.GetContext(strPage);
            strPage = strPage.Substring(ti.EndPos);           

            while (strTRLine != "")
            {                
                ti.TagName = "tr";
                ti.GetPos(strPage);
                strTRLine = ti.GetContext(strPage);
                if (strTRLine == "") return "";
                strPage = strPage.Substring(ti.EndPos);

                //AV Product
                ti.TagName = "td";
                ti.GetPos(strTRLine);
                strAVProductName = ti.GetContext(strTRLine);
                

                //Version
                strTRLine = strTRLine.Substring(ti.EndPos);
                ti.TagName = "td";
                ti.GetPos(strTRLine);
                strVersion = ti.GetContext(strTRLine);
                
                //Last Update
                strTRLine = strTRLine.Substring(ti.EndPos);
                ti.TagName = "td";
                ti.GetPos(strTRLine);
                strUpdate = ti.GetContext(strTRLine);
                
                //Result
                strTRLine = strTRLine.Substring(ti.EndPos);
                ti.TagName = "td";
                ti.GetPos(strTRLine);
                strVirusName = ti.GetContext(strTRLine);
                if (strVirusName != "-")
                    return strVirusName;                        
            }

            return ""; 
        }
        static void Main(string\[\] args)
        {
 
            string strResult;
            string strFileName="";
            long lTime;

            if (args.Length < 1)
            {
                Console.WriteLine("Usage: CloudScanner.exe FileName");
                return;
            }

            strFileName = args\[0\];

            if (!File.Exists(strFileName))
            {
                Console.WriteLine("文件不存在！");
                return;
            }

            lTime = DateTime.Now.Ticks;            
            strResult = GetScanResult(strFileName);

            strResult = ExtractResult(strResult);
            lTime = (DateTime.Now.Ticks - lTime) / TimeSpan.TicksPerSecond;

            if(strResult != "")               
                Console.WriteLine("发现病毒 " + strResult + "，扫描用时" + lTime.ToString() + "秒。");
            else               
                Console.WriteLine("正常文件，扫描用时" + lTime.ToString() + "秒。");

        }

 

    }

    public class TagInfo
    {
        public string TagName;
        public int StartPos = -1;
        public int EndPos = -1;

        public void GetPos(string strIn)
        {


            string strScriptStart = "<" + TagName;
            string strScriptEnd = "/" + TagName + ">";

            StartPos = strIn.IndexOf(strScriptStart);
            if (StartPos != -1)
                EndPos = strIn.IndexOf(strScriptEnd, StartPos);
        }

        public string GetContext(string strIn)
        {

            string strTemp = "";
            string strScriptStart = ">";
            string strScriptEnd = "<";
            int iScriptStart;
            int iScriptEnd;

            if (StartPos != -1 && EndPos != -1)
            {
                iScriptStart = strIn.IndexOf(strScriptStart, StartPos) + 1;
                iScriptEnd = strIn.LastIndexOf(strScriptEnd, EndPos);
                strTemp = strIn.Substring(iScriptStart, iScriptEnd - iScriptStart);
            }
            return strTemp;
        }

    }

}

```

思路非常简单，将本地文件的 MD5 值发送到 VirusTotal，分析返回结果，如果查找到病毒名称，则报出。对上述代码稍加改动，可实现更多云查杀客户端功能。  
  
1、由于 VirusTotal 在云端集成了 43 款不同的杀毒引擎，所以此客户端虽然简单，但查杀率极高。  
2、直接使用链接查询方式，而非 VirusTotal 提供的 API，所以不受查询次数限制（限制为 5 分钟 20 次）。  
  
附件增加了文件夹递归扫描和病毒文件删除功能。  
![](img/3e65abbb6e0568fe793b3cf8b9f956cd.jpg)

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || CloudScanner.rar  
 |