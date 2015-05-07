## 用户表 用于存储用户识别数据、鉴权数据，以及用户个人信息资料  
// basic info  
id required auto 主键  
createtime required auto  

avatar qiniu serveice key auto  
username case-sensitive contains non-numeric, indexed  
cellphone required, indexed  
password md5-wrapped  

// emailauth  
email lowercase 唯一, indexed  
emailvalidate bool  
// realnameauth  

realname  
idtype 0:personal id  
idnumber 与idtype组合需唯一 indexed  
realnamevalidate bool  

// security  
securitytype required 0:disabled 1:low(no pass) 2:medium(email un-validate) 3:high(email validate)  

---

## 商户表  
// basic info  
id required auto 主键  
merchantlabel required auto 商户标签  
merchantcode required auto 15 ascii 商户代号 主键  
createtime required auto  
userid id 可以关联一个用户，参与中汇用户管理 indexed  

---

## 业务字典表
// basic info  
id required auto 主键  
businesscode required 主键  
businessname required '收单业务'等业务名称  
businesstablename required 实际业务的表名称，例如'收单业务'的表名称  

---

## 商户业务关联表 用于存储商户业务数据的，包括收单业务，或者其他在线收款业务  
// basic info  
id required auto 主键  
createtime required auto  
merchantid required 与商户表中的id关联 indexed  
businesscode required 从业务字典中选区code，将和实际的商户业务详情表一一对应 indexed  
extendid required 与商户业务详情表的id关联 indexed  

---

## 手机收单SDK商户业务详情表  
// basic info  
id required auto 主键  
createtime required auto  
merchantno 中汇商户号 indexed  
type required 0:standard agent 1:big merchant 2:personal  
status required 0:disabled 1: enabled auto 1  

---

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

## 订单表  
// basic info  
id required auto 主键  
createtime required auto  
orderno required auto like 20150423096325154638 主键，年月日时分秒6位循环流水(流水号存内存，线程安全)  
amount required  

// business  
merchantid required  
businesscode 业务code，将和实际的商户业务详情表一一对应  
transno 产生交易时对应实际的商户收款业务关联信息，不同的业务关联方式不一样，这一个字段需要和businesscode联合使用才有意义，收单业务可以是交易数据库id  
ordername the order name   
orderinfo comment  
expired required  
status required 0:init 1:pending(got an ordertype and the keypair info) 2:trading 3:success 4:failure  (atomic read write)  

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

  
  
  
  
  
