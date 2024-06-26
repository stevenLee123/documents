# linux
操作系统时计算机硬件和用户之间的桥梁

## linux的启动流程
docker启动与kvm启动的区别是啥
docker启动需要宿主机的内核
kvm启动
1. bios启动： 基本输入输出系统，写入主板的固件
2. MBR（master boot record）加载 硬盘的主引导记录，磁盘驱动器最开始部分的一个特殊的启动扇区
3. grub启动引导阶段：执行bootloader主程序（stage1）、调用core.img(stage 1.5)、加载grub（stage 2）
4. initrd/initramfs协助内核启动
5. rootfs 

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
### ssh命令
* 构建ssh密钥对
ssh-keygen -t rsa
* 查看主机是否添加密钥对
ssh-kengen -F 192.168.1.101
* 删除主机密钥对
ssh-kengen -R 192.168.1.101
* 通过用户名和主机直接登陆
ssh  user@hostname   
ssh root@192.168.1.101
* 指定端口登陆
ssh -p 8080 root@192.168.1.101
* 绑定源地址(指定客户端用哪个IP链接到SSH)
ssh -b 192.168.1.102 root@192.168.1.101
* 使用ssh在远程主机上执行一条命令
ssh root@192.168.1.101 ls -l
* 在远程主机上运行一个图形界面程序
ssh -X root@192.168.1.101



## vim/vi

三种模式：
> 正常模式，刚用vi打开文件，进入正常模式
  快捷键：
  *  yy 拷贝当前行
  *  p 粘贴拷贝内容
  *  4yy 拷贝当前行（包含当前行）往下的4行
  *  dd删除当前行
  *  4dd 删除当前行（包含当前行）往下的4行
  *  shift +g 定位到最后一行
  *  gg定位到第一行
  *  u 撤销，撤销上一次操作
  *  定位到20行，20+ shift+g，定位到第20行
  *  0 0键跳转到行首
  *  $ $键跳转到行尾
  

> 插入模式：按i（insert），o, O,I,A,R,r 都可以进入输入模式

> 命令行模式：esc，再按shift+: 进入命令行模式，输入wq（保存并退出），输入q（退出）
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
sync 将内存数据同步到磁盘（在上面的关机命令中实际上已经先执行了sync，然后再执行关机）

## 登陆注销
su - root 切换到root用户
logout 注销用户（图形界面（运行级别3）注销无效）
exit 退出终端或注销用户

## 用户管理
useradd abc 添加用户
添加用户abc默认会在/home下创建abc目录作为用户的工作目录

useradd -d /home/test abc 指定/home/test为新用户abc的工作目录

passwd abc ABC123 修改用户abc的密码

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


/etc/passwd 文件：用户的配置文件，记录用户的各种信息,记录用户的信息，含义 ： 用户名：口令：用户标识号：组标识号：注释性描述：主目录：登陆shell
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

### 找回root密码
使用单用户模式修改密码

## 帮助指令
man 获取帮助信息
man ls
help cd

## alias  在shell会话中定义临时别名
alias ll = " ls -l"
移除别名
unalias ls
# 文件目录指令
## cd
cd .. 返回上一级
cd - 返回上一个目录

## cp 命令 复制命令
cp -r dir1/ dir2/ 将dir1目录下的文件递归复制到dir2中

## rm 删除命令
rm file.txt 删除文件
rm -r dir 递归删除目录下的文件
rm -rf dir2 强制递归删除目录下的文件

## mv 在文件系统中移动文件和目录
mv source_file des_folder/ 移动文件到des_folder目录下

## mkdir 创建文件夹
mkdir images/
mkdir -p abc/efg 创建多级目录
## rmdir 删除目录
rmdir images 删除空目录
rm -rf abc/efg 强制删除整个目录

## touch： 创建空文件或更新文件的时间戳
touch example.txt 文件不存在，则创建文件，如果文件存在，则更新文件的访问时间和修改时间
touch -t 202309251200 example.txt 创建文件时指定特定的时间戳
touch -m old_file
## pwd
显示当前所在目录

## cp 拷贝文件命令
cp helloworld abc/ 拷贝文件到abc目录下
cp -r /home/bbb/ /opt/ 拷贝文件夹
\cp -r /home/bbb/ /opt/ 强制覆盖/opt/下的重复文件

## rm 删除文件
rm helloworld 删除
rm -f helloworld 强制删除
rm -rf helloworld 强制递归删除

## mv 移动文件、目录或重命名
mv helloworld helloworld1 重命名
mv helloworld abc/helloworld1 移动文件到abc目录下，并对文件重命名

