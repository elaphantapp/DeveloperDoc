## DID login with Elaphant App



**Raw Request URL Scheme**

```
elaphant://identity?
AppID=<AppID>
&AppName=<AppName>
&RandomNumber=<A random number>
&DID=<Developer DID>
&PublicKey=<Developer Publickey>
&ReturnUrl=<URL to which the elephant wallet will go>
&RequestInfo=ELAAddress,BTCAddress,ETHAddress,Email
```

**A sample (you can directly use it)**

```
elaphant://identity?AppID=ac89a6a3ff8165411c8426529dccde5cd44d5041407bf249b57ae99a6bfeadd60f74409bd5a3d81979805806606dd2d55f6979ca467982583ac734cf6f55a290
&AppName=Mini%20Apps
&RandomNumber=123456789
&DID=ibxNTG1hBPK1rZuoc8fMy4eFQ96UYDAQ4J
&PublicKey=034c51ddc0844ff11397cc773a5b7d94d5eed05e7006fb229cf965b47f19d27c55
&ReturnUrl=https%253A%252F%252Fcryptoname.org%252FeditInfo.html%253Fname%253Dsjunsong
&RequestInfo=ELAAddress%2CBTCAddress%2CETHAddress%2CEmail
```

**PC or Mobile auto fit page**

```
https://launch.elaphant.app/?appName=<AppName>&appTitle=<AppTitle>&autoRedirect=True
&redirectURL=<RequestURL>
```

**A sample**

```
https://launch.elaphant.app/?appName=CryptoName&appTitle=CryptoName&autoRedirect=True
&redirectURL=elaphant%3A%2F%2Fidentity%3FAppID%3Dac89a6a3ff8165411c8426529dccde5cd44d5041407bf249b57ae99a6bfeadd60f74409bd5a3d81979805806606dd2d55f6979ca467982583ac734cf6f55a290%26AppName%3DMini%20Apps%26RandomNumber%3D123456789%26DID%3DibxNTG1hBPK1rZuoc8fMy4eFQ96UYDAQ4J%26PublicKey%3D034c51ddc0844ff11397cc773a5b7d94d5eed05e7006fb229cf965b47f19d27c55%26ReturnUrl%3Dhttps%253A%252F%252Fcryptoname.org%252FeditInfo.html%253Fname%253Dsjunsong%26RequestInfo%3DELAAddress%2CBTCAddress%2CETHAddress%2CEmail
```

[More info](./elaphant_uri_schemes.md#identity-command)
