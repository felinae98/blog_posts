不装虚拟机的开发环境不是好开发环境，踩了好多坑终于在deepin中装好了Homestead，Ubuntu应该和这个教程类似
# 安装vagrant
	sudo aptitude install vagrant
*推荐使用aptitude,解决包依赖愉快地一批*
# 导入box
	vagrant box add laravel/homestead
但是在国内几乎不可能顺利地把它下下来，有两个解决办法：
借一把梯子，proxychains4 
```
proxychains4 vagrant box add laravel/homestead
```
或者是用迅雷什么的把安装包下下来
https://app.vagrantup.com/laravel/boxes/homestead/versions/4.0.0/providers/virtualbox.box
把4.0.0改成现在的版本号就好
然后
```
vagrant box add laravel/homestead homestead.box
```
1G左右，哪种方式都不会很快
然后运行
```bash
vagrant box list
```
![](https://code.felinae98.cn/wp-content/uploads/2017/11/cc2227b4745cc138934c050d27aa3ba7.png)
# 下载更改配置
### 下载与初始化
然后继续官方教程，下载homestead
```bash
git clone https://github.com/laravel/homestead.git Homestead
cd Homestead
bash init.sh
```
更改Homestead.yaml文件， 这个文件是homestead的设置文件
### 更改ip和provider
```yaml
ip: "192.168.10.10"
provider: virtualbox
```
如果连了192.168局域网的把ip改成不同的网段，不然你的网卡可能会直接懵逼
provider直接填virtualbox就可以了
### 更改共享文件夹
```yaml
folders:
- map: /home/你的用户名/code  #（laravel项目的所在地）
to: /home/vagrant/Code # 这个就不用改了
```
这个配置的是虚拟机的共享文件夹，一般共享laravel的项目所在文件夹
Homestead.yaml文件中的folders属性列出了所有主机和 Homestead 虚拟机共享的文件夹，一旦这些目录中的文件有了修改，将会在本地和 Homestead 虚拟机之间保持同步，如果有需要的话，你可以配置多个共享文件夹（一般一个就够了）
### 添加站点
```yaml
sites:
- map: liang.app
to: /home/vagrant/Code/public
```
对 Nginx 不熟？没问题，通过sites属性你可以方便地将“域名”映射到 Homestead 虚拟机的指定目录，Homestead.yaml中默认已经配置了一个示例站点。和共享文件夹一样，你可以配置多个站点
### 改Hosts文件
更改`/etc/hosts`加入你的虚拟机ip与虚拟站点名称
例如`192.168.10.10 liang.app`
# 启动vagrant虚拟机
首先保证你的电脑中已经生成过公钥和私钥
如果没有的话首先运行一下
```bash
ssh-keygen
```
在Homestead文件夹下运行
```bash
vagrant up
```
成功运行就直接进入下一步
如果出现了报错
```
        default: Running: inline script
==> default: mesg:
==> default: ttyname failed
==> default: :
==> default: Inappropriate ioctl for device
```
修改Vagrantfile， 在Vagrant.configure后面加一行
```
config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
```
再运行
```
vagrant  provision
```
初始化虚拟机
这时候ping一下192.168.10.10如果通说明虚拟机正常启动了，可跳过这个步骤剩下的内容
如果不能连接到该ip说明配置有问题
打开虚拟机发现虚拟机正常运行
ifconfig发现并没有虚拟vboxnet0的虚拟网卡，需要手动启动网卡
```
sudo ifconfig vboxnet0 up
```
可ping通app.liang
# 安装laravel
切换到要安装laravel的目录下
先换个composer的中国源
```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```
然后安装laravel
```
composer create-project laravel/laravel --prefer-dist
```
最后访问app.liang,大功告成

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">知识共享署名-相同方式共享 4.0 国际许可协议</a>进行许可。