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
    -H "Date: Wed, 8 Apr 2015 15:51 GMT"
    -H "x-order-id: 0020150303120864123456" \
    # 其他可选参数，参数以键值对呈现...
```
> **注：**
> `x-order-id` 为交易请求的订单号，需携带订单号，作为一笔交易的唯一凭证

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
| 刷卡设备初始化| [/pay/init](#init)                      | urlencoded           |
| 用户         | [/user](#user)                          | urlencoded        |
| 单一用户     | /user/:id                                | urlencoded        |
| 用户实名资料  | /user/:id/realname                      | urlencoded         |
| 商户        | /merchant                                | GET / POST         |
| 单一商户     | /merchant/:id                            | GET / PUT / DELETE |
| 设备        | /device                                  | GET / POST         |
| 单一设备     | /device/:ksnno                           | GET / PUT / DELETE |

  
----------------------------------------------------------------------------------
<a id="init"></a>
### 刷卡设备初始化  /pay/init
#### 1\. 初始化一个设备以及交易所需数据
请求：  
```
POST /pay/init HTTP/1.1
Host: payment.vcpos.cn
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 30
x-order-id: 0020150303120864123456

ksnNo=600012345678&
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
  "msg": "验证码发送成功",
  "code": 0
}

// or

{
  "msg": "验证码请求次数过多",
  "code": 1
}

// or

{
  "msg": "手机号已被注册",
  "code": 2,
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

<a id="user"></a>
### 用户 /user
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
  "user": {
    "cellphone": "13811122334",
    "username": "vcposuser",
    "realname": "小明"
  }
}
```
响应：  
```
{
  "msg": "用户注册成功",
  "code": 0,
  "user": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "avatar": "",
    "username": "vcposuser",
    "cellphone": "13811190292",
    "realname": "小明"
  }
}
```

#### 2\. 获取指定用户手机号的用户信息
请求：  
```
GET /user?cellphone=13811190292 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
```
响应：  
```
{
  "status": 0,
  "total": 1,
  "index": 0,
  "user": [
    {
      "_id": 3579246,
      "_createtime": 1928739283,
      "avatar": "",
      "username": "vcposuser",
      "cellphone": "13811190292",
      "realname": "小明",
      ""
    }
  ]
}
```