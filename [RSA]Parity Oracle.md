在CTF密码学中有一种RSA题目，题目通过某种方法提供一个解密机，但是只会返回解密后明文的最后一比特位，既返回解密出明文的奇偶。我们可以用log(N)的复杂度确定m，举例如下：
首先我们有$n, e, c$，并且有$m=c^d\bmod n$
我们可以先传入$c$，服务器算出$m$，返回m的奇偶性，当然这并没有什么用所以我们考虑下一步。
我们在服务器中传入$c* 2^e\bmod n$，服务器计算出$m'=(c* 2^e)^d\bmod n = 2m\bmod n$，我们考虑到如果$2m\lt n$，那么$m'$一定是一个偶数；如果$2m\geqslant n$， 那么$m'=2m-n$，同时我们注意到$n=pq$，$p$和$q$都是大质数，所以它们都是奇数，所以$n$也是奇数，那么$m'$就一定是个奇数。
我们现在已经可以通过一次查询把m的范围缩小到了n/2，我们是否可以通过二分法确定m的准确的值？以数学归纳法的思想，我们现在已经完成了第一步的证明，现在要完成的是对之后部分的证明。
假设现在已经把m的范围确定到了$\frac{xn}{2^i}\leqslant m\lt\frac{(x+1)n}{2^i}$，那么下面我们查询$c* 2^{(i+1)e}\bmod n$，服务器会返回给我们$m'=2^{(i+1)}m\bmod n=2^{(i+1)}m-kn$，那么：
$$
\begin{aligned}
&0\leqslant2^{(i+1)}m-kn\lt n\\\\
&\frac{kn}{2^{(i+1)}}\leqslant m\lt \frac{(k+1)n}{2^{(i+1)}}
\end{aligned}
$$
并且考虑到初始条件：
$$
\begin{aligned}
\frac{xn}{2^i}&\leqslant m\lt\frac{(x+1)n}{2^i}\\\\
\frac{(2x)n}{2^{(i+1)}}&\leqslant m\lt\frac{(2x+2)n}{2^{(i+1)}}
\end{aligned}
$$
因为x和k都是整数，要满足原先的条件，$k$只能取$2x$或$(2x+1)$。
$2^{(i+1)}$是偶数，$n$是奇数。所以当$m'$是奇数的时候，$2^{(i+1)}-kn$是奇数，$k$必然是奇数；反之，如果$2^{(i+1)}-kn$是偶数，$k$必然是偶数。
如果$k$是奇数，$k$只能取$(2x+1)$，m被约束到$\frac{(2x+1)n}{2^{(i+1)}}\leqslant m\lt\frac{(2x+2)n}{2^{(i+1)}}$；
如果$k$是偶数，$k$只能取$2x$，m被约束到$\frac{(2x)n}{2^{(i+1)}}\leqslant m\lt\frac{(2x+1)n}{2^{(i+1)}}$。
这样二分下去直到上下界只差小于1.
```python
#!/usr/bin/python3

def parity_oracle(n, query):
    """input: n: the modulus of the RSA
    query: query is a function which inputs an int i, returns if the m*(2^i)%n is odd
    return: int m
    """
    i = 0
    x = 0
    while n >> i:
        res = query(i+1)
        if res:
            x = 2 * x + 1
        else:
            x = 2 * x
        i += 1
    return (x+1) * n // 2 ** i

if __name__ == "__main__":
    from Crypto.PublicKey import RSA
    from Crypto.Util import number
    key = RSA.generate(2048)
    n = key.n
    m = number.getRandomRange(0, n)
    print(m)
    m2 = parity_oracle(n, lambda x: (pow(2, x, n) * m % n) & 1)
    print(m2)
    print(m2 == m)
```
