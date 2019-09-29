---
title: blockchain 之 eos 钱包实战
categories:
- project
---
这章介绍 eos（Enterprise Operation System）区块链钱包实战。
<!--more-->
### 一、基础知识点
- EOS 带着免费的口号来，不过事实上注册一个账号就要 100 多人民币， DAPP 的开销就更大了。但是确实也有一些好东西是免费的，一是消耗的 CPU 和 NET 以一天为周期自动清零，二是不涉及到 table 的数据是免费的
- 去中心化和高 TPS 其实是一对矛盾体，越去中心化，效率越低，TPS 自然就低。因此 DPOS 应运而生，投票选出 21 个代表，决策者只有 21 个代表，而不是 POW 的全民运动，自然决策效率高
- 账号执行的 action 需要写数据时就需要消耗 ram
- staked 是自己的 EOS 量，delegated 是别人帮忙垫付的 EOS 量，这两部分都是抵押的 EOS，用来预购买 cpu bandwidth 和 net bandwidth 的。可以把 staked 中的 EOS 提现到 balance 里，让这些 EOS 恢复自由。提现是有时间延时的，3 天后才能退回到 balance 里，同时已经花掉的 eos 没法立马提现，必须随着系统总带宽增加慢慢提现
- balance: 和网上所说的 unstaked eos 是一个东西，即用户可以自由转账的 EOS 数值
- memory: 就是 ram， 新建账号时，账号数据会写到数据库里，因而需要购买 ram。但是先有账号才能购买 ram, 而创建账号又必须要有 ram，所以这里存在一个先有鸡还是先有蛋的问题，一般需要有账号的人帮忙创建
- stake 抵押原理: 当我们抵押一部分 EOS 时，其实会引发一次转账。会从当前账号转走对应的 EOS 至 eosio.stake 账号(可认为是往 eosio.stake 账号充值)，同时记录来源，方便后期从 eosio.stake 提现，提现相当于充值一个负值
- ram 购买: 扣除 0.5% 手续费后的 EOS 转到购买 ram 的账号 eosio.ram，根据 ram 市场里的 EOS 和 ram 实时汇率计算出能够购买的 ram 量，然后更新该账号的 ram 拥有量。ram 购买的量是一个绝对值，是根据购买时市场内 EOS 和 ram 的汇率计算出来的。一般来说在 ram 总量不增加的情况下，ram 会越来越贵。所以如果早期你购买了 ram，然后过段时间后卖掉 ram 可能还能挣钱
- cpu、net 收费逻辑: cpu，net 的消耗量是绝对值，以 kb 计量，而 cpu， net 的拥有量不是以 kb 计量的，而是以抵押的 EOS 计量保存，在余额检测时才实时转为具体的绝对 kb 值进行比较。全网的网络抵押的 EOS 越多，账号能用的网络带宽就越少，即和抵押的 EOS 总量相关，而不是和全部发行的 EOS 量相关。首先获取窗口时间内的全网网络流量总量、该账号的网络 EOS 抵押量、全网所有用户的网络总 EOS 抵押量，根据占据 EOS 抵押量的比例，获取窗口时间内账号可用最大网络量
- cpu、net 购买: 购买 cpu、 net 资源是通过抵押 EOS 实现的
- cpu，net 的消耗量是临时的，过一段时间可以清空，可以继续使用。间隔的时间越长，上次的消耗量占比越低；历史消耗量要乘上一个衰减系数，即总消耗量不是直接历史消耗量 + 本次消耗量；超过一个窗口时间后，历史消耗量清零，也就相当于未曾消耗过；源码窗口时间是 1 天。有可能执行一次转账后，cpu used 反而变小了，就是因为衰减
- 存在账号 ram，cpu 资源用尽了，无法执行如何操作，比如创建新账号失败，因为 cpu 不够。然后想着购买 cpu，然后因为 ram、net 不够失败。那购买 ram、net 仍旧失败，因为没有足够的 cpu。所以进入一个死循环，没有 ram 没法购买 cpu，没有 cpu 没法购买 ram。要解决这个问题，只能再找一个账号，让别人给你购买一些 ram 或 cpu
- 存在突发事件导致账号冻结。2018.7.22 的 23 点到 24 点短短 1 个小时，全网 CPU 抵押的 EOS 快速激增了一倍，导致每个人可以用的 CPU 减少了一半，进而很多账号的 CPU 都超支了，就是已使用的 CPU 超过了用户拥有的 CPU 量，进而处于僵死状态，啥都不能做。因为用户的可用 CPU 量是动态变化的，按用户的 CPU 抵押量占全网的总抵押量的比例来分配可使用的 CPU 量，而用户的 CPU 已使用量(used)是不变的，所以当用户的 CPU 可使用量(limit)动态下降到一定程度就出现超支了。全网 CPU 抵押量激增主要是 TPS 大增导致，但是短时间的激增是因为突发事件导致每个账号的 CPU 可使用量快速下降，进而导致很多账号失活，然后解决失活的 CPU 抵押操作进一步增加全网 CPU 的抵押量，进一步降低了账号的 CPU 可使用量，从而更多账号失活，因而这是一个恶性循环。但是这个恶性循环有它的截止点，因为大部分用户在账号失活后(当前账号没法抵押 EOS 换取更多 CPU)，没有渠道快速抵押 EOS 换取 CPU 来重新激活。不着急操作的账号就等，等一天或者两天不操作，哪怕你 CPU 超支了，超支的再多也不用怕，你只需执行很小的 CPU 抵押操作，系统就会帮你清零已使用的 CPU
- EOS 钱包天生就是多签钱包，每一个账户都有两个以上的私钥与之对应, 但是多把私钥, 可以是相同的。比如, 一个 EOS 账号为 imtokensimon, 这个账户的 owner 私钥是 123, active 的私钥可以设定为 123, 也可以是 456。Owner 私钥代表着一个 EOS 账户的最高权限, 它可以用来分配并重置 Active 私钥的权限。Active 私钥可以做除了修改 Owner 以外所有的事, 基本来说, 用户的转账授权等操作, 都是通过 Active 私钥来完成的
- 用户可以使用冷存储的方式, 存储 Owner 私钥, 这样如果你的 Active 私钥泄露了, 可以使用 Owner 私钥重置你的 Active 私钥, 但需要注意的是, 如果你的 Active 私钥泄露了, 黑客也能够转移你的资产
- memo（备注）最多存 256 字节,可以放中文
- 抵押/赎回 CPU 和 NET 资源: A 以租借的形式为 B 抵押资源，A 可以赎回，B 无法赎回； A 以赠送的方式为 B 抵押资源，A 无法赎回，B 可以赎回； B 自己抵押资源并且可以赎回。赎回资源时，EOS 不会马上到账，而是会在三天后到账。你无法赎回已经使用的资源，需要在该资源自动释放后才可赎回。CPU 和 NET 资源是系统自动释放的，也就是已使用的这部分资源会在一段时间后恢复并可以再次使用。抵押和赎回也是通过 eosio.token 的 transfer 方法，与系统账户 eosio.stake 相互转账，系统不会收取手续费
- 购买/出售内存: 内存资源是通过使用 EOS Token 购买获得的，可以由其他账户为你购买，也可以自己购买。不管是前面哪种形式为你购买的内存，都是属于你的，可以出售，获得的 EOS Token 也都会立刻打入到你的账户。本质上，购买和出售内存，都是通过 eosio.token 的transfer 方法，与系统账户 eosio.ram 相互转账。同时，系统会在每次交易中(包括购买和出售)收取本次交易中 EOS 总额的 5% 作为手续费，也就是向 eosio.ramfee 转账。需要注意的是，内存资源是不可自动恢复的
- 其实只需要为新账户购买内存资源，否则会创建失败。而抵押 CPU 和 NET 资源可以不在创建账户时同时进行。但一般来说，为了使新账户可以操作一些事情，例如转账，都会在创建时同时为其抵押 CPU 和 NET 资源。这时就会出现一笔 transaction 中包含多个 action 的情况

