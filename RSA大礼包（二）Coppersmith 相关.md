需要sagemath进行计算，这里贴上[github上的轮子](https://github.com/mimoo/RSA-and-LLL-attacks)和[某位日本友人的博客](http://inaz2.hatenablog.com/entry/2016/01/20/022936)
sagemath下载安装需要4G，如果不想安装可以使用[在线sage计算](http://sagecell.sagemath.org/)
## Stereotyped messages
若e较小，并且知道明文m的高位，可以这样进行攻击：
用sage运行以下代码
下载[相关源码](https://github.com/mimoo/RSA-and-LLL-attacks)，启动sage
```python
load("coppersmith.sage")
N = #N的值
e = 3 #e的值
m = #m的大概值
c = #c的值
ZmodN = Zmod(N)
P.<x> = PolynomialRing(ZmodN)
f = (m + x)^e - c
dd = f.degree()
beta = 1
epsilon = beta / 7
mm = ceil(beta**2 / (dd * epsilon))
tt = floor(dd * mm * ((1/beta) - 1))
XX = ceil(N**((beta**2/dd) - epsilon))
roots = coppersmith_howgrave_univariate(f, N, beta, mm, tt, XX)
```
求出的结果是与m的差值，既$$f(x)=(m+x)^e-c$$的根
特别的，XX是根的上界，根据你知道的m的位数进行调整
## Partial Key Exposure Attack(部分私钥暴露攻击)
若e较小，已知d的低位
```python
def partial_p(p0, kbits, n):
    PR.<x> = PolynomialRing(Zmod(n))
    nbits = n.nbits()

    f = 2^kbits*x + p0
    f = f.monic()
    roots = f.small_roots(X=2^(nbits//2-kbits), beta=0.3)  # find root < 2^(nbits//2-kbits) with factor >= n^0.3
    if roots:
        x0 = roots[0]
        p = gcd(2^kbits*x0 + p0, n)
        return ZZ(p)

def find_p(d0, kbits, e, n):
    X = var('X')

    for k in xrange(1, e+1):
        results = solve_mod([e*d0*X - k*X*(n-X+1) + k*n == X], 2^kbits)
        for x in results:
            p0 = ZZ(x[0])
            p = partial_p(p0, kbits, n)
            if p:
                return p


if __name__ == '__main__':
    n = 0x00bef498e6eb2cffe71312da47ab89d2c47db7438ea2cfa992ddddbc2a01978001fc51e286e6ebf028396cdb8b3323c60e6b9d50cd84187cf7f48e3875a2f0890f70b02333ad89db2923863ce146562286f63fb0a1d0198e3a6862ba5ac12e85a5c6d0d27cb1c81bdf69cc5bc95b8001a2f744517f9437b4ddd5a076fc0e9a5de1a7a268c40f31aa29e8dc27c0b3a182299ca7a9335b4bd4585452f6107c238e486c98dd73a5f9862e9e80b152f53381c72f897107551c281259ac3ee32c4b4f46cc03127d1bf699acd0266f3c6729253c70da0c69b1560fa172735709866b375b6eba294e1ce8b46fba798ba380080b4bf9603998cac199d9cd46e30ae8da9e7f
    e = 3
    d = 0x7f4dbb449cc8aa9a0cb73c2fc7b1372da924d7b46c8a710c93e9281c010faaabfd8bec59ef47f5702648925cccc284099d138b33ad65a8a54db425a3c1f5b0b4f5cac22273b13cc617aed340d98ec1af4ed5206be011097c459726e72b7459192f35e1a8768567ea46883d30e7aaabc1fa2d8baa62cfcde93915a4a809bc3e9547bb07e1ecca16e51078312e89f0561e31b55db8b0ea5bc87a6ca7464a3d7c28a68c60e2ba88fe6a7d2b300d723e549910a987da89fc0a1c0de197a3d62c501b1f0e819891b1c32a0d6c233f2a285df87bb9e5c6c72d983ff3e706696bba639f573f9c3646968f02f3a615a438e20bb7c38d53621079f2899547a95350f3abeb

    beta = 0.5
    epsilon = beta^2/7

    nbits = n.nbits()
    kbits = floor(nbits*(beta^2+epsilon))
    d0 = d & (2^kbits-1)
    print "lower %d bits (of %d bits) is given" % (kbits, nbits)

    p = find_p(d0, kbits, e, n)
    print "found p: %d" % p
    q = n//p
    print d
    print inverse_mod(e, (p-1)*(q-1))
```
d0代表已知的d的较低位，kbits是已知的d0的位数。
## Factoring with high bits known
已知因子中的高位
```python
load("coppersmith.sage")
N = #N的值
ZmodN = Zmod(N)
P.<x> = PolynomialRing(ZmodN)
qbar = #q的近似值
f = x - qbar
beta = 0.5
dd = f.degree()
epsilon = beta / 7
mm = ceil(beta**2 / (dd * epsilon))
tt = floor(dd * mm * ((1/beta) - 1))
XX = ceil(N**((beta**2/dd) - epsilon)) + 1000000000000000000000000000000000
roots = coppersmith_howgrave_univariate(f, N, beta, mm, tt, XX)
```
我们要有$$q\geqslant N^{beta}$$,XX是解的上界