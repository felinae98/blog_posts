### RSA相关
#### 查看公钥信息
`openssl rsa -pubin -text -modulus -in public.pem`
#### 自定义参数生成私钥
假设给定以下参数
p =17, q = 11，n = 187, e= 7 & d = 23
要生成相关的私钥解密文件
首先计算以下参数
$$d \bmod(p-1) = 23 \bmod 16 = 7$$
$$d \bmod(q-1) = 23 \bmod 10 = 3$$
$$q^{-1} \bmod p = 14$$
把以下内容写入文件
```
asn1=SEQUENCE:rsa_key

[rsa_key]
version=INTEGER:0
modulus=INTEGER:187
pubExp=INTEGER:7
privExp=INTEGER:23
p=INTEGER:17
q=INTEGER:11
e1=INTEGER:7
e2=INTEGER:3
coeff=INTEGER:14
```
生成私钥
`openssl asn1parse -genconf [config file] -out newkey.der`
进行验证
`openssl rsa -in newkey.der -inform der -text -check`
#### 加密解密
`openssl rsautl -encrypt -pubin -inkey pubkey.pem [-keyform ...] -in a.txt -out b.txt`
`openssl rsautl -decrypt -inkey prikey.pem [-keyform ...] -in b.txt`