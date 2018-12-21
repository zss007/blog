---
title: blockchain 之 eth 发币、映射、钱包
categories:
- essay
---
这章介绍以太坊发币，映射 eos 公链和钱包开发价值。
<!--more-->
### 一、发币
#### 1.1、简介
ERC20 是以太坊代币的标准，以太坊 token （即代币）就是基于 ERC20 标准的数字货币，token 兼容以太坊钱包。    
通俗上我们可以把以太坊当成一条大街，然后街上有很多店。为了经营，每个店都发布了自己的会员卡，这个会员卡就是 token。
#### 1.2、ERC20 内容
```
// 返回 string 类型的 ERC20 代币的名字
name
// 返回 string 类型的 ERC20 代币的符号，也就是代币的简称
symbol
// 支持几位小数点后几位，如果设置为 3，也就是支持 0.001 表示
decimals
// 发行代币的总量
totalSupply
// 获取该地址代币的余额
balanceOf
// 调用 transfer 函数将 token 转账给 _to 地址，_value 为转账个数
transfer
// 批准账户从自己的账户转移 _value 个 token，可以分多次转移
approve
// 与 approve 搭配使用，approve 批准之后，调用 transferFrom 函数来转移 token
transferForm
// 返回其他账户还能提取 token 的个数
allowance
// 当成功转移 token 时，一定要触发 Transfer 事件
Transfer
// 当调用 approval 函数成功时，一定要触发 Approval 事件
Approval
```
#### 1.3、ERC20 实现案例
```
pragma solidity ^0.4.16;

contract TokenERC20 {
    // token 的共有变量
    string public name;
    string public symbol;
    // 强烈建议使用 18 位小数
    uint8 public decimals = 18;
    uint256 public totalSupply;

    // 创建一个地址的余额映射
    mapping (address => uint256) public balanceOf;
    // 创建一个地址的授权映射
    mapping (address => mapping (address => uint256)) public allowance;

    // 生成一个公共事件用来提醒客户端
    event Transfer(address indexed from, address indexed to, uint256 value);

    // 提醒客户端销毁多少token
    event Burn(address indexed from, uint256 value);

    // 构造函数
    function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
        balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
    }

    // 内部转换，只能被合约内容调用
    function _transfer(address _from, address _to, uint _value) internal {
        // Prevent transfer to 0x0 address. Use burn() instead
        require(_to != 0x0);
        // Check if the sender has enough
        require(balanceOf[_from] >= _value);
        // Check for overflows （检查溢出）
        require(balanceOf[_to] + _value >= balanceOf[_to]);
        // Save this for an assertion in the future
        uint previousBalances = balanceOf[_from] + balanceOf[_to];
        // Subtract from the sender
        balanceOf[_from] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        emit Transfer(_from, _to, _value);
        // Asserts are used to use static analysis to find bugs in your code. They should never fail
        assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
    }

    // 从账户转移 token 到其他账户
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
    }

    // 从 _from 转移 token 到 _to 上，代替 _from 账户
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= allowance[_from][msg.sender]);     // Check allowance
        allowance[_from][msg.sender] -= _value;
        _transfer(_from, _to, _value);
        return true;
    }

    // 授权 _spender 使用账户 token
    function approve(address _spender, uint256 _value) public
        returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    // 销毁 token
    function burn(uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough
        balanceOf[msg.sender] -= _value;            // Subtract from the sender
        totalSupply -= _value;                      // Updates totalSupply
        emit Burn(msg.sender, _value);
        return true;
    }

    // 销毁其他账户的 token
    function burnFrom(address _from, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value);                // Check if the targeted balance is enough
        require(_value <= allowance[_from][msg.sender]);    // Check allowance
        balanceOf[_from] -= _value;                         // Subtract from the targeted balance
        allowance[_from][msg.sender] -= _value;             // Subtract from the sender's allowance
        totalSupply -= _value;                              // Update totalSupply
        emit Burn(_from, _value);
        return true;
    }
}
```
#### 1.4、自定义合约
虽然我们可以一行代码都不敲，直接粘贴复制合约代码发币，但是往往这样满足不了我们自定义的需求。现在来添加以下自定义功能：
###### 1、权限控制
###### 2、添加token
###### 3、资源冻结
###### 4、交易和购买
```
pragma solidity ^0.4.16;

// 权限控制
contract owned {
    address public owner;

    function owned() public {
        owner = msg.sender;
    }

    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }

    function transferOwnership(address newOwner) onlyOwner public {
        owner = newOwner;
    }
}

contract TokenERC20 {
    // token 的共有变量
    string public name;
    string public symbol;
    // 强烈建议使用 18 位小数
    uint8 public decimals = 18;
    uint256 public totalSupply;

    // 创建一个地址的余额映射
    mapping (address => uint256) public balanceOf;
    // 创建一个地址的授权映射
    mapping (address => mapping (address => uint256)) public allowance;

    // 生成一个公共事件用来提醒客户端
    event Transfer(address indexed from, address indexed to, uint256 value);

    // 提醒客户端销毁多少token
    event Burn(address indexed from, uint256 value);

    // 构造函数
    function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
        balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
    }

    // 内部转换，只能被合约内容调用
    function _transfer(address _from, address _to, uint _value) internal {
        // Prevent transfer to 0x0 address. Use burn() instead
        require(_to != 0x0);
        // Check if the sender has enough
        require(balanceOf[_from] >= _value);
        // Check for overflows （检查溢出）
        require(balanceOf[_to] + _value >= balanceOf[_to]);
        // Save this for an assertion in the future
        uint previousBalances = balanceOf[_from] + balanceOf[_to];
        // Subtract from the sender
        balanceOf[_from] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        emit Transfer(_from, _to, _value);
        // Asserts are used to use static analysis to find bugs in your code. They should never fail
        assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
    }

    // 从账户转移 token 到其他账户
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
    }

    // 从 _from 转移 token 到 _to 上，代替 _from 账户
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= allowance[_from][msg.sender]);     // Check allowance
        allowance[_from][msg.sender] -= _value;
        _transfer(_from, _to, _value);
        return true;
    }

    // 授权 _spender 使用账户 token
    function approve(address _spender, uint256 _value) public
        returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    // 销毁 token
    function burn(uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough
        balanceOf[msg.sender] -= _value;            // Subtract from the sender
        totalSupply -= _value;                      // Updates totalSupply
        emit Burn(msg.sender, _value);
        return true;
    }

    // 销毁其他账户的 token
    function burnFrom(address _from, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value);                // Check if the targeted balance is enough
        require(_value <= allowance[_from][msg.sender]);    // Check allowance
        balanceOf[_from] -= _value;                         // Subtract from the targeted balance
        allowance[_from][msg.sender] -= _value;             // Subtract from the sender's allowance
        totalSupply -= _value;                              // Update totalSupply
        emit Burn(_from, _value);
        return true;
    }
}

/******************************************/
/*               高级自定义               */
/******************************************/

contract MyAdvancedToken is owned, TokenERC20 {

    uint256 public sellPrice;
    uint256 public buyPrice;

    mapping (address => bool) public frozenAccount;

    // 触发一个公共事件提醒客户端
    event FrozenFunds(address target, bool frozen);

    // 合约初始化
    function MyAdvancedToken(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) TokenERC20(initialSupply, tokenName, tokenSymbol) public {}

    // 内部传输，只在合约内部被调用
    function _transfer(address _from, address _to, uint _value) internal {
        require (_to != 0x0);                               // Prevent transfer to 0x0 address. Use burn() instead
        require (balanceOf[_from] >= _value);               // Check if the sender has enough
        require (balanceOf[_to] + _value >= balanceOf[_to]); // Check for overflows
        require(!frozenAccount[_from]);                     // Check if sender is frozen
        require(!frozenAccount[_to]);                       // Check if recipient is frozen
        balanceOf[_from] -= _value;                         // Subtract from the sender
        balanceOf[_to] += _value;                           // Add the same to the recipient
        emit Transfer(_from, _to, _value);
    }

    // 铸币给 target 账户
    function mintToken(address target, uint256 mintedAmount) onlyOwner public {
        balanceOf[target] += mintedAmount;
        totalSupply += mintedAmount;
        emit Transfer(0, this, mintedAmount);
        emit Transfer(this, target, mintedAmount);
    }

    // 冻结账户
    function freezeAccount(address target, bool freeze) onlyOwner public {
        frozenAccount[target] = freeze;
        emit FrozenFunds(target, freeze);
    }

    // 设置价格
    function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner public {
        sellPrice = newSellPrice;
        buyPrice = newBuyPrice;
    }

    // 购买 token
    function buy() payable public {
        uint amount = msg.value / buyPrice;               // calculates the amount
        _transfer(this, msg.sender, amount);              // makes the transfers
    }

    // 售卖 token
    function sell(uint256 amount) public {
        address myAddress = this;
        require(myAddress.balance >= amount * sellPrice);      // checks if the contract has enough ether to buy
        _transfer(msg.sender, this, amount);              // makes the transfers
        msg.sender.transfer(amount * sellPrice);          // sends ether to the seller. It's important to do this last to avoid recursion attacks
    }
}
```
### 二、映射
EOS 首先是在以太坊上发布代币众筹，后面 EOS  有了自己独立的公链。等众筹结束后，将以太坊上的代币放到 EOS 公链上转成真正的 EOS 币种，用户需要自己做映射。
#### 2.1、映射教程（https://www.zhihu.com/question/265707806 ）
##### 2.1.1、到EOS公链上生成EOS密钥对
生成密钥对的网址:  https://nadejde.github.io/eos-token-sale/  
密钥对生成网站代码在:  https://github.com/Nadejde/eos-token-sale/  
EOS官方的完整代码在:  https://github.com/EOSIO/eos-token-distribution/
##### 2.1.2、注册一个 myetherwallet 的钱包，并进行映射
#### 2.2、映射部分代码
映射合约源代码： https://etherscan.io/address/0xd0a6e6c54dbc68db5db3a091b171a77407ff7ccf#code
```
contract EOSSale is DSAuth, DSExec, DSMath {
    DSToken  public  EOS;                  // 详情见源代码
    uint128  public  totalSupply;          // Token 总量
    uint128  public  foundersAllocation;   // 开发团队保留份额
    string   public  foundersKey;          // 保留份额 Holder Address

    uint     public  openTime;             // window 0 开始时间
    uint     public  createFirstDay;       // window 0 供应量

    uint     public  startTime;            // window 1 开始时间
    uint     public  numberOfDays;         // window 总数
    uint     public  createPerDay;         // 每日供应量
    /**
    * EOS 的 ICO 规则为：前五天为一个 window 总共发行 2 亿 Token。五天之后，每23个小时为一个window，发行 2 百万 Token。总共发行 10 亿 Token
    */

    mapping (uint => uint)                       public  dailyTotals;
    mapping (uint => mapping (address => uint))  public  userBuys;
    mapping (uint => mapping (address => bool))  public  claimed;
    mapping (address => string)                  public  keys;

    event LogBuy      (uint window, address user, uint amount);
    event LogClaim    (uint window, address user, uint amount);
    event LogRegister (address user, string key);
    event LogCollect  (uint amount);
    event LogFreeze   ();

    function EOSSale(
        uint     _numberOfDays,
        uint128  _totalSupply,
        uint     _openTime,
        uint     _startTime,
        uint128  _foundersAllocation,
        string   _foundersKey
    ) {
        numberOfDays       = _numberOfDays;
        totalSupply        = _totalSupply;
        openTime           = _openTime;
        startTime          = _startTime;
        foundersAllocation = _foundersAllocation;
        foundersKey        = _foundersKey;

        // // window 0 Token 供应量 (由于 solidity 里面没有 float 类型，想要乘以 0.2 可以通过 乘以 0.2 ether 实现)
        createFirstDay = wmul(totalSupply, 0.2 ether);
        // // window 1 以及以后的每天 Token 供应量
        createPerDay = div(
            sub(sub(totalSupply, foundersAllocation), createFirstDay),
            numberOfDays
        );

        assert(numberOfDays > 0);
        assert(totalSupply > foundersAllocation);
        assert(openTime < startTime);
    }

    function initialize(DSToken eos) auth {
        assert(address(EOS) == address(0));
        assert(eos.owner() == address(this));
        assert(eos.authority() == DSAuthority(0));
        assert(eos.totalSupply() == 0);

        EOS = eos;
        // mint 方法是 dapp_token 提供，用来设置 Token 供应总量
        EOS.mint(totalSupply);

        // 开发团队保留的份额被发送到 0xb1 。这个地址是无人可用的，等于丢掉此数量的 Token ，但是又保证 Token 总供应量不变。EOS 团队不需要这个代币，因为在 EOS 上线之后，现在的 Token 会兑换为 EOS 链上的 Token。
        EOS.push(0xb1, foundersAllocation);
        keys[0xb1] = foundersKey;
        LogRegister(0xb1, foundersKey);
    }

    function time() constant returns (uint) {
        return block.timestamp;
    }

    function today() constant returns (uint) {
        return dayFor(time());
    }

    // 前五天都是 window 0 。之后每23小时为一个 window 。这么做的好处是让每个 window 的开始时间是滚动的，不同时区的投资者更方便参与。
    function dayFor(uint timestamp) constant returns (uint) {
        return timestamp < startTime
            ? 0
            : sub(timestamp, startTime) / 23 hours + 1;
    }

    function createOnDay(uint day) constant returns (uint) {
        return day == 0 ? createFirstDay : createPerDay;
    }

    // This method provides the buyer some protections regarding which
    // day the buy order is submitted and the maximum price prior to
    // applying this payment that will be allowed.
    function buyWithLimit(uint day, uint limit) payable {
        // 限制时间
        assert(time() >= openTime && today() <= numberOfDays);
        assert(msg.value >= 0.01 ether);

        assert(day >= today());
        assert(day <= numberOfDays);

        // 购买记录
        userBuys[day][msg.sender] += msg.value;
        dailyTotals[day] += msg.value;

        if (limit != 0) {
            assert(dailyTotals[day] <= limit);
        }

        LogBuy(day, msg.sender, msg.value);
    }

    function buy() payable {
       buyWithLimit(today(), 0);
    }

    function () payable {
       buy();
    }

    function claim(uint day) {
        assert(today() > day);
        
        // 防止重复兑换
        if (claimed[day][msg.sender] || dailyTotals[day] == 0) {
            return;
        }

        // This will have small rounding errors, but the token is
        // going to be truncated to 8 decimal places or less anyway
        // when launched on its own chain.

        var dailyTotal = cast(dailyTotals[day]);
        var userTotal  = cast(userBuys[day][msg.sender]);
        // 指定 window 的 Token 供应量除以此 window 的 eth 总量，得到兑换比例
        var price      = wdiv(cast(createOnDay(day)), dailyTotal);
        // 兑换比例乘以指定 window 中此用户支付的 eth 数量得到兑换总量
        var reward     = wmul(price, userTotal);

        // 记录兑换标志
        claimed[day][msg.sender] = true;
        // 执行转账
        EOS.push(msg.sender, reward);

        LogClaim(day, msg.sender, reward);
    }

    function claimAll() {
        for (uint i = 0; i < today(); i++) {
            claim(i);
        }
    }

    // 注册 EOS 共钥，EOS 团队要求投资者在众筹结束之前自行生成 EOS 公私钥，并将生成的共钥注册在合约中。这样在 EOS 正式上线之后，用户可以兑换 EOS 上的 Token 。
    function register(string key) {
        assert(today() <=  numberOfDays + 1);
        assert(bytes(key).length <= 64);

        keys[msg.sender] = key;

        LogRegister(msg.sender, key);
    }

    // ETH 转移（auth 是 dapp_auth 提供的权限控制方法，保证 collect 函数只能被合约 owner 执行）
    function collect() auth {
        assert(today() > 0); // window 0 不能转移
        exec(msg.sender, this.balance);  // 将 eth 转移给调用者
        LogCollect(this.balance);
    }

    // Anyone can freeze the token 1 day after the sale ends
    function freeze() {
        assert(today() > numberOfDays + 1);
        EOS.stop();
        LogFreeze();
    }
}
```
### 三、钱包
#### 3.1、钱包简介
区块链钱包目前很多人的认识就是数字资产的管理软件，用来转账和收款。由于区块链只要私钥丢失，就意味着所有的数字资产很容易就可以被人盗走，所以私钥通常都是保存在本地。私钥保存在本地的代价就是任何区块链交易都需要在本地完成私钥签名，才能将整个交易流程走通，这就使得现有的很多 DAPP 无法做到无缝地体验，换言之区块链钱包之于 DAPP，就如同支付宝之于各种电商 APP。 

