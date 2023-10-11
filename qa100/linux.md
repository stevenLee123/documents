# linux
操作系统时计算机硬件和用户之间的桥梁

## 虚拟机中的三种网络连接模式

桥接模式:虚拟系统可以和外部系统相互通信，但是容易造成IP冲突
NAT模式：网络地址转换，虚拟系统可以和外部系统通信，外部系统无法访问虚拟机，不会造成IP冲突
仅主机模式： 独立系统，不和外部发生联系

## 磁盘分区
swap分区：临时充当内存
boot引导分区
root根分区

通过vmtools 来实现宿主机和虚拟机的共享文件夹

kdump 内存崩溃转储机制

## linux 目录介绍
层级式的树状目录结构
在linux中，一切皆是文件
/boot ：启动系统用到的文件
/dev   硬件映射成对应的文件
/bin    存放经常使用的目录 /usr/bin  /usr/local/bin 
/sbin super bin 存放管理员使用的命令
/lib 开机所需要最基本的动态链接共享库
/lost+found 存放系统非法关机后的文件
/etc 系统管理所需要的配置文件，如mysql配置文件 my.conf
/usr 一般用户的应用程序和文件放在这个目录下
/proc 虚拟目录，系统内存映射回去系统信息
/srv service，存放服务启动后需要提取的数据
/sys 系统目录
/tmp 存放临时文件
/mnt 让用户临时挂载别的文件系统的，可以将外部的存储挂在在mnt下，让后进入该目录就能看到里面的内容
/media linux自动识别一些设备，如u盘，linux会把识别的设备挂载到该目录
/opt 给主机额外安装软件所存放的目录
/usr/local 软件安装的目标目录
/var  存放日志等不断扩充的文件
/selinux 安全子系统



## 远程登录






## vim/vi

三种模式：
> 命令模式，刚用vi打开文件，进入命令模式
  快捷键：
  *  yy 拷贝当前行
  *  p 粘贴拷贝内容
  *  4yy 拷贝当前行（包含当前行）往下的4行
  *  dd删除当前行
  *  4dd 删除当前行（包含当前行）往下的4行
  *  G 定位到最后一行
  *  gg定位到第一行
  *  u 撤销，撤销上一次操作
  *  20G，定位到第20行
  

> 输入模式：按i（insert），o, O,I,A,R,r 都可以进入输入模式

> 底线模式：esc，再按shift+: 进入底线模式，输入wq（保存并退出）
  常用命令：
  * q! 强制退出，不保存
  * /abc 查找abc字符串，使用n（next）进行下一个匹配查找
  * set nu 显示行号
  * set nonu 取消显示行号

## 关机重启
关机：
shutdown -h now 立即关机 h表示halt
shutdown -h 1 一分钟后关机
shutdown -r now 立即重启
halt 关机
reboot 立即重启
sync 将内存数据同步到磁盘

## 登陆注销
su - root 切换到root用户
logout 注销用户（图形界面（运行级别3）注销无效）
exit 退出终端或注销用户

## 用户管理
useradd 添加用户

useradd -d /home/test abc 指定/home/test为新用户abc的工作目录

passwd abc 修改用户abc的密码

userdel abc 删除abc用户，不删除用户目录
userdel -r abc 删除abc用户，并删除用户目录

id abc 查询用户abc的信息
who/whoami 查看当前登陆用户

## 用户组
创建组
groupadd testgroup1
删除
groupdel testgroup1
添加一个用户并分配组
useradd -g test1 testgroup1
修改组
usermod -g test1 testgroup2


/etc/passwd 文件：用户的配置文件，记录用户的各种信息
/etc/shadow 文件：口令配置文件
/etc/group 文件： 组配置文件记录linux包含的组信息

## 指定运行级别
0 关机
1 单用户（找回丢失密码）
2 多用户状态没有网络服务
3 多用户有网络服务
4 系统未使用保留给用户的
5 图形界面
6 系统重启
常用3 和 5

