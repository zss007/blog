---
title: blockchain 之比特币钱包实战
categories:
- project
---
最近几个月都在搞区块链，是时候总结一波了，这章介绍比特币区块链钱包实战。
<!--more-->
### 一、基础知识点
- 矿工关心每字节的费用（或千字节）。这是总费用除以事务中的字节数，例如 40 satoshis / byte 或 0.0004 bitcoins / kilobyte。这是矿工最重要的衡量标准。原因是他们使用它来决定是否将您的事务包含在他们尝试生成的块中，因为它们只能在其块中包含大约 100 万字节的事务。因此，他们更愿意包括每字节支付更多费用的交易
- 1 bitcoin (BTC 比特币) = 1000 millibitcoins (mBTC 毫比特) = 1000000 microbitcoins (uBTC 微比特) = 100000000 Satoshi （satoshi 聪）
- 如果一个人持有私钥，他就可以使用私钥对任意的消息进行签名。即通过私钥 sk 对消息 message 进行签名，得到 signature： sign(message, sk)
- imtoken 普通地址和隔离见证地址路径不同。普通地址： "m/44'/0'/0'/0/0"，隔离见证地址： "m/49'/0'/0'/0/0"
- testnet 测试网络环境下，助记词生成 btc 私钥，需要显性设置网络才能通过后面的签名，如：HDKey.fromMasterSeed(seed, network)
- 私钥可以表示为 64 位 16 进制串、Base58、WIF、WIF-compressed。64 位 16 进制串就是把私钥直接转换；Base58 编码就是对 64 位 16 进制串直接编码；WIF(Wallet Import Format) 就是在 64 位 16 进制串对应的字节串前加上前缀 0x80， 并用 Base58Check 编码；WIF-compressed 就是在 64 位 16 进制串对应的字节串前加上前缀 0x80，并加上后缀 0x01， 并用 Base58Check 编码

### 二、测试网络
- testnet 是一条比特币的备用链，用于开发测试的使用。测试用币从现实币中隔离区分开来，并且没有任何实际价值，仅作为开发的实验使用，不用担心在这条链上的交易会影响甚至使现实链崩溃。
- Faucets 网站是一类免费分享测试用币的网站，在这上面你可以按需获取一定数量的测试用币，但是为了网站的长期使用，所以希望各位开发者们在使用完以后，能够将测试用币归还到给定的收集地址。
  - 测试网查询浏览器 https://test-insight.bitpay.com
  - Faucets类网站 https://testnet.manu.backend.hamburg/faucet (最大方，给很多，推荐)

### 三、实战之账号生成
#### 1、生成助记词
```
const bip39 = require('bip39')

<!-- 如：that flock okay rubber promote right lift sell grow turtle camp shoulder -->
bip39.generateMnemonic()
```
#### 2、生成私钥
```
const bitcoin = require('bitcoinjs-lib')
const bip39 = require('bip39')
const bip32 = require('bip32')

// 在 imtoken 中使用助记词生成一般地址路径是 "m/44'/0'/0'/0/0"，隔离见证地址路径是 "m/49'/0'/0'/0/0"
let path = "m/44'/0'/0'/0/0"
const seed = bip39.mnemonicToSeed(mnemonic)

<!-- 测试网络 -->
const testnet = bitcoin.networks.testnet
const hdWallet = bip32.fromSeed(seed, testnet)
const keyPair = hdWallet.derivePath(path)
const privateKey = keyPair.toWIF()

<!-- 正式网络 -->
const hdWallet = bip32.fromSeed(seed)
const keyPair = hdWallet.derivePath(path)
const privateKey = keyPair.toWIF()
```
#### 3、私钥生成地址
```
const bitcoin = require('bitcoinjs-lib')

const testnet = bitcoin.networks.testnet

<!-- 测试网络一般地址 -->
const keyPair = bitcoin.ECPair.fromWIF(privateKey, testnet)
const address = bitcoin.payments.p2pkh({ pubkey: keyPair.publicKey, network: testnet }).address

<!-- 测试网络隔离见证地址 -->
const keyPair = bitcoin.ECPair.fromWIF(privateKey, testnet)
const address = bitcoin.payments.p2sh({
    redeem: bitcoin.payments.p2wpkh({ pubkey: keyPair.publicKey, network: testnet }),
    network: testnet
}).address

<!-- 正式网络一般地址 -->
const keyPair = bitcoin.ECPair.fromWIF(privateKey)
const address = bitcoin.payments.p2pkh({ pubkey: keyPair.publicKey }).address

<!-- 正式网络隔离见证地址 -->
const keyPair = bitcoin.ECPair.fromWIF(privateKey, testnet)
const address = bitcoin.payments.p2sh({
    redeem: bitcoin.payments.p2wpkh({ pubkey: keyPair.publicKey })
}).address
```
### 四、实战之交易查询
如果是个人研究学习，可以直接使用 insight 的 api，如果是企业级应用，可以使用解决方案 https://bitcore.io/guides/full-node ，源码及文档见 https://github.com/bitpay/insight-api
- 测试网络 insight 地址：https://test-insight.bitpay.com
- 正式网络 insight 地址：https://insight.bitpay.com

