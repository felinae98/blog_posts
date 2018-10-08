ElGamal算法的安全性是基于求解离散对数问题的困难性，于1984年提出，也是一种双钥密码体制，既可以用于加密又可用于数字签名。
假设p是一个大素数，并且p-1有大素因子，$$\alpha$$是$$Z^{\ast}_p$$的生成元.,并且$$y\in Z^{\ast}_p$$，那么要寻找a，使$$\alpha^a\equiv y\pmod p$$是很难实现的，我们把a记为$$a=\log _{\alpha}{y}$$.
### 算法实现
我们先选取一个大质数p，取$$Z_p^\ast$$乘法运算生成元$$\alpha$$.
选取$$a$$，计算$$\beta = \alpha^a\pmod p$$
其中私钥为$$(a)$$，公钥为$$(\alpha, \beta, p)$$
加密时选取一个随机数k，定义$$e_k(m,k) = (y_1,y_2)$$
其中$$y_1=\alpha^k\bmod p,\ y_2 = m\beta^k\bmod p=m\alpha^{ak}\bmod p$$
同时定义解密函数$$d_k(y_1,y_2)=m$$
$$m = y_2(y_1^a)^{-1}\bmod p$$
$$y_2(y_1^a)^{-1}\bmod p=m\alpha^{ak}(\alpha^{ak})^{-1}\bmod p=m$$

要求m就必须求$$(\beta^k)^{-1}\bmod p$$也就必须求$$\beta^k\bmod p=\alpha^{ak}\bmod p$$，从密文和公钥中可以获取到$$\beta=\alpha^a\bmod p,a^k\bmod p$$，但是并不能很快的求出$$\beta^k\bmod p$$