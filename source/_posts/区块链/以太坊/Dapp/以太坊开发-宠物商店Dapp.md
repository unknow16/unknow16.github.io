---
title: 以太坊开发-宠物商店
date: 2018-04-19 19:43:44
tags: BlockChain
---




### 环境搭建
1. 安装Node
2. 安装Truffle，是一个基于node.js的开发Dapp框架

```
sudo npm install -g truffle
```

3. 安装Ganache，是一个图形化以太坊私有链工具
```
//1. ubuntu下载ganache
wget https://github.com/trufflesuite/ganache/releases/download/v1.0.1/ganache-1.0.1-x86_64.AppImage  

//2. 修改权限
chmod +x ganache-1.0.1-x86_64.AppImage

//3. 启动ganache
sudo ./ganache-1.0.1-x86_64.AppImage 
```

4. Chrome安装MetaMask


### 创建项目
1. 建立项目目录

```
> mkdir pet-shop-tutorial
> cd pet-shop-tutorial
```

2. 使用truffle unbox创建项目

```
> truffle unbox pet-shop
```

3. 项目目录说明
* contracts/ 智能合约的文件夹，所有的智能合约文件都放置在这里，里面包含一个重要的合约Migrations.sol（稍后再讲）
* migrations/ 用来处理部署（迁移）智能合约 ，迁移是一个额外特别的合约用来保存合约的变化。
* test/ 智能合约测试用例文件夹
* truffle.js/ 配置文件

### 编写智能合约
智能合约承担着分布式应用的后台逻辑和存储。智能合约采用Solidity编写，如下：

在contracts目录下，添加合约文件Adoption.sol
```
pragma solidity ^0.4.17;

contract Adoption {

  address[16] public adopters;  // 保存领养者的地址

    // 领养宠物
  function adopt(uint petId) public returns (uint) {
    require(petId >= 0 && petId <= 15);  // 确保id在数组长度内

    adopters[petId] = msg.sender;        // 保存调用这地址 
    return petId;
  }

  // 返回领养者
  function getAdopters() public view returns (address[16]) {
    return adopters;
  }

}
```

### 编译智能合约
Solidity是编译型语言，需要把可读的Solidity代码编译成EVM能执行的字节码才能运行。

在pet-shop-tutorial下，执行下列命令：

```
truffle complile
```

### 部署智能合约
合约编译只后就可以部署到区块链上了。

在migrations文件夹下已经有一个1_initial_migration.js部署脚本，用来部署Migrations.sol合约。
Migrations.sol 用来确保不会部署相同的合约。

1. 现在我们来创建一个自己的部署脚本2_deploy_contracts.js

```
var Adoption = artifacts.require("Adoption");

module.exports = function(deployer) {
  deployer.deploy(Adoption);
};
```

2. 启动Ganache来开启一个私链来部署合约，默认会在7545端口运行,如果未启动，请参考上文环境搭建。
3. 部署合约命令，之后Ganache区块链中查看到产生新的区块。

```
> truffle  migrate
```

### 测试智能合约
1. 在test目录下新建一个TestAdoption.sol，编写测试合约


```
pragma solidity ^0.4.17;

import "truffle/Assert.sol";   // 引入的断言
import "truffle/DeployedAddresses.sol";  // 用来获取被测试合约的地址
import "../contracts/Adoption.sol";      // 被测试合约

contract TestAdoption {
  Adoption adoption = Adoption(DeployedAddresses.Adoption());

  // 领养测试用例
  function testUserCanAdoptPet() public {
    uint returnedId = adoption.adopt(8);

    uint expected = 8;
    Assert.equal(returnedId, expected, "Adoption of pet ID 8 should be recorded.");
  }

  // 宠物所有者测试用例
  function testGetAdopterAddressByPetId() public {
    // 期望领养者的地址就是本合约地址，因为交易是由测试合约发起交易，
    address expected = this;
    address adopter = adoption.adopters(8);
    Assert.equal(adopter, expected, "Owner of pet ID 8 should be recorded.");
  }

    // 测试所有领养者
  function testGetAdopterAddressByPetIdInArray() public {
  // 领养者的地址就是本合约地址
    address expected = this;
    address[16] memory adopters = adoption.getAdopters();
    Assert.equal(adopters[8], expected, "Owner of pet ID 8 should be recorded.");
  }
}
```
2. 运行测试用例，之后会显示是否通过信息。

```
> truffle test
```

