## 用户表 用于存储用户数据  
// basic info  
userid required auto 主键  
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

## 时效表 用于验证码、邮箱验证、token验证等各类验证，或者token、session等各类会话  
// basic info  
verifyid required auto 主键  
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
merchantid required auto 主键  
createtime required auto  
merchantno required 主键  
userid required  
type required 0:standard agent 1:big merchant 2:personal  

// business  
allowapi required 0:disallow 1:allow  
appuser redundant info  
agencycode required redundant info  
...  

---

## 订单表  
// basic info  
orderid required auto 主键  
createtime required auto  
orderno required auto like 20150423096325154638 主键  
amount required  

// business  
orderinfo type standard json string  
merchantno required  
expired required  
transid fill in when triggering trading  
status 0:init 1:pending 2:trading 3:success 4:failure  (atomic read write)  
tradeposition coordinate: logitude|latitude  
resultcode string '00'/'55'  
resultic hexstring string  

---

## 服务商appkey  
// basic info  
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

  
  
  
  
  
