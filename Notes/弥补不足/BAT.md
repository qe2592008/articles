1. 命令
- @
> 处于行首，表示当前行其后面的命令本身不显示
- echo
> 1. 一个开关，echo on表示显示其后面所有的命令本身，echo off表示不显示其后面的命令本身
> 2. 显示后面的内容
> 3. 编辑文件
- ::
> 注释命令
- pause
> 暂停命令
- :和goto
> 标签与跳转标签命令
- %
> 参数标签，用于参数替换
- if
> 1. 条件判断命令：if [not] "%1"=="" goto end
> 2. 存在判断命令：if [not] exist C:\Progra~1\Tencent\AD.gif del C:\Progra~1\Tencent\AD.gif
> 3. 结果判断命令：if [not] errorlevel 1 link %1.obj
- call
> 调用命令

