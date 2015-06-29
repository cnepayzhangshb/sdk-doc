# 中汇移动支付，订单交易接口规范
> **Beta:**
> 该API仍处于 Beta 阶段

```
1. 该接口适用对象：SDK或中汇自有APP
2. 该接口实现功能：用户的订单检验，交易
3. 该接口调用规范：采用HTTPS请求与中汇的前置POSP进行通信
```
> **注：**
> 文中所有 `<>` 标注的字段，均需根据你的实际情况替换（无需 `<>` 符号，仅作标注之用）
> 文中所有 `:id` 标注的字段，均需根据该资源的实际 `id` 值替换
> 文中所有 `{x|y|...}` 标注的字段，均需根据你的实际情况用其中一个 `x` 或者 `y`（ `|` 分割）替换

## API 接口地址
```
https://payment.vcpos.cn # 生产环境
http://zftpay.21er.net:15080 # 测试环境
```

## 标准请求
```sh
curl -X POST \
    http://zftpay.21er.net:15080/<资源路径> \
    -H "x-order-code:ca8c0353-f53d-493b-95dd-6364565be8d7" \
    -H "x-order-no: 20150303120864123456" \
    # 其他可选参数，参数以键值对呈现...
```
> **注：**
> `x-order-no` 为交易请求的订单号，需携带订单号，作为一笔交易的唯一凭证
> `x-order-code` 为交易请求的令牌，需携带令牌，校验订单的时效

## 标准响应
* 订单校验通过的情况下  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100
x-zftapi-request-id: <uuid>

...body...
```
> **注：**
> `x-zftapi-request-id` 是由API服务创建，并唯一标识这个response的UUID。如果在使用API服务时遇到问题，可以凭借该字段联系技术人员，快速定位问题。 

* 订单校验未通过的情况下  
```
HTTP/1.1 401 Unauthorized
```
* 请求 Method 不被支持的情况下  
```
HTTP/1.1 405 Method Not Allowed
```
* 请求参数不正确的情况下  
```
HTTP/1.1 403 Forbidden
```

## 功能路径列表
| 资源名称     | 路径                                     | Content-Type         |
|-------------|-----------------------------------------|----------------------|
| 设备签到| [/sdk/checkin](#checkin)                      | urlencoded           |
| 消费| [/sdk/sale](#sale)                      | urlencoded           |
| IC脚本回调| [/sdk/saleNotify](#saleNotify)                      | urlencoded           |
| 交易查询| [/sdk/transStatus](#transStatus)                      | urlencoded           |
| 更新设备状态| [/sdk/finishDownloadIcPublic](#finishDownloadIcPublic)                      | urlencoded           |

  
----------------------------------------------------------------------------------
<a id="checkin"></a>
### 设备签到  /sdk/checkin
#### 1\. 获取当前设备相关信息并进行签到
请求：  
```
POST /sdk/checkin HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30