### 二、测试网络
- JungleTestnet 是一条 eos 的备用链，用于开发测试的使用。测试用币从现实币中隔离区分开来，并且没有任何实际价值，仅作为开发的实验使用，不用担心在这条链上的交易会影响甚至使现实链崩溃。
- 可在 http://jungle.cryptolions.io/#faucet 上免费领取到测试币
- 可在 http://jungle.cryptolions.io/#account 上免费创建新账号
- 可在 http://jungle.cryptolions.io/#apiendpoints 上获取免费的测试节点

### 三、实战之账号
#### 1、种子生成秘钥对
```
const EOS = require('eosjs')
const { ecc } = EOS.modules

<!-- seed 为生成账号种子，相同种子生成的私钥相同，可借助助记词 -->
let privateKey = ecc.seedPrivate(seed)
let publicKey = ecc.privateToPublic(privateKey)
```
#### 2、私钥导入
```
const EOS = require('eosjs')
const { ecc } = EOS.modules

let publicKey = ecc.privateToPublic(privateKey)
```
#### 3、判断公钥是否有效
```
const EOS = require('eosjs')
const { ecc } = EOS.modules

ecc.isValidPublic(publicKey)
```
#### 4、判断私钥是否有效
```
const EOS = require('eosjs')
const { ecc } = EOS.modules

ecc.isValidPrivate(privateKey)
```
#### 5、根据公钥获取账号名
```
let config = {
    httpEndpoint: 'http://jungle.cryptolions.io:18888',  // eos 测试网络节点
    expireInSeconds: 60,
    chainId: '038f4b0fc8ff18a4f0842a8f0564611f6e96e8535901dd45e43ac8691a1c4dca',  // 节点的 chainId 信息，可通过 eos.getInfo({}) 获取
    broadcast: true,
    debug: false,
    sign: true
}
let eos = EOS(config)

eos.getKeyAccounts(publicKey).then(res => {
    if (res.account_names.length > 0) {
        console.log(res.account_names)  // 账号名，一个公钥可对应多个账号
    } else {
        console.log('该地址在网络中无账号，请先注册账号！')
    }
}).catch(err => {
    console.log(err)
})
```
#### 6、根据账号名获取账号信息
```
<!-- config 同根据公钥获取账号名 -->
let eos = EOS(config)

eos.getAccount({ account_name: name }).then(res => {
    cosnole.log(res)
}).catch(e => {
    console.log(e)
})
```
#### 7、创建账号
```
let config = {
    httpEndpoint: 'http://jungle.cryptolions.io:18888',  // eos 测试网络节点
    expireInSeconds: 60,
    chainId: '038f4b0fc8ff18a4f0842a8f0564611f6e96e8535901dd45e43ac8691a1c4dca',
    keyProvider: privateKey,  // 创建者账户私钥
    broadcast: true,
    debug: false,
    sign: true
}
let eos = EOS(config)

eos.transaction(tr => {
    tr.newaccount({
        creator: creator,   // 创建者账户名称
        name: accountName,  // 新账户名称
        owner: owner,       // owner 权限公钥
        active: active      // active 权限公钥
    })
    tr.buyrambytes({
        payer: creator,
        receiver: accountName,
        bytes: Number(bytes)// 购买存储空间字节数，如 8192
    })
    tr.delegatebw({
        from: creator,
        receiver: accountName,
        stake_net_quantity: stake_net_quantity,  // 抵押 net 数
        stake_cpu_quantity: stake_cpu_quantity,  // 抵押 cpu 数
        transfer: transfer                       // transfer 设置为 1 时，抵押的 EOS，会转到新账户
    })
}).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
### 四、实战之资源
#### 1、抵押带宽
```
<!-- config 同账号创建 -->
let eos = EOS(config)

