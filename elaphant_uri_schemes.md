# **Introduction**

In order to give third-party apps the ability to integrate quickly and easily into the blockchain, as well as the power to use the blockchain, Elephant Wallet is open for integration with third-party apps. Third-party apps can now complete complete payments with digital currency, use DID identification and write data onto the blockchain, all through Elephant Wallet.

# **Deployment Method**

Third-parties can call and deploy Elephant through a special scheme, "Elaphant://". They can also turn the deployed URI code into a QR code. Elephant can automatically scan the QR code to execute the corresponding operation. A scheme has three parts: command, required parameters, and command parameters.

## **Scheme format:**

elaphant://<command></RequiredParameters...>[/CommandParameters...]

### **The commands include:**

- [identity](#identity-command)
- [elapay](#elapay-command)
- [eladposvote](#eladposvote-command)
- [sign](#sign-command)
- [setproperty](#setproperty-command)
- [getproperty](#getproperty-command)

### **Required parameters:**

Any third-party initiating a deployment to Elephant must provide relevant parameters about information pertaining to the app, which is used to verify the truthfulnesstrustfulness of the deployer's identity and data.

| **Fieldname** | **Category**       | **Required?** | **Description**                                              |
| ------------- | ------------------ | ------------- | ------------------------------------------------------------ |
| AppName       | String & URLEncode | Required      | Name of your app; may not exceed 64 characters               |
| AppID         | String             | Required      | The App ID is a string, which is produced after the developer's DID creates a signature for the App Name. |
| Description   | String & URLEncode | Required      | Descriptions of the app and previous operations so that the user can decide whether to give authorization |
| DID           | String             | Required      | Developer's DID                                              |
| PublicKey     | String             | Required      | DID Public Key                                               |

Verification Method:

1. Verify whether the AppID value is the the DID signature for the App Name, and use the public key to conduct verification.
2. When verifying whether the DID corresponds with the public key, the Public Key can derive from the DID.

### **Callback and backtrack parameters:**

| **Fieldname** | **Category**       | **Required?** | **Description**                                              |
| ------------- | ------------------ | ------------- | ------------------------------------------------------------ |
| CallbackUrl   | String & URLEncode | Optional      | Callback URL resulting from the request                      |
| ReturnUrl     | String & URLEncode | Optional      | Request backtrack page URL                                   |
| Target        | String             | Optional      | The Return URL can be opened in the Elephant internal webview or in external browsers. "Internal" indicates that the internal webview is being used to open it, where "browser" indicates the use of an external browser to open it. |

** Note that when the request is made, the field for the URL Encode must be coded using the URL Encode method in UTF8 characters, as it will be decoded by Elephant using URL Decode.

Elephant provides two kinds of backtrack information to third parties: [Callback URL] and [Return URL]. The differences between the two is as follows:

- Callback URL: Elephant uses HTTP Post or other equivalent methods to asynchronously backtrack information on the back end.
- Return URL: Elephant utilizes HTTP Get or equivalent methods to jump and open the corresponding URL page on the front end.

Third-party apps can select to use one or both kinds.

If third-party apps simultaneously serve multiple users, tracking parameters can be appended to the callback URL of the Return URL.When Elephant backtracks, it will maintain its original parameter settings to callback the third-party app.

# **Identity Command**

## **Outline**

When third-party app programs need to verify user DID identities, or need personal information, they can initiate a request to Elephant, and after the user gives their authorization, Elephant will return the information and signature to the third-party.

A classic use case is login:

1. A third-party generates request content, including requestor's information and a random number.
2. After receiving user authorization, signature of the random number and DID information is carried out.
3. A notification of the information and signature is sent to the third-party through the two methods of callback and return.

Third-party requests should be accompanied by the random number. The random number should consist of unrepeatable, unguessable random content, to prevent attackers from carrying out replay attacks, in which pre-signed content is used to deceive the third-party and user identities are illegally used.

After the Elephant return information is received, the random number accompanying the return must be verified, and incorrect numbers are invalid.

## **Step 1, Third-party initiation of requests**

Request format:
```
elephant://identity?
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

Command parameter:

| **Fieldname** | **Category**       | **Required?** | **Description**                                              |
| ------------- | ------------------ | ------------- | ------------------------------------------------------------ |
| RandomNumber  | String & URLEncode | Required      | Utilize the "request/response" method to verify whether users possess a DID. Users must sign by using the DID and the random number, and return it to the requestor. |
| RequestInfo** | String & URLEncode | Optional      | Request to obtain the user's DID information list            |

** RequestInfo field is an information field required by the third-party, separated by commas. The fields that can be requested include:

Nickname, ELAAddress, BTCAddress, BCHAddress, ETHAddress, IOEXAddress, Email, PhoneNumber, ChineseIDCard

** Note that when the request is made, the field for the URL Encode must be coded using the URL Encode method in UTF8 characters, as it will be decoded by Elephant using URL Decode.

RequestInfo Example
```
RequestInfo="Nickname,ELAAddress,Email,PhoneNumber"
```
Identity Request Example:
```
elaphant://identity?
AppID=cc053c61afc22dda9a309e96943c1734&
AppName=RedPacket&
Description=red&
PublicKey=028971D6DA990971ABF7E8338FA1A81E1342D0E0FD8C4D2A4DF68F776CA66EA0B1&
RequestInfo=BTCAddress%2CETHAddress%2CEmail%2CPhoneNumber
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
```

It is also possible to generate a QR Code using the URI. The user scans the QR code in Elephant to complete the request.

## **Step 2, Return Information**

After the user gives authorization, Elephant will return the information pertaining to the user DID according to the third-party's request.

- Currently, the only callback method supported is HTTP Post. A post call will be initiated according to the address of the callback URL at the time of request, and the return information will be attached to the body.
- Currently, the only Return method supported is HTTP Get. A page will be opened in an internal Webview or external browser according to the return URL address at the time of request.

Return information format:

The return information includes two parts: Data and Sign. Data is a JSON object serialized character string. Sign is a signature that uses DID to provide a signature for data character string content.

Return format example
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
	ChineseIDCard\":[ChineseIDCard]
}"

Sign="<signature>"
```

Instructions for trackback parameter:

| **Fieldname** | **Category** | **Required?** | **Description**                                              |
| ------------- | ------------ | ------------- | ------------------------------------------------------------ |
| DID           | String       | Required      | User DID                                                     |
| PublicKey     | String       | Required      | DID Public Key                                               |
| RandomNumber  | String       | Required      | Third-party request random number                            |
| Nickname      | String       | Optional      | User nickname                                                |
| ELAAddress    | String       | Optional      | User's ELA wallet address                                    |
| BTCAddress    | String       | Optional      | User's BTC wallet address                                    |
| BCHAddress    | String       | Optional      | User's BCH wallet address                                    |
| ETHAddress    | String       | Optional      | User's ETH wallet address                                    |
| IOEXAddress   | String       | Optional      | User's IOEX wallet address                                   |
| Email         | String       | Optional      | User's email address                                         |
| PhoneNumber   | String       | Optional      | User's telephone number                                      |
| ChineseIDCard | String       | Optional      | Use JSON format to return user's Chinese identity card information |
| Signature     | String       | Required      | Regarding signatures for return information                  |

HTTP Callback Example:

Callback returns information using Post, and uses JSON formatting in the body, including in the two fields of Data and Sign.
```
[Method]: POST
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
[Body]:
{
    "Data":"{\"BTCAddress\":\"1Fmb9MLJ74ShWG5WcJWqMTDBfPTJPnQzVP\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"ELAAddress\":\"EHeSKgaFNxPJiLScvase437D2pM9isw4pX\",\"Email\":\"\",\"ETHAddress\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"NickName\":\"中文测试\",\"PhoneNumber\":\"\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}",
    "Sign": "02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176"
}
```

HTTP Return Example:

Return returns information using the Get method, and appends Data and Sign parameters inside the URL parameters. Third-parties should avoid using these two parameter names in the ReturnURL, to avoid conflicts.
```
[Method]: GET
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&Data={\"BTCAddress\":\"1Fmb9MLJ74ShWG5WcJWqMTDBfPTJPnQzVP\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"ELAAddress\":\"EHeSKgaFNxPJiLScvase437D2pM9isw4pX\",\"Email\":\"\",\"ETHAddress\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"NickName\":\"中文测试\",\"PhoneNumber\":\"\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}&Sign= 02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176
```


## **Step 3, Analyze and Verify**

1. Turn the parameter data string content into a JSON object;
2. Recover the PublicKey and DID field from the JSON object;
3. Verify that the Public Key and DID match;
4. Verify that the data character string content, the PublicKey and Sign signature content match;
5. Verify the returned random number field and the Random Number value requested in "Step 1" are consistent;

# **Elapay Command**

## **Outline**

When a third-party program requires the use of digital currency for payments, the program can send a request to Elephant, and after users give authorization, the currency will be transferredtransfered to the requested address. This process is similar to third-party requests for Alipay and Paypal to complete payments.

- Step 1: The third-party initiated the request, including the recipient's address, balance, name of the digital currency, and the unique order identifier.
- Step 2: Elephant will display the payment transfer page, and after the user has confirmed, the transfer is be initiated.
- Step 3: The transfer transaction TXID will be returned, and the third-party can use it to check the status of the transaction.

## **Third-party initiations of payment requests**

Request format:
```
elephant://elapay?
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

Command parameter:

| **Fieldname**    | **Category** | **Required?** | **Description**                                              |
| ---------------- | ------------ | ------------- | ------------------------------------------------------------ |
| OrderID          | String       | Optional      | Order number, used for third-party tracking of transaction results. |
| CoinName         | String       | Required      | Name of digital currency type                                |
| Amount           | Number       | Required      | Transfer balance                                             |
| ReceivingAddress | String       | Required      | Address used for receiving funds                             |

Elapay request example:
```
elaphant://elapay?
AppID=cc053c61afc22dda9a309e96943c1734&
AppName=RedPacket&
Description=red&
PublicKey=028971D6DA990971ABF7E8338FA1A81E1342D0E0FD8C4D2A4DF68F776CA66EA0B1&
OrderID=354199ds9213k1f&
CoinName=ELA&
Amount=1.234&
ReceivingAddress=EXRNP8Pm3KQR7EeLXFGRtfLdkBaeZsMLyy&
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
```

## **Return payment information**

Return information format:

Regardless of whether Callback or Return method is used, information is returned in a  character string format. A character string has two parameters: "TXID" and "OrderID." Return information includes two parts: TXID (the transaction hash) and OrderID (the third-party order number). The TEXID is this transaction's hash, and the OrderID is the OrderID value input by the third-party at the time of request, for the purpose of the tracking the payment process by the third-party.
```
TXID="bb14a5898ca81ee0207ab9178df1d1e4617be3aa5e9dfec9e185d41ff13786fc"&OrderID=“354199ds9213k1f"
```
Instructions for trackback parameter:

| **Fieldname** | **Category** | **Required?** | **Description**                           |
| ------------- | ------------ | ------------- | ----------------------------------------- |
| TXID          | String       | Required      | Transfer transaction TXID                 |
| OrderID       | String       | Required      | OrderID corresponding to the request time |

HTTP Callback example:

Callback returns information using the Post method, utilizes JSON formatting in the body, and includes the two fields of TXID and OrderID.
```
Method: POST
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
Body:
{
"TXID":"895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718",
"OrderID":"354199ds9213k1f"
}
```

HTTP Return return example:

Return uses the Get method to return information by appending TXID and OrderID parameters into the URL parameters. Third-parties should not use these two parameters in ReturnURLs, in order to avoid conflicts.

```
Method: GET
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&TXID=895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718&OrderID=354199ds9213k1f
```

You can confirm the status of transactions at blockchain.elastos.org.

<https://blockchain.elastos.org/tx/895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718>

# **Eladposvote Command**

## **Outline**

Third-party apps can issue DPoS votes through the vote command. After users give authorization, the entire ELA balance will be used to issue the vote transaction.

- Step 1: The third-party issues the request, including the candidate public key list.
- Step 2: Elephant displays the vote information, and the vote transaction is issue after the user confirms.
- Step 3: The TXID of the vote transaction is returned to the third-party, and the third party can use it to check the transaction status.

## **Third-Party initiation of vote requests**

Request format:
```
elephant://eladposvote?
	AppName=<AppName>&
	AppID=<AppID>&
	Description=<Description>&
	DID=<DeveloperDID>&
	PublicKey=<DID PublicKey>&
	CandidatePublicKeys=<Supernode public key list>
	[&CallbackUrl=<Callback URL>]
	[&ReturnUrl=<Return URL>&Target=<"internal" | "browser">]
```

Request parameters:

| **Fieldname**       | **Category**       | **Required?** | **Description**                                           |
| ------------------- | ------------------ | ------------- | --------------------------------------------------------- |
| CandidatePublicKeys | String & URLEncode | Required      | Candidate public key lists should be separated by a comma |

eladposvote request example:
```
elaphant://eladposvote?
AppID=cc053c61afc22dda9a309e96943c1734&
AppName=DPoSVote&
Description=DPoSVote%20mini%20app&
PublicKey=028971D6DA990971ABF7E8338FA1A81E1342D0E0FD8C4D2A4DF68F776CA66EA0B1&
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
CandidatePublicKeys=03ef5f8b0534c82aa4db218f7cead278124efc0411e4ca38b6131954a58e8ae3c0%2C03ef5f8b0534c82aa4db218f7cead278124efc0411e4ca38b6131954a58e8ae3c0
```

## **Return Vote Information**

Return information format:

Regardless of whether the Callback or Return method is used, information is returned in a character string format.
```
TXID="bb14a5898ca81ee0207ab9178df1d1e4617be3aa5e9dfec9e185d41ff13786fc"
```
Instructions for trackback parameter:

| **Fieldname** | **Category** | **Required?** | **Description**        |
| ------------- | ------------ | ------------- | ---------------------- |
| TXID          | String       | Required      | Vote Transaction TXIDs |

HTTP Callback example:

Callbacks use the Post method to return information, returning information in the Body in JSON format.
```
Method: POST
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
Body:
{
"TXID":"895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718"
}
```

HTTP Return example: Return uses the Get method to return information, the TXID corresponding to vote transactions are returned using the parameter method. Third-parties should not use TXID parameter names in the ReturnURL, to avoid conflicts.
```
Method: GET
URL: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&TXID=895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718
```

You can confirm the status of transactions at blockchain.elastos.org.

<https://blockchain.elastos.org/tx/895630890fa53437cdd09e1f3bb4934585e1914eb45eb3a6b1e4a4d6bd789718>

# **Sign Command**

## **Outline**

When third-party app programs need to verify user DID identities, or need personal information, they can initiate a request to Elephant, and after the user gives their authorization, Elephant will construct a signature certification and return it to the third-party.

For example, third-party requests a user's signature on an article they wrote:

1. The third-party will use the article's hash as the signature request content and sends it to Elephant.
2. Elephant displays the requestor's information, the content of the signature request and the "use statementstatemetn" input window
3. The user voluntarily inputs the "use statement" as: article signature, and packages and signs the request content, use statement, timestamp, requester information, creating an authorization certificate.
4. The third-party app is notified of the authorization certificate through the two methods of callback and return.

## **Step 1, Third-party initiation of requests**

Request format:
```
elephant://sign?
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

Command parameter:

| **Fieldname**    | **Category**       | **Required?** | **Description**                                              |
| ---------------- | ------------------ | ------------- | ------------------------------------------------------------ |
| RequestedContent | String & URLEncode | Required      | Third-party requests user-signed content.                    |
| UseStatement     | String & URLEncode | Optional      | Display the reason for this instance of collecting the user's signature |

Sign Request Example:
```
elaphant://Sign?
AppID=cc053c61afc22dda9a309e96943c1734&
AppName=RedPacket&
Description=red&
PublicKey=028971D6DA990971ABF7E8338FA1A81E1342D0E0FD8C4D2A4DF68F776CA66EA0B1&
RequestedContent=7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322
UseStatement=Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block
CallbackUrl=http%3A%2F%2Flocalhost%3A8081%2Fpacket%2Fgrab%2F1509893100600982-0&
```

It is also possible to generate a QR Code using the URI. The user scans the QR code in Elephant to complete the request.

## **Step 2, Return Information**

After the user has provided authorization, Elephant will generate a authorization certificate and return it to the third-party.

Return information format:

The return information includes two parts: Data and Sign. Data is a JSON object serialized character string. Sign is a signature that uses DID to provide a signature for data character string content.

Return format example
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

Instructions for trackback parameter:

| **Fieldname**   | **Category**       | **Required?** | **Description**                                              |
| --------------- | ------------------ | ------------- | ------------------------------------------------------------ |
| DID             | String             | Required      | User DID                                                     |
| PublicKey       | String             | Required      | DID Public Key                                               |
| RequesterDID    | String             | Required      | DID of the third-party requester                             |
| AppName         | String & URLEncode | Required      | Name of the third-party request application; length must not exceed 64 characters |
| AppID           | String             | Required      | The App ID is a string, which is produced after the developer's DID creates a signature for the App Name. |
| RequestedContent | String & URLEncode | Required      | Content for which signature is being requested               |
| Timestamp       | String             | Required      | User authorization timestamp                                 |
| UseStatement    | String & URLEncode | Required      | User statement for the reason for collecting this signature  |
| Signature       | String             | Required      | Regarding signatures for return information                  |

HTTP Callback Example:

Callback returns information using Post, and uses JSON formatting in the body, including in the two fields of Data and Sign.
```
[Method]: POST
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480
[Body]:
{
    "Data":"{\"RequesterDID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"AppID\":\"cc053c61afc22dda9a309e96943c1734\",\"AppName\":\"RedPacket\",\"RequestedContent\":\"7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322\",\"UseStatement\":\"Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block\",\"Timestamp\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}",
    "Sign": "02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176"
}
```

HTTP Return Example:

Return returns information using the Get method, and appends Data and Sign parameters inside the URL parameters. Third-parties should avoid using these two parameter names in the ReturnURL, to avoid conflicts.
```
[Method]: GET
[URL]: https://redpacket.elastos.org/packet/grab/3176517663416268-1?_locale=zh_CN&offset=-480&Data={\"RequesterDID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"DID\":\"iYarsyzWeaSRa7XVqeCeNpa1J9sGB4Uwyr\",\"AppID\":\"cc053c61afc22dda9a309e96943c1734\",\"AppName\":\"RedPacket\",\"RequestedContent\":\"7aae6c4f8da0799063f1db14024413ba7aba70e61312f291527dfde3605ea322\",\"UseStatement\":\"Sign%20the%20hash%20of%20the%20block%2C%20indicating%20the%20recognition%20of%20the%20contents%20of%20the%20block\",\"Timestamp\":\"0x6601F031E63cb2C477beE183D671F0517a37CdC7\",\"PublicKey\":\"03b27a7ae64d9f11c9685475f76d9e3ba6498953c49f69962b6566655146820d71\"}&Sign= 02e1df21bc11744421ece9be874691ebace8f6ce7ea4443d0f96500e9a9db3978ce90eecc9863be8de27648eb316560df904eaf1c6fc2dc8bf630822cdb9c176
```

## **Step 3, Analyze and Verify**

1. Turn the parameter data string content into a JSON object;
2. Recover the PublicKey and DID field from the JSON object;
3. Verify that the Public Key and DID match;
4. Verify that the data character string content, the PublicKey and Sign signature content match;
5. Verify the returned RequestedContent field and the RequestedContent value requested in "Step 1" are consistent;


# **Request create multi-sig wallet**

## **Step1, Request**

Parameters:

Field Name           | Type            | Description
----------------------| ------------------- | -------------------
AppName               | String     | Your app name; the length is no more than 64 bytes.
AppID                 | String     | Your app id; the length is no more than 64 bytes, base58 encoding and is unique within the developer's DID.
DID                   | String     | The developer's DID.
PublicKey             | String     | Public key of the DID.
PublicKeys            | String     | Public keys to create multi-sig wallet，Separated by commas
RequiredCount         | String     | The least number of signatures
CallbackUrl           | String     | Callback URI of the request result
ReturnUrl             | String     | Returned page URI after the callback

scheme sample：
```
elaphant://multicreate?AppID=6d3936396579af659f8faa82e2105e33d2b8adbe77be97f698e4e19ac9e7711ed341936aa2641d03e5238fe81f404f273d7932ff63019ad8e50c09b853204f01&AppName=elastos.multisign.test&PublicKey=02ab56c5493d1b26639dcdbc53a664c5eceb4739043ce06eae358b2d8366f358b1&DID=iWgMpqouJPK2H4WM4r2zvLp8XpdZwDihZS&ReturnUrl=http%3A%2F%2Fapp.51aiu.com%2Felastos%2Fhtml%2Fwallet.html&PublicKeys=031ed85c1a56e912de5562657c6d6a03cfe974aab8b62d484cea7f090dac9ff1cf%2C02afa15bf70b80b9398e467613e5ffecb491e339be7e49ef6687bab46e564218d0%2C02aa8bc1c0f935be03c6c4d3c7c200944f1ab5413f15925ba0ee4186b88d695a20&RequiredCount=2
```

## **Step2, Response**

Parameters:

Field Name           | Type             | Description
-------------------------- | ------------------ | -------------------
DID                 | String        | User's DID.
PublicKey           | String        | Public key of the DID.
Address             |  String       | Multi-sig wallet address.

Return url sample：
```
 http://app.51aiu.com/elastos/html/wallet.html?Data=%7B%22Address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%2C%22DID%22%3A%22iii89hfPzr4vKkhJhg1vKAyP5XvP72BASn%22%2C%22PublicKey%22%3A%2202afa15bf70b80b9398e467613e5ffecb491e339be7e49ef6687bab46e564218d0%22%7D&Sign=d194ffed53f04a7cf0a8816b020ad989cbcf05c352d94e911b6e236390b968a25798994faaaebbcb92df66fea5f16b64daf7d72ea5959afd799880f7a921ea8e&Scheme=multicreate
```

## **Step 3, Verify & Return**

1. Turn the parameter data string content into a JSON object;
2. Recover the PublicKey and DID field from the JSON object;
3. Verify that the Public Key and DID match;
4. Verify that the data character string content, the PublicKey and Sign signature content match;


# **Request multi-sig wallet payment**

## **Step1, Request**

Parameters:

Field Name           | Type            | Description
----------------------| ------------------- | -------------------
AppName               | String     | Your app name; the length is no more than 64 bytes.
AppID                 | String     | Your app id; the length is no more than 64 bytes, base58 encoding and is unique within the developer's DID.
DID                   | String     | The developer's DID.
PublicKey             | String     | Public key of the DID.
Tx                    | String     | Json string of the transaction.
CallbackUrl           | String     | Callback URI of the request result
ReturnUrl             | String     | Returned page URI after the callback

scheme sample:
```
elaphant://multitx?AppID=6d3936396579af659f8faa82e2105e33d2b8adbe77be97f698e4e19ac9e7711ed341936aa2641d03e5238fe81f404f273d7932ff63019ad8e50c09b853204f01&AppName=elastos.multisign.test&PublicKey=02ab56c5493d1b26639dcdbc53a664c5eceb4739043ce06eae358b2d8366f358b1&DID=iWgMpqouJPK2H4WM4r2zvLp8XpdZwDihZS&Tx=%7B%22Transactions%22%3A%5B%7B%22UTXOInputs%22%3A%5B%7B%22address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%2C%22txid%22%3A%222e4d2a77fadce79873468f1ce650d5e559631736717c61329e8842199ddfb9cf%22%2C%22index%22%3A1%7D%5D%2C%22Fee%22%3A4860%2C%22Outputs%22%3A%5B%7B%22amount%22%3A10000000%2C%22address%22%3A%22EgCrEaSfKrLAXP4DFqaxQVFYUjDHzDfxGZ%22%7D%2C%7B%22amount%22%3A899951400%2C%22address%22%3A%228UKQAU6AjphwAehoeD4f4cknodah2A9qsY%22%7D%5D%2C%22Memo%22%3A%22%22%7D%5D%7D
```

## **Step2, Response**

No return data.

## **Step 3, Verify & Return**

