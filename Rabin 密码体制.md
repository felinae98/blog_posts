>Thm1 (Rabin密码体制)设$$n=pq$$，其中p和q是素数，且$$p,q\equiv3\mod 4$$，设$$ P=C=Z_n^*$$，且定义
κ={(n,p,q)}
对K=(n,p,q)，定义
$$e_K(x)=x^2\mod n$$
和
$$d_K=\sqrt y\mod n$$
n为公钥，p和q为私钥。
注：条件$$p,q\equiv3\mod 4$$
可以省去，条件P=C=Zn⋆也可以弱化为P=C=Zn，只是在使用更多的限制性描述的时候，简化了许多方面的计算和密码体制分析。

当$$p\equiv3\mod 4$$时，有一个简单公式来计算模p的二次剩余的平方根，假定y是一个模p的二次剩余，且$$y\equiv3\pmod 4$$那么有
$$(\pm y^\frac{p+1}{4})^2 \equiv y^\frac{p+1}{2}\pmod p\equiv y^\frac{p-1}{2}y\pmod p$$
由欧拉准则我们可以知道，当y是一个模p的二次剩余时，有$$y^\frac{p-1}{2}\equiv1\pmod p$$
所以就有$$(\pm y^\frac{p+1}{4})^2 \equiv y\pmod p$$
$$\sqrt{y}\pmod p\equiv \pm y^\frac{p+1}{4}$$
所以我们可以解出$$\pm y_1\equiv x\pmod p, \pm y_2\equiv x\pmod q$$
运用中国剩余定理构造$$s_1,s_2$$
$$s_1=q(q^{-1}\pmod p)$$
$$s_2=p(p^{-1}\pmod q)$$
则x可能的四个值为
$$x\equiv\pm y_1s_1\pm y_2s_2\pmod {n}$$
