#聊天SDK相关文档 

##一.Elastos.SDK.ElephantWallet.Contact

###下载地址：https://github.com/elastos/Elastos.SDK.ElephantWallet.Contact/
###文档地址：https://elastossdkcontact.readthedocs.io/en/latest/
###编译方法：

> PC(MacOS，Ubuntu): $`./scripts/build.sh`, target libraries are in build/sysroot/{Darwin,Linux}/x86_64/lib/.

> Android:           $`./scripts/build-android.sh`, target libraries are in build/package/Elastos.SDK.Contact-android-vx.x.x.zip.

> iOS:               $`./scripts/build-ios.sh`, target libraries are inbuild/package/Elastos.SDK.Contact-ios-vx.x.x.zip.


##二.PersonalStorage.SDK.OSS

###下载地址：https://github.com/elaphantapp/PersonalStorage.SDK.OSS
###文档地址：无
###编译方法：

> PC(MacOS，Ubuntu): $`./scripts/build.sh`, target libraries are in build/sysroot/{Darwin,Linux}/x86_64/lib/.

> Android:           $`./scripts/build.sh -f Android`, target libraries are in build/sysroot/Android/armeabi-v7a/lib/.

> iOS:               $`./scripts/build.sh -f iOS`, target libraries are in build/sysroot/iOS/arm64/lib/.

     
##三.微服务

###微服务类型
> - Moments（朋友圈）
    https://github.com/zuohuahua/Moments
> - MicroService.PersonalStorage（个人存储微服务）
    https://github.com/elaphantapp/MicroService.PersonalStorage
> - MicroService.ChatGroup（群聊微服务）
    https://github.com/elaphantapp/MicroService.ChatGroup
> - MicroService.HashAddressMapping（地址重映射微服务）
    https://github.com/elaphantapp/MicroService.HashAddressMapping
> - MicroService.Manager（综合管理微服务）
    https://github.com/elaphantapp/MicroService.Manager
> - MicroService.Feedback（回馈测试）
    https://github.com/elaphantapp/PeerNodeSDK/tree/master/plugins/MicroService.Feedback


###编译方法
    选择上述微服务中的一个或几个，download到PeerNodeSDK的plugins文件夹，执行./scripts/build.sh，微服务会被PeerNodeSDK自动编译出so，并置于build/sysroot/Linux/x86_64/lib/PeerNodePlugins目录下。

###执行微服务
    类似./build/sysroot/Linux/x86_64/bin/PeerNodeLauncher.sh -name build/sysroot/Linux/x86_64/lib/PeerNodePlugins/libChatGroupService.so -path /tmp/PeerNode/ -key 4138db5bd521d810dbd5863bda6f18af8dbaa8382fd6a5260ee09cf05d2dc276 

###已部署的服务地址
> - CharGroup
    privkey:	4138db5bd521d810dbd5863bda6f18af8dbaa8382fd6a5260ee09cf05d2dc276
	did:	iinVuWZXaUU4QVhHMVvDS3wdaNausvW85M
	carrieraddr:	HuBkrBkUJatE2QVCtRpbsZgi27v47XGnzg6XcYeYqqfb92jmDyDW

> - Feedback
    privkey:	bee987a36419cce7c13cee162ab140b6986c02669bb4f954783ee7d933b5ad93
	did:	ifk5sgfJS2qF5Qha8rPHtHcwh2BraaLFi5
	carrieraddr:	FXzQuPsDnnruqGD2iVmrWpGiksa4wMGivfqMJBz7LfbEd11bNh4v

> - PersonalStorage
    privkey:	dd6d06cef60c2a8819c1fd8a1dec213c63bdc55ee59c055ea47a50f03c86fbee
	did:	iaagUjbceqWqME9iRE4KcD7WHiV1MC2FTq
	carrieraddr:	6zEgAhLns7s5LugSBKnxwC9s8m72qrkbUmCi3xmkq7bHtXAF5MsK

> - HashAddressMapping
    privkey:	ec1fd17239f4bf9c05b68ec30fccc7839750109fbe62895bcec50fdec34265db
	did:	iT36DAhPHCnrwtEyECM6N8LfapbmXEQmFd
	carrieraddr:	NbXwoUs79Loz3KADUjNbKAUdsUuxojb4BRNBy4yRehJALumLEfsA

> - Moments
    privkey:	9d890c9270e67d772cccb2ea347312b4a63e1a51cc685a02c7d9bfc9822b0d74
	did:	ikrf3Pny75bLvdJSAyJHPQi8LszDtap23F
	carrieraddr:	To6HsbmTNYhbrakRqLXMYVSAeJw9cEbqnLS8zEsXWXpspiiqZaee


###遗留问题：未集成新版本PeerNodeSDK和ContactSDK，接口与新SDK不兼容。

