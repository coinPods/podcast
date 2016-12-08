# Segwit

## Segwit好处都有啥

[https://bitcoincore.org/en/2016/01/26/segwit-benefits/]

1. 延展性
2. 扩容
3. 脚本升级
4. SPV和硬件钱包
5. 减小UTXO
6. 签名O(n^2)问题

	> 超大交易，通过重复计算Hash、多次签名，进行攻击的成本
	> 
	> - http://rusty.ozlabs.org/?p=522
	> - 鱼池的1MB交易 https://btc.com/bb41a757f405890fb0f5856228e23b715702d714d59bf2b1feb70d8b2b4e3e08
	>   - 5569输入
	>   - 25 秒验证
	> - 优化方案
	>   - 经过优化验证上述交易可以减少至数秒
	>   - 1MB， 3300 inputs, 406KB  output(s):  仅做Hash运算需要 10.9 seconds
	>   - 但是如果是8MB 交易，22,500输入， 3.95M输出，仅做Hash运算需要11分钟

## Segwit的机制
* 到底改动多大

	[https://github.com/bitcoin/bitcoin/pull/7910/files]

	> 128 次提交
	> 代码行数变化，包含功能代码、测试代码、脚本以及文档等所有变更
	> 新增：5,305
	> 删除：   571
	> 3000+测试代码
	> - 涉及80个文件
	> - 开发周期： 2016.04.19 - 2016.06.22，共计：64 天
	>   - 04.19 - 04.28 完成主要功能
	>   - 05.10 -06.22 测试，修复bug，部署验证，测试网激活
	> - 发布版本
	>   - 0.12.x 未激活
	>   - 0.13.0 未激活
	>   - 0.13.1 可激活


* 矿工，开发者，用户需要注意什么

[https://bitcoincore.org/en/2016/10/27/segwit-upgrade-guide/]

* 交易相关

	什么时候可以开始使用SegWit

### 现有六类交易输出类型

* PubKey (compressed and un-compressed) 
	   - 公钥 04xxxxx or 02/03xxxxx
	   - 地址形式：`1xxxxxxxxxxx`
	   - `047211a824f55b505228e4c3d5194c1fcfaa15a456abdf37f9b9d97a4040afc073dee6c89064984f03385237d92167c13e236446b417ab79a0fcae412ae3316b77 OP_CHECKSIG`

* PubKeyHash
	   - Hash160: `RIPEMD160(pubkey)`
	   - 地址形式：`1xxxxxxxxxxx`
	   - `OP_DUP OP_HASH160 721afdf638d570285d02d3076d8be6a03ee0794d OP_EQUALVERIFY OP_CHECKSIG`
* Pay To ScriptHash(P2SH)
	   - n/m 多重签名
	   - hash160 of `redeem script`
	   - 地址形式：`3xxxxxxxxxxx`
	   - `OP_HASH160 886f833d15aec316a6e98d7b63f3a5bceaca2868 OP_EQUAL`
* MultiSig
	   - 最早的多重签名，目前已不推荐使用
	   - 地址形式：`1xxxxxxxxxxx   1xxxxxxxxxxx  1xxxxxxxxxxx`
	   - `N pubkey_1 pubkey_2 ... pubkey_n M`
* Null-Data
	 - `OP_RETURN xxxxxxx`
* Non-Standard
	  - 无法解析的非标准脚本


### SegWit 新增四类输出类型

[https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki]

- P2WPKH
	  - WITNESS_V0_KEYHASH
	  - `HASH160(pubkey_33)` 20Bytes
	  - `0x0014{20-byte keyhash}`
	- `0014 cbb97f673c5fa8db49e6182951edb87c8f9c0ac4`
- P2WSH
	  - WITNESS_V0_SCRIPTHASH
	  - `SHA256(pubkey_33)` 32Bytes
	  - `0x0020{32-byte keyhash}`
	- `0020 925fe0a6cde95bdc7a21b08925c246cae17005f8a013efffdb5e5cb7b7f8d0c2`
- P2SH-P2WPKH
	  - PubKey is compressed
	  - `RIPEMD160( SHA256( PubKey ) )`
- P2SH-P2WSH
	  - `scripPubKey  = OP_HASH160 RIPEMD160(redeemScript) OP_EQUAL`
	- `redeemScript = 0x0020{32-byte scripthash}`
	- `scripthash = SHA256(witnessScript)`

* 矿池
	  - GetBlockTemplate
	  - 多出一个字段 `default_witness_commitment`
	  - 放入 coinbase tx 的输出中，新增一个输出
	- 代码改动应该小于100行

## Segwit如何修复延展性

* 对比下Flexible Transaction?

[https://zander.github.io/posts/Flexible\_Transactions]
[https://github.com/bitcoin/bips/blob/master/bip-0134.mediawiki]

* 闪电网络与Segwit

[http://www.coindesk.com/whats-left-bitcoins-lightning-network-goes-live/]


## Segwit如何实现扩容

* 扩容效果究竟如何

## 硬分叉实现与软分叉实现对比

* 欺骗了旧节点？

> [https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki]
> To judge whether or not more than 50% of hashing power supports this BIP, miners are asked to upgrade their software and put the string "/P2SH/" in the input of the coinbase transaction for blocks that they create.
> 
> On February 1, 2012, the block-chain will be examined to determine the number of blocks supporting pay-to-script-hash for the previous 7 days. If 550 or more contain "/P2SH/" in their coinbase, then all blocks with timestamps after 15 Feb 2012, 00:00:00 GMT shall have their pay-to-script-hash transactions fully validated. Approximately 1,000 blocks are created in a week; 550 should, therefore, be approximately 55% of the network supporting the new feature.
> 
> If a majority of hashing power does not support the new validation rules, then rollout will be postponed (or rejected if it becomes clear that a majority will never be achieved).
> 

* 有没有技术债？

* 雾件

## 其他

* Script Version和MAST

[https://github.com/jl2012/bips/blob/mast/bip-mast.mediawiki]

* Schnorr Signature和Aggregated Signature

[https://bitcoinmagazine.com/articles/the-power-of-schnorr-the-signature-algorithm-to-increase-bitcoin-s-scale-and-privacy-1460642496]
[https://bitcointalk.org/index.php?topic=1377298.0]
