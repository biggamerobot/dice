# Big.game 骰子公平性验证说明

验证工具网页请访问：https://big.game/balance

在使用此网页工具之前，请仔细阅读以下说明。你可以按照如下步骤，自行开发程序验证。

## 第一步：	解析下注记录的memo

在区块链浏览器（如eosflare）查询您下注的转账记录，查看其memo信息。

memo信息是一个以”-“字符将信息连接起来的字符串,类似“81-44287e8f0d21...qySS36H5d-1542455142-knightplay11-SIG_K1_JuR6XiFuWCc...VY”:
![memo](https://github.com/biggamerobot/dice/blob/master/bet_memo.png)

其结构如下：
* 第一个信息是下注的数字
* 第二个是服务器的种子哈希（house_seed_hash），玩家下注前，在“玩法与公平”页面可看到此预先公布的值
* 第三个是玩家的种子（player_seed）,玩家下注前，在“玩法与公平”页面可看到玩家种子，使用通用的哈希函数sha1对玩家的种子进行运算，可以得到玩家种子（player_seed_hash)
* 第四个是服务器种子的有效时间戳
* 第五个是下注推荐人（这部分有可能没有）
* 第六个是服务器的种子签名（House Signature），玩家下注前，在“玩法与公平”页面可看到此预先公布的值

## 第二步：	查询开奖记录
在www.eosflare.com自己账号的记录中找到类型为“eosbiggame44 - receipt”的交易记录，找到信息中的house_seed_hash和player_seed_hash与第一步的memo中找到的一致的记录，即为开奖记录,如下图所示：
![开奖记录](https://github.com/biggamerobot/dice/blob/master/receipt.png)

（注意：在老的交易记录中，receipt中的seed和seed_hash对应为house_seed和house_seed_hash, user_seed_hash为player_seed_hash)

## 第三步：	验证开奖记录的服务器种子
在开奖receipt中我们可以获取到服务器种子（house_seed），使用通用的哈希函数sha256对house_seed进行运算，如果结果为下注记录memo中的house_seed_hash，那么这个house_seed是可信的。
## 第四步： 验证签名
验证签名使用biggame的公钥（EOS8aBhWoUfbLPAMwccqd2pRxjd1SVB3vuaEfGyjGtBjySr2oCZPM），对house_seed_hash和memo中的（house_seed_sign）进行ecc签名验证，验证结果通过即可证明服务器种子未被篡改。
## 第五步：	计算开奖结果
分别将服务器种子（house_seed）和玩家种子哈希（player_seed_hash)从字符串转换为16进制的数值数组，进行以下运算即可得到开奖结果：
		
        result = uint64(house_seed[0]) +uint64(player_seed_hash[1]) + uint64(house_seed[2]) + uint64(player_seed_hash[3]) + uint64(house_seed[4]) + uint64(player_seed_hash[5]) + uint64(house_seed[6]) + uint64(player_seed_hash[7]))%100
