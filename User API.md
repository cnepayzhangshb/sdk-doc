# 中汇移动支付，用户接口规范
> **Beta:**
> 该API仍处于 Beta 阶段

```
1. 该接口适用对象：服务商的服务器
2. 该接口实现功能：用户的创建与管理，商户的创建与管理，设备的绑定与管理
3. 该接口调用规范：采用REST规范的HTTPS请求与中汇的服务器进行通信
4. 该接口调用权限：暂未对资源独立控制
```
> **注：**
> 文中所有 `<>` 标注的字段，均需根据你的实际情况替换（无需 `<>` 符号，仅作标注之用）
> 文中所有 `:id` 标注的字段，均需根据该资源的实际 `id` 值替换
> 文中所有 `{x|y|...}` 标注的字段，均需根据你的实际情况用其中一个 `x` 或者 `y`（ `|` 分割）替换

## API 接口地址
```
https://api.vcpos.cn # 生产环境
http://zftapi.21er.net:15080 # 测试环境
```

## 标准请求
```sh
curl -X {GET|POST|PUT|DELETE} \
    http://zftapi.21er.net:15080/<资源路径> \
    -H "Authorization: SIGN <appid>:<signature>" \
    -H "Date: Wed, 8 Apr 2015 15:51 GMT"
    # 其他可选参数...
```

## 标准响应
* 签名校验通过的情况下  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100
x-zftapi-request-id: <uuid>

{
  "code": "00",
  "msg": "成功",
  "data": {}
}

...body...
```
> **注：**
> `x-zftapi-request-id` 是由API服务创建，并唯一标识这个response的UUID。如果在使用API服务时遇到问题，可以凭借该字段联系技术人员，快速定位问题。 

* 签名校验未通过的情况下  
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

## 时间校验
调用所有接口需要进行Date时间校验，通过HTTP Request Header 中添加 `Date` 的方式进行校验：
```
Date: Thu, 09 Apr 2015 11:36:53 GMT
```
*`Date`* 的检验主要验证服务器时间与调用者的时间是否基本一致，时间差需要控制在前后3分钟内。例如，它与订单创建者设定的订单有效期相关，这个有效期可以使用相对时间，也可以使用服务器绝对时间

## 授权认证
调用所有接口需要进行授权认证，通过在 HTTP Request Header 中添加 `Authorization` 的方式来进行权限验证：  
```
Authorization: SIGN <appid>:<signature>
```
其中 *`appid`* 为服务商获取的 **appid** 身份，*`signature`* 为使用服务商获取的 **appkey**，根据参数计算出来的签名。
> **注：**
> **`appkey`** 作为服务商或者API使用者使用接口的特定凭证，是不能对外公开或在客户端存储使用的，是一个私有数字凭证

### 签名算法

授权认证过程中的 *`signature`* 参数通过下面的方式获得：
> 1. 将HTTP请求头中的数据当成头部数据 `<head>`
> 2. 如果该请求含有 body，将整个 body 数据按照以下规则转换为 `<body>`  
> **a)** 如果该请求 `Content-Type` 为 `application/json` 或者 `application/x-www-form-urlencoded` ，则直接将body内容当作 `<body>`  
> **b)** 如果该请求 `Content-Type` 为 `multipart/form-data`， 我们会在后面的 **附录** 中解释如何转化为 `<body>`  
> 3. 将 *`appkey`* 作为 `<key>`
> 4. 将 *`appid`* 作为 `<id>`
> 5. 连接整个数据 `<id><head><body><key>` 并对其进行MD5签名，得到 *`signature`*

特别地，参数字符串都必须使用 Encoding `utf-8` 编码处理


## <a name="content-title"></a>RESTful资源路径
| 资源名称     | 路径                                     | 可使用的方法         |
|-------------|-----------------------------------------|--------------------|
| 注册验证码    | [/registercode](#regcode)               | POST               |
| 用户         | [/user](#user)                          | POST               |
| 手机用户     | [/user/:cell-{phoneno}](#cellphone)      | GET                |
| 单一用户     | /user/:id                                | GET / PUT          |
| 用户实名资料  | /user/:id/realname                      | GET / PUT          |
| 商户        | [/merchant](#merchant)                   | POST               |
| 单一商户     | [/merchant/:idOrCode](#merchant1)        | GET / PUT / DELETE |
| 商户业务     | [/merchant/:idOrCode/business](#business)| GET / PUT          |
| 设备        | /device                                  | GET / POST         |
| 单一设备     | /device/:ksnno                           | GET / PUT / DELETE |
| 订单创建     | [/order](#order)                         | POST               |
| 单一订单     | [/order/:orderNo](#order1)               | GET / PUT          |

  
----------------------------------------------------------------------------------

### <a name="regcode"></a>注册验证码  /registercode
#### 1\. 通过注册手机号发送注册验证码
请求：  
```
POST /registercode HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 30

