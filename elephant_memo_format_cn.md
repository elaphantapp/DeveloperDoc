# Format of the note message in the elephant transaction

## Usage

Elephant Wallet支持在交易中填写备注信息，这些备注信息会与交易一起打包上链。同时在Elephant Wallet中也会显示这些备注信息。

## Raw transcation

交易的备注信息记录在attributes数组里，直接构造一个attribute并打包在交易里。
```
attribute.usage=Memo(0x81)
```

## Memo format

备注内容的格式如下：
```
type:text,msg:[plain text]
```

对于备注消息的文本，我们采用UTF-8编码。


