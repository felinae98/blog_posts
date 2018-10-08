 database  | script  | usage
 :------------ | :------------ | :------------
 MySQL > 5.6  | 0x2char | 把0x...的字符串转换成CONCAT(CHAR(12),CHAR(23))的形式
 all  | apostrophemask  | 用utf-8全角的单引号代替英文的单引号
  all | apostrophenullencode  |  把单引号替换为%00%27
  Access | appendnullbyte  | 在payload最后添加%00
 all  | base64encode  | 字面意思（Unicode编码）
 all  | between  | 把>和=用between来替换
  all |  bluecoat | 把sql关键字后的空格替换为%09，把=替换为like
 all | chardoubleencode| 进行两次URL编码
all |charencode|对全部字符URL编码
all|charunicodeencode|对所有字符进行Unicode-url编码（%u0052）
all|charunicodeescape|Unicode-escapes所有字符（\\u0052）
Mysql|commalesslimit|去掉LIMIT的逗号
Mysql|commalessmid|去掉MID（）的逗号
all|commentbeforeparentheses|在函数与参数间加上/\*\*/
MySQL| concat2concatws| 把concat替换为concat_ws
!PosgreSql| equaltolike| 把=替换成LIKE
all| escapequotes| 在‘和"前加\
all| greatest| 替换>符号
all| least| 替换<符号
MySQL < 5.1| halfversionedmorekeywords| 在关键字前加注释
all| htmlencode|html实体编码
all|ifnull2casewhenisnull|用CASE WHEN ISNULL替换IFNULL
all| ifnull2ifisnull| 用IF ISNULL替换IFNULL
all|informationschemacomment|把information_schema后加/\*\*/
all|lowercase| 字面意思
MySQL| modsecurityversioned| Embraces complete query with versioned comment(看不懂) 用于ModSecurity
MySQL | modsecurityzeroversioned| 同上
all | multiplespaces| 在有空格的地方多加几个空格
all | nonrecursivereplacement| 双写关键词(UNION, SELECT, INSERT, UPDATE, FROM. WHERE)
all | overlongutf8 |利用utf-8 decoder的漏洞，转换非字母字符
all| overlongutf8more| 同上，转换所有字符
ASP|percentage| 在每个字母前加%
MSSQL| plus2concat| 用CONCAT替换+
MSSQL| plus2fnconcat |用{fn CONCAT}替换+
all| randomcase| 随缘大小写
all | randomcomments| 随缘块注释
all | securesphere| 在payload后面加 and '0having'='0having'(贼蠢)
MSSQL| sp_password | 如果payload有行注释，在最后价格sp_password
all | space2comment| 用块注释替换空格
MSSQL, SQLite| space2dash | 用行注释(--)替换空格
MySQL| space2hash| 用行注释(#)替换空格
all| space2morecomment| 用块注释(/\*\*\_\*\*/)替换空格
MySQL| space2morehash| 用块注释(\#)替换空格，并且加戏
MSSQL| space2mssqlblank| 用随机空格字符替换空格
MSSQL |space2mssqlhash | 用行注释(#)替换空格
MySQL | space2mysqlblank| 用随机空白字符替换空格
MySQL| space2mysqldash | 用行注释(--)替换空格
all | space2plus| 用+替换空格
all| space2randomblank| 用随机空白字符替换空格
all | symboliclogical|用\|\|和&&替换OR和AND
all | unmagicquotes| 宽字节注入(%bf)
all | uppercase | 字面意思
all | varnish | 加X-originating-IP的http头
MySQL |versionedkeywords| 给关键字加/\*!keyword\*/
MySQL| versionedmorekeywords| 同上，加的范围更多
all | xforwardedfor| 随机xff头
