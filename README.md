# blockchain
Just a blockchain, Nothing in particular.

## 1.What's blockchain ?

I will only give you a one-minute introduction to my block system.
if you want a further reading, read the following page and try it by a demo(their author really finished an outstanding job).   
[blockchain_guide](https://github.com/Blockchain-CN/blockchain_guide)    
[blockchain_demo](https://blockchaindemo.io/)

### 2.1 What's block ?
A block contains these data members.
A block is legal when meet all the following conditions.
- Hash = sha256(PVHash+Timestamp+Data+Index+Nonce)
- Hash value meets the right difficulty.
``` go
// Block struct.
type Block struct {
	PVHash    string `json:"pv_hash"`
	Timestamp int64  `json:"timestamp"`
	Data      string `json:"data"`
	Index     int64  `json:"index"`
	Nonce     int64  `json:"nonce"`
	Hash      string `json:"hash"`
}
```
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片0.jpg)    

### 2.2 What's chain?
a blockchain just contains a lot of blocks
```go
// TheChain BlockChain struct.
type TheChain struct {
	Chain []*Block `json:"chain"`
}
```
Chain means we organised these block like a list, PVHash data equals to the previous block's Hash data.
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片1.jpg)   
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片2.jpg)   

### 2.3 How does your data transfer to the whole network?
In transport layer, blockchain system use the P2P network to spread your latest block to your peers, and after passing your peers' Validity test, peer will append it to their chain's tail, and spread it to their peers.   
- The complete transfer protocal.   
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片3.jpg)   
- How does peers do when they received a legal block and append to their chain's tail?   
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片4.jpg)   
- How does peers do when they received a legal block and it's index is longer than their chain?   
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片5.jpg)   
- How does peers do when they received a illegal block or it's index is shorter than their chain?   
![image](https://github.com/Blockchain-CN/blockchain/raw/master/readme_image/幻灯片6.jpg)   

### 2.4 What's bit-coin ?
It's a protocol about the data form inside a block.
and using RSA algorithm to guarantee your account's security.

### 2.5 How to crack or destroy it ?
- In order to crack someone's account
It's a same problem to creak RSA.
- In order to crack a block
You need to find an algorithm to generate a certain sha256 result, without brute force attacks。
- In order to crack a blockchain
As far as i knew, you need a longer blockchain. You publish the longer chain, and naturally all the network will trust you until there is a logger one appears.

## 2.General design
### 2.1 organization
	main.go             // 入口
	server              // 传输层入口 
		- http              // HTTP server to support remote operation
			- create            // create a block and spread it to all the peers
		    - join              // join a peer
		- command line      // TODO standard inputs to support local operation
	handlers            // 函数入口层
		- http              // HTTP server to support remote operation
            - create            // create a block and spread it to all the peers
            - join              // join a peer
	models
		- block             // block object
		- blockchain        // blockchain object
		- translaction      //transaction object
		- user              // user object
	protocal
	    - protocal          //blockchain spread protocal
	    - singleton         // maintain the singleton
	common
		- errno             // defined the error numbers
		- const             // defined const variables
	idl
	    - create            // defined the data struct of input and output
	    - join              // defined the data struct of input and output  

### 2.2 How to use

```
// step1: 构建&运行
go build
./blockchain -server :10024 -p2p :12345
```

```
// step2: 加入一个临近peer
curl -H 'content-type: application/json' -X POST -d '{"peer_addr":"xxxxxx"}' http://127.0.0.1:10024/blockchain/join
```

```
// step3: 发布一个区块
curl -H 'content-type: application/json' -X POST -d '{"name":"luda", "data":"first blockchain"}' http://127.0.0.1:10024/blockchain/create
```

```
// step4: 查看区块链
curl -H 'content-type: application/json' -X POST -d '{"chain":true, "peer":true}' http://127.0.0.1:10024/blockchain/show
```

```json
// 获得的json为
{
  "chain": {
    "chain": [
      {
        "pv_hash": "0",
        "timestamp": 1521084679901628400,
        "data": "This is Genesis Block, Copyright Belong to Blockchain-CN",
        "index": 0,
        "nonce": 520,
        "hash": "0068aa0c6393b0d617fb7f297eeaa1bf857dae2371fe7be55321f71363bb7a5b"
      },
      {
        "pv_hash": "0068aa0c6393b0d617fb7f297eeaa1bf857dae2371fe7be55321f71363bb7a5b",
        "timestamp": 1521084685874480600,
        "data": "{\"account\":\"MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDWzESbJDsfzkYpMtFqKr266wNin3A9+Gz6KICuL6VxwkHISCdaaMZ3joRvv5L01ld8BDki80Wi75srINIuTy+aXhwa6uywLsFg6K9i0Drqimde7ny4ie3WNgBaOz2h0ZshQeMzgkOryJ0wL24EQ02BejtM/vPL0o1Pf73QoxwsVwIDAQAB\",\"cipher\":\"Pel+VgGy2tCem66lMjbQ2/hXSLR+jYJ3PMFt8C7xgdTeDQAar3jiQvV3lLjfvi4Hde4M/LGGlbzrV42OdeJnHHIkYp6gaEXf+c7tR2Jwbg9qm0mljjv6rT6A2nFYfmxbSoxgd5cyGdzJ2l6v3IwPpk34sSo12DcBlG+SGWYe5z0=\",\"transaction\":\"block 1\"}",
        "index": 1,
        "nonce": 2207,
        "hash": "00fb9507a21e2bdf64a5e773b1cf5e43d0c8caafe2e2ab240ccd0e5a6ea23ff1"
      }
    ]
  },
  "peer": {}
}
```