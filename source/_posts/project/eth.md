---
title: blockchain 之 eth 钱包实战
categories:
- project
---
这章介绍 eth（以太坊）区块链钱包实战。
<!--more-->
### 一、实战之账号生成
#### 1.1、生成助记词
```
const bip39 = require('bip39')

<!-- 如：that flock okay rubber promote right lift sell grow turtle camp shoulder -->
bip39.generateMnemonic()
```
#### 1.2、助记词生成私钥
```
const bip39 = require('bip39')
const HDKey = require('hdkey')

// 在 imtoken 中使用助记词地址路径是 "m/44'/60'/0'/0/0"
let path = "m/44'/60'/0'/0/0"
const seed = bip39.mnemonicToSeed(mnemonic)
const hdWallet = HDKey.fromMasterSeed(seed)
const keyPair = hdWallet.derivePath(path)
const privateKey = '0x' + keyPair.privateKey.toString('hex')
```
#### 1.3、私钥生成账号
```
const Web3 = require('web3')
<!-- 可在 infura 申请免费连接节点 -->
const web3 = new Web3(new Web3.providers.HttpProvider("https://mainnet.infura.io/v3/f06735caf2964f6881c7a219b8196471"))

const account = web3.eth.accounts.privateKeyToAccount(privateKey)
```
#### 1.4、随机创建账号
```
<!-- web3 配置同私钥生成账号 -->
const account = web3.eth.accounts.create()
```
#### 1.5、生成备份 keystore
```
<!-- web3 配置同私钥生成账号，privateKey 为私钥，password 为加密密码 -->
const keyStore = JSON.stringify(web3.eth.accounts.encrypt(privateKey, password))
```
#### 1.6、keyStore 导入
```
<!-- web3 配置同私钥生成账号，password 为加密密码 -->
const account = web3.eth.accounts.decrypt(keyStore, password)
```
### 二、实战之交易
#### 2.1、获取账户余额
```
const BigNumber = require('bignumber.js')

<!-- 获取比特币余额 -->
const balance = await web3.eth.getBalance(address)
const res = web3.utils.fromWei(balance, 'ether')

<!-- 获取代币余额，contractAddress 为合约地址，contractAbi 为以太坊 erc20 标准的 abi -->
const contractAbi = [
    { "constant": true, "inputs": [], "name": "name", "outputs": [{ "name": "", "type": "string" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "symbol", "outputs": [{ "name": "", "type": "string" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "decimals", "outputs": [{ "name": "", "type": "uint8" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [{ "name": "", "type": "uint256" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [{ "name": "_owner", "type": "address" }], "name": "balanceOf", "outputs": [{ "name": "balance", "type": "uint256" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [{ "name": "_to", "type": "address" }, { "name": "_value", "type": "uint256" }], "name": "transfer", "outputs": [{ "name": "", "type": "bool" }], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": false, "inputs": [{ "name": "_spender", "type": "address" }, { "name": "_value", "type": "uint256" }], "name": "approve", "outputs": [{ "name": "", "type": "bool" }], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": false, "inputs": [{ "name": "_from", "type": "address" }, { "name": "_to", "type": "address" }, { "name": "_value", "type": "uint256" }], "name": "transferFrom", "outputs": [{ "name": "", "type": "bool" }], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [{ "name": "_owner", "type": "address" }, { "name": "_spender", "type": "address" }], "name": "allowance", "outputs": [{ "name": "", "type": "uint256" }], "payable": false, "stateMutability": "view", "type": "function" },
    { "inputs": [], "payable": false, "stateMutability": "nonpayable", "type": "constructor" },
    { "anonymous": false, "inputs": [{ "indexed": true, "name": "owner", "type": "address" }, { "indexed": true, "name": "spender", "type": "address" }, { "indexed": false, "name": "value", "type": "uint256" }], "name": "Approval", "type": "event" },
    { "anonymous": false, "inputs": [{ "indexed": true, "name": "from", "type": "address" }, { "indexed": true, "name": "to", "type": "address" }, { "indexed": false, "name": "value", "type": "uint256" }], "name": "Transfer", "type": "event" }
]
const contract = new web3.eth.Contract(contractAbi, contractAddress)
const balance = await contract.methods.balanceOf(address).call()
const decimals = await contract.methods.decimals().call()
const res = BigNumber(balance).div(Math.pow(10, decimals)).decimalPlaces(4)
```
#### 2.2、获取推荐 GasPrice
```
const gasPrice = await web3.eth.getGasPrice()
const res = web3.utils.fromWei(gasPrice, 'gwei')
```
#### 2.3、获取推荐 GasLimit
```
<!-- 如果是以太币交易直接返回 21000，payeeAddress 为收款人地址，address 为转账人地址，contractAddress 为合约地址 -->
let formatPayeeAddress = _addPreZero(payeeAddress.substr(2))
let formatAmount = _addPreZero(web3.utils.toHex(web3.utils.toWei(amount, 'ether')).substr(2))
const gasLimit = await web3.eth.estimateGas({
    from: address,
    to: contractAddress,
    <!-- '0xa9059cbb' 为固定前缀；formatPayeeAddress 先去掉 '0x'，再补全 64 位；formatAmount 先转为 wei 单位，然后转成 hex 格式，去掉前面 '0x'，再补全 64 位 -->
    data: `0xa9059cbb${formatPayeeAddress}${formatAmount}`
})

// 补齐64位，不够前面用0补齐
function _addPreZero(num) {
    var t = (num + '').length,
        s = ''
    for (var i = 0; i < 64 - t; i++) {
        s += '0'
    }
    return s + num
}
```
### 三、实战之查询
通过自建 mongodb 数据库，读取整个 eth 链上的数据到库中，实现代币查询和交易历史查询功能。具体实现可参考 https://github.com/TrustWallet/trust-ray/ 。
#### 3.1、遇到的问题
- web3.js 版本问题带来的解析错误，解决方法：
```
 // 兼容性处理，新版本 web3 报错，换一种写法
 if (abi.name.toLowerCase() == 'decimals') {
    const value = await web3.eth.call({
        to: contractAddress,
        data: contractInstance.methods[abi.name]().encodeABI()
    })
    return web3.utils.hexToNumber(value)
 }
```
- 特殊数据报错，解决方法：
```
 // 0xe065f822c8184971ab2598663f5db8d4cce72ca7bd8f3b69666129eaea32af26
 let decodedLogs
 try {
    decodedLogs = abiDecoder.decodeLogs(transaction.receipt.logs).filter((log) => log)
 } catch(error) {
    logger.info(`Could not parse transaction with error: ${transaction._id}`);
    decodedLogs = []
 }
```
- 部分合约不规范，导致无法解析，拟手动修复。或在代币搜索页面用户提交未覆盖的代币，然后进行修改
- 内存溢出，默认 v8 内存固定，当超出内存时导致溢出。手动修改内存空间，解决方案如下：
```
 <!--添加 --max_old_space_size=4096 参数配置-->
 "dev": "cross-env EGG_SERVER_ENV=prod NODE_ENV=prod node --max_old_space_size=4096 dispatch.js"
```
- mongodb 空间不足，预计需要六七百 G
- 测试过不同并发量，最终确定最佳并发量为 10。