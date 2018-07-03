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