init 3 进入多用户有网络服务但无图形界面
init 5 切换回图形界面
init 0 关机

systemctl get-default 查看默认运行级别
systemctl set-default multi-user.target 修改默认级别为3
systemctl set-default graphical.target 修改默认级别为5

找回root密码
使用单用户模式修改密码

## 帮助指令
man 获取帮助信息
man ls
help cd

## 文件目录指令









## 根据端口号杀进程
```shell
 kill -9 `lsof -i:6379| awk 'NR==2{print $2}'`
```
## sed替换文件内容
将/var/www/test文件夹下的所有文件内容中的abc字符串换成123
sed -i "s/abc/123/g" `grep abc -rl /var/www/test`
替换文件，将index.html中的abc替换成123
sed -i "s/abc/123/g" /var/www/test/index.html

### 查询/卸载软件
```shell
rpm -qa | grep java
rpm -e --nodeps xxxxxxx-java-xxxx.rpm
```

## 创建用户
useradd test
修改密码
passwd test

## 授权
修改文件所有者
chown -R test:root mysql/*

## scp 方便的在linux机器之间复制文件
前提条件：linux机器之间配置ssh免密登陆
scp -r test.text root@node2:/tmp/

## 检查xml文件内容是否正确
xmllint -noout conf/hbase-site.xml

## 创建文件
touch filename

## ACL(access control list) 访问控制列表
当使用 文件所属人和文件所属组来进行权限控制无法满足要求时，可以使用acl来单独指定用户，或组对文件/文件夹的访问权限控制
相关命令
添加用户user对project文件夹下的所有文件的r，x权限
`setfacl -m u:user1:rx project/`
添加对组usergroup1对文件夹下所有文件的r，w权限
`setfacl -m g:usergroup1:rwx project/`
其他参数
-m：设定 ACL 权限。如果是给予用户 ACL 权限，则使用"u:用户名：权限"格式赋予；如果是给予组 ACL 权限，则使用"g:组名：权限" 格式赋予；
-x：删除指定的 ACL 权限；
-b：删除所有的 ACL 权限；
-d：设定默认 ACL 权限。只对目录生效，指目录中新建立的文件拥有此默认权限；
-k：删除默认 ACL 权限；
-R：递归设定 ACL 权限。指设定的 ACL 权限会对目录下的所有子文件生效；
获取project文件夹的acl权限控制信息
`getfacl project`

## 创建软连接
`ln -s spark-3.3.2-bin-hadoop3/ test-spark`
创建软连接后，软链接可以当作文件或文件夹使用

## linux的ugo权限模型
u: user
g: usergroup
o: other
针对每一种用户，都有读r、写w、执行x三种权限
通过ls -l 命令查看某个文件的信息能得到文件所属人，文件所属组，以及other用户对文件的访问权限
```shell
ls -l 
-rw-r--r--  1 lijie3  staff  0  4 12 11:37 test.txt
lrwxr-xr-x  1 lijie3  staff        8  4 12 11:39 s-test -> test.txt
```
-rw-r--r--说明
第一位表示：目录（d）、文件（-）软连接（l）
第2-4位：user（文件所属人的权限）： rwx 
第5-7位：group（文件所属组的权限）rwx
第8-10位：other（其他人的权限）

常用命令
* 变更文件所属人 -R 递归变更
`chown root -R archive`  
* 修改文件/目录所属组
`chgrp testgroup -R archive`  

修改权限
```shell
chmod u+rw test.txt #给所属用户权限位添加读写权限
chmod g+rw test.txt #给所属组权限位添加读写权限
chmod o+rw test.txt #给其他用户权限位添加读写权限
chmod u=rw test.txt #设置所属用户权限位的权限位读写
chmod a-x test.txt #所有权限为去掉执行权限
chmod 755 test.txt
```
## 向tcp端口号9999发送数据
nc -lk 9999

## 查询程序路径
`which java`
/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home/bin/java