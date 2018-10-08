![](/wp-content/uploads/2018/08/U74P2DT20110906075017.jpg)
相关概念请参阅[朱双印的博客](http://www.zsythink.net/archives/1199)
### 命令总览
```
  --append  -A chain		添加到链
  --check   -C chain		检查规则是否存在
  --delete  -D chain		删除链中匹配的规则
  --delete  -D chain rulenum
				删除链中第几条规则（从1开始）
  --insert  -I chain [rulenum]
				在链中指定位置添加规则
  --replace -R chain rulenum
				替换链中指定位置的规则
  --list    -L [chain [rulenum]]
				列出指定链中或所有链中的规则
  --list-rules -S [chain [rulenum]]
				打印出指定链或所有链中的规则
  --flush   -F [chain]		删除指定链中或所有链中的规则
  --zero    -Z [chain [rulenum]]
				Zero counters in chain or all chains
  --new     -N chain		新建一条链
  --delete-chain
            -X [chain]		删除自定义链
  --policy  -P chain target
				改变链的策略
  --rename-chain
            -E old-chain new-chain
				改变链的名字
```
### 选项
```
 --protocol			-p proto				protocol: by number or name, eg. `tcp'
 --source			-s address[/mask][...]	source specification
 --destination		-d address[/mask][...]	destination specification
 --in-interface		-i input name[+]		network interface name ([+] for wildcard)
 --jump				-j target				target for rule (may load target extension)
 --goto				-g chain				jump to chain with no return
 --out-interface	-o output name[+]		network interface name ([+] for wildcard)
 --table			-t table				table to manipulate (default: `filter')
 --line-numbers								print line numbers when listing
 --fragment			-f						match second or further fragments only
```
### 自带链
```
    INPUT		处理输入数据包。
    OUTPUT		处理输出数据包。
    PORWARD		处理转发数据包。
    PREROUTING	用于目标地址转换（DNAT）。
    POSTOUTING	用于源地址转换（SNAT）。
```
### 表名
```
    raw		高级功能，如：网址过滤。
    mangle	数据包修改（QOS），用于实现服务质量。
    net		地址转换，用于网关路由器。
    filter	包过滤，用于防火墙规则。
```
### 常用命令
查看现有规则：
`iptables -L -n -v --line-numbers`
放行指定端口：
`iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
禁止其他链接访问：
`iptables -A INPUT -j REJECT`
或
`iptables -P INPUT REJECT`
屏蔽ip：
`iptables -I INPUT -s 123.4.56.7 -j REJECT`