let form = {
    from,                   // 账户名称
    receiver,               // 抵押接收人
    stake_net_quantity,     // 抵押 net 数量，如 10.0000 EOS
    stake_cpu_quantity,     // 抵押 cpu 数量，如 10.0000 EOS
    transfer,               // transfer 设置为 1 时，抵押的 EOS，会转到新账户（如果自己给自己抵押，不能设置为 1）
}

eos.delegatebw(form).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
#### 2、赎回带宽
```
<!-- config 同账号创建 -->
let eos = EOS(config)

let form = {
    from,                   // 账户名称
    receiver,               // 赎回接收人
    unstake_net_quantity,   // 赎回 net 数量，如 10.0000 EOS
    unstake_cpu_quantity,   // 赎回 cpu 数量，如 10.0000 EOS
}

eos.undelegatebw(form).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
#### 3、手动赎回
```
<!-- config 同账号创建 -->
let eos = EOS(config)

<!-- owner: 账户名称，用于 undelegatebw 操作三天后仍未被赎回，手动触发 -->
eos.refund({ owner }).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
#### 4、购买 ram
```
<!-- config 同账号创建 -->
let eos = EOS(config)

let form = {
    payer,      // 付款账户名称
    receiver,   // 接收人
    quant,      // 购买数量，如 1.0000 EOS
}

eos.buyram(form).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
#### 5、出售 ram
```
<!-- config 同账号创建 -->
let eos = EOS(config)

