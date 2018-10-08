## 故事背景
我的学生乞丐服务器里跑着三个人的博客，用的是一个巨老的Mariadb做服务器，燃鹅这个数据库三天崩了两次，所以必须把服务器docker化提上日程了，以前尝试但是失败了，现在是时候完成它了。
## 前期准备
* 卸载wp-supercache，因为这个插件里配置文件一堆绝对路径，怕被坑随意先卸载吧
* 备份数据库
* 备份网页文件
* systemctl stop/disable httpd
* systemctl stop/disable mariadb
* 宣布网站倒闭！

## 数据库的docker化
我们使用的是[mariadb](https://hub.docker.com/_/mariadb/)。可以新开一个mariadb的容器，把数据-v到其他的地方，但是我们的机器上原来跑着一个数据库，/var/lib/mysql理还有原来的数据。按照mariadb官方文档的说法，我们可以在原来已有数据的基础上运行一个数据库。
> If you start your mariadb container instance with a data directory that already contains a database (specifically, a mysql subdirectory), the $MYSQL_ROOT_PASSWORD variable should be omitted from the run command line; it will in any case be ignored, and the pre-existing database will not be changed in any way.

不设置密码就行，所以我们
`docker run -v /var/lib/mysql:/var/lib/mysql --name main_sql mariadb`
运行数据库容器。
但这时候我们遇到了一个尴尬的问题，原来的数据库是默认不能远程登录root的，但是在容器里运行数据库必须能远程登录root，否则其他的容器会连不上这个数据库，我们必须进入容器登录数据库然后完成授权操作。
`docker exec -it main_sql mysql -uroot -p`
进入数据库后执行：
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your_password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
成功授权其他容器访问数据库。
## web服务的docker化
我们的服务器中运行着不同域名的多个博客，不同的博客可以放置在不同容器中，再使用一个[nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/)容器实现把流量反向代理到各个容器。只要在目标容器中加入一条环境变量，就能自动反向代理到该容器。同时，这个容器还能实现https，集中管理证书，外部https流量解密成http再代理到对应的容器；同时配合[docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)，自动获取letsencrypt证书，再也不用担心证书过期了。
具体来说，我们先运行两个容器：
```
$ docker run -d -p 80:80 -p 443:443 \
    --name nginx-proxy \
    -v /path/to/certs:/etc/nginx/certs:ro \
    -v /etc/nginx/vhost.d \
    -v /usr/share/nginx/html \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    --label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy \
    jwilder/nginx-proxy
```
```
$ docker run -d \
    -v /path/to/certs:/etc/nginx/certs:rw \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    --volumes-from nginx-proxy \
    --name nginx-letsencrypt \
    jrcs/letsencrypt-nginx-proxy-companion
```
之后运行web容器时，只需要加上`-e VIRTUAL_HOST=<域名> -e LETSENCRYPT_HOST=<域名> -e LETSENCRYPT_EMAIL=<邮箱>`就行
> 注：nginx-proxy默认启用HSTS，如果你不确定你会不会遇到不能成功上https，mixed content之类的问题，建议加上`-e HSTS=off`

然后是WordPress的容器化，在[官方页面](https://hub.docker.com/_/wordpress/)里并没有提到-v的用法，但是根据docker里的脚本我们可以发现，可以把相关数据-v进/var/www/html中。同时我们要注意，虽然在外部使用了https但是最终到达WordPress容器的流量是http，所以WordPress会认为你没有使用https而疯狂302，在WordPress官方文档上我们发现：
> Using a Reverse Proxy
>
>If WordPress is hosted behind a reverse proxy that provides SSL, but is hosted itself without SSL, these options will initially send any requests into an infinite redirect loop. To avoid this, you may configure WordPress to recognize the HTTP_X_FORWARDED_PROTO header (assuming you have properly configured the reverse proxy to set that header).

所以要在wp-config.php中加入：
```php
define('FORCE_SSL_ADMIN', true);
// in some setups HTTP_X_FORWARDED_PROTO might contain 
// a comma-separated list e.g. http,https
// so check for https existence
if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false)
       $_SERVER['HTTPS']='on';
```
应该就可以顺利登录了。
如果发现不能装主题插件，对目录没有写权限，需要`docker exec <容器名> chown -R www-data:www-data /var/www/html`
到此迁移工作全部完成