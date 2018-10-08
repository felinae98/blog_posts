axel是Linux下一个不错的HTTP/ftp高速下载工具。支持多线程下载、断点续传，且可以从多个地址或者从一个地址的多个连接来下载同一个文件。
安装：
```shell
sudo apt-get install axel
```
用法：
```shell
axel [options] url1 [url2] [url...]
```
参数设置：
```shell
--max-speed=x		-s x	指定最大速率（字节/秒）
--num-connections=x	-n x	指定最大连接数
--output=f			-o f	指定本地输出文件
--search[=x]		-S [x]	搜索镜像并从 X 服务器下载
--header=x			-H x	添加报头字符串
--user-agent=x		-U x	设置用户代理
--no-proxy			-N	不使用任何代理服务器
--insecure			-k	不校验 SSL 证书
--quiet				-q	使用输出简单信息模式
--verbose			-v	更多状态信息
--alternate			-a	另一种进度指示器
--help				-h	帮助信息
--version			-V	版本信息
```
