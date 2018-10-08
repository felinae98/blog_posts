Crypto模块是Python中一个很成熟的现代密码学模块，无论是做题/出题，还是里面的一些小工具都很好用。
安装：
`pip(3) install pycrypto`
### 常用工具
* 异或两个bytes
```python
from Crypto.Util.strxor import strxor
strxor(b'abc', b'def')
```
* 给一个bytes的每一位异或一个数字
```python
from Crypto.Util.strxor import strxor_c
strxor_c(b'abc', ord('a'))
```
* 最大公约数
```python
from Crypto.Util.number import GCD
GCD(36,12)
```
* 模逆
```python
from Crypto.Util.number import inverse
inverse(3,5)
```
* 取随机质数
```python
from Crypto.Util.number import getPrime
getPrime(512)
```
* bytes和数字互相转化
```python
from Crypto.Util.numbet import bytes_to_long, long_to_bytes
bytes_to_long(b'felinae')
long_to_bytes(28821963924201829)
```