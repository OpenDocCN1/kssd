
从该函数中不难发现里面包含了许多抓包中的参数信息，有理由怀疑程序就是在此处进行组包并请求信息！

# 0x05 hook getUnifiedSign

分析到这了，先 hook 看看 `getUnifiedSign` 函数请求和返回都是些啥，hook 代码：

```
function main() {
  Java.perform(function () {
    var InnerSignImpl = Java.use("mtopsdk.security.InnerSignImpl");
    InnerSignImpl["getUnifiedSign"].implementation = function (params, ext, appKey, authCode, useWua, requestId) {
      console.log('\ngetUnifiedSign is called' + ', ' + '\nparams: \n' + params + '\n' + 'ext: \n' + ext + '\n' + 'appKey: \n' + appKey + '\n' + 'authCode: \n' + authCode + '\n' + 'useWua: \n' + useWua + '\n' + 'requestId: \n' + requestId);
      var ret = this.getUnifiedSign(params, ext, appKey, authCode, useWua, requestId);
      console.log('getUnifiedSign ret value is ' + ret);
      return ret;
    };
  });
}
setImmediate(main)
```

搜索关键字：黑丝。

结果：

> getUnifiedSign is called,
> params:
> {data={"activeSearch":false,"bizFrom":"home","disableHierarchicalSort":0,"forceUseInputKeyword":false,"forceUseTppRepair":false,"fromFilter":false,"fromKits":false,"fromLeaf":false,"fromShade":false,"fromSuggest":false,"keyword":"黑丝","pageNumber":1,"resultListLastIndex":0,"rowsPerPage":10,"searchReqFromActivatePagePart":"searchButton","searchReqFromPage":"xyHome","searchTabType":"SEARCH_TAB_MAIN","shadeBucketNum":-1,"suggestBucketNum":27}, deviceId=AnlVbDHuTb2u0LWMPSEZxO4CdI4PNLcEAjN85BBOipB9, sid=null, uid=null, x-features=27, appKey=21407387, api=mtop.taobao.idle.search.glue, lat=0, lng=0, utdid=Y6mM0d1XDnwDAAZc4d8Tk60B, extdata=openappkey=DEFAULT_AUTH, ttid=231200@fleamarket_android_7.4.70, t=1672065081, v=8.0}
> ext:
> {pageId=, pageName=}
> appKey:
> 21407387
> authCode:
> null
> useWua:
> false
> requestId:
> r_342
> getUnifiedSign ret value is
> {x-sgext=JAc6QkgEOGWLbN43MhWbokILcghxC2EIdAJ0A2EJdghhCW4PbgpuC24LbgtuC24LbgtuC24MdRZzFnQNbgx1FnIWchZyFnIWchZyFnIWchZyFnIWcxZzDncWcw53FnEWchZyFnACYQt0DXAKdwxyC3UZcgghW3IKcgtxXHUOIA17CGEJdgphA2EJdhlyCnINYQlhC2ELYQthC2ELYQhhC2EOcRlxGXUZcw5hCmEKYQphCmEKYRknGSRfYQphXCdfJwInGXIKcgpy, x-umt=2QMB7AlLPMcI7wKFTpWcJNO9Tq3ykFES, x-mini-wua=HHnB_LsOm2MbDDQX8pocsAv844s/AJ3eeRpQBvQ0ruCym5E4E9z73i+wqyWX+kYoOCLjd0M+Af0hvQxs8NJyeS1/+qAd+g60eGM0Y7snvKtTeCvVhBnNESbEFrPu+orzouidZjoRxOAXN2Cpe1icpSFPKMA==, x-sign=azU7Bc002xAAJAe6xWI/sfnl+vxS1Be0CqzNIjAW/Hwc8p+i1dC0d4rAl5xTWTXVcWTzU1+mIZmU3sPAV41DwKbwpS1llAe0BZQHtA}

参数都在这了，这就好办了，rpc 调用就能解决，注意这边的 data 数据是进行了 url 编码的，需进行进一步转化。

rpc 调用代码可参考我写的：[原创] 安卓协议逆向 cxdx 分析与实现 一文。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
