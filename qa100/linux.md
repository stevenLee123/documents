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