### 运用web3与智能合约交互
web3是一个实现了与以太坊节点通信的库，我们利用web3来和合约进行交互。

我们已经编写和部署及测试好了我们的合约，接下我们为合约编写UI，让合约真正可以用起来。在Truffle Box pet-shop里，已经包含了应用的前端代码，代码在src/文件夹下。

在编辑器中打开src/js/app.js 可以看到用来管理整个应用的App对象，init函数加载宠物信息，就初始化web3.

1. 初始化web3:

修改app.js中initWeb3()如下：

```
//其中优先使用Mist 或 MetaMask提供的web3实例，如果没有则从本地环境创建一个。

initWeb3: function() {
  // Is there an injected web3 instance?
  if (typeof web3 !== 'undefined') {
    App.web3Provider = web3.currentProvider;
  } else {
    // If no injected web3 instance is detected, fall back to Ganache
    App.web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');
  }
  web3 = new Web3(App.web3Provider);

  return App.initContract();
}
```

2. 实例化合约：

使用truffle-contract会帮我们保存合约部署的信息，就不需要我们手动修改合约地址，修改initContract()代码如下：

```
initContract: function() {
  // 加载Adoption.json，保存了Adoption的ABI（接口说明）信息及部署后的网络(地址)信息，它在编译合约的时候生成ABI，在部署的时候追加网络信息
  $.getJSON('Adoption.json', function(data) {
    // 用Adoption.json数据创建一个可交互的TruffleContract合约实例。
    var AdoptionArtifact = data;
    App.contracts.Adoption = TruffleContract(AdoptionArtifact);

    // Set the provider for our contract
    App.contracts.Adoption.setProvider(App.web3Provider);

    // Use our contract to retrieve and mark the adopted pets
    return App.markAdopted();
  });
  return App.bindEvents();
}
```

3. 处理领养逻辑：

修改markAdopted()代码：
```
markAdopted: function(adopters, account) {
  var adoptionInstance;

  App.contracts.Adoption.deployed().then(function(instance) {
    adoptionInstance = instance;

    // 调用合约的getAdopters(), 用call读取信息不用消耗gas
    return adoptionInstance.getAdopters.call();
  }).then(function(adopters) {
    for (i = 0; i < adopters.length; i++) {
      if (adopters[i] !== '0x0000000000000000000000000000000000000000') {
        $('.panel-pet').eq(i).find('button').text('Success').attr('disabled', true);
      }
    }
  }).catch(function(err) {
    console.log(err.message);
  });
}
```
修改handleAdopt()代码：
```
handleAdopt: function(event) {
  event.preventDefault();

  var petId = parseInt($(event.target).data('id'));

  var adoptionInstance;

  // 获取用户账号
  web3.eth.getAccounts(function(error, accounts) {
    if (error) {
      console.log(error);
    }
  
    var account = accounts[0];
  
    App.contracts.Adoption.deployed().then(function(instance) {
      adoptionInstance = instance;
  
      // 发送交易领养宠物
      return adoptionInstance.adopt(petId, {from: account});
    }).then(function(result) {
      return App.markAdopted();
    }).catch(function(err) {
      console.log(err.message);
    });
  });
}
```

4. 修改src/index.html中引用的jquery.js

因为index.html中引用的jquery为google域名下的,由于某些问题，不能访问，所以需自己从其他渠道下载jquery.js,放到src/js下，并修改index.html中引用路径。

### 配置web服务器lite-server
接下来需要本地的web 服务器提供服务的访问， Truffle Box pet-shop里提供了一个lite-server可以直接使用，我们看看它是如何工作的。

1. bs-config.json指示了lite-server的工作目录。

```
{
  "server": {
    "baseDir": ["./src", "./build/contracts"]
  }
}

// ./src 是网站文件目录
// ./build/contracts 是合约输出目录
```

2. package.json文件的scripts中添加了dev命令：

```
"scripts": {
  "dev": "lite-server",
  "test": "echo \"Error: no test specified\" && exit 1"
},

// 当运行npm run dev的时候，就会启动lite-server
```

4. 启动服务
```
> npm run dev

// 之后会自动打开浏览器，显示领养宠物商店首页
```

### 领养宠物
1. 当我们点击Adopt时，MetaMask会弹窗提示我们交易的确认
2. 点击Submit确认后，就可以看到成功领养了这次宠物。
3. 在MetaMask中，也可以看到交易的清单：