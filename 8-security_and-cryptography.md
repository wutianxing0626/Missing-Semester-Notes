# Entropy

$log_2(x)$

# Cryptographic hash function 密码散列函数
是一种特殊的哈希函数, 将任意长度的数据映射为固定长度的输出

一般的表示为: 
```
hash(value: array<byte>) -> vector<byte, N>  (N对于该函数固定)
```

例如`sha1sum <file>`可以输出一段字节(不论是文件还是输入流)的SHA-1哈希后的结果(160 bit = 40位16进制字符串)

但这里有一个很有趣的结果:

```shell
> echo "Hello World" | sha1sum
648a6a6ffffdaa0badb23b8baf90b6168dd16b3a  -
> printf "Hello World" | sha1sum
0a4d55a8d778e5022fab701977c5d840bbc486d0  -
```

原因是因为echo默认会加上一个"\n", 给echo开启-n或者printf加上\n均能解决这一问题

与普通的哈希方法不同, 给密码学用的散列函数需要一定的安全性质: 

- 不可逆性: 对于 hash(m) = h, 难以通过已知的输出 h 来计算出原始输入 m
- 目标碰撞抵抗性/弱无碰撞: 对于一个给定输入 $m_1$, 难以找到 $m_2$ != $m_1$ 且 $hash(m_1) = hash(m_2)$
- 碰撞抵抗性/强无碰撞: 难以找到一组满足 $hash(m_1) = hash(m_2)$ 的输入 $m_1, m_2$(该性质严格强于目标碰撞抵抗性)
- 正向计算不能太快(防止暴力破解)

一些应用: 
- git
- 文件的SHA-256对比
- 单向性 -> 承诺机制

如果在网络上实现密码登录机制, 注意不能直接存储明文密码(防止数据库泄露), 同时考虑到对抗彩虹表(即提前计算一批文本和哈希值的对应)一般采用在密码后加随机字符串的方式(即加盐)`hash(pass + salt)`, 数据库记录salt和哈希值即可

# 对称加密
加密和解密使用相同的密钥, 适用于需要快速加密和解密的场景, 密钥分发和管理较为复杂

伪代码如下: 
```
keygen() -> key  #随机

encrypt(plaintext: array<byte>, key) -> array<byte>  #加密
decrypt(ciphertext: array<byte>, key) -> array<byte>  #解密
```

性质上需要保证: 
- 只有密文而无密钥的情况下难以强行得到明文
- 正确性: decrypt(encrypt(m, k), k) = m

密钥的生成可以借助相关函数


# 非对称加密
使用两个具有不同功能的密钥：不向外公布的私钥(private key)和可以公布的公钥(public key), 加密和解密速度较慢

伪代码如下: 
```
keygen() -> (public key, private key)  #随机

encrypt(plaintext: array<byte>, public key) -> array<byte>  #加密
decrypt(ciphertext: array<byte>, private key) -> array<byte>  #解密

sign(message: array<byte>, private key) -> array<byte>  #生成签名
verify(message: array<byte>, signature: array<byte>, public key) -> bool  #验证签名是否由和这个公钥相关的私钥生成
```

同样的, 性质上需要保证: 
- 只有密文而无私钥的情况下难以强行得到明文
- 正确性: decrypt(encrypt(m, public key), private key) = m
- 在缺失私钥的情况下, 对于任意信息均难以强行找到签名使得verify(message, signature, public key)通过
- 签名正确性: verify(message, sign(message, private key), public key) = true

非对称性使得
- 我们可以公布公钥, 并单向安全的接受由公钥加密的信息
- 签名机制: 用私钥加密一段文字, 并让收信者用公钥验证是否为你发送

# 两种加密方式的对比
引用一下讲义: 

对称加密就好比一个防盗门: 只要是有钥匙的人都可以开门或者锁门.  

非对称加密好比一个可以拿下来的挂锁. 你可以把打开状态的挂锁(公钥)给任何一个人并保留唯一的钥匙(私钥). 这样他们将给你的信息装进盒子里并用这个挂锁锁上以后, 只有你可以用保留的钥匙开锁. 