## 低加密指数广播攻击
如果选取的加密指数较低，并且使用了相同的加密指数给一个接受者的群发送相同的信息，那么可以进行广播攻击得到明文。
即，选取了相同的加密指数e（这里取e=3），对相同的明文m进行了加密并进行了消息的传递，那么有：
$$c_1\equiv m^e\ mod \ n_1$$
$$c_2\equiv m^e\ mod \ n_2$$
$$c_3\equiv m^e\ mod \ n_3$$
 对上述等式运用中国剩余定理，在e=3时，可以得到：
 $$c_x\equiv m^3mod\ n_1n_2n_3 $$
所以$m=\sqrt[3]{c_x}$

> 中国剩余定理求解过程
	假设有
	$$y_1\equiv x\bmod n_1$$
	$$y_2\equiv x\bmod n_2$$
	$$y_3\equiv x\bmod n_3$$
	那么我们可以构造$z_1$使
	$$z_1=n_1n_2\ ((n_1n_2)^{-1}mod\ n_3)$$
	对于$z_1$有这样几点性质
	$$z_1\equiv0\bmod n_1, z_1\equiv0\bmod n_2, z_1\equiv1\bmod n_3$$
	所以我们按相同的方法构造$z_2, z_3$
	最后得到$z=z_1y_1+z_2y_2+z_3y_3$
	同时满足$x$的性质
	所以容易得到$x\equiv z \bmod n_1n_2n_3$

## 低解密指数攻击
如果e很大，可以考虑低解密指数攻击。
与低加密指数相同，低解密指数可以加快解密的过程，但是者也带来了安全问题。Wiener表示如果满足：
$d<\frac{1}{3}n^\frac{1}{4}$
那么一种基于连分数(一个数论当中的问题)的特殊攻击类型就可以危害RSA的安全。此时需要满足：
$p\lt q\lt 2p\$
就可以在多项式时间内分解n。
在github有现成的[轮子](https://github.com/pablocelayes/rsa-wiener-attack)
```python
import  RSAwienerHacker
d =  RSAwienerHacker.hack_RSA(e,n)
if d:
	print(d)
```
## 共模攻击
如果在RSA的使用中使用了相同的模n对相同的明文m进行了加密，那么就可以在不分解n的情况下还原出明文m的值。
$$c_1\equiv m^{e_1}\mod n$$
$$c_2\equiv m^{e_2}\mod n$$
 此时不需要分解n，不需要求解私钥，如果两个加密指数互素，就可以通过共模攻击在两个密文和公钥被嗅探到的情况下还原出明文m的值。
过程如下，首先两个加密指数互质，则：
$$gcd(e_1,e_2)=1$$
即存在$s_1,s_2$使$s_1e_1+s_2e_2=1$
对上式代入化简可得
$$c_1^{s_1}c_2^{s_2}\equiv m^{s_1e_1+s_2e_2}\mod n$$
$$c_1^{s_1}c_2^{s_2}\equiv m\mod n$$
可以通过拓展欧几里得算法得出$s_1,s_2$，最终算出原文。