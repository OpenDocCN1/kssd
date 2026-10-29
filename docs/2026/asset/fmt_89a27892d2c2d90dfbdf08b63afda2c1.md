
# [原创][首发]cve-2015-0569 安卓手机提权ROOT漏洞 分析

测试机器：`nexus4` `android版本：4.4` 内核版本`3.4.0`

漏洞介绍：函数进行拷贝时没有对长度进行判断，导致用户可以修改内核栈中值。

漏洞利用：通过修改函数返回地址，来进行提权操作

## 1，首先找到官方的修补代码

修补代码地址：`c8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8$3c8W2j5i4g2J5L8%4u0S2i4K6u0W2L8%4u0Y4i4K6u0r3j5$3N6A6N6q4)9J5c8Y4q4#2K9h3y4Q4x3V1k6D9j5g2)9J5c8Y4m8D9j5i4c8X3L8%4u0E0i4K6u0r3N6X3g2F1k6r3!0J5i4K6u0r3M7h3y4G2L8g2)9J5k6r3!0H3k6h3&6K6L8%4g2J5j5$3g2Q4x3V1k6%4L8r3q4F1i4K6u0r3M7h3y4S2j5$3I4V1i4K6u0V1x3W2)9J5k6e0m8Q4x3V1k6V1K9h3k6X3i4K6u0r3b7@1!0d9c8g2)9J5c8V1S2p5c8q4)9J5c8Y4y4J5j5#2)9J5c8Y4N6D9j5h3&6Q4y4h3k6Z5k6r3c8Q4y4h3k6%4k6i4S2@1i4K6u0W2j5#2)9K6c8X3W2V1i4K6y4p5j5e0l9%4z5h3b7%4x3e0k6T1y4e0b7^5x3e0t1J5x3$3j5H3x3e0j5$3j5K6j5@1y4r3f1&6k6h3x3%4j5K6M7#2j5e0x3I4j5U0l9J5j5H3`.`.

```
@@ -9741,6 +9741,9 @@ int wlan_hdd_set_filter(hdd_context_t *pHddCtx, tpPacketFilterCfg pRequest,
                 hddLog(VOS_TRACE_LEVEL_INFO, "Data Offset %d Data Len %d",
                         pRequest->paramsData[i].dataOffset, pRequest->paramsData[i].dataLength);
+                if ((sizeof(packetFilterSetReq.paramsData[i].compareData)) <
+                           (pRequest->paramsData[i].dataLength))
+                    return -EINVAL;
                memcpy(&packetFilterSetReq.paramsData[i].compareData,
                        pRequest->paramsData[i].compareData, pRequest->paramsData[i].dataLength);
                memcpy(&packetFilterSetReq.paramsData[i].dataMask,
                        pRequest->paramsData[i].dataMask, pRequest->paramsData[i].dataLength);
```

## 2，推测漏洞的产生原理

从官方的修补来看，这里只是添加了一个拷贝长度的限制，那么漏洞产生的原因，应该是`memcpy`拷贝时数据越界有关。

为了证实我的推测，我们看看`packetFilterSetReq` ， `pRequest` 两个结构体是从何而来。

```
int wlan_hdd_set_filter(hdd_context_t *pHddCtx, tpPacketFilterCfg pRequest,
                            tANI_U8 sessionId)
{
    tSirRcvPktFilterCfgType    packetFilterSetReq = {0};
    tSirRcvFltPktClearParam    packetFilterClrReq = {0};
    int i=0;
```

从内核代码上看，`packetFilterSetReq` 是局部变量，而`pRequest`则是作为参数传入。

结合起来看，我推测，漏洞产生的原因应该是，`memcpy`拷贝时越界修改掉了栈的值，覆盖掉了函数的返回地址。

## 3，查找调用漏洞函数的过程

为了利用，以及验证我的推测，现在要找出从漏洞函数到入口函数的整条调用的线

首先从产生漏洞的函数`wlan_hdd_set_filter;`开始找

`wlan_hdd_set_filter`在内核源码`drivers/staging/prima/core/hdd/src/Wlan_hdd_wext.c`文件中

查看`wlan_hdd_set_filter`的调用，发现了两处，分别是同文件下的`wlan_hdd_set_mc_addr_list` 和 `iw_set_packet_filter_params`

首先看 `wlan_hdd_set_mc_addr_list` 函数

```
void wlan_hdd_set_mc_addr_list(hdd_adapter_t *pAdapter, v_U8_t set)
{
    v_U8_t filterAction;
    tPacketFilterCfg request;
    v_U8_t i;
    hdd_context_t *pHddCtx = WLAN_HDD_GET_CTX(pAdapter);
    filterAction = set ? HDD_RCV_FILTER_SET : HDD_RCV_FILTER_CLEAR;
    /*set mulitcast addr list*/
    for (i = 0; i < pAdapter->mc_addr_list.mc_cnt; i++)
    {
        memset(&request, 0, sizeof (tPacketFilterCfg));
        request.filterAction = filterAction;
        request.filterId = i;
        if (set)
        {
            request.numParams = 1;
            request.paramsData[0].protocolLayer = HDD_FILTER_PROTO_TYPE_MAC;
            request.paramsData[0].cmpFlag = HDD_FILTER_CMP_TYPE_EQUAL;
            request.paramsData[0].dataOffset = WLAN_HDD_80211_FRM_DA_OFFSET;
            request.paramsData[0].dataLength = ETH_ALEN; //长度变成固定，不符合触发漏洞的要求
            memcpy(&(request.paramsData[0].compareData[0]),
                    &(pAdapter->mc_addr_list.addr[i][0]), ETH_ALEN);
            /*set mulitcast filters*/
            hddLog(VOS_TRACE_LEVEL_INFO,
                    "%s: %s multicast filter: addr ="
                    MAC_ADDRESS_STR,
                    __func__, set ? "setting" : "clearing",
                    MAC_ADDR_ARRAY(request.paramsData[0].compareData));
        }
        wlan_hdd_set_filter(pHddCtx, &request, pAdapter->sessionId);
    }
    pAdapter->mc_addr_list.isFilterApplied = set ? TRUE : FALSE;
}
```

从代码上来看，再看看这个宏的值

`#define ETH_ALEN                6`

发现这个长度，根本无法达到产生漏洞的条件，因此可以判断出，这个函数不是我们所要找的目标函数。

回过头看另外一个函数`iw_set_packet_filter_params`

```
static int iw_set_packet_filter_params(struct net_device *dev, struct iw_request_info *info,
        union iwreq_data *wrqu, char *extra)
{
    hdd_adapter_t *pAdapter = WLAN_HDD_GET_PRIV_PTR(dev);
    tpPacketFilterCfg pRequest = (tpPacketFilterCfg)wrqu->data.pointer;
return wlan_hdd_set_filter(WLAN_HDD_GET_CTX(pAdapter), pRequest, pAdapter->sessionId)
}
```

发现这个`pRequest`的值是作为参数传入，那么继续找他的调用点，但是发现，却找不到调用点，那么这个函数应该是函数指针调用的，在文件中搜索函数名发现了一个结构体中保存了函数地址

```
static const iw_handler we_private[] = {
```

其中有这一项

```
static const iw_handler we_private[] = {
   [WLAN_PRIV_SET_INT_GET_NONE      - SIOCIWFIRSTPRIV]   = iw_setint_getnone,  //set priv ioctl
   [WLAN_PRIV_SET_NONE_GET_INT      - SIOCIWFIRSTPRIV]   = iw_setnone_getint,  //get priv ioctl
   [WLAN_PRIV_SET_CHAR_GET_NONE     - SIOCIWFIRSTPRIV]   = iw_setchar_getnone, //get priv ioctl
   [WLAN_PRIV_SET_THREE_INT_GET_NONE - SIOCIWFIRSTPRIV]  = iw_set_three_ints_getnone,
   [WLAN_PRIV_GET_CHAR_SET_NONE      - SIOCIWFIRSTPRIV]  = iw_get_char_setnone,
   [WLAN_PRIV_SET_NONE_GET_NONE     - SIOCIWFIRSTPRIV]   = iw_setnone_getnone, //action priv ioctl
   [WLAN_PRIV_SET_VAR_INT_GET_NONE  - SIOCIWFIRSTPRIV]   = iw_set_var_ints_getnone,
   [WLAN_PRIV_ADD_TSPEC             - SIOCIWFIRSTPRIV]   = iw_add_tspec,
   [WLAN_PRIV_DEL_TSPEC             - SIOCIWFIRSTPRIV]   = iw_del_tspec,
   [WLAN_PRIV_GET_TSPEC             - SIOCIWFIRSTPRIV]   = iw_get_tspec,
#ifdef FEATURE_OEM_DATA_SUPPORT
   [WLAN_PRIV_SET_OEM_DATA_REQ - SIOCIWFIRSTPRIV] = iw_set_oem_data_req, //oem data req Specifc
   [WLAN_PRIV_GET_OEM_DATA_RSP - SIOCIWFIRSTPRIV] = iw_get_oem_data_rsp, //oem data req Specifc
#endif
#ifdef FEATURE_WLAN_WAPI
   [WLAN_PRIV_SET_WAPI_MODE             - SIOCIWFIRSTPRIV]  = iw_qcom_set_wapi_mode,
   [WLAN_PRIV_GET_WAPI_MODE             - SIOCIWFIRSTPRIV]  = iw_qcom_get_wapi_mode,
   [WLAN_PRIV_SET_WAPI_ASSOC_INFO       - SIOCIWFIRSTPRIV]  = iw_qcom_set_wapi_assoc_info,
   [WLAN_PRIV_SET_WAPI_KEY              - SIOCIWFIRSTPRIV]  = iw_qcom_set_wapi_key,
   [WLAN_PRIV_SET_WAPI_BKID             - SIOCIWFIRSTPRIV]  = iw_qcom_set_wapi_bkid,
   [WLAN_PRIV_GET_WAPI_BKID             - SIOCIWFIRSTPRIV]  = iw_qcom_get_wapi_bkid,
#endif /* FEATURE_WLAN_WAPI */
#ifdef WLAN_FEATURE_VOWIFI_11R
   [WLAN_PRIV_SET_FTIES                 - SIOCIWFIRSTPRIV]   = iw_set_fties,
#endif
   [WLAN_PRIV_SET_HOST_OFFLOAD          - SIOCIWFIRSTPRIV]   = iw_set_host_offload,
   [WLAN_GET_WLAN_STATISTICS            - SIOCIWFIRSTPRIV]   = iw_get_statistics,
   [WLAN_SET_KEEPALIVE_PARAMS           - SIOCIWFIRSTPRIV]   = iw_set_keepalive_params
#ifdef WLAN_FEATURE_PACKET_FILTERING
   ,
   [WLAN_SET_PACKET_FILTER_PARAMS       - SIOCIWFIRSTPRIV]   = iw_set_packet_filter_params
#endif
#ifdef FEATURE_WLAN_SCAN_PNO
   ,
   [WLAN_SET_PNO                        - SIOCIWFIRSTPRIV]   = iw_set_pno_priv
#endif
   ,
   [WLAN_SET_BAND_CONFIG                - SIOCIWFIRSTPRIV]   = iw_set_band_config,
   [WLAN_PRIV_SET_MCBC_FILTER           - SIOCIWFIRSTPRIV]   = iw_set_dynamic_mcbc_filter,
   [WLAN_PRIV_CLEAR_MCBC_FILTER         - SIOCIWFIRSTPRIV]   = iw_clear_dynamic_mcbc_filter,
   [WLAN_SET_POWER_PARAMS               - SIOCIWFIRSTPRIV]   = iw_set_power_params_priv,
   [WLAN_GET_LINK_SPEED                 - SIOCIWFIRSTPRIV]   = iw_get_linkspeed,
};
```

那么在看看哪里使用 `we_private`

```
const struct iw_handler_def we_handler_def = {
   .num_standard     = sizeof(we_handler) / sizeof(we_handler[0]),
   .num_private      = sizeof(we_private) / sizeof(we_private[0]),
   .num_private_args = sizeof(we_private_args) / sizeof(we_private_args[0]),
```