### 五、实战之发起交易
这里主要给交易签名，交易数据的广播用到的是 insight 的 api，这里就不介绍了。值得注意的是，如果 unspent 的交易未被确认或者确认次数过少，是存在风险的，但是确认次数过多会很影响交易速度，导致交易无法进行。
#### 1、获取转账矿工费和转账所需 upsents
```
const BigNumber = require('bignumber.js')

// 注意 txrefs 是 unspent 的交易数据，最好从大到小排序，减少转账手续费；amount 为转账金额；rate 是 satoshi/byte 值
function getFee(txrefs, amount, rate) {
    let total = 0, inputsNum = 0, txs = [], fee

    for (let ref of txrefs) {
        total += ref.value
        txs.push(ref)
        inputsNum++
        // 如果总额大于要转账金额
        if (total > amount) {
        fee = satoshi2Btc((148 * inputsNum + 34 * 1 + 10) * rate)  // 148 * inputsNum + 34 * 1 + 10 为 size 的估算公式
        if (total == (amount + fee)) {  // 如果总额等于矿工费加转账金额
            return { fee, txs, change: 0 }
        } else if (total > (amount + fee)) {  // 如果总额大于矿工费加转账金额
            // 更新旷工费用，设置找零地址
            fee = satoshi2Btc((148 * inputsNum + 34 * 2 + 10) * rate)
            if (total >= (amount + fee)) {
            return { fee, txs, change: total - amount - fee }
            }
        }
        }
    }

    return -1
}

function satoshi2Btc(amount) {
  return Number(BigNumber(amount).div(Math.pow(10, 8)).toString())
}
```
#### 2、交易签名
```
const bitcoin = require('bitcoinjs-lib')

let feeObj = getFee(txrefs, amount, rate)

<!-- 测试网络一般地址 -->
const fromObj = bitcoin.ECPair.fromWIF(privateKey, testnet)
const txb = new bitcoin.TransactionBuilder(testnet)
for (let tx of feeObj.txs) {  // 添加输入 unspent 交易
    txb.addInput(tx.tx_hash, tx.tx_output_n)
}
txb.addOutput(to, btc2Satoshi(amount))  // 添加收款人
feeObj.change != 0 ? txb.addOutput(from, btc2Satoshi(feeObj.change)) : ''  // 如果有余额，设置找零地址为付款人
for (let flag = 0; flag < feeObj.txs.length; flag++) {      // 签名交易
    txb.sign(flag, fromObj)
}
const rawtransaction = txb.build().toHex() // 得到签名交易数据

<!-- 测试网络隔离见证地址 -->
const fromObj = bitcoin.ECPair.fromWIF(privateKey, testnet)
const txb = new bitcoin.TransactionBuilder(testnet)
const fromObjP2SH = bitcoin.payments.p2sh({
    redeem: bitcoin.payments.p2wpkh({ pubkey: fromObj.publicKey, network: testnet }),
    network: testnet
})
txb.addOutput(to, btc2Satoshi(amount))  // 添加收款人
feeObj.change != 0 ? txb.addOutput(from, btc2Satoshi(feeObj.change)) : ''  // 如果有余额，设置找零地址为付款人
for (let flag = 0; flag < feeObj.txs.length; flag++) {      // 签名交易
    txb.sign(flag, fromObj, fromObjP2SH.redeem.output, null, btc2Satoshi(feeObj.txs[flag].value))
}
const rawtransaction = txb.build().toHex() // 得到签名交易数据

<!-- 正式网络 -->
// 将测试网络对应地址方法中的 testnet 删除即可。
```
