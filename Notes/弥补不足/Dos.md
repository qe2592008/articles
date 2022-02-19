1. 常用命令
- dir   罗列文件夹与文件
```dos
dir         #罗列当前目录的所有目录与文件列表
dir /p      #分页展示，回车键翻页 
dir /s      #罗列当前目录下所有文件与子目录及子目录文件，循环深入
dir /a      #查看包括隐含文件的所有文件 
dir /ah     #只显示出隐含文件
dir /w      #以紧凑方式（一行显示5个文件）显示文件和文件夹
```
- cd    切换目录
```dos
cd \        #跳转根目录
cd ..       #返回上级目录
cd java     #进入当前目录下的java目录
```
- md    创建文件夹
```dos
md java     #在当前目录下创建java文件夹
```
- rd    删除文件夹
```dos
rd java     #删除当前目录下的java文件夹，注意改文件必须是空文件夹才能被删除
```
- del   删除文件
```dos
del file    #删除当前目录下的file文件
del *.*     #删除当前目录下的所有文件
```
- move  移动文件
```dos
move 路径\文件名 路径\文件名        将前面目录下的文件移动到另一个地方
```
- copy  复制文件
```dos
copy 路径\文件名 路径\文件名        执行文件拷贝
```
- cls   清屏
```dos
cls         #清屏
```
- ren   更名
```dos
ren 旧文件名 新文件名   #修改文件名称
```
- edit  编辑文件
```dos
edit file       #编辑文件file
```
- ping  主要用来诊断网络连接
- ipconfig  显示TCP/IP的有关配置
- netstat   监控TCP/IP网络
- net       网络命令
- echo 创建文件