let form = {
    account,      // 账户名称
    bytes,        // 出售大小，必须为整数
}

eos.sellram(form).then(res => {
    console.log(res.transaction_id)
}).catch(e => {
    console.log(e)
})
```
### 五、实战之交易
#### 1、获取某种资源详情
```
<!-- config 同根据公钥获取账号名 -->
let eos = EOS(config)

<!-- code: 代币合约; symbol: 代币符号 -->
eos.getCurrencyStats(code, symbol).then(res => {
    console.log(res)
}).catch(e => {
    console.log(e)
})
```
#### 2、获取账号的某种资源
```
<!-- config 同根据公钥获取账号名 -->
let eos = EOS(config)

let form = {
    code,       // 代币合约
    account,    // 账号名称
    symbol,     // 代币符合
}

eos.getCurrencyBalance(form).then(res => {
    console.log(res)
}).catch(e => {
    console.log(e)
})
```
#### 3、获取交易详情
测试网络 JungleTestnet 的 eos.getTransaction 接口存在 bug，不能正常返回，可使用 eospark 浏览器的接口。
```
<!-- config 同根据公钥获取账号名 -->
let eos = EOS(config)

<!-- id: 交易 id -->
eos.getTransaction(id).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})
```
#### 4、获取交易列表
测试网络 JungleTestnet 的 eos.getActions 接口存在 bug，不能正常返回，可使用 eospark 浏览器的接口。
```
<!-- config 同根据公钥获取账号名 -->
let eos = EOS(config)

<!-- account_name: 账号名称 -->
eos.getTransaction({ account_name }).then(res => {
    res.actions = res.actions.filter(action => {
        let action_trace = action.action_trace || {}
        // 去重（如果是转账操作，则可能出现多笔相同交易 id 的 action，因为可能存在多个 inline_action）
        if (action_trace.act && action_trace.act.name == 'transfer') {
            if (action_trace.receipt && action.action_trace.receipt.receiver == account_name) {
                return action
            }
        } else {
            return action
        }
    })

    console.log(res)
}).catch(err => {
    console.log(err)
})
```
#### 5、发起交易
```
<!-- config 同账号创建 -->
let eos = EOS(config)

let form = {
    from,           // 转账人名称
    to,             // 收款人名称
    quantity,       // 转账数量 例: 2.0000 EOS
    memo,           // 备注（可选）
}

<!-- account_name: 账号名称 -->
eos.transfer(form, { broadcast: true }).then(res => {
    console.log(res.transaction_id)
}).catch(err => {
    console.log(err)
})
```