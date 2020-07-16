
# **区块链核心技术-Fabric网络实践**

## 实验名称

掌握联盟区块链分布式网络原理及使用

## 案例说明

- 与公共区块链分布式网络不同，联盟区块链分布式网络有准入机制，有方便的的智能合约发布机制，
- 有高效可靠的共识机制，因此，联盟区块链分布式网络将更贴合实际的业务场景，将会在区块链的发展中脱颖而出。
- 本实验分3个场景，介绍联盟区块链分布式网络的原理及使用

## 实验目的

1. 掌握联盟区块链分布式网络原理
2. 熟悉分布式系统的使用
3. 掌握智能合约的编写及发布

## 实验任务

1. 搭建联盟区块链分布式网络
2. 数据上链及数据链上查询
3. 编写并发布智能合约，实现通过hash比较的方法，判断数据是否相等

## 实验环境

需要一台linux机器和一台PC，PC不做要求

硬件环境

- 双核(及以上)处理器
- 4G(及以上)内存
- 30G(及以上硬盘)
- 千兆网口

软件环境

- ubuntu 18及以上 64位操作系统
- ssh远程连接工具(如sourceCRT)

## 实验者要求

- 了解linux系统及基本操作命令
- 了解go语言基本语法知识
- 了解hash原理

## 实验指南

### 任务一 搭建联盟区块链分布式网络

实验步骤：

1. 启动前清理之前的Fabric网络信息

```shell
./network-mng.sh -m down
```
在提示信息中输入y清理
 ![Alt text](./img/clear-fabric.png)

2. 启动Fabric网络

```shell
./network-mng.sh -m up
```
启动成功提示
 ![Alt text](./img/up-fabric-ok.png)
 
3. 创建账本

```shell
./create-ledger.sh
```
创建成功提示
![Alt text](./img/create-ledger.png)

4. 加入账本

```javascript
./join-ledger.sh 0 1 // 在组织1的0节点加入账本
./join-ledger.sh 1 1 // 在组织1的1节点加入账本
./join-ledger.sh 0 2 // 在组织2的0节点加入账本
./join-ledger.sh 1 2 // 在组织2的1节点加入账本
```
加入成功提示
![Alt text](./img/join-ledger.png)

5. 安装链码

```javascript
./install.sh 0 1 // 为组织1的0节点安装链码
./install.sh 1 1 // 为组织1的1节点安装链码
./install.sh 0 2 // 为组织2的0节点安装链码
./install.sh 1 2 // 为组织2的1节点安装链码
```
安装成功提示
![Alt text](./img/i-chaincode.png)

6. 初始化链码

```javascript
./init.sh 0 2 OR // 为组织2的0节点初始化链码
```
初始化成功提示
![Alt text](./img/init-chaincode.png) 

### 任务二 数据上链及数据链上查询

实验步骤：

1. 数据记录到区块链上

```javascript
./invoke.sh key1 'name:sjr,age:18' '0,2' // 将信息'name:sjr,age:18'以key1为关键字，请求组织2的节点0签名并记录到区块链上
```
记录成功提示
![Alt text](./img/upload-common.png)

3. 分别从两个节点上查询数据，如成功则说明区块链分布式账本成功同步

```javascript
./query.sh key1 0 1 // 查询组织1下的0节点key=key1的合约
./query.sh key1 1 1 // 查询组织1下的1节点key=key1的合约
```

### 任务三 编写并发布智能合约，实现通过hash比较的方法，判断数据是否相等

实验步骤：

1. 在原来的智能合约基础上增加compare方法，比较数据是否一致

* 智能合约相对路径chaincode/traceability/go/agent.go

