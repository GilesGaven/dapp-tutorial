将智能合约部署到 Rinkeby 测试网
---------------

在前一篇文章中, 我们分享了如何创建智能合约并部署到 Ganache 区块链模拟器[链接](https://github.com/Michael-Jing/dapp-tutorial/blob/master/docs/create_and_deploy_cards_game.md), 本次我们接着上次的内容分享如何将智能合约部署到 Rinkeby 测试网, 当然也可以用这种方法来部署到以太坊主网. 
# 环境准备
- geth
- Mist
- truffle

# 开始部署
启动 geth
```
geth  --rinkeby --rpc --rpcapi db,eth,net,web3,personal
```
启动 mist, 并连接到 geth. 这里需要注意在不同的系统上面, mist 和 rinkeby/geth.ipc 的路径会不同.
```
/Applications/Mist.app/Contents/MacOS/Mist --rpc ~/Library/Ethereum/rinkeby/geth.ipc
```

等 mist 打开之后复制自己的账户地址, 比如0x03fd8c397Bd9aC3Ef86D2C1aa9618E161da5c08d.
, 后面的部署操作需要消耗以太币, 所以如果自己的账户里面没有以太币, 可以去https://faucet.rinkeby.io/领取.
回到 geth 的命令行窗口, 按 Ctrl + C 结束掉进程, 然后输入
```
geth  --rinkeby --rpc --rpcapi db,eth,net,web3,personal --unlock "0x03fd8c397Bd9aC3Ef86D2C1aa9618E161da5c08d" --rpccorsdomain http://localhost:3000
```
注意这里要用自己的账户地址替换上面的0x03fd8c397Bd9aC3Ef86D2C1aa9618E161da5c08d

编辑 truffle.js
```
   module.exports = {
     // See <http://truffleframework.com/docs/advanced/configuration>
     // for more about customizing your Truffle configuration!
     networks: {
        development: {
        host: "127.0.0.1",
         port: 7545,
         network_id: "*" // Match any network id
       }
        , rinkeby: {
            host: "127.0.0.1",
            port: 8545,
            from: "0x03fd8c397Bd9aC3Ef86D2C1aa9618E161da5c08d",
            network_id: 4,
            gas: 4712388
        }
    }
};

```
注意from 后面的地址也要替换成自己的账户地址
到这里准备工作就做完了, 执行下面的命令就可以部署了.
```
truffle compile && truffle migrate --network rinkeby
```
这种部署方法需要同步区块, 比较麻烦, 想要学习更加简单快速的方法吗, 请加入我们的dapp 开发课程, 相关信息请关注星号区块链咨询.



