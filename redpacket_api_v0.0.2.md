# Red Packet Api :

## 1. Create a red packet 

### request url : 

`/api/v1/packet/create`

### method :

`POST`

### request Data :
```js
{
	// number of packet , max 2000 , if packet_amt = 1ELA , you can send maximum 500 red packet , 2ELA for 1000 packet etc..
	// must
	"packet_num": 500,
	
	// packet ela amount
	// must
	"packet_amt": 1.0,

        // only specified addrs can receive the packet
	// optional
	"packet_allowed_rcv_addrs":[],
	
	// 0 stand for random red packet, 1 stand for fixed red packet , 2 stand for super node red packet
	// must
	"packet_type":0,

	// if red packet type is 2 then this should fill in the super node owner public key
	// optional
	"super_node_owner_public_key":"030fce722c5e8caf9ff7c4672e4f67c9a1b3090a35b224640e71524bb2a0c7292a",
	
	// red packet blessing message  
	// optinal
	"packet_blessing":"Good luck ,Good fortune",
	
	// name of the red packet creator
	// must
	"packet_creator":"Elaphant wallet",
	
	// define the red packet end timestamp , default 1 day . timestamp which is the number of milliseconds since January 1, 1970, 00:00:00 GMT represented by this date(utc time)
	// optional
	"packet_end_timestamp":1572950076764,
	
	//current locale language . only support `zh_CN` or `en_US` , default `en_US` 
	//optional
	"language":"en_US"
}
```

### response :
```js
{
    	// response result only exist when status is 200
    	// optional
	"result":{
	
	    	// pay this address to create a red packet
	    	// must
		"pay_address":"EbDLm4dndRAmTyEhh1zryY9Q8R5PoaZ64V",
		
        	// red packet identity hash
		// must 
		"packet_hash":"2229495168234378",
    	
		// should always stay "Ok"
		// must
		"desc":"Ok"
	},
	
	// enum value of 200 or 400 or 500 , 200 menas everything is fine , 400 indicate this is a bad request ,maybe because request param is not correct , 500 indicate unexpeced behavious happened in server side, maybe you need to contact the admin
	// must
	"status":200,
	
	// description of the request error only exist when status is not 200
	// optinal
	"error_msg":"something is not right"
}
```

## 2. Peek red packet

### request url : 

`/api/v1/packet/peek`

### method :

`GET`

### request Data :
```string
// packet hash
// must
packet_hash=2229495168234378
&
// whether or not show the details of receivers
// must
show_receivers=true
```

### response :
```js
{
    	// response result only exist when status is 200
    	// optional 
	"result":{
	
		// red packet detail info
		// must
		"packet_details":{
		
            	// red packet identity hash
    		// must 
    		"packet_hash":"2229495168234378",
		
            	// number of packet , max 2000 , if packet_amt = 1ELA , you can send maximum 500 red packet , 2ELA for 1000 packet etc..
        	// must
        	"packet_num": 500,
        	
        	// packet ela amount
        	// must
        	"packet_amt": 1.0,
            
        	// 0 stand for random red packet, 1 stand for fixed red packet , 2 stand for super node redpacket
        	// must
        	"packet_type":0,
        
        	// if packet type is 2 then this should fill in the super node owner public key
        	// optional
        	"super_node_owner_public_key":"030fce722c5e8caf9ff7c4672e4f67c9a1b3090a35b224640e71524bb2a0c7292a",
        	
        	// packet blessing message  
        	// optinal
        	"packet_blessing":"Good luck ,Good fortune",
        	
        	// if red packet type is 2 then this should fill in the super node owner public key
        	// optional
        	"super_node_owner_public_key":"030fce722c5e8caf9ff7c4672e4f67c9a1b3090a35b224640e71524bb2a0c7292a",
        	
        	// name of red packet creator
        	// must
        	"packet_creator":"Elaphant wallet",
        	
        	// red packet already received number
        	// must
        	"packet_rcv_num":100,
        	
        	// red packet already received money
        	// must
        	"packet_rcv_amt":0.8762333,
        	
		// red packet start timestamp
		// must
		"packet_start_timestamp":1572940076764,
		
		// red packet end timestamp
		// must
		"packet_end_timestamp":1572950076764,
		
        	// receiver details , only exist when request param `show_receivers = true`
        	// optional
        	"packet_rcver_details":[
        	    {
        	    
        	        // receiver name
        	        // must
        	        "name":"Your NickName",
        	        
        	        // receiver address
        	        // must
        	        "address" :"EMxQnmDJAoCTjFXXKhtwrGN9Ab4Jn6CMJE",
        	        
        	        // packet receiving timestamp which is the number of milliseconds since January 1, 1970, 00:00:00 GMT represented by this date(utc time)
        	        // must
        	        "time_stamp":1572940076764,
        	        
        	        // receiving red packet amount
        	        // must
        	        "amount":0.002319
        	    }
        	]
		}
		
		// enum value , may be the following values , `Ready` or `Not_ready`, `Ready` means the transaction is confirmed on the block , `Not_ready` means the transaction is not yet include in the block ,maybe currently in the transactionpool
		// must 
		"desc":"Not_ready"
	},
	
	// enum value of 200 or 400 or 500 , 200 menas everything is fine , 400 indicate this is a bad request ,maybe because request param is not correct , 500 indicate unexpeced behavious happened in server side, maybe you need to contact the admin
	// must
	"status":200,
	
	// description of the request error only exist when status is not 200
	// optinal
	"error_msg":"something is not right"
}
```

## 3. Grab a red packet 

### request url :    

`/api/v1/packet/grab`

### method :     

`Get` 

### request Data: 
```string
// red packet identity hash
// must 
packet_hash=6719330022590231
&
// red packet receiving address
// must
address=EbDLm4dndRAmTyEhh1zryY9Q8R5PoaZ64V
&
// red packet graber name
// must
name=alice
```

### response Data:
```js
{


    	// response result only exist when status is 200
    	// optional 
	"result":{
	
		// only return when the desc is Normal
		// optinal
		"amount": 0.0002,
		
		// only return when the desc is Normal, tell you if this is your first time grabing this red packet . only the first time grab a red packet will actually send you the red packet.
		// optional
		"first_grab": true,
		
		// enum value , may be the following values , `Normal` or `Too_late` or `Not_allowed`
		// must 
		"desc": "Normal"
	},
	
	// enum value of 200 or 400 or 500 , 200 menas everything is fine , 400 indicate this is a bad request ,maybe because request param is not correct , 500 indicate unexpeced behavious happened in server side, maybe you need to contact the admin
	// must
	"status":200
	
    	// description of the request error only exist when status is not 200
	// optinal
	"error_msg":"something is not right"
}
```