```go
/*
Author: sjr
*/

package main

import (
	"fmt"
	"bytes"
	"time"
	"strconv"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

type marble struct {
	ObjectType string `json:"docType"` //docType is used to distinguish the various types of objects in state database
	Name       string `json:"name"`    //the fieldtags are needed to keep case from bouncing around
	Color      string `json:"color"`
	Size       int    `json:"size"`
	Owner      string `json:"owner"`
}


var logger = shim.NewLogger("receivable_issue")

// ReceivableChaincode receivable chaincode implementation
type ReceivableChaincode struct {
}

// Init initializes the chaincode state
func (t *ReceivableChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	//logger.Info("########### ReceivableChaincode Init ###########")
	return shim.Success(nil)

}

// Invoke makes a value setting to a key
func (t *ReceivableChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	//logger.Info("########### ReceivableChaincode Invoke ###########")
	
	function, args := stub.GetFunctionAndParameters()
	//logger.Infof("function: %s, args: \n", function, args)
	if function == "get" {
		// get an entity state
		return t.get(stub, args)
	}

	if function == "set" {
		// set an entity from its state
		return t.set(stub, args)
	}
	
	if function == "gethistory" {
		// get an entity state
		return t.gethistory(stub, args)
	}	

	logger.Errorf("Unknown action, check the first argument, must be one of 'get', or 'set'. But got: %v", args[0])
	return shim.Error(fmt.Sprintf("Unknown action, check the first argument, must be one of 'get', or 'set'. But got: %v", args[0]))
}

func (t *ReceivableChaincode) set(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	// must be an invoke
	var key, val string
	var err error

	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 key and 1 value")
	}

	key = args[0]
	val = args[1]

	if key == "" {
		return shim.Error("'key' can not be null")
	}

	if val == "" {
		return shim.Error("'val' can not be null")
	}

	// Write the state back to the ledger
	err = stub.PutState(key, []byte(val))
	if err != nil {
		return shim.Error(err.Error())
	}


	return shim.Success(nil)
}

// get callback representing the get of a chaincode
func (t *ReceivableChaincode) get(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var err error
	var key string

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting a key to get")
	}

	key = args[0]
	
	if key == "" {
		return shim.Error("'key' can not be null")
	}

	// Get the state from the ledger
	valbytes, err := stub.GetState(key)
	if err != nil {
		jsonResp := "{\"Error\":\"Failed to get state for " + key + "\"}"
		return shim.Error(jsonResp)
	}

	if valbytes == nil {
		jsonResp := "{\"Error\":\"Nil amount for " + key + "\"}"
		return shim.Error(jsonResp)
	}

	//jsonResp := "{\"key\":\"" + key + "\",\"value\":\"" + string(valbytes) + "\"}"
	//logger.Info("get Response:%s\n", jsonResp)
	return shim.Success(valbytes)
}

func (t *ReceivableChaincode) gethistory(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var key string

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting a key to get")
	}

	key = args[0]
	
	if key == "" {
		return shim.Error("'key' can not be null")
	}

	// Get the state from the ledger
	resultsIterator, err := stub.GetHistoryForKey(key)
	if err != nil {
		jsonResp := "{\"Error\":\"Failed to get history for " + key + "\"}"
		return shim.Error(jsonResp)
	}
	
	defer resultsIterator.Close()

    // buffer is a JSON array containing historic values for the marble
    var buffer bytes.Buffer
    buffer.WriteString("[")

    bArrayMemberAlreadyWritten := false
    for resultsIterator.HasNext() {
        response, err := resultsIterator.Next()
        if err != nil {
            return shim.Error(err.Error())
        }
        
        // Add a comma before array members, suppress it for the first array member
        if bArrayMemberAlreadyWritten == true {
            buffer.WriteString(",")
        }
        buffer.WriteString("{\"TxId\":")
        buffer.WriteString("\"")
        buffer.WriteString(response.TxId)
        buffer.WriteString("\"")

        buffer.WriteString(", \"Value\":")
        // if it was a delete operation on given key, then we need to set the
        //corresponding value null. Else, we will write the response.Value
        //as-is (as the Value itself a JSON marble)
        if response.IsDelete {
            buffer.WriteString("null")
        } else {
            buffer.WriteString(string(response.Value))
            //logger.Info(response)            
        }

        buffer.WriteString(", \"Timestamp\":")
        buffer.WriteString("\"")
        buffer.WriteString(time.Unix(response.Timestamp.Seconds, int64(response.Timestamp.Nanos)).String())
        buffer.WriteString("\"")

        buffer.WriteString(", \"IsDelete\":")
        buffer.WriteString("\"")
        buffer.WriteString(strconv.FormatBool(response.IsDelete))
        buffer.WriteString("\"")

        buffer.WriteString("}")
        bArrayMemberAlreadyWritten = true
    }
    buffer.WriteString("]")

    fmt.Printf("- getHistoryForMarble returning:\n%s\n", buffer.String())

    return shim.Success(buffer.Bytes())
}

func main() {
	err := shim.Start(new(ReceivableChaincode))
	if err != nil {
		logger.Errorf("Error starting receivable chaincode: %s", err)
	}
}
```

