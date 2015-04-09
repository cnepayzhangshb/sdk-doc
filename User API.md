# 中汇移动支付，用户接口规范
> **Beta:**
> 该API仍处于 Beta 阶段
1. 该接口适用对象：服务商的服务器
2. 该接口实现功能：用户的创建与管理，商户的创建与管理，设备的绑定与管理
3. 该接口调用规范：采用REST规范的HTTPS与中汇的服务器进行通信
> **注：**
> 文中所有 `<>` 标注的字段，均需根据你的实际情况替换（无需 `<>` 符号，仅作标注之用）
> 文中所有 `:id` 标注的字段，均需根据该资源的实际id值替换
> 文中所有 `{x|y|...}` 标注的字段，均需根据你的实际情况用其中一个x或者y（`|` 分割）替换

## API 接口地址
```
https://api.vcpos.cn # 生产环境
http://zftapi.21er.net:15080 # 测试环境
```

## 标准请求方法
```sh
curl -X {GET|POST|PUT|DELETE} \
    http://zftapi.21er.net:15080/<资源路径> \
    -H "Authorization: SIGN <appid>:<signature>" \
    -H "Date: Wed, 8 Apr 2015 15:51 GMT"
    # 其他可选参数...
```

## 授权认证
调用所有接口需要进行授权认证，通过在 HTTP Request Header 中添加 `Authorization` 的方式来进行权限验证：
```
Authorization: SIGN <appid>:<signature>
```
其中 *`appid`* 为服务商获取的 `appid` 身份，*`signature`* 为使用服务商获取的 `appkey` ，根据参数计算出来的签名。
> **注：**
> `appkey` 作为服务商或者api使用者使用接口的特定凭证，是不能对外公开或在客户端存储使用的私有数字凭证

### 签名算法

授权认证过程中的 *`signature`* 参数通过下面的方式获得：
1\. 将HTTP请求头中的数据当成头部数据 <head>
2\. 如果该请求含有 body，将整个 body 数据按照以下规则转换为 <body>
2\.1 如果该请求 `Content-Type` 为 `application/json` 或者 `application/x-www-form-urlencoded` ，则直接将body内容当作<body>
2\.2 如果该请求 `Content-Type` 为 `multipart/form-data`， 我们会在后面的 *附录* 中解释如何转化为 <body>
3\. 将 *`appkey`* 作为 <key>
4\. 将 *`appid`* 作为 <id>
5\. 连接整个数据 <id><head><body><key> 并对其进行MD5签名，得到 *`signature`*
特别地，参数字符串都必须使用 encoding `utf-8` 编码处理

## 基于RESTful的接口资源路径
| 资源名称     | 路径                                     | 可使用的方法         |
|-------------|------------------------------------------|--------------------|
| 用户         | /user                                   | GET / POST         |
| 单一用户     | /user/:id                                | GET / PUT          |
| 用户实名资料  | /user/:id/realname                       | GET / PUT          |
| 商户        | /merchant                                | GET / POST         |
| 单一商户     | /merchant/:id                            | GET / PUT / DELETE |
| 设备        | /device                                  | GET / POST         |
| 单一设备     | /device/<ksnno>                          | GET / PUT / DELETE |