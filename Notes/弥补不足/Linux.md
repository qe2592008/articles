## 1. Linux常用命令
### 1.1 查看命令文档
- --help
    ```linux
    ls --help
    ```
- man 
    ```linux
    man ls
    ```
### 1.2 history 查看历史命令
```linux
history
```
### 1.3 查看当前磁盘使用情况
```linux
df -h
```
### 1.4 查看当前系统内存使用情况
```linux
free -h
```
### 1.5 执行shell脚本
- 使用sh命令执行
    ```linux
    sh start.sh
    ```
- 通过`./`执行
    ```linux
    ./start.sh
    ```
### 1.6 打包解压命令

#### 1.6.1 zip压缩
- zip打包
    ```linux
    zip test.zip fileZip
    ```
    > 将文件fileZip打包成一个名为test.zip的压缩包
- zip解压
    ```linux
    unzip test.zip
    ```
    > 解压test.zip压缩包
#### 1.6.2 tar压缩
- tar打包
    ```linux
    tar -czvf png.tar.gz test.png
    ```
    > 将当前目录下的test.png文件打包成一个名为png.tar.gz的包
- tar解压
    ```linux
    tar -xzvf png.tar.gz
    ```
    > 解压png.tar.gz文件

### 1.7 查看文件信息
```linux
ls      列出当前目录的内容 
ls -l   列表形式显示文件详细信息
ls -a   显示所有文件，包含隐藏文件
ls -h   友好显示文件大小，配合-l使用
```
### 1.8 分屏显示
```linux
more <file>
```
### 1.9 管道命令 |
管道左侧的结果作为管道右侧的输入
```linux
ps -ef | grep java
```
左侧查询所有线程，右侧从其中查找包含java的线程
### 1.10 切换目录命令
```linux
cd ~    切换到当前用户主目录
cd .    切换到当前目录
cd ..   切换到上级目录
cd -    切换到上次所在目录
```
### 1.11 显示当前路径
```linux
pwd
```
### 1.12 创建目录
```linux
mkdir <dirName>
```
> 只有当前用户对当前目录拥有写权限，才能创建目录
### 1.13 删除文件
```linux
rm <file>   删除某个文件    
rm -i       删除目录或文件前需要询问用户是否删除
rm -r       执行递归删除
rm -v       显示指令详细执行过程
rm -f       强制删除文件或目录
```
### 1.14 删除目录
```linux
rmdir <dirName>
```
> 只有该目录为空目录，才能使用rmdir命令删除
> 只有离开待删除目录，才能进行删除

### 1.15 文件搜索命令
```linux
find / -name test.log   从根目录开始搜索名称未test.log的文件
```
## 2. Linux文件权限
```txt
r   可读权限（4）
w   可写权限（2）
x   可执行权限（1）
-   无对应权限
```
权限组
- 第一组是文件所有者的权限
- 第二组是文件所有组的权限
- 第三组是其他人的权限

```linux
chmod 754 test.txt
```
> 7=4+2+1，表示可读，可写，可执行
> 5=4+1，表示可读，可执行，不可写
> 4，表示可读权限，不可写，不可执行