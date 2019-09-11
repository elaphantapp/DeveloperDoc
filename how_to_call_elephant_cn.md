### 从手机上发起调用

如果用户在手机上安装了Elephant Wallet，那么无论是原生App或者是H5 App都可以使用特殊URI前缀的URL来唤起Elephant Wallet并传递参数。
我们支持两种URI前缀，它们是等价的：elastos:// 或者 elaphant://

请求Elephant Wallet用户身份的示例：
```
**elaphant://identity?**
ReturnUrl=http%3A%2F%2Fbing.com&
AppID=7d80c7e800f5842b3b8e7ec7318189f66b7fd5b6db13bb80fbd89d2b1c444772c1d0202fea1e9cbabbf3258b3d91685484c02c2ae52d78ca39e2e54593ec81dd&
PublicKey=032f6347b27401dc0bced2de0ab4531e62c496841cd8e67a58c572e3018dcb72d9&
DID=iXzenTELVRDc712tmt2Qvbtk3KcAwV2tU8&
AppName=Hackathon&
RequestInfo=elaaddress,Email,Nickname&
description=Hackathon&
RandomNumber=4284432979
```

其中identity是调用elephant wallet的命令，更多命令和参数可以查看我们的URI scheme说明。

### 从PC Web上发起调用

对于非手机环境，或者没有安装Elephant Wallet的环境，还可以通过二维码来发起调用。第三方应用可以将调用请求的URI制作成二维码显示，在Elephant Wallet中扫描二维码来获取调用参数和信息。

为了方便跨平台的Web项目调用Elephant wallet，我们提供一个launch elephant的代理页面的项目

- [Github Repo: launch.app](https://github.com/elaphantapp/launch.app) 

第三方应用可以使用一个固定的URL格式来调用Elephant wallet，而无需再次判断发起端是从手机端还是PC、或者是微信。

* 如果是手机端，它会直接唤起Elephant Wallet，如果用户手机还没有安装，它也会展示安装页面；
* 如果是从微信里发起调用，会先提示用户从微信里跳出到手机浏览器，然后再唤起Elephant Wallet；
* 如果是PC端，它会显示二维码，提示用户打开大象钱包进行扫码。

### 获取Elephant的返回信息

在调用Elephant Wallet的URL参数中有两个参数：return_url和callback_url，

- return_url: Elephant Wallet在完成请求以后，会调用手机端浏览器打开return_url指定的页面；
- callback_url: Elephant Wallet在完成请求以后，会使用HTTP Post方法回调callback_url指定的页面。

#### 返回到手机前端

直接在return_url中指定一个H5页面的URL，或者另一个App注册的scheme的URL，都可以让Elephant Wallet唤起它，并且附带调用结果。

#### 返回到后台服务

直接在callback_url中指定后台服务的URL，Elephant wallet会使用HTTP Post方法回调，并将调用结果作为参数传递。

#### 返回到PC Web前端

如果PC Web想调用Elephant Wallet并获得结果，通常需要需要自己实现一个后端服务中转，为此我们提供了一个转发服务

- [Github Repo: Echo Service](https://github.com/elaphantapp/EchoService)

两个例子：

- [使用Elephant Wallet的DID登录的例子](./samples/how_to_login_with_did_callbackUrl.html)

- [使用Elephant Wallet支付的例子](./samples/how_to_pay_ela_callbackUrl.html)