{
    "x-order-code": "eb62561b-bc2b-4690-a9af-b2facdb8fd9a",
    "x-order-no": "20150629131005000063",
    "reqTime": "201506291315",
    "ksnNo": "4880000000004613",
    "appVersion": "设备名.系统名.系统版本号"
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100

{
  "respMsg": "签到成功",
  "respCode": "success",
  "isSuccess": true,
  "respTime": "201506291315",
  "name": "cnepay",
  "macAddress": "20-89-84-D3-A9-30",
  "ksnNo": "4880000000004613",
  "bluetoothName": "fdsfssdg",
  "amount": 1,
  "model": 4ffd5d,
  "DES3": Des3Tpk,
  "needUpdateIC": "true",
  "data": {
    "rids": {
        "0x9F06": "...",
        "0x9F22": "...",
        "0xDF05": "...",
        "0xDF06": "...",
        "0xDF07": "...",
        "0xDF02": "...",
        "0xDF04": "...",
        "0xDF03": "..."
    },
    "aids": {
        "0x9F06": "...",
        "0xDF01": "...",
        "0x9F08": "...",
        "0x9F09": "...",
        "0xDF11": "...",
        "0xDF12": "...",
        "0xDF15": "...",
        "0xDF13": "...",
        "0x9F1B": "...",
        "0xDF16": "...",
        "0xDF17": "...",
        "0xDF14": "...",
        "0xDF18": "..."
    }
  },
  "isBluetooth": "true"
}
```

<a id="sale"></a>
### 消费  /sdk/sale
#### 1\. 验证交易信息并消费 
请求：  
```
POST /sdk/sale HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30

{
    "x-order-code": "20150629131005000063",
    "x-order-no": "eb62561b-bc2b-4690-a9af-b2facdb8fd9a",
    "reqTime": "201506291315",
    "amount": 1,
    "encTracks": "...",
    "ksnNo": "4880000000004613",
    "checksum": 10,
    "appVersion": "设备名.系统名.系统版本号"
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100

{
  "respMsg": "成功",
  "respCode": "success",
  "isSuccess": true,
  "respTime": "201506291315",
  "reqNo": "...",
  "merchantName": "cnepay",
  "merchantNo": "fd-32-yt-45-34",
  "terminalNo": "45453453",
  "operatorNo": "01",
  "resultCode": "00",
  "cardNoWipe": 12,
  "amount": 32,
  "currency": "CNY",
  "issuer": "erwf",
  "voucherNo": "werwer",
  "batchNo": "gdssf",
  "transTime": "2201506291315456354634734734645645",
  "refNo": "...",
  "authNo": "...",
  "script": "f55"
}
```

<a id="saleNotify"></a>
### IC脚本回调  /sdk/saleNotify
#### 1\. 效验交易信息并响应
请求：  
```
POST /sdk/saleNotify HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30

{
    "x-order-code": "eb62561b-bc2b-4690-a9af-b2facdb8fd9a",
    "x-order-no": "20150629131005000063",
    "reqTime": "201506291315",
    "cardNo": "...",
    "origReqTime": "201506291315",
    "origReqNo": "...",
    "origBatchNo": "...",
    "ksnNo": "4880000000004613",
    "icData": "...",
    "origTransTime": "201506291315",
    "origTransType": 1,
    "appVersion": "设备名.系统名.系统版本号"
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100

{
  "reqNo": null,
  "respMsg": "成功",
  "respCode": "success",
  "isSuccess": true,
  "respTime": "201506291315"
}
```

<a id="transStatus"></a>
### 交易查询  /sdk/transStatus
#### 1\. 根据订单编号查询交易信息
请求：  
```
POST /sdk/transStatus HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30

{
    "x-order-code": "eb62561b-bc2b-4690-a9af-b2facdb8fd9a",
    "x-order-no": "20150629131005000063",
    "reqTime": "201506291315",
    "origReqNo": "...",
    "origReqTime": "201506291315",
    "origTransType": "sale",
    "amount": 999
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100

{
  "respMsg": "成功",
  "respCode": "success",
  "isSuccess": true,
  "respTime": "201506291315",
  "reqNo": "...",
  "merchantName": "cnepay",
  "merchantNo": "...",
  "terminalNo": "...",
  "operatorNo": "01",
  "cardNoWipe": "...",
  "amount": 999,
  "currency": "CNY",
  "issuer": "xiaozhang",
  "voucherNo": "...",
  "batchNo": "...",
  "transTime": "201506291315",
  "refNo": "...",
  "authNo": "..."
}
```

<a id="finishDownloadIcPublic"></a>
### 更新设备状态  /sdk/finishDownloadIcPublic
#### 1\. 根据kns编号更新设备状态
请求：  
```
POST /sdk/finishDownloadIcPublic HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30

{
    "x-order-code": "eb62561b-bc2b-4690-a9af-b2facdb8fd9a",
    "x-order-no": "20150629131005000063",
    "reqTime": "201506291315",
    "ksnNo": "4880000000004613"
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100

{
  "respMsg": "已更新状态!",
  "respCode": "success",
  "isSuccess": true,
  "respTime": "201506291315"
}
```