支付宝占据了人民币交易的大部分市场份额，于是支付宝就成了很多交易场景无法绕过去的支付工具，我们熟悉的电商、旅游订票、知识服务等平台，都会在手机端唤起支付宝APP完成最终的支付流程。同样的，区块链上的数字资产未来极有可能也会因为大规模 DAPP 的出现而使得交易更加频繁，于是方便快捷的钱包产品就显得非常有必要了。
#### 3.2、钱包之应用分发中心
有人调研了国内外很多区块链钱包产品，很多钱包已经开始集成有限的 DAPP 了，用户在使用 DAPP 时，只要有需要支付交易的场景，都会唤起本地钱包完成私钥签名和支付，这就像很多第三方电商 APP 会在需要交易的时候唤起支付宝。区块链钱包的价值就在于给各种第三方 DAPP 提供了一套完整的支付体验流程，基于此，我们有理由相信未来的钱包很有可能会是一个应用分发中心。

过往很多商业模式都是基于流量思维，对于线下零售来说，好地段永远都是最重要的选址标准，因为好地段就意味着大流量。同样的，对于互联网应用来说，流量依然是大家极其看重的指标，微信垄断了社交流量、阿里垄断了电商流量、百度垄断了搜索流量、抖音垄断了短视频流量等等，对于一家手握大量流量的公司来说，商业变现都是迟早的事情。
#### 3.3、钱包拓展背景
在区块链领域，过去很多人都觉得中心化的交易所未来极有可能是流量入口，但是从过往交易所监守自盗的劣迹来看，人们极有可能会最终抛弃中心化交易所。数字资产交易所和传统的股票交易所本质上并没有太大的区别，数字资产交易所今天上演的各种行为，在很多人看来完全是在复制早期股票交易所，所不同的是，数字资产交易所未来是否会引入监管还存在极大的不确定性。于是各种内幕交易、操盘价格走势，再加上做空和期货的引入，交易所既可以成倍放大收益，也可以成倍放大市场的恐慌情绪。

