先摘点金句: 

code does not do what you expect it to do, but what you tell it to do

“The most effective debugging tool is still careful thought, coupled with judiciously placed print statements” — Brian Kernighan, Unix for Beginners.

# 怎么调试呢？

## 快去请printf老祖
如上面金句而言, printf仍然是很好用的方法, 此事在我的算法coding中亦有记载

## 使用logs
日志是一种有效的调试方法, 具有以下优势: 
- 可以写入文件, 从而使用文件的相关接口
- 支持严重等级(例如 INFO, DEBUG, WARN, ERROR 等), 结合命令行操作可以直接过滤
- 即使还有别的新bug也能从日志里直接得出

目前大多数应用程序会将日志写入同一个系统日志. 可以使用 `logger <cmd>` 将输出重定向至日志. 

## pdb or gdb
前两种都不好用的话建议使用调试器(惨遭跳过)

# 从多种方面分析一段代码
讲讲怎么从代码本身/系统资源等方面找到你的代码的运行情况

## 静态分析
在不运行的情况下基于编码规则直接分析, 一般来说已经集成在IDE中

## 性能分析

最简单的方法当然是使用 `import time` 测量程序的总运行时间, 但这种方法得到的是从开始到结束的总时间, 中间可能夹杂其他进程的执行时间、阻塞消耗的时间、内核态执行的时间等. 

为此你真正需要的指令是: `time [-o[a] file] <cmd>`, 其中参数`-o`会设定输出文件, `-a`会将文件写入模式改为append而非write

公式化输出: 
```shell
real    0m2.561s #实际时间
user    0m0.015s #CPU执行用户态代码的时间
sys     0m0.012s #CPU执行内核态代码的时间
```

当然, time只能看到总的时间, 如果想深入程序内部观察的话更好的东西是性能分析工具(profilers)

profiler主要分为两种: 
-  追踪分析器(tracing profiler): 将分析器的代码和源代码一起运行从而得到结果, 代价是开销相对更高
-  采样分析器(sampling profiler): 每隔定长时间对程序状态进行一次采样, 并得到结果

例如用CPU的追踪分析器cProfile来分析一段代码: 
```shell
python -m cProfile -s tottime <program>

> ...

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
   8000    0.266    0.000    0.292    0.000 {built-in method io.open}
   8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
  17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
  ...
```
这样就看到了程序哪些地方调用次数和总时间比较多

但针对python这种解释性语言而言, 使用line profiler这种基于行的分析器是更好的

当然, 分析器不仅可以只针对CPU, 也可以针对内存: 
```shell
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

## 外部事件分析

有时候我们想关注程序运行时的一些外部事件, 比如占用的CPU周期数/缓存相关内容, 可以使用`perf`: 

- `perf record <cmd>` 记录命令执行的采样信息并将统计数据储存在 `perf.data` 中
- `perf report` 格式化并打印 `perf.data` 中的数据

## 但我想要可视化

以上的这些东西固然都很好, 但是最大的痛点都是基于命令行, 而flamegraph或者pycallgraph可以用来做一些可视化工作进行辅助

# 系统资源分析

有时候, 程序变慢通常是因为它所需要的资源不够了, 因此需要一些能显示系统整体资源状况的指令

## htop

功能很多的一个指令, 直接上图: 
![alt text](figure\htop.png)

能够显示每个CPU的占用情况, 内存占用, SWAP占用, 进程的PID,时间, CPU和内存占用, 进程结构等等信息, 下方的F1-F10快捷键集成了常用功能

## du

用于查看磁盘占用

使用方法: 
```shell
du [OPTIONS][-L <符号连接>][--exclude=<目录或文件>][--max-depth=<目录层数>][目录或文件]
```

| 常用参数 | 说明 |
| --- | --- |
| `-c` | 除了显示个别目录或文件的大小外, 同时也显示所有目录或文件的总和 |
| `-h` | 同`ls -h`, 提高可读性 |
| `-l` | 重复计算硬件连接的文件 |
| `-s` | 仅显示指定目录或文件的总大小, 而不显示其子目录的大小 |
| `-D` | 显示指定符号连接的源文件大小 |
| `-L <符号连接>` | 显示选项中所指定符号连接的源文件大小 |
| `--exclude=<目录或文件>` | 略过指定的目录或文件 |
| `--max-depth=<目录层数>` | 超过指定层数的目录后, 予以忽略 |

最常用的`du -sh`就是显示整个目录文件的每一项在磁盘上的占用

# MISC

`sudo strace <cmd> >/dev/null` 可以查看指令本身的系统调用, 而将指令本身的输出去除

网页截图不可信, 因为浏览器支持直接修改内容

`sudo lsof`可以列出被进程打开的文件信息, 但由于信息量非常大建议结合`grep`一起使用
