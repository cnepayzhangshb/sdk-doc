## 用户表 用于存储用户识别数据、鉴权数据，以及用户个人信息资料  
// basic info  
id required auto 主键  
createtime required auto  

avatar qiniu serveice key auto  
username case-sensitive contains non-numeric  
cellphone required 主键  
password md5-wrapped  

// emailauth  
email lowercase 唯一  
emailvalidate bool  
// realnameauth  

realname  
idtype 0:personal id  
idnumber 唯一  
realnamevalidate bool  

// security  
securitytype required 0:disabled 1:low(no pass) 2:medium(email un-validate) 3:high(email validate)  

---

## 用户业务关联表 用于存储用户业务数据的，包括收单业务，信用卡还款业务，转账业务，水电煤缴费业务，每个业务限制每个用户只能关联一次  
// basic info  
id required auto 主键  
createtime required auto  
userid required  
businessname required 'bankcardMerchant:收单业务'等业务名称，和业务表一一对应  
businessid required 业务关联id  

---

## 收单业务表信息  
// basic info  
id required auto 主键  
createtime required auto  
maxmerchantcount required auto 1 最大商户数  
status 0:disabled 1: enabled auto 1  


## 时效表 用于验证码、邮箱验证、token验证等各类验证，或者token、session等各类会话  
// basic info  
id required auto 主键  
createtime required auto  
situation required enum 'register'/'forgetpass'/'emailvalidate'/'session'   (可分表字段)  
key required '13811100000'/'13811122222'/'sss@sss.ss'/'13811190292'  
value required '0000'/'1111'/'1bad3d2ba22'/'12312479129827391273021'  
expired required  
createcount required auto 0  
failurecount required auto 0  

---

## 商户表  
// basic info  
id required auto 主键  
createtime required auto  
merchantno required 索引，等同于实际商户的商户号，商户号关联  
businessid required 关联业务表id  
type required 0:standard agent 1:big merchant 2:personal  

// business info  
allowapi required 0:disallow 1:allow  
// appuser redundant info  
// agencycode required redundant info  
...  

---

## 订单表  
// basic info  
id required auto 主键  
createtime required auto  
orderno required auto like 20150423096325154638 主键，年月日时分秒6位循环流水(流水号存内存，线程安全)  
amount required  

// business  
ordertype 'cardpayment'/'zhifubao'/'weixin'  
transid fill in when triggering trading  
ordername the order name   
orderinfo comment  
expired required  
status 0:init 1:pending(got an ordertype and the keypair info) 2:trading 3:success 4:failure  (atomic read write)  

---

## 服务商appkey  
// basic info  
id required auto
appid required uuid 主键  
createtime required auto  
appkey required  
appuser required 'PM1010'/'android' should be corresponding to the level 1 agencycode  
appgroup required 'agencyCode'/'mobile'  

// business  
status 0:off 1:on  
notifyurl for receiving notification  
notifyurlhost only host  
bindip  
bindport  

---

## 交易表  
// remain nearly the same  

---

## 图片服务器数据库 暂定

  
  
  
  
  
