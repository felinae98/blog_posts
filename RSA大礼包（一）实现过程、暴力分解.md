# RSA大礼包（一）
![](/wp-content/uploads/2018/08/RSA-CMYK-COATED-POS-HI.jpg)
## 前言
RSA基于一个简单的数论事实，两个大素数相乘十分容易，将其进行因式分解确实困难的。在量子计算机还没有成熟的今天，RSA算法凭借其良好的抵抗各种攻击的能力，在公钥密码体制中发挥着中流砥柱的作用。然而即便RSA算法目前来说是安全可靠的，但是错误的应用场景，错误的环境配置，以及错误的使用方法，都会导致RSA的算法体系出现问题，从而也派生出针对各种特定场景下的RSA攻击方法。
## 算法实现
RSA是一种非对称加密，涉及三个参数n, e, d，其中n，e是公钥，n，d是私钥。
n是连个大素数p和q的积
$$\varphi(n)=(p-1)(q-1)$$
$$\varphi(n)$$是n的欧拉函数
e是与$$\varphi(n)$$互素的数字
一般e取65537，也就是0x10001
$$ d\equiv e^{-1} mod \varphi(n)$$
d是e模$$\varphi(n)$$的逆
设c是密文，m是明文，则加密过程如下：
$$c=m^e mod n$$
解密过程如下：
$$m=c^d mod n$$
只有知道d才能解密原文，要知道d就要知道$$\varphi(n)$$，就需要知道p和q，但是在目前的条件下，2048位以上的n被认为是安全的。
## 算法理论
> 占坑
## 模数分解
如果给的n比较小，可以直接分解pq，通过$$\varphi(n)=(p-1)(q-1)$$算出$$\varphi(n)$$，然后通过求模逆运算算出d。
```python
def egcd(a, b):
	if a == 0:
		return (b, 0, 1)
	else:
		g, y, x = egcd(b % a, a)
		return (g, x - (b // a) * y, y)
def modinv(a, m):
	g, x, y = egcd(a, m)
	if g != 1:
		raise Exception('modular inverse does not exist')
	else:
		return x % m
```
然后直接直接解出m
`pow(c,d,n)`
如果数字稍微大一点，可以去[factordb](http://factordb.com/)上查质因数。
也可以用yafu试试，[yafu](https://sourceforge.net/projects/yafu/)。
yafu用于自动整数因式分解，在RSA中，当p、q的取值差异过大或过于相近的时候，使用yafu可以快速的把n值分解出p、q值，原理是使用Fermat方法与Pollard rho方法等。
如果所给的n不能直接分解，就要想想其他的办法了。
## 相同素因子
如果有$$n_1,n_2$$有相同的素因子$$p$$，那么可以用辗转相除法算出$$p$$
$$p=gcd(n_1,n_2)$$
识别条件，如果给了一堆n，就可以考虑公因数攻击。
## 低加密指数攻击
当e=3时，如果明文过小，那么$$m^e < n$$，则可以对c直接开根，$$m=\sqrt[3]{c}$$。
如果$$m^e >n$$，但并不是足够大，可以尝试$$m=\sqrt[3]{c+kn} \quad k=1,2,3...$$，知道得到m是整数。
因为要开根的数字很大，Python自带的开根并不能满足要求，建议安装gmpy2
`sudo aptitude install python3-gmpy2`
```python
i = 0
while True:
	if gmpy2.root(c + i * N, e).is_interger():
		print(int(gmpy2.root(c + i * N, e)))
		break
	i += 1
```