{
  "cellphone": "13811190292",
  "action": "create",
  "timeout": 180000
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
x-zftapi-request-id: 81e1d8cf-60b1-426c-bd96-8e39c9f57235

{
  "msg": "验证码发送成功",
  "code": "00"
}

// or

{
  "msg": "验证码请求次数过多",
  "code": "Q4"
}

// or

{
  "msg": "手机号已被注册",
  "code": "P2",
}
```
#### 2\. 通过接收验证码验证手机号，以获取registertoken，用于注册
请求：  
```
POST /registercode HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 30

{
  "cellphone": "13811190292",
  "action": "verify",
  "registercode": "4028"
}
```
响应：  
```
{
  "msg": "验证码验证成功",
  "code": 0,
  "registertoken": "1aef1dacf4424acaf1e3"
}

// or

{
  "msg": "验证码错误",
  "code": 1,
}

// or

{
  "msg": "验证码已过期",
  "code": 2,
}
```
##### [返回目录↑](#content-title)
### <a name="user"></a>用户 /user
#### 1\. 通过registertoken创建用户
请求：  
```
POST /user HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 100

{
  "registertoken": "1aef1dacf4424acaf1e3",
  "cellphone": "13811122334",
  "username": "vcposuser",
  "realname": "小明"
}
```
响应：  
```
{
  "msg": "用户注册成功",
  "code": "00",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "avatar": "",
    "username": "vcposuser",
    "cellphone": "13811190292",
    "realname": "小明"
  }
}
```
##### [返回目录↑](#content-title)
### <a name="cellphone"></a>手机用户 /user/:cell-{phoneno}
#### 1\. 获取指定用户手机号的用户信息
请求：  
```
GET /user/cell-13811190292 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
```
响应：  
```
{
  "code": "00",
  "total": 1,
  "index": 0,
  "data": [
    {
      "_id": 3579246,
      "_createtime": 1928739283,
      "avatar": "",
      "username": "vcposuser",
      "cellphone": "13811190292",
      "realname": "小明"
    }
  ]
}
```
##### [返回目录↑](#content-title)
### <a name="merchant"></a>商户 /merchant
#### 1\. 创建一个商户
请求：  
```
POST /merchant HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 10

{
  "label": "小刚的马家堡火锅店"
}

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "label": "小刚的马家堡火锅店",
    "merchantcode": "M12130000000001",
    "userid": null
  }
}
```
##### [返回目录↑](#content-title)
### <a name="merchant1"></a>单一商户 /merchant/:idOrCode
> `idOrCode` 指内容为纯 `id` 或者 `code-{merchantCode}`

#### 1\. 获取商户信息
请求：  
```
GET /merchant/3579246 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "label": "小刚的马家堡火锅店",
    "merchantcode": "M12131",
    "userid": null
  }
}
```
##### [返回目录↑](#content-title)
### <a name="business"></a>商户业务 /merchant/:idOrCode/business
> `idOrCode` 指内容为纯 `id` 或者 `code-{merchantCode}`

#### 1\. 获取商户业务数据
请求：  
```
GET /merchant/3579246/business HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "SJSDSDK": {
      "_id": 123414512,
      "_createtime": 1928739283,
      "businessname": "手机收单SDK",
      "businesscode": "SJSDSDK",
      "merchantno": "500100002000120",
      "type": 2,
      "status": 1
    }
  }
}
```
#### 2\. 修改商户业务数据
请求：  
```
PUT /merchant/3579246/business HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 25

{
  "add": {   // 表示新增业务
    "SJSDSDK": {  // 手机收单业务
      "merchantno": "500100002000120",
      "type": 2, // 0表示标准进件商户，1表示大商户，2表示个人商户
      "status": 1 // 0表示禁用，1表示启用
    }
  }
}

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "SJSDSDK": {  // 手机收单业务
      "merchantno": "500100002000120",
      "type": 2, // 0表示标准进件商户，1表示大商户，2表示个人商户
      "status": 1 // 0表示禁用，1表示启用
    }
  }
}
```
##### [返回目录↑](#content-title)
### <a name="order"></a>订单创建 /order
#### 1\. 创建一个订单
请求：  
```
POST /order HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 100

{
  "amount": 12300,  // ￥123.00
  "currency": "CNY", // default
  "merchantcode": "M12130000000001",
  "ordername": "",
  "orderinfo": {
    "url": null,
    "orderdetail": null
  },
  "expired": "2d5h2m10s" // or numeric time acquired like new Date().getTime()
}

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "orderno": "20150504091240100001",
    "amount": 12300,  // ￥123.00
    "currency": "CNY", // default
    "merchantcode": "M12130000000001",
    "businesscode": ""
    "expired": 1928739998,
    "status": 0
  }
}
```
##### [返回目录↑](#content-title)