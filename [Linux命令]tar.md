没错，死也学不会的tar
```
  -c, --create               创建一个新归档
  -t, --list                 列出归档内容
  -x, --extract, --get       从归档中解出文件
  -f, --file=ARCHIVE         使用归档文件或 ARCHIVE 设备
  -a, --auto-compress        使用归档后缀名来决定压缩程序
  -j, --bzip2                通过 bzip2 过滤归档
  -J, --xz                   通过 xz 过滤归档
  -z, --gzip, --gunzip, --ungzip   通过 gzip 过滤归档
  -v, --verbose              详细地列出处理的文件
```
### 新建压缩文件
首先新建压缩文件，
`-c`是create的意思，`-f`是file，指定归档文件
所以可以通过
`tar -cf example.tar file/dir`
创建包含file的归档文件example.tar，但是这个tar文件是没有压缩过的，
`-j`代表bzip2，后缀bz2，`-z`代表gzip，后缀gz
所以可以
`tar -czf example.tar.gz file/dir`
或者
`tar -cjf example.tar.bz2 file/dir`
建立压缩文件。
也可以使用`-a`flag自动通过后缀识别压缩类型
`tar -caf example.tar.gz file/dir` 或者`tar -caf example.tar.bz2 file/dir`
建立压缩文件
### 查看文件
`-v`是详细列出文件的意思
`tar -tvf example.tar`
查看压缩包中的文件
### 解压文件
`-x`代表extract，意味提取解压
解压自动识别归档文件压缩类型不需要手动制定压缩方式
`tar -xf example.tar.gz`
解压文件
`tar -xvf example.tar.gz`
解压时显示解压的文件