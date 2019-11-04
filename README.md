## 参数说明
 - APPID (初始化SDK必要的参数)
 - APPKEY (初始化SDK必要的参数)
 - APPKEY_SERVER (用于服务端回调通讯验签密钥)

## 服务端充值回调

当用户支付成功后我方会向游戏方充值回调地址POST以下参数

参数 | 类型 | 说明
---|---|---
account_id | int | 我方玩家唯一标识
amount | float | 订单总金额(元)
appid | int | 我方提供的APPID
order_no | string | 我方订单号
out_trade_no | string | 游戏方订单号
create_time | int | 订单创建时间戳
nonce | string | 随机字符串
sign | string | 签名

 * 签名过程 假设数据如下：
```
account_id = 123  
amount = 5.88  
appid = 888  
order_no = 1904081028419740  
out_trade_no = 2019040822001498171028043287  
create_time = 1554722177  
nonce = V6ySGphc6fYmFzCL
```
 1. 对以上参数进行ksort排序后再组成字符串得到：
```
account_id=123&amount=5.88&appid=888&create_time=1554722177&nonce=V6ySGphc6fYmFzCL&order_no=1904081028419740&out_trade_no=2019040822001498171028043287
```
 2. 拼接APPKEY_SERVER(假设为 tDMlAG4gDePQVloFvBp4mYKBSjgp6aOz)得到：
 ```
account_id=123&amount=5.88&appid=888&create_time=1554722177&nonce=V6ySGphc6fYmFzCL&order_no=1904081028419740&out_trade_no=2019040822001498171028043287tDMlAG4gDePQVloFvBp4mYKBSjgp6aOz
 ```
 3. 计算签名 md5后转为大写
 ```
   strtoupper(md5(str)) = F61CFB54B79AC349D23C11133A203BC5
 ```
 4. 最终POST数据
 ```
 account_id = 123  
 amount = 5.88  
 appid = 888  
 order_no = 1904081028419740  
 out_trade_no = 2019040822001498171028043287  
 create_time = 1554722177  
 nonce = V6ySGphc6fYmFzCL
 sign = F61CFB54B79AC349D23C11133A203BC5
 ```
 
 * 游戏方验签过程
 
 1. 获取到所有的POST参数
 2. 取出sign字段
 3. 对所有的参数(sign除外)按照上述ksort的方式进行计算签名
 4. 对比sign
 5. 签名校验通过 进行发货等其他逻辑处理
 
 * 游戏方返回：
 
 游戏方处理成功后 `success`字符串  
 如果游戏方反馈给我方的字符不是`success`这7个字符，我方服务器会不断重发通知，直到超过24小时22分钟。  
 一般情况下，25小时以内完成8次通知（通知的间隔频率一般是：4m,10m,10m,1h,2h,6h,15h）
 
 * 特别注意：
  ```diff
  - 因为同一订单可能会重复发送通知，所以请保证物品只发送一次。如果订单已完成，则直接返回 success
  ````


