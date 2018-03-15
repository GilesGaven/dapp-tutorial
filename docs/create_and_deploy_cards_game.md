创建卡牌游戏合约并部署到区块链模拟器
-----
# 用truffle 创建工程


打开终端, 创建一个空目录作为本次项目的根目录, 然后输入

```
truffle unbox react
```
truffle 会开始下载相关的文件, 请耐心等待一会儿.

找到truffle.js, 填入如下内容, 是为truffle指定要去链接的区块链网络
```
module.exports = {
    networks: {
        dev: {
            host: "127.0.0.1",
            port: 7545,
            network_id: "*"
        }
    }
};
```

# 编写一个智能合约
目前以太网络上已经有比较多的智能合约了, 我们可以找别人的智能合约来参考.

## 安装Metamask
Metamask是一款浏览器插件, 可以让我们从浏览器中访问到以太坊区块链的数据. 因为Google插件商店需要爱国上网才能访问, 所以这里推荐使用Firefox, 并安装Metamask插件.

## 找合约模板
访问一个已经上线的dapp的网站, 比如[cryptobeauty.cc](http://cryptobeauty.cc), 走一遍购买流程, 在最后就能拿到合约的地址, 然后上etherscan.io就可以看到合约的源码.
这里我们拿到的cryptobeauty的合约源码如下:

```
pragma solidity ^0.4.19;

contract AccessControl {
    address public owner;
    address[] public admins;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier onlyAdmins {
        bool found = false;

        for (uint i = 0; i < admins.length; i++) {
            if (admins[i] == msg.sender) {
                found = true;
                break;
            }
        }
        require(found);
        _;
    }

    function addAdmin(address _adminAddress) public onlyOwner {
        admins.push(_adminAddress);
    }

    function transferOwnership(address newOwner) public onlyOwner {
        if (newOwner != address(0)) {
            owner = newOwner;
        }
    }

}

contract ERC721 {
    // Required Functions
    function implementsERC721() public pure returns (bool);
    function totalSupply() public view returns (uint256);
    function balanceOf(address _owner) public view returns (uint256);
    function ownerOf(uint256 _tokenId) public view returns (address);
    function transfer(address _to, uint _tokenId) public;
    function approve(address _to, uint256 _tokenId) public;
    function transferFrom(address _from, address _to, uint256 _tokenId) public;

    // Optional Functions
    function name() public pure returns (string);
    function symbol() public pure returns (string);

    // Required Events
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);
}


contract CryptoBeauty is AccessControl, ERC721 {
    // Event fired for every new beauty created
    event Creation(uint256 tokenId, string name, address owner);

    // Event fired whenever beauty is sold
    event Purchase(uint256 tokenId, uint256 oldPrice, uint256 newPrice, address prevOwner, address owner, uint256 charityId);

    // Event fired when price of beauty changes
    event PriceChange(uint256 tokenId, uint256 price);

    // Event fired when charities are modified
    event Charity(uint256 charityId, address charity);

    string public constant NAME = "Crypto Beauty"; 
    string public constant SYMBOL = "BEAUTY"; 

    // Initial price of card
    uint256 private startingPrice = 0.005 ether;
    uint256 private increaseLimit1 = 0.5 ether;
    uint256 private increaseLimit2 = 50.0  ether;
    uint256 private increaseLimit3 = 100.0  ether;

    // Charities enabled in the future
    bool charityEnabled;

    // Beauty card
    struct Beauty {
        // unique name of beauty
        string name;

        // selling price
        uint256 price;

        // maximum price
        uint256 maxPrice;
    }

    Beauty[] public beauties;

    address[] public charities;
    
    mapping (uint256 => address) public beautyToOwner;
    mapping (address => uint256) public beautyOwnershipCount;
    mapping (uint256 => address) public beautyToApproved;

    function CryptoBeauty() public {
        owner = msg.sender;
        admins.push(msg.sender);
        charityEnabled = false;
    }

    function implementsERC721() public pure returns (bool) {
        return true;
    }

    function totalSupply() public view returns (uint256) {
        return beauties.length;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return beautyOwnershipCount[_owner];
    }
    function ownerOf(uint256 _tokenId) public view returns (address owner) {
        owner = beautyToOwner[_tokenId];
        require(owner != address(0));
    }

    function transfer(address _to, uint256 _tokenId) public {
        require(_to != address(0));
        require(beautyToOwner[_tokenId] == msg.sender);

        _transfer(msg.sender, _to, _tokenId);
    }
    function approve(address _to, uint256 _tokenId) public {
        require(beautyToOwner[_tokenId] == msg.sender);
        beautyToApproved[_tokenId] = _to;
        Approval(msg.sender, _to, _tokenId);
    }
    function transferFrom(address _from, address _to, uint256 _tokenId) public {
        require(beautyToApproved[_tokenId] == _to);
        require(_to != address(0));
        require(beautyToOwner[_tokenId] == _from);

        _transfer(_from, _to, _tokenId);
    }
    function name() public pure returns (string) {
        return NAME;
    }
    function symbol() public pure returns (string) {
        return SYMBOL;
    }

    function addCharity(address _charity) public onlyAdmins {
        require(_charity != address(0));

        uint256 newCharityId = charities.push(_charity) - 1;

        // emit charity event
        Charity(newCharityId, _charity);
    }

    function deleteCharity(uint256 _charityId) public onlyAdmins {
        delete charities[_charityId];

        // emit charity event
        Charity(_charityId, address(0));
    }

    function getCharity(uint256 _charityId) public view returns (address) {
        return charities[_charityId];
    }

    function createBeauty(string _name, address _owner, uint256 _price) public onlyAdmins {
        if (_price <= 0.005 ether) {
            _price = startingPrice;
        }
        
        Beauty memory _beauty = Beauty({
            name: _name,
            price: _price,
            maxPrice: _price
        });
        uint256 newBeautyId = beauties.push(_beauty) - 1;

        Creation(newBeautyId, _name, _owner);

        _transfer(address(0), _owner, newBeautyId);


    }
    
    function newBeauty(string _name, uint256 _price) public onlyAdmins {
        createBeauty(_name, msg.sender, _price);
    }

    function getBeauty(uint256 _tokenId) public view returns (
        string beautyName,
        uint256 sellingPrice,
        uint256 maxPrice,
        address owner
    ) {
        Beauty storage beauty = beauties[_tokenId];
        beautyName = beauty.name;
        sellingPrice = beauty.price;
        maxPrice = beauty.maxPrice;
        owner = beautyToOwner[_tokenId];
    }


    function purchase(uint256 _tokenId, uint256 _charityId) public payable {
        // seller
        address oldOwner = beautyToOwner[_tokenId];
        // current price
        uint sellingPrice = beauties[_tokenId].price;
        // buyer
        address newOwner = msg.sender;
        
        require(oldOwner != newOwner);
        require(newOwner != address(0));
        require(msg.value >= sellingPrice);
        
        uint256 devCut;
        uint256 nextPrice;

        if (sellingPrice < increaseLimit1) {
          devCut = SafeMath.div(SafeMath.mul(sellingPrice, 5), 100); // 5%
          nextPrice = SafeMath.div(SafeMath.mul(sellingPrice, 200), 95);
        } else if (sellingPrice < increaseLimit2) {
          devCut = SafeMath.div(SafeMath.mul(sellingPrice, 4), 100); // 4%
          nextPrice = SafeMath.div(SafeMath.mul(sellingPrice, 135), 96);
        } else if (sellingPrice < increaseLimit3) {
          devCut = SafeMath.div(SafeMath.mul(sellingPrice, 3), 100); // 3%
          nextPrice = SafeMath.div(SafeMath.mul(sellingPrice, 125), 97);
        } else {
          devCut = SafeMath.div(SafeMath.mul(sellingPrice, 2), 100); // 2%
          nextPrice = SafeMath.div(SafeMath.mul(sellingPrice, 115), 98);
        }

        uint256 excess = SafeMath.sub(msg.value, sellingPrice);

        if (charityEnabled == true) {
            
            // address of choosen charity
            address charity = charities[_charityId];

            // check if charity address is not null
            require(charity != address(0));
            
            // 1% of selling price
            uint256 donate = uint256(SafeMath.div(SafeMath.mul(sellingPrice, 1), 100));

            // transfer money to charity
            charity.transfer(donate);
            
        }

        // set new price
        beauties[_tokenId].price = nextPrice;
        
        // set maximum price
        beauties[_tokenId].maxPrice = nextPrice;

        // transfer card to buyer
        _transfer(oldOwner, newOwner, _tokenId);

        // transfer money to seller
        if (oldOwner != address(this)) {
            oldOwner.transfer(SafeMath.sub(sellingPrice, devCut));
        }

        // emit event that beauty was sold;
        Purchase(_tokenId, sellingPrice, beauties[_tokenId].price, oldOwner, newOwner, _charityId);
        
        // transfer excess back to buyer
        if (excess > 0) {
            newOwner.transfer(excess);
        }  
    }

    // owner can change price
    function changePrice(uint256 _tokenId, uint256 _price) public {
        // only owner can change price
        require(beautyToOwner[_tokenId] == msg.sender);

        // price cannot be higher than maximum price
        require(beauties[_tokenId].maxPrice >= _price);

        // set new price
        beauties[_tokenId].price = _price;
        
        // emit event
        PriceChange(_tokenId, _price);
    }

    function priceOfBeauty(uint256 _tokenId) public view returns (uint256) {
        return beauties[_tokenId].price;
    }

    function tokensOfOwner(address _owner) public view returns(uint256[]) {
        uint256 tokenCount = balanceOf(_owner);

        uint256[] memory result = new uint256[](tokenCount);
        uint256 total = totalSupply();
        uint256 resultIndex = 0;

        for(uint256 i = 0; i <= total; i++) {
            if (beautyToOwner[i] == _owner) {
                result[resultIndex] = i;
                resultIndex++;
            }
        }
        return result;
    }


    function _transfer(address _from, address _to, uint256 _tokenId) private {
        beautyOwnershipCount[_to]++;
        beautyToOwner[_tokenId] = _to;

        if (_from != address(0)) {
            beautyOwnershipCount[_from]--;
            delete beautyToApproved[_tokenId];
        }
        Transfer(_from, _to, _tokenId);
    }

    function enableCharity() external onlyOwner {
        require(!charityEnabled);
        charityEnabled = true;
    }

    function disableCharity() external onlyOwner {
        require(charityEnabled);
        charityEnabled = false;
    }

    function withdrawAll() external onlyAdmins {
        msg.sender.transfer(this.balance);
    }

    function withdrawAmount(uint256 _amount) external onlyAdmins {
        msg.sender.transfer(_amount);
    }

}


/**
 * @title SafeMath
 * @dev Math operations with safety checks that throw on error
 */
library SafeMath {

  /**
  * @dev Multiplies two numbers, throws on overflow.
  */
  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  /**
  * @dev Integer division of two numbers, truncating the quotient.
  */
  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  /**
  * @dev Substracts two numbers, throws on overflow (i.e. if subtrahend is greater than minuend).
  */
  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  /**
  * @dev Adds two numbers, throws on overflow.
  */
  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```

##  在工程中创建智能合约文件
进入工程目录下面的contracts目录, 会看到里面已经有Migrations.sol和SimpleStorage.sol两个文件, 我们创建CryptoBeauty.sol, 并填入上面的合约内容. 

## 打开区块链模拟器
找到我们下载的ganache.AppImage文件, 双击运行或者在命令行输入./ganache-*.AppImage 来运行ganache. 
ganache打开之后我们可以看到左上角的current block 显示为0, 同时ganache创建的每个账户都有100ETH的资金.

## 部署合约
进入migrations文件夹, 创建3_deploy_beauty.js并在里面输入下面的内容
```
var CryptoBeauty = artifacts.require("./CryptoBeauty.sol");

module.exports = function(deployer) {
  deployer.deploy(CryptoBeauty);
};
```
保存文件并退出
然后在工程的根目录下面输入
```
truffle compile && truffle migrate --network dev
```
看到Saving successful miration to network...的消息就表示已经部署成功了. 我们也可以调出ganache, 会看到这个时候current block已经不在是0, 同时第一个账户里面的资金表少了, 因为部署合约的过程支付了gas.

## 创建角色
CryptoBeauty里面的卡牌是怎么来的呢, 答案就是: 通过与智能合约交互创建的. 我们下面就来创建角色.

首先我们回过头去看下智能合约, 里面有一个叫newBeauty的函数用来创建角色, 还有一个 totalSupply()函数返回合约总共有多少个角色.
```
function newBeauty(string _name, uint256 _price) public onlyAdmins {
        createBeauty(_name, msg.sender, _price);
}

function totalSupply() public view returns (uint256) {
        return beauties.length;
}    
```
newBeauty函数有两个参数, 一个是字符串型的角色的名字, 另一个是角色的价格, 注意这里的单位是wei.

如何创建角色呢?.
在工程的根目录下输入
```
truffle console --network dev
```
然后在打开的truffle console中输入
```
CryptoBeauty.deployed().then(function(instance) {instance.newBeauty('FBB', 10000)})

CryptoBeauty.deployed().then(function(instance) {instance.newBeauty('LBB', 10000)})

CryptoBeauty.deployed().then(function(instance) {instance.newBeauty('LYF', 10000)})

```

这就完成了创建角色, 怎么验证是不是成功了呢?, 我们可以通过下面的命令调用合约的totalSupply()方法
```
CryptoBeauty.deployed().then(function(instance) {instance.totalSupply.call().then(function(tot){console.log('total supplay is ' + tot)})})

```
这里输出的数字和上面我们创建的角色的数量一致就表明角色已经创建成功了.

好了, 本期的课程就到这里, 如果需要视频课程, 请扫描下面的二维码加入课程群.

![群二维码](./images/dapp_group2.jpeg)
![老师个人微信](./images/michael_wechat.jpeg)