* 参考的智能合约相对路径chaincode/traceability_ref/go/agent.go

```go
/*
Author: sjr
*/

package main

import (
	"crypto/md5"
	"io"
	"fmt"
	"bytes"
	"time"
	"strconv"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

type marble struct {
	ObjectType string `json:"docType"` //docType is used to distinguish the various types of objects in state database
	Name       string `json:"name"`    //the fieldtags are needed to keep case from bouncing around
	Color      string `json:"color"`
	Size       int    `json:"size"`
	Owner      string `json:"owner"`
}


var logger = shim.NewLogger("receivable_issue")

// ReceivableChaincode receivable chaincode implementation
type ReceivableChaincode struct {
}

// Init initializes the chaincode state
func (t *ReceivableChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	//logger.Info("########### ReceivableChaincode Init ###########")
	return shim.Success(nil)

}

// Invoke makes a value setting to a key
func (t *ReceivableChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	//logger.Info("########### ReceivableChaincode Invoke ###########")
	
	function, args := stub.GetFunctionAndParameters()
	//logger.Infof("function: %s, args: \n", function, args)
	if function == "get" {
		// get an entity state
		return t.get(stub, args)
	}

	if function == "set" {
		// set an entity from its state
		return t.set(stub, args)
	}

	if function == "compare" {
		// set an entity from its state
		return t.compare(stub, args)
	}
	
	if function == "gethistory" {
		// get an entity state
		return t.gethistory(stub, args)
	}	

	logger.Errorf("Unknown action, check the first argument, must be one of 'get', or 'set'. But got: %v", args[0])
	return shim.Error(fmt.Sprintf("Unknown action, check the first argument, must be one of 'get', or 'set'. But got: %v", args[0]))
}

func (t *ReceivableChaincode) compare(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	// must be an invoke
	var str1, str2 string
	
	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 key and 1 value")
	}

	str1 = args[0]
	str2 = args[1]
	
	if str1 == "" {
		return shim.Error("'str1' can not be null")
	}
	
	if str2 == "" {
		return shim.Error("'str2' can not be null")
	}

	t1 := md5.New()
	io.WriteString(t1, str1)
	s1 := fmt.Sprintf("%x", t1.Sum(nil))

	t2 := md5.New()
	io.WriteString(t2, str2)
	s2 := fmt.Sprintf("%x", t2.Sum(nil))

	var buffer bytes.Buffer
	if s1 != s2 {
		buffer.WriteString("Unequal strings!")
	} else {
		buffer.WriteString("Equal string!")
	}

	return shim.Success(buffer.Bytes())
}

func (t *ReceivableChaincode) set(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	// must be an invoke
	var key, val string
	var err error

	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 key and 1 value")
	}

	key = args[0]
	val = args[1]

	if key == "" {
		return shim.Error("'key' can not be null")
	}

	if val == "" {
		return shim.Error("'val' can not be null")
	}

	// Write the state back to the ledger
	err = stub.PutState(key, []byte(val))
	if err != nil {
		return shim.Error(err.Error())
	}


	return shim.Success(nil)
}

// get callback representing the get of a chaincode
func (t *ReceivableChaincode) get(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var err error
	var key string

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting a key to get")
	}

	key = args[0]
	
	if key == "" {
		return shim.Error("'key' can not be null")
	}

	// Get the state from the ledger
	valbytes, err := stub.GetState(key)
	if err != nil {
		jsonResp := "{\"Error\":\"Failed to get state for " + key + "\"}"
		return shim.Error(jsonResp)
	}

	if valbytes == nil {
		jsonResp := "{\"Error\":\"Nil amount for " + key + "\"}"
		return shim.Error(jsonResp)
	}

	//jsonResp := "{\"key\":\"" + key + "\",\"value\":\"" + string(valbytes) + "\"}"
	//logger.Info("get Response:%s\n", jsonResp)
	return shim.Success(valbytes)
}

func (t *ReceivableChaincode) gethistory(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var key string

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting a key to get")
	}

	key = args[0]
	
	if key == "" {
		return shim.Error("'key' can not be null")
	}

	// Get the state from the ledger
	resultsIterator, err := stub.GetHistoryForKey(key)
	if err != nil {
		jsonResp := "{\"Error\":\"Failed to get history for " + key + "\"}"
		return shim.Error(jsonResp)
	}
	
	defer resultsIterator.Close()

    // buffer is a JSON array containing historic values for the marble
    var buffer bytes.Buffer
    buffer.WriteString("[")

    bArrayMemberAlreadyWritten := false
    for resultsIterator.HasNext() {
        response, err := resultsIterator.Next()
        if err != nil {
            return shim.Error(err.Error())
        }
        
        // Add a comma before array members, suppress it for the first array member
        if bArrayMemberAlreadyWritten == true {
            buffer.WriteString(",")
        }
        buffer.WriteString("{\"TxId\":")
        buffer.WriteString("\"")
        buffer.WriteString(response.TxId)
        buffer.WriteString("\"")

        buffer.WriteString(", \"Value\":")
        // if it was a delete operation on given key, then we need to set the
        //corresponding value null. Else, we will write the response.Value
        //as-is (as the Value itself a JSON marble)
        if response.IsDelete {
            buffer.WriteString("null")
        } else {
            buffer.WriteString(string(response.Value))
            //logger.Info(response)            
        }

        buffer.WriteString(", \"Timestamp\":")
        buffer.WriteString("\"")
        buffer.WriteString(time.Unix(response.Timestamp.Seconds, int64(response.Timestamp.Nanos)).String())
        buffer.WriteString("\"")

        buffer.WriteString(", \"IsDelete\":")
        buffer.WriteString("\"")
        buffer.WriteString(strconv.FormatBool(response.IsDelete))
        buffer.WriteString("\"")

        buffer.WriteString("}")
        bArrayMemberAlreadyWritten = true
    }
    buffer.WriteString("]")

    fmt.Printf("- getHistoryForMarble returning:\n%s\n", buffer.String())

    return shim.Success(buffer.Bytes())
}

func main() {
	err := shim.Start(new(ReceivableChaincode))
	if err != nil {
		logger.Errorf("Error starting receivable chaincode: %s", err)
	}
}
```

2. 将修改后的智能合约发布到联盟区块链网络上

```javascript
./install.sh 0 1 // 为组织1的0节点安装链码
./install.sh 1 1 // 为组织1的1节点安装链码
./install.sh 0 2 // 为组织2的0节点安装链码
./install.sh 1 2 // 为组织2的1节点安装链码
./update.sh 0 2 OR // 为组织2的0节点初始化链码
```
部署成功提示
![Alt text](./img/upload-common.png)

3. 分别从两个节点上查询数据

```javascript
./query.sh key1 0 1 // 查询组织1下的0节点key=key1的数据 data01
./query.sh key1 0 2 // 查询组织2下的0节点key=key1的数据 data02
```

4. 调用新加的智能合约功能，完成数据的比较

```js
./compare.sh data01 data02 0 2 // 调用组织2的节点0的智能合约比较方法，比较data01和data02是否相等，如不相等则发生了篡改
```
对比结果
![Alt text](./img/compare-equal.png)

*******************************
.*(END)*
