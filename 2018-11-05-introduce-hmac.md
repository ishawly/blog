---
title: 数据防篡改之HMAC
date: 2018-11-05 10:02:33
tags:
---

# 前言
有一串不希望做任何修改的带参数的URL，一般做法是按`key=value[&]`拼接参数并散列函数加密，再将加密值附在URL上，服务器收到请求后对请求参数以同样的方式获得加密值并比对。但这样任然有可能被伪造。随之产生了HMAC(Hash-based Message Authentication Code)解决方案。

<!-- more -->

# 实现原理
HMAC算法使用一个加密的键值对明文进行双重散列处理

1. 如果键值的长度小于64字节，就用\0把键值扩展到64字节；如果键值长度大于64字节，先用散列函数处理(一般散列函数处理后长度为64字节，若不足则\0填充)
2. 构建 opad (用0x5C异或的64字节的键值)和 ipad (用0x36异或的64字节的键值)
3. 散列函数处理ipad拼接明文
4. 散列函数处理opad拼接第三部得到值

其公式如下：`H(K XOR opad, H(K XOR ipad, text))`
- H：用到的散列函数(md5, sha1等等)
- K：用零(0x0)扩展到64字节的键值
- opad：64字节长度的0x5C
- ipad：64字节长度的0x36
- text：明文

# PHP实现
## 1. 调用内置的函数`hash_hmac('md5', $text, $K)`

## 2. 根据上述原理实现
```
class Crypt_HMAC {
    private $_func, $_ipad, $_opad;

    function __construct($key, $method="md5")
    {
        if (!in_array($method, ['sha1', 'md5'])) {
            die("Unsupported hash function '$method'.");
        }
        $this->_func = $method;

        // 填充关键字
        if( strlen($key) > 64) {
            $key = pack('H32', $method($key));
        }

        if( strlen($key) < 64 ) {
            $key = str_pad($key, 64, chr(0));
        }

        // 计算填充的关键字, 并且保存他们
        $this->_ipad = substr($key, 0, 64) ^ str_repeat(chr(0x36), 64);
        $this->_opad = substr($key, 0, 64) ^ str_repeat(chr(0x5C), 64);
    }

    function hash($data)
    {
        $func = $this->_func;
        $inner = pack('H32', $func($this->_ipad . $data));
        $digest = $func($this->_opad . $inner);

        return $digest;
    }
}

// 测试
// 键值K
$secretKey = 'secret'; 
// 明文text
$data = 'The quick brown fox jumped over the lazy dog.';

// 使用内置函数
echo hash_hmac('md5', $data, $secretKey);
echo PHP_EOL;

$h = new Crypt_HMAC($secretKey, 'md5');
$hash = $h->hash($data);
echo $hash;
echo PHP_EOL;
```

# 后记
使用HMAC可以有效的防止数据被伪造。本文参考了**PHP5权威编程**，感谢作者。