最重要的一点是人们在交易所上存放的数字资产本质上并不掌握所有权，人们只是把数字资产转到了交易所指定的账号上，由交易所代为管理。所以交易所完全有可能直接用这些数字资产去做短期套利，而只需要在个人账户界面上显示原来的余额即可。对于刚刚过去的 EOS 投票上线主网，由于散户的投票热情很低，EOS 因为投票率低迟迟无法上线主网，最后是交易所推高了投票率，才使得 EOS 主网最终上线主网，那么我们就有理由相信交易所私自动用了未经用户授权的 EOS，强行代用户进行了投票。     

最后最为致命的一点就是交易所由于缺乏监管，理论上完全有可能携款跑路，至少目前国家法律并不保护用户的数字资产，所以散户们除了道德谴责好像也找不到更好的维权途径，不要以为我在危言耸听，很多小规模的交易所已经开始无法提现了。
#### 3.4、DAPP 的价值
钱包由于满足了用户管理数字资产的刚需，所以用户量可以做到很高，今天中国币圈投资者基本上没有人不知道 imToken 钱包的。随着去中心化交易所技术的不断改善和提高，我们有理由相信钱包内置去中心化交易平台会是一个大趋势，一旦人们的交易需求被满足，那么人们极有可能会抛弃中心化交易所。

除了交易需求，未来更大的想象空间应该来自 DAPP 的大爆发，互联网以及随后的移动互联网最为火热的阶段都是大量应用层出不穷的时候，诞生了很多世界级的大公司，人们也因此享受到了更加便捷的生活方式。我们有理由相信随着区块链基础设施的不断完善，未来基于区块链的去中心化应用（DAPP）会开始呈现爆发的态势。

Token 可以让区块链上的信息以极低的成本在用户之间完成交易，智能合约则可以通过代码编程的方式定制利益分配规则，促成更多的分工合作。当越来越多的 DAPP 爆发时，所有的交易都需要私钥签名，出于安全性考虑，私钥又不得不放在钱包 APP 中，所以就需要钱包充当媒介为 DAPP 提供最终的支付解决方案。如果钱包本身就是一个应用分发中心的话，那么就意味着人们可以在钱包中使用完成从 DAPP 体验到支付的所有环节。
#### 3.5、多链资产和跨链交易
如果未来钱包真的要充当应用分发中心和支付媒介，那么就肯定会有多链资产和跨链交易的需求，目前我们看到的很多钱包都还是只支持单一区块链平台，随着区块链开始在各个行业渗透，未来的钱包大概率会出现多条链并存，DAPP 横跨多条链的场景。

想象一下，如果一款钱包可以同时支持比特币、以太坊以及各种 ERC20 代币、EOS、IOTA 等，是不是会觉得非常方便，如果还能在钱包中完成这几种资产的比例自由配置，估计币圈的投资者会有一种相见恨晚的感觉。

我们不妨再想象一下，如果今天的加密猫游戏不仅仅是基于以太坊的 ERC721 代币，还包括其他区块链数字货币，那么游戏体验会不会更好，用户可以通过比特币充值，然后用以太币完成交易，还可以换成某条侧链上的资产去玩其他周边游戏，比如说给猫买一顶帽子等。一旦跨链交易成为可能，游戏的扩展性和可玩性都会得到极大的提高。