## cat 查看文件
cat helloworld 查看文件
cat -n helloworld 查看文件并显示行号
cat -n /etc/profile|more 使用管道命令符号，将第一个命令的结果返回给后面的指令

## more 分屏显示命令

## less 分屏查看文件内容，允许文件查找
less helloworld.txt
可以在末尾通过？或/ 进行查找
/abc 查找abc

## echo 输出内容到控制台

## tail 输出文件尾部内容
tail -n 200 -f tomcat.txt 实时监控日志信息

## head 显示文件头几行
head -n 200 tomcat.txt 显示文件头200行

## > 输出重定向
echo 'helloworld' > helloword.txt 
## >>  追加
echo 'helloworld' > helloword.txt



## ln 软连接，符号链接（将一个文件指向另一个文件）
ln -s /root /home/myroot

## history 显示命令执行记录
history 10显示最近执行的10条命令

# 时间日期指令
## date 显示当前时间
date %Y 显示年
date '+%Y-%m-%d' 显示年月日 2023-12-05
date -s '2023-12-05 00:00:00' 设置当前时间

## cal 日历命令

# 查找指令
## find 查找命令
find /home -name helloword.txt 按文件名查找
find /home/lijie3 -user lijie3  按用户名查找文件
find / -size +200M/G/k  按文件大小查找,查找大于200M的文件

## locate 快速定位文件路径
updatedb 更新文件数据库
locate helloworld.txt 在updatedb之后执行

## grep 过滤查找，与管道符号｜一起使用
cat helloworld.txt|grep helloworld  查找helloworld字符
cat helloworld.txt|grep -n helloworld 查找helloworld并显示行号
cat helloworld.txt|grep -i helloworld 忽略大小写查找helloworld
grep -n 'helloworld' helloworld.txt 查找helloworld字符


# 用户管理
一个文件、一个目录有所有者和所属组
## chown 修改文件所有者
chown abc helloworld.txt 修改文件所有者为abc
## chgrp 修改文件所在组
chgrp dxy helloworld.txt 修改文件所属组为dxy
## groupadd 创建组
groupadd dxy 创建组dxy
useradd -g dxy lijie 创建用户并添加到组
## usermod 修改用户所在组
usermod -g dxy lijie 修改lijie所在组为dxy
usermod -d myroot lijie 修改用户的工作目录为myroot

## chmod 修改权限
chmod u=rwx,g=rx,o=x nginx 给所有者、组、其他组分别赋予权限
chmod o+r helloworld 给其他人赋予r的权限 
chmod o-w helloworld 给其他人移除w权限
chmod 751 helloworld 给所有者读写执行、组读执行、其他用户执行权限

## chown 修改所有者
chown lijie3 /home/test 修改文件所有者权限
chown -R lijie3 /home/folder 递归修改文件夹所有者权限

## chgrp 修改文件所在组
chgrp goup1 /home/test 修改所有组为group1

# 任务调度
## crond定时任务调度
crontab -e 开启定时任务的编辑
*/1 * * * * ls -l /etc/ > /tmp/to.txt 执行调度任务的信息配置，每分钟执行一次

## at定时任务，执行一次

# 磁盘分区与挂载
磁盘以文件的方式挂载在目录下
linux 硬盘分为IDE和SCSI硬盘，目前基本上是SCSI硬盘
ide硬盘标识为hdx～ ，sda 表示一块硬盘、sda1，sda2 标识磁盘的两个分区
scsi硬盘的标识为sdx～

## lsblk -f查看磁盘的分区情况
## 磁盘分区并挂载
fdisk /dev/sdb
再输入n
选择主分区（p）或者扩展分区(e)
选分区号1
输入w保存并退出
mkfs -t ext4 /dev/sdb1 对sdb1分区进行格式化
使用lsblk -f 查看磁盘分区情况及磁盘挂载情况

mkdir newdisk 创建文件夹
mount /dev/sdb1 /newdisk 挂载分区到目录

## 卸载磁盘
umount /dev/sdb1 卸载磁盘文件
或
umount /newdisk 卸载挂载目录

**使用命令行挂载的方式重启系统后挂载会失效**

## 永久挂载
修改/etc/fstab文件添加要挂载的磁盘分区及挂载点

## 查询磁盘情况
df -h 查看磁盘使用情况，挂载点
du -ha /newdisk --max-depth=1 查询指定路径的磁盘占用情况

## 统计目录下文件个数
ls -l .|grep "^-" |wc -l
## 统计目录及子目录下的文件个数
ls -lR .|grep "^-" |wc -l
## 树状显示目录结构
tree .

## 设置host映射
hostname 返回主机名称
修改/etc/hosts文件，添加ip与主机名的映射关系

