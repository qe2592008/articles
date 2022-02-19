1. 运行方式
    - cd到程序目录，执行
        ```shell
        ./test.sh
        ```
        > 其中`./`表示在当前目录找执行程序
    - 执行如下命令
        ```shell
        /bin/sh test.sh
        /bin/php test.php
        ```
2. 变量
- 定义变量
```shell
your_name="runoob.com"
```
> 注意：
>   1. 变量名和等号之间不能有空格
>   2. 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头
>   3. 中间不能有空格，可以使用下划线 _
>   4. 不能使用标点符号
>   5. 不能使用bash里的关键字（可用help命令查看保留关键字）
>   6. 已定义的变量，可以被重新定义
>   7. 使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变
>   ```shell
>   your_name="runoob.com"
>   readonly your_name
>   ```
- 使用变量
```shell
your_name="qinjx"
echo $your_name
echo ${your_name}
```
> 注意：
>   1. 使用一个定义过的变量，只要在变量名前面加美元符号即可
>   2. 变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界,推荐都加上
- 删除变量
```shell
unset variable_name
```
> 注意：
>   1. 变量被删除后不能再次使用
>   2. unset 命令不能删除只读变量
3. 字符串
- 字符串定义
```shell
str='this is a string'
str1="this is another string"
```
> 注意： 
>   1. 字符串可以用单引号，也可以用双引号，也可以不用引号。
>   2. 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的,单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用
>   3. 双引号里可以有变量,双引号里可以出现转义字符
- 字符串操作
```shell
# 操作1：提取字符串长度
string="abcd"
echo ${#string} #输出 4
# 操作2：提取子字符串
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
# 操作3：查找子字符串
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```
> 注意：
>   1. 第一个字符的索引值为 0
>   2. 以上脚本中 ` 是反引号，而不是单引号 '，不要看错了哦
