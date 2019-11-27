
# 介绍

为了让第三方应用简单方便地接入区块链，拥有使用区块链的能力，Elephant钱包开放接口给第三方应用，让第三方应用可以通过Elephant完成支付数字币、使用DID身份、向区块链写入数据。

# 调用方式

第三方可以通过特殊的Scheme “elaphant://”来唤起和调用Elephant，也可以将调用的URI编码成二维码，让Elephant主动扫描二维码来执行相应操作。
scheme包括三部分：指令，必选参数和指令的参数。

## scheme格式：
```
elaphant://<command></RequiredParameters...>[/CommandParameters...]
```

### 指令包括：
- [身份鉴权identity](#identity指令)
- [支付elapay](#elapay指令)
- [DPoS投票eladposvote](#eladposvote指令)
- [签名授权sign](#sign指令)
- [setproperty](#setproperty指令)
- [getproperty](#getproperty指令)
- [请求创建多签钱包multicreate](#multicreate指令)
- [多签交易请求签名multitx](#multitx指令)

### 必选参数：
任何第三方向Elephant发起调用都必须提供关于App相关信息的参数，用于验证调用者身份和数据可信性。

字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
AppName                 | String & URLEncode | 必选 | 你的应用名称; 长度不超过64个字节
AppID                   | String     | 必选 | 由开发者DID对AppName签名后产生的签名字符串，作为该App的ID。
Description             | String & URLEncode | 必选 | 对于App和当前操作的描述，方便用户判断是否授权
DID                     | String     | 必选 | 开发者DID
PublicKey               | String     | 必选 | DID的公钥


**验证方法：**
1. 验证AppID的值是否是DID对AppName的签名，使用PublicKey进行验证。
2. 验证DID与PublicKey是否对应，可以由PublicKey推导出DID。


### 回调和返回参数：
字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
CallbackUrl             | String & URLEncode | 可选 | 请求结果的回调URL
ReturnUrl               | String & URLEncode | 可选 | 请求返回的页面URL
Target                  | String | 可选 | 可以在Elephant内部webview或者在外部浏览器打开Return URL。"internal"表示用内部webview打开，"browser"表示用外部浏览器打开。

** 其中标注URLEncode的字段，在请求时必须使用UTF8字符集和URL Encode方法编码，Elephant处理时会使用URL Decode解码。

Elephant提供两种方式返回信息给第三方：[Callback URL]和[Return URL]。
它们的区别是：
- Callback URL：Elephant采用HTTP Post方法或等同方法，在后台异步地返回信息。
- Return URL：Elephant采用HTTP Get方法或等同方法，在前台跳转、打开对应的URL页面。

第三方应用可以自己选择哪一种或者两种。

如果第三方应用同时面向多位用户服务，可以在Callback URL或者Return URL里附加追踪参数，Elephant在返回的时候也会保持原始参数设置来回调第三方App。

# identity指令

## 概要
当第三方应用程序需要验证用户DID身份、需要个人信息时，它可以向Elephant 发起请求，用户授权以后，Elephant将第三方请求的信息和签名一起返回给第三方。

一个典型的用例是登录：

1. 第三方生成请求内容，包括请求者信息和随机数。
2. 获得用户授权后，对随机数和DID信息进行签名。
3. 将信息和签名通过回调和返回两种方式通知第三方应用。

第三方请求时要附带随机数，这个随机数应该是一个不会重复的、不可推测的随机内容，通过以防止攻击者进行重放攻击——使用事先准备好的签名内容来欺骗第三方，冒用其他用户的身份。

当收到Elephant返回的信息时，必须验证返回的随机数是否正确，如果不正确则无效。


## 步骤1, 第三方发起请求

**请求格式：**
```
elaphant://identity?
	AppName=<AppName>&
	AppID=<AppID>&
	Description=<Description>&
	RandomNumber=<RandomNumber>&
	DID=<DeveloperDID>&
	PublicKey=<DID PublicKey>
	[&RequestInfo=<Required Fields>]
	[&CallbackUrl=<Callback URL>]
	[&ReturnUrl=<Return URL>&Target=<"internal" | "browser">]

```
**指令参数：**

字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
RandomNumber            | String & URLEncode | 必选 | 采用“请求/应答”方式验证用户是否是DID持有者，用户必须使用DID对Random Number进行签名，并将它返回给请求者。
RequestInfo**           | String & URLEncode | 可选 | 请求获取用户DID的信息列表



** RequestInfo字段是第三方需要的信息字段，以逗号分隔，可请求的字段包括
```
Nickname, ELAAddress, BTCAddress, BCHAddress, ETHAddress, ELAETHSCAddress, IOEXAddress, Email, PhoneNumber, ChineseIDCard
```

** 其中标注URLEncode的字段，在请求时必须使用UTF8字符集和URL Encode方法编码，Elephant处理时会使用URL Decode解码。

RequestInfo示例
```
RequestInfo="Nickname,ELAAddress,Email,PhoneNumber"
```

**identity请求示例：**
```
elaphant://identity?
AppID=e104892291bebf6047681f415165067b8e167e18920291a3c791ca3daec6020fc4c09640c790f2138f50b17a029029acb74ddd9406ff041611d3fbde9f5eb542&
AppName=FooBar&
Description=FooBar&
DID=iWwPFLS8Gp18LqM6GJgLz5QrPwjBqqshxF&
PublicKey=03684d22dd6de91cc5b504ff0499eff919bd2a1507826475d5c6b314217ea96417&
RequestInfo=BTCAddress%2CETHAddress%2CEmail%2CPhoneNumber
CallbackUrl=http%3A%2F%2Fbing.com
```

也可以将URI生成二维码，用户通过Elephant扫描二维码完成请求。


## 步骤2, 返回信息
在用户授权以后，Elephant将用户DID相关信息按第三方请求的方式返回。
- 对于Callback方式，目前仅支持HTTP Post方法，将按照请求时Callback URL的地址发起Post调用，并在Body里附带返回信息。
- 对于Return方式，目前仅支持HTTP Get方法，将按照请求时Return URL的地址在内部Webview或者外部浏览器打开页面。

**返回信息格式：**

返回信息包括两部分：Data（数据）和Sign（签名）。Data是一个JSON对象序列化后的字符串。Sign是使用DID对Data字符串内容的签名。

返回格式示例
```
Data="{
	\"DID\":<DID>,
	\"PublicKey\":<PublicKey>,
	\"RandomNumber\":<RandomNumber>,
	\"Nickname\":[Nickname],
	\"ELAAddress\":[ELAAddress],
	\"BTCAddress\":[BTCAddress],
	\"ETHAddress\":[ETHAddress],
	\"BCHAddress\":[BCHAddress],
	\"IOEXAddress\":[IOEXAddress],
	\"Email\":[Email],
	\"PhoneNumber\":[PhoneNumber],
	\"ChineseIDCard\":[ChineseIDCard]
}"

Sign="<signature>"
```

**返回参数说明：**


字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
DID             | String    | 必选 | 用户的DID
PublicKey       | String    | 必选 | DID的公钥
RandomNumber    | String    | 必选 | 第三方请求的随机数
Nickname        | String    | 可选 | 用户的昵称
ELAAddress      | String    | 可选 | 用户的ELA钱包地址
BTCAddress      | String    | 可选 | 用户的BTC钱包地址
BCHAddress      | String    | 可选 | 用户的BCH钱包地址
ETHAddress      | String    | 可选 | 用户的ETH钱包地址
IOEXAddress     | String    | 可选 | 用户的IOEX钱包地址
Email           | String    | 可选 | 用户的电子邮件
PhoneNumber     | String    | 可选 | 用户的电话号码
ChineseIDCard   | String    | 可选 | 以JSON格式返回用户的中国身份证信息
Signature       | String    | 必选 | 针对返回信息的签名

**HTTP Callback示例**：

Callback是以Post方式返回信息，在Body中使用JSON格式,包含Data和Sign两个字段。
```
[Method]: POST
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
[Body]:
{
    "Data":"{\"BTCAddress\":\"1Fmb9MLJ74ShWG5WcJWqMTDBfPTJPnQzVP\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"ELAAddress\":\"EHeSKgaFNxPJiLScvase437D2pM9isw4pX\",\"Email\":\"\",\"ETHAddress\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"NickName\":\"中文测试\",\"PhoneNumber\":\"\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}",
    "Sign": "02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176"
}

```

**HTTP Return示例**：

Return是以Get方式返回信息，在URL的参数中附加Data和Sign参数，请第三方在其ReturnURL中避免使用这两个参数名，以免冲突。
```
[Method]: GET
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&Data={\"BTCAddress\":\"1Fmb9MLJ74ShWG5WcJWqMTDBfPTJPnQzVP\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"ELAAddress\":\"EHeSKgaFNxPJiLScvase437D2pM9isw4pX\",\"Email\":\"\",\"ETHAddress\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"NickName\":\"中文测试\",\"PhoneNumber\":\"\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}&Sign= 02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176

```

## 步骤3, 解析和验证
1. 将参数Data的字符串内容转换为JSON对象；
2. 从JSON对象中恢复PublicKey、DID字段内容；
3. 验证PublicKey与DID是否匹配；
4. 验证Data字符串内容、PublicKey和Sign的签名内容是否匹配；
5. 验证返回的RandomNumber字段与“步骤1”请求时的RandomNumber值是否一致；

# elapay指令

## 概要
当第三方应用程序需要使用数字币支付时，它可以向Elephant发起请求，用户授权以后会转账到请求的地址，这个过程类似于第三方请求支付宝和Paypal完成支付一样。

- 步骤1：第三方发起请求，包括收款地址，金额，数字币种名称和订单唯一标识符。
- 步骤2：Elephant显示转账支付页面，用户确认后发起转账。
- 步骤3：向第三方返回转账交易的TXID，第三方可以通过它查询交易状态。


## 第三方发起支付请求
**请求格式：**
```
elaphant://elapay?
	AppName=<AppName>&
	AppID=<AppID>&
	Description=<Description>&
	DID=<DeveloperDID>&
	PublicKey=<DID PublicKey>&
	CoinName=<Coin or token name>&
	Amount=<Amount>&
	ReceivingAddress=<Receiving Address>
	[&CallbackUrl=<Callback URL>]
	[&ReturnUrl=<Return URL>&Target=<"internal" | "browser">]

```
**指令参数：**

字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
OrderID                 | String     | 可选 | 订单编号，用于第三方追踪支付结果，如果提供这个参数，它会作为交易的备注上链
CoinName                | String     | 必选 | 数字币种名称
Amount                  | Number     | 必选 | 转账的金额
ReceivingAddress          | String     | 必选 | 用于收款的地址


**elapay请求示例：**
```
elaphant://elapay?
DID=iWwPFLS8Gp18LqM6GJgLz5QrPwjBqqshxF&
AppID=e104892291bebf6047681f415165067b8e167e18920291a3c791ca3daec6020fc4c09640c790f2138f50b17a029029acb74ddd9406ff041611d3fbde9f5eb542&
AppName=FooBar&
Description=FooBar&
PublicKey=03684d22dd6de91cc5b504ff0499eff919bd2a1507826475d5c6b314217ea96417&
OrderID=354199ds9213k1f&
CoinName=ELA&
Amount=1.234&
ReceivingAddress=EXRNP8Pm3KQR7EeLXFGRtfLdkBaeZsMLyy&
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
```

## 返回支付信息

**返回信息格式：**

无论是Callback或者Return哪种方式，都以字符串格式返回信息，字符串包含两个参数"TXID"和"OrderID"。
返回信息包括两部分：TXID（交易hash）和OrderID（第三方订单号）。TXID是本次交易的Hash，OrderID是第三方请求时传入的OrderID值，便于第三方跟踪支付过程。
```
TXID="bb14a5898ca81ee0207ab9178df1d1e4617be3aa5e9dfec9e185d41ff13786fc"&OrderID=“354199ds9213k1f"
```


**返回参数说明：**

字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
TXID             | String    | 必选 | 转账交易的TXID
OrderID          | String    | 必选 | 对应请求时的OrderID


**HTTP Callback回调示例:**

Callback是以Post方式返回信息，在Body中使用JSON格式,包含TXID和OrderID两个字段。
```
Method: POST
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
Body:
{
"TXID":"895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718",
"OrderID":"354199ds9213k1f"
}
```

**HTTP Return返回示例:**

Return是以Get方式返回信息，在URL的参数中附加TXID和OrderID参数，请第三方在其ReturnURL中避免使用这两个参数名，以免冲突。
```
Method: GET
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&TXID=895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718&OrderID=354199ds9213k1f
```

您可以通 `blockchain.elastos.org` 确认交易状态.
<https://blockchain.elastos.org/tx/895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718>

# eladposvote指令

## 概要
第三方应用可以通过投票指令发起DPoS投票，用户授权以后，将使用全部ELA余额发起投票交易。

- 步骤1：第三方发起请求，包括候选人公钥列表。
- 步骤2：Elephant显示投票信息，用户确认后发起投票交易。
- 步骤3：向第三方返回投票交易的TXID，第三方可以通过它查询交易状态。


## 第三方发起投票请求
**请求格式：**

```
elaphant://eladposvote?
	AppName=<AppName>&
	AppID=<AppID>&
	Description=<Description>&
	DID=<DeveloperDID>&
	PublicKey=<DID PublicKey>&
	CandidatePublicKeys=<Supernode public key list>
	[&CallbackUrl=<Callback URL>]
	[&ReturnUrl=<Return URL>&Target=<"internal" | "browser">]

```
**请求参数：**


字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
CandidatePublicKeys                 | String & URLEncode     | 必选 | 候选人的公钥列表，用逗号分隔


**eladposvote请求示例：**

```
elaphant://eladposvote?
AppName=FooBar&
Description=FooBar&
DID=iWwPFLS8Gp18LqM6GJgLz5QrPwjBqqshxF&
PublicKey=03684d22dd6de91cc5b504ff0499eff919bd2a1507826475d5c6b314217ea96417&
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
CandidatePublicKeys=03ef5f8b0534c82aa4db218f7cead278124efc0411e4ca38b6131954a58e8ae3c0%2C03ef5f8b0534c82aa4db218f7cead278124efc0411e4ca38b6131954a58e8ae3c0

```

## 返回投票信息

**返回信息格式：**

无论是Callback或者Return哪种方式，都以字符串格式返回信息。
```
TXID="bb14a5898ca81ee0207ab9178df1d1e4617be3aa5e9dfec9e185d41ff13786fc"
```


**返回参数说明：**


字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
TXID             | String    | 必选 | 投票交易的TXID



**HTTP Callback回调示例:**

Callback以Post方法返回信息，在Body里以JSON格式返回信息。
```
Method: POST
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
Body:
{
"TXID":"895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718"
}
```

**HTTP Return返回示例:**
Return以Get方法返回信息，投票交易对应的TXID以参数方式返回，请第三方避免在ReturnURL参数中使用TXID参数名，以免冲突。
```
Method: GET
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&TXID=895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718
```

您可以通 `blockchain.elastos.org` 确认交易状态.
<https://blockchain.elastos.org/tx/895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718>


# sign指令

## 概要
当第三方应用程序需要验证用户DID身份、需要个人信息时，它可以向Elephant 发起请求，用户授权以后，Elephant将构造一个签名证书返回给第三方。

比如第三方请求用户对写对文章进行签名：

1. 第三方将文章的Hash作为签名请求的内容发送给Elephant。
2. Elephant显示请求者信息、请求签名的内容和“用途声明”输入框
3. 用户主动填写“用途声明”为：文章署名，并将请求内容、用途声明、时间戳、请求者信息打包并签名，构造成为授权证书。
4. 将授权证书通过回调和返回两种方式通知第三方应用。

## 步骤1, 第三方发起请求

**请求格式：**
```
elaphant://sign?
	AppName=<AppName>&
	AppID=<AppID>&
	Description=<Description>&
	DID=<DeveloperDID>&
	PublicKey=<DID PublicKey>
	RequestedContent=<RequestedContent>&
	UseStatement=<UseStatement>&
	[&CallbackUrl=<Callback URL>]
	[&ReturnUrl=<Return URL>&Target=<"internal" | "browser">]

```
**指令参数：**

字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
RequestedContent       | String & URLEncode | 必选 | 第三方请求用户签名的内容。
UseStatement           | String & URLEncode | 可选 | 提示用户此次签名的用途


**sign请求示例：**
```
elaphant://sign?
AppName=FooBar&
Description=FooBar&
DID=iWwPFLS8Gp18LqM6GJgLz5QrPwjBqqshxF&
PublicKey=03684d22dd6de91cc5b504ff0499eff919bd2a1507826475d5c6b314217ea96417&
RequestedContent=7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322
UseStatement=Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
```

也可以将URI生成二维码，用户通过Elephant扫描二维码完成请求。


## 步骤2, 返回信息
在用户授权以后，Elephant将生成授权证书返回给第三方。

**返回信息格式：**

返回信息包括两部分：Data（数据）和Sign（签名）。Data是一个JSON对象序列化后的字符串。Sign是使用DID对Data字符串内容的签名。

返回格式示例
```
Data="{
	\"DID\":<DID>,
	\"PublicKey\":<PublicKey>,
	\"RequesterDID\":<RequesterDID>,
	\"RequestedContent\":<RequestedContent>,
	\"Timestamp\":<Timestamp>,
	\"UseStatement\":[UseStatement]
}"

Sign="<signature>"
```

**返回参数说明：**


字段名称           | 类型              | 是否必选 | 描述
----------------------| ------------------- | ------------------- | -------------------
DID             | String    | 必选 | 用户的DID
PublicKey       | String    | 必选 | DID的公钥
RequesterDID    | String    | 必选 | 第三方请求者的DID
AppName                 | String & URLEncode | 必选 | 第三方请求应用的名称; 长度不超过64个字节
AppID                   | String     | 必选 | 由开发者DID对AppName签名后产生的签名字符串，作为该App的ID。
RequestedContent | String & URLEncode    | 必选 | 请求签名的内容
Timestamp       | String    | 必选 | 用户授权的时间戳
UseStatement    | String & URLEncode    | 必选 | 用户声明的此次签名的用途
Signature       | String    | 必选 | 针对返回信息的签名

**HTTP Callback示例**：

Callback是以Post方式返回信息，在Body中使用JSON格式,包含Data和Sign两个字段。
```
[Method]: POST
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
[Body]:
{
    "Data":"{\"RequesterDID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"AppID\":\"cc053c61afc22dda9a309e96943c1734\",\"AppName\":\"RedPacket\",\"RequestedContent\":\"7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322\",\"UseStatement\":\"Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block\",\"Timestamp\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}",
    "Sign": "02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176"
}

```

**HTTP Return示例**：

Return是以Get方式返回信息，在URL的参数中附加Data和Sign参数，请第三方在其ReturnURL中避免使用这两个参数名，以免冲突。
```
[Method]: GET
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&Data={\"RequesterDID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"AppID\":\"cc053c61afc22dda9a309e96943c1734\",\"AppName\":\"RedPacket\",\"RequestedContent\":\"7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322\",\"UseStatement\":\"Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block\",\"Timestamp\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}&Sign= 02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176

```

## 步骤3, 解析和验证
1. 将参数Data的字符串内容转换为JSON对象；
2. 从JSON对象中恢复PublicKey、DID字段内容；
3. 验证PublicKey与DID是否匹配；
4. 验证Data字符串内容、PublicKey和Sign的签名内容是否匹配；
5. 验证返回的RequestedContent字段与“步骤1”请求时的RequestedContent值是否一致；

# multicreate指令

## 步骤1, 请求

**指令参数：**

字段名称           | 类型            | 描述
----------------------| ------------------- | -------------------
AppName               | String     | 你的应用名称; 长度不超过64个字节
AppID                 | String     | 你的app id; 长度不超过64字节，base58编码，并且在开发人员的DID中是唯一的。
DID                   | String     | 开发人员的DID
PublicKey             | String     | DID的公钥
PublicKeys            | String     | 创建多签钱包的所有公钥，用逗号分隔
RequiredCount         | String     | 交易最少的签名数
CallbackUrl           | String     | 请求结果的回调URI，返回多签钱包地址
ReturnUrl             | String     | 回调后返回的页面URI

**通过scheme调用请求的示例：**
```
elaphant://multicreate?AppID=6d3936396579af659f8faa82e2105e33d2b8adbe77be97f698e4e19ac9e7711ed341936aa2641d03e5238fe81f404f273d7932ff63019ad8e50c09b853204f01&AppName=elastos.multisign.test&PublicKey=02ab56c5493d1b26639dcdbc53a664c5eceb4739043ce06eae358b2d8366f358b1&DID=iWgMpqouJPK2H4WM4r2zvLp8XpdZwDihZS&ReturnUrl=http%3A%2F%2Fapp.51aiu.com%2Felastos%2Fhtml%2Fwallet.html&PublicKeys=031ed85c1a56e912de5562657c6d6a03cfe974aab8b62d484cea7f090dac9ff1cf%2C02afa15bf70b80b9398e467613e5ffecb491e339be7e49ef6687bab46e564218d0%2C02aa8bc1c0f935be03c6c4d3c7c200944f1ab5413f15925ba0ee4186b88d695a20&RequiredCount=2
```

## 步骤2, 返回信息

参数：

字段名称           | 类型             | 描述
-------------------------- | ------------------ | -------------------
DID                 | String        | 用户的DID
PublicKey           | String        | DID的公钥
Address             |  String       | 多签钱包地址

Return url回调示例：
```
 http://app.51aiu.com/elastos/html/wallet.html?Data=%7B%22Address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%2C%22DID%22%3A%22iii89hfPzr4vKkhJhg1vKAyP5XvP72BASn%22%2C%22PublicKey%22%3A%2202afa15bf70b80b9398e467613e5ffecb491e339be7e49ef6687bab46e564218d0%22%7D&Sign=d194ffed53f04a7cf0a8816b020ad989cbcf05c352d94e911b6e236390b968a25798994faaaebbcb92df66fea5f16b64daf7d72ea5959afd799880f7a921ea8e&Scheme=multicreate
```

## 步骤3, 解析和验证
1. 将参数Data的字符串内容转换为JSON对象；
2. 从JSON对象中恢复PublicKey、DID字段内容；
3. 验证PublicKey与DID是否匹配；
4. 验证Data字符串内容、PublicKey和Sign的签名内容是否匹配；


# multitx指令

## 步骤1, 请求

**指令参数：**

字段名称           | 类型            | 描述
----------------------| ------------------- | -------------------
AppName               | String     | 你的应用名称; 长度不超过64个字节
AppID                 | String     | 你的app id; 长度不超过64字节，base58编码，并且在开发人员的DID中是唯一的。
DID                   | String     | 开发人员的DID
PublicKey             | String     | DID的公钥
Tx                    | String     | 交易json
CallbackUrl           | String     | 请求结果的回调URI
ReturnUrl             | String     | 回调后返回的页面URI

**通过scheme调用请求的示例：**
```
elaphant://multitx?AppID=6d3936396579af659f8faa82e2105e33d2b8adbe77be97f698e4e19ac9e7711ed341936aa2641d03e5238fe81f404f273d7932ff63019ad8e50c09b853204f01&AppName=elastos.multisign.test&PublicKey=02ab56c5493d1b26639dcdbc53a664c5eceb4739043ce06eae358b2d8366f358b1&DID=iWgMpqouJPK2H4WM4r2zvLp8XpdZwDihZS&Tx=%7B%22Transactions%22%3A%5B%7B%22UTXOInputs%22%3A%5B%7B%22address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%2C%22txid%22%3A%222e4d2a77fadce79873468f1ce650d5e559631736717c61329e8842199ddfb9cf%22%2C%22index%22%3A1%7D%5D%2C%22Fee%22%3A4860%2C%22Outputs%22%3A%5B%7B%22amount%22%3A10000000%2C%22address%22%3A%22EgCrEaSfKrLAXP4DFqaxQVFYUjDHzDfxGZ%22%7D%2C%7B%22amount%22%3A899951400%2C%22address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%7D%5D%2C%22Memo%22%3A%22%22%7D%5D%7D
```

## 步骤2, 返回信息

当前版本不返回任何信息，回调仅用于通知转账完成。

## 步骤3, 解析和验证

