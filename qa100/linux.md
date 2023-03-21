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