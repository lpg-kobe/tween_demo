
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

### 任务二 数据上链及数据链上查询

实验步骤：

1. 创建账本

```shell
./create-ledger.sh
```
创建成功提示
![Alt text](./img/create-ledger.png)

2. 加入账本

```javascript
./join-ledger.sh 0 1 // 在组织1的0节点加入账本
./join-ledger.sh 1 1 // 在组织1的1节点加入账本
./join-ledger.sh 0 2 // 在组织2的0节点加入账本
./join-ledger.sh 1 2 // 在组织2的1节点加入账本
```
加入成功提示
![Alt text](./img/join-ledger.png)

3. 安装链码

```javascript
./install.sh 0 1 // 为组织1的0节点安装链码
./install.sh 1 1 // 为组织1的1节点安装链码
./install.sh 0 2 // 为组织2的0节点安装链码
./install.sh 1 2 // 为组织2的1节点安装链码
```
安装成功提示
![Alt text](./img/i-chaincode.png)

4. 初始化链码

```javascript
./init.sh 0 2 OR // 为组织2的0节点初始化链码
```
初始化成功提示
![Alt text](./img/init-chaincode.png)

### 任务二 编写并发布智能合约，实现通过hash比较的方法，判断数据是否相等

实验步骤：

1. 部署合约

```javascript
./invoke.sh key 'name:sjr,age:18' '0,2' // 为组织2的0节点部署{key:key,value:'name:sjr,age:18'}合约
```
部署成功提示

2. 查询部署的合约

```javascript
./query.sh key 0 2 // 查询组织2下的0节点key=key的合约
```

*******************************
.*(END)*
