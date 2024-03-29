## 对接参数
 - APPID (初始化SDK必要的参数)
 - APPKEY (初始化SDK必要的参数)
 - APPKEY_SERVER (用于后台充值回调、登陆验证等)

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
 5. 额外字段说明
  ```diff
  + 由于部分厂商要求增加代金券(coupon)和福利币(coin)金额回传，单位(元)；
  + 为了不影响已接入的游戏验签，新增POST的额外字段请不要参与验签
  ````
 
 ###### 游戏方验签过程
 
 1. 获取到所有的POST参数
 2. 取出account_id、amount、appid、order_no、out_trade_no、create_time、nonce字段（不含sign及上述第5点中的额外字段）
 3. 按照上述第1步到第3步进行排列组装字符串后计算签名
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

## SDK登陆验证

 - 登录验证接口地址：https://sdk.kepan365.com/account/verify
 - 请求方式POST 
 - Content-Type：application/x-www-form-urlencoded

请求参数：
参数 | 类型 | 说明
---|---|---
appid | int | 对接游戏的APPID参数
account_id | int | 登录用户ID
nonce | string | 8位随机字符串
timestamp | int | 10位unix时间戳
sign | string | 签名

sign 计算规则，假设数据如下

```
appid = 10000
account_id = 123
nonce = V6ySGphc
timestamp = 1554722177
```

 1. ksort排序后再组成字符串得到：
```
account_id=123&appid=10000&nonce=V6ySGphc&timestamp=1554722177
```

 2. 在最后拼接APPKEY_SERVER参数(假设为 O5SuutGvaWqRwB9r5F3froQFUJmZsm9E)：
```
account_id=123&appid=10000&nonce=V6ySGphc&timestamp=1554722177O5SuutGvaWqRwB9r5F3froQFUJmZsm9E
```

 3. 计算签名 md5后转为大写
 ```
   sign = strtoupper(md5(str)) = CDEE69AC322CCA5EBA8F4F8E97D3AE5A
 ```

 4. 最终POST数据
 ```
 appid = 10000
 account_id = 123
 nonce = V6ySGphc
 timestamp = 1554722177
 sign = CDEE69AC322CCA5EBA8F4F8E97D3AE5A
 ```


 5. 返回参数(code 200表示成功, 新增中宣部实名认证pi返回)：
```json
{
    "code": 200,
    "msg": "success",
    "data": {"pi": "1hpfml09b57f3f8185f8cb5094ea3f26278efb"}
}

```
