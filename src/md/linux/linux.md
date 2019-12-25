# Linux

## 按键
Tab  
Ctrl+c  
Ctrl+d(exit)  
[shift]+{[PageUP]|[Page Down]}  

### 基本指令
date  
cal  
bc  
man -k(apropos)  
man -f(whatis)  
info  
who 谁在线  
netstat -a
#### 关机  
sync  
shutdown  -k -h -r c 时间(now, 20:25, +10)
reboot  
poweroff  
halt  
systemctl  

#### 权限与目录配置
chgrp  
chown -R 目录文件和次文件夹  
chmod -R xyz  
u g o a  
chmod u=rwx,go=rx filename  


元件       内容           叠代物件         r             w            x
文件    详细数据data     文件数据夹    读到文件内容   修改文件内容    执行文件内容
目录    文件名           可分类抽屉    读到文件名     修改文件名      进入该目录的权限（key）

Filesystem Hierarchy Standard （FHS） 


可变| 可分享的 | 不可分享的 
- | :-: | -: 
不变的（static）|usr （软件放置处）|etc （配置文件）
不变的（static）|opt （第三方协力软件） |boot （开机与核心档） 
可变的（variable）|var mail （使用者邮件信箱）|var run （程序相关） 
可变的（variable）|var spool news （新闻群组）|var lock （程序相关）



根目录（ ）所在分区应该越小越好， 且应用程序所安装的软件最好不要与根目录放在同一个分区内，保持根目录越小越好。 如此不但性能较佳，根目录所在的文件系统也较不容易发生问题。  


usr=Unix Software Resource 

Linux Standard Base （LSB） 
uname -r  
uname -m  

#### 文件与目录管理

cd  
pwd -P 非链接路径
mkdir -m 权限 -p 递归创建  
rmdir -p 删除上层空的目录  

ls  -a 全部，隐藏 . ..
	-A 不包含. ..
 	-d 仅目录本身，无数据
 	-l 列表，权限
 	
cp  [-adfilprsu]  (source)  (destination)  
* -a: -dr --preserve=all  
* -d: link file  
* -f: force  
* -i: 若已经存在，覆盖时会先询问  
* -l: hard link  
* -p: 连同文件属性（权限用户时间）一起复制，不使用默认（备份） 
* -r: 递归复制（目录） 
* -s: 符号链接  


rm  
rm [-fir]  
* -f: force   
* -i: 提示  
* -r: 递归删除  

mv  
mv -[fiu] source destination
* -f: force   
* -i: 提示  
* -u: 若已经存在，source较新，才更新



ps aux | grep bc| xargs kill -9


java -jar bc-0.0.1-SNAPSHOT.jar > log.file 2>&1 &



## awk

```bash
# 格式
$ awk 动作 文件名

# 示例
$ awk '{print $0}' demo.txt
```

`$0`代表当前行

`awk`会根据空格和制表符，将每一行分成若干字段，依次用`$1`、`$2`、`$3`代表第一个字段、第二个字段、第三个字段等等。

```bash
$ echo 'this is a test' | awk '{print $3}'
```

`-F`参数指定分隔符

```bash
awk -F ':' '{ print $1 }' demo.txt
```



除了`$ + 数字`表示某个字段，`awk`还提供其他一些变量。

变量`NF`表示当前行有多少个字段，因此`$NF`就代表最后一个字段。

```bash
$ echo 'this is a test' | awk '{print $NF}'
```

`$(NF-1)`代表倒数第二个字段。

```bash
$ echo 'this is a test' | awk '{print $(NF-1)}'
```

print里的逗号代表输出时两部分空格分隔。

```bash
$ awk -F ':' '{print $1, $(NF-1)}' demo.txt
```



变量`NR`表示当前处理的是第几行

```bash
awk -F ':' '{print NR ") " $1}' demo.txt
```

```bash
echo 'this is a test' | awk '{print NR ")", $1}'
```

`awk`的其他内置变量如下。

> - `FILENAME`：当前文件名
> - `FS`：字段分隔符，默认是空格和制表符。
> - `RS`：行分隔符，用于分割每一行，默认是换行符。
> - `OFS`：输出字段的分隔符，用于打印时分隔字段，默认为空格。
> - `ORS`：输出记录的分隔符，用于打印时分隔记录，默认为换行符。
> - `OFMT`：数字输出的格式，默认为`％.6g`。



`awk`还提供了一些内置函数，方便对原始数据的处理。

函数`toupper()`用于将字符转为大写。

其他常用函数如下。

> - `tolower()`：字符转为小写。
> - `length()`：返回字符串长度。
> - `substr()`：返回子字符串。
> - `sin()`：正弦。
> - `cos()`：余弦。
> - `sqrt()`：平方根。
> - `rand()`：随机数。



`awk`允许指定输出条件，只输出符合条件的行。

输出条件要写在动作的前面。

```bash
$ awk '条件 动作' 文件名
```

正则

```bash
awk -F ':' '/usr/ {print $1}' demo.txt
```

