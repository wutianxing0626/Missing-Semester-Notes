# Cheat sheet 三周目
`nohup + <cmd>` 强制前台运行

`<cmd> &` 强制后台运行, 但输入流和错误流不重定向的话还是会输出到终端

`ssh`后可以直接跟命令, 此时会返回命令结果

# 什么是信号?

shell 通过unix提供的信号(等效于软件中断)实现IPC, 可以在`man signal`中查看

`^C` 等效于向进程发送一个`SIGINT`, 而`^\`等效于`SIGQUIT`, 都可以用来退出

`^Z` 对应`SIGSTOP`, 会暂停进程

`SIGKILL`是强行终止程序(对内核态/僵尸进程不起作用), 不可以被捕获, 否则会有僵尸进程之类的东西

python 的signal库可以处理一部分能够捕获的信号, 例如: 

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

signal.signal初始化中利用handler函数中表明了如何处理`SIGINT`

# kill: 杀了吗? 如杀
虽然写作kill, 但实际上是向进程发送信号

使用方法: 
```shell
kill [-<signal>] <PID>
```
参数signal需要传入信号描述(信号名称去除"SIG"), 或者这个信号对应的值, 用于指定要给进程传入什么信号, 默认值为SIGTERM(15), 即正常停止

一些常用的信号有: 

- SIGKILL(9): 立即结束进程, 不能被捕获或忽略
- SIGTERM(15): 正常结束进程, 可以被捕获或忽略
- SIGSTOP(19): 暂停进程, 不能被捕获、忽略或结束
- SIGCONT(18): 继续执行被暂停的进程
- SIGINT(2): 通常是`^C`产生的信号, 可以被进程捕获或忽略
- SIGHUP(1): 挂起进程

利用`jobs`命令可以列出当前终端会话中尚未完成的全部任务, 可以用`jobs`中进程的代号替换PID, 例如`kill -9 %1`即为杀死第一个进程

# tmux
小抄一手讲义快速复习一下: 

会话相关: 
- `tmux new [-s NAME]` (以指定名称)开始一个新的会话
- `tmux ls` 列出当前所有会话
- `tmux a[ttach] [-t NAME]` 重新连接最后一个/指定名称的会话
  
窗口相关: 
- `<C-b> c` 创建一个新的窗口, 使用 `<C-d>` 关闭
- `<C-b> N` 跳转到第 `N` 个窗口, 注意每个窗口都是有编号的
- `<C-b> p` 切换到前一个窗口
- `<C-b> n` 切换到下一个窗口
- `<C-b> ,` 重命名当前窗口
- `<C-b> w` 列出当前所有窗口

分屏相关: 
- `<C-b> "` 水平分割
- `<C-b> %` 垂直分割
- `<C-b> <方向>` 切换到指定方向的面板, `<方向>` 指的是键盘上的方向键
- `<C-b> <空格>` 在不同的面板排布间切换

# 命令别名
有时候你会嫌一个指令太长/手滑容易打错等等问题, 别名可以很轻松的解决这一点

使用方式: 
```shell
alias <new_cmd>="<old_cmd>"
```
以后打new_cmd就可以了

我们再次小抄一手讲义展示一下用法: 
```shell
# 创建常用命令的缩写
alias ll="ls -lh"

# 能够少输入很多
alias gs="git status"

# 手误打错命令也没关系
alias sl=ls

# 重新定义一些命令行的默认行为
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed

# 别名可以组合使用
alias la="ls -A"
alias lla="la -l"

# 获取别名的定义
alias ll
# 会打印 ll='ls -lh'
```

和变量一样, 单纯的别名设置无法被保存, 除非写入配置文件中

# 配置文件相关

命令行中当然没有GUI的设置给你使用, 取而代之的是很多程序的配置都是通过纯文本格式的被称作`dotfile`的配置文件来完成的, 这类文件一般文件名以.开头, `ls -A`无法列出

一个好的方法是把各个配置文件总结在一起(然后利用`git`等版本控制软件管理), 然后打一个软连接过去