# 进程管理
## 查看进程信息
ps -aux|more

## 以全格式显示当前所有进程
ps -ef   -f 全格式   -e 显示所有进程

ps -aux|grep sshd 查看通过sshd登陆到当前主机的用户进程信息

## pstree 
树状结构显示

## service 管理
服务的本质是进程，运行在后台的服务通常都会监听某个端口，等待其他程序的请求
在centos7之后service不在使用（service命令管理的服务在/etc/init.d下），而是使用systemctl命令
service start｜stop｜restart｜reload｜status mysqld

## chkconfig 给服务的各个运行级别设置自启动/关闭
chkconfig命令管理的服务在/etc/init.d下
chkconfig --list 查看课管理的服务列表
chkconfig --level 5 network off
chkconfig --level 3 network on

## systemctl管理命令
systemctl [start|stop|restart|status] 服务名
systemctl指令管理的服务在/usr/lib/systemd/system下
systemctl enable mysqld 设置mysqld 自启动
systemctl disable mysqld 设置mysqld 非自启动
systemctl is-enable mysqld 查看服务是否自启动
systemctl list-unit-files|grep 查看服务开机自启动状态

netstat -aut


## kill 命令
kill [-s q] pid 指定要发送的信息
kill -l pid 列出所有可用的信息名称
kill pid 杀死进程
kill -KILL pid 强制杀死进程
kill -HUP pid 发送sigHUB 信号
kill -9 pid 彻底杀死进程

killall  杀死进程及子进程

# 压缩解压命令
gzip helloworld.txt
gunzip helloworld.txt.gz 解压

zip -r test.zip . 递归压缩当前目录
unzip -d /test test.zip 解压到test目录下

tar 
-c 产生.tar打包文件
-v 显示详细信息
-f 指定压缩后文件名
-z 打包同时压缩
-x 解压.tar文件

tar -zcvf test.tar.gz helloworld.txt 将helloworld.txt压缩成test.tar.gz
tar -zxvf test.tar.gz 解压文件
tar -zxvf test.tar.gz -C /ouput 解压文件，并制定解压目录

## 文档编辑命令

## 文件传输命令

## 磁盘管理命令

## 磁盘维护命令

## 网络通信命令

### centos8下的防火墙设置
* 查看防火墙状态
systemctl status firewalld
firewall-cmd --state
* 查看已开放端口
firewall-cmd --list-ports
* 开启防火墙
systemctl start firewalld
* 开放端口
firewall-cmd --zone=public --add-port=8800/tcp --permanent
命令含义：
--zone #作用域
--add-port=8800/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效
* 重启防火墙
firewall-cmd --reload

## rpm （redhat package manager）
rpm -qa |grep xxx 查询当前系统是否安装了某个软件
rpm -q firefox 查看某个软件是否安装
rpm -qi firefox 查看安装的软件包的信息
rpm -ql firefox 查询软件包中的文件
rpm -qf /etc/passwd 查询文件归属于哪个软件包

rpm -e firefox 删除软件包（erase 擦除）
rpm -e --nodeps firefox 强制删除，忽略依赖警告信息， no depedencies

rpm -ivh firefox.rpm 安装某个软件 
-i install
-v verbose 提示
-h hash 进度条

## yum shell前端软件包管理器，基于rpm包管理
yum list|grep xxx 查询yum服务器是否需有需要安装的软件
yum install firefox 安装某个软件

## source /etc/profile 让新的环境变量生效


## 系统管理命令

## 系统设置命令

## 备份压缩命令

## 设备管理命令





## 查看端口占用情况lsof(list open files) 列出当前系统打开文件的工具
list -i:2181

COMMAND   PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java    60366 lijie3   75u  IPv6 0x7c72a1d3184302ef      0t0  TCP *:eforward (LISTEN)

## 查看端口占用情况
netstat -tunlp|grep 2181



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

## free 查看系统内存使用情况
              total        used        free      shared  buff/cache   available
Mem:         792772      265232       82672       46828      444868      339988
Swap:             0           0           0
total 总物理内存大小
used 已使用多少
free 可用内存多少
shared 多个进程共享的内存总额
buffers/cached 磁盘缓存大小
avaliable 估算的可用于启动新应用程序而不必使用交换空间的内存量

free -b/--bytes 以字节为单位显示内存使用情况
free -k/--kilo 以KB显示（默认情况）
free -m/--mega 以MB显示
free -g/--giga 以GB显示
free -h/--human 以人类刻度格式显示
free -t /--total 显示总计的内存使用情况
free --si 使用1000而不是1024作为单位的转换因子