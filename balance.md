# 骰子公平性验证说明
## 第一步：	解析下注记录的memo
查询下注的转账记录并获取到其中的memo信息，memo信息是一个以”-“字符将信息连接起来的字符串:

    第一个信息是下注的数字，
    第二个是服务器的种子哈希（seed_hash），
    第三个是用户的种子（user_seed）,
    第四个是服务器种子的有效时间戳，
    第五个是下注推荐人（这部分有可能没有）
    第六个是服务器的种子签名（seed_sign）

使用通用的哈希函数sha1对用户的种子进行运算，可以得到用户种子（user_seed_hash)。

## 第二步：	查询开奖记录
在www.eosflare.com自己账号的记录中找到类型为“eosbiggame44 - receipt”并且信息中的seed_hash和user_seed_hash与第一步的memo中找到的一致的记录，即为开奖记录,如下图所示：
![开奖记录](https://github.com/biggamerobot/dice/blob/master/receipt.png)

## 第三步：	验证开奖记录的服务器种子
在开奖记录中我们可以获取到服务器种子（seed），使用通用的哈希函数sha256对seed进行运算，如果结果为下注记录memo中的seed_hash，那么这个服务器种子seed是可信的。
## 第四步： 验证签名
验证签名使用biggame的公钥（EOS8aBhWoUfbLPAMwccqd2pRxjd1SVB3vuaEfGyjGtBjySr2oCZPM），对服务器的种子哈希（seed_hash）和memo中的（seed_sign）进行ecc签名验证，验证结果通过即可证明服务器种子未被篡改。
## 第五步：	计算开奖结果
分别将服务器种子和用户种子哈希从字符串转换为16进制的数值数组，进行以下运算即可得到开奖结果：
		
        result = (uint64(seed[0]) +uint64(user_seed_hash[1]) + uint64(seed[2]) + uint64(user_seed_hash[3]) + uint64(seed[4]) + uint64(user_seed_hash[5]) + uint64(seed[6]) + uint64(user_seed_hash[7])) % 100
