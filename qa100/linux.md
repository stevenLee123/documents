# linux

## vim/vi
三种模式：
> 命令模式，刚用vi打开文件，进入命令模式
> 输入模式：按i（insert）进入输入模式
> 底线模式：esc，再按shift+: 进入底线模式

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