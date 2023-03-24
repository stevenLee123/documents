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