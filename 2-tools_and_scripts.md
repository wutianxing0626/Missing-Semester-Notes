# 依旧是bash的cheating paper

字符串机制: 双引号字符串会替换变量, 但是单引号一定不会转义

`source file.sh` 运行脚本文件

bash语言有短路机制

`;`用于多条命令在一行内的分割

关于脚本参数: 

- `$0` 脚本文件的名字
- `$1` - `$9` 可以接受9个参数
- `$_` 上个命令的最后一个参数
- `!!` 指代上一个命令, 在前面要加东西的时候很有用, 比如sudo
- `$?` 上个命令的返回值(错误代码), 可以用逻辑运算符结合其他指令
- `$@` 所有参数
- `$#` 参数个数
- `$$` PID


变量替换机制: 即在`$(<cmd>)`会将这个变量替换为cmd的输出

进程替换机制: 即在`<(<cmd>)`会执行cmd并将结果输出到一个临时**文件**中, 并将本身替换成临时文件名, 可以用在所有需要文件名字作为参数的地方

# 事已至此, 先举例吧
这是一个简单的搜索程序: 

```shell
#!/bin/bash

echo "Starting program at $(date)"

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

一些解析: 
- 参数需要接受文件名
- `for <var> in "$@"`:  一个for循环, 类似于python中 `for iter in iterator`
- `> /dev/null 2> /dev/null`:  由于不关心输出, 将输出流和错误流均定向到黑洞文件
- `if [[ $? -ne 0 ]]: ` 利用浅显的汇编知识可以得出等效于返回值不为0, 即发生错误, 注意shell语言请尽可能使用双中括号以避免一些可能的错误, 虽然不能完全避免()
- 一些if语句和for循环的公式化使用
  
# 通配二周目

实质上是一种简化的正则表达式

* 表示匹配任意字符, ? 表示匹配一个字符, [...] 表示匹配指定范围内的字符, **表示匹配任意中间目录

例如: 
```shell
ls *.txt         # 列出所有扩展名为.txt的文件
ls file?.txt     # 列出文件名为file?.txt的文件, 其中?表示任意一个字符
ls [abc]*.txt    # 列出以a, b或c开头, 扩展名为.txt的文件
```

在指令中增加{a,b,c,...} 用于有公共子串的指令展开, 例如
```shell
cp /path/to/project/{foo,bar,baz}.sh /newpath
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath
```
这一指令支持集合的笛卡尔积(bushi)

{a..h}等价于{a,b,c,...,h}

# 一些奇妙小指令

`diff <file1> <file2>`: 比较两个东西下面的内容有何不同, 公式化输出为: 
```shell
< <只在file1有的内容>
---
> <只在file2有的内容>
```

`find <路径> <匹配条件> [<动作>]`: 在路径下寻找所有符合匹配条件的文件, 并执行一些动作

其中路径和匹配条件均支持多个, 用逗号分割即可

一些常用的匹配条件有: 
| 常用条件 | 作用 |
| --- | --- |
| -name <pattern> | 按文件名查找, 支持使用通配符 |
| -type <type> | 按文件类型查找, 同ls -l内的文件类型 |
| -size <[+-]size[cwbkMG]> | 按文件大小查找, 支持使用 + 或 - 表示大于或小于指定大小, 单位可以是 c(字节), w(字数), b(块数), k(KB), M(MB)或 G(GB) |
| -user/group <name> | 按文件的user或者group查找 |
| -perm <数字权限> | 按chmod数字模式表示的权限查找 |

动作最常用的是-exec,  支持在这个指令后加入想要对结果执行的指令, 但数据不能太多, 使用管道以及 xargs (使用输入流中的内容作为参数) 可以避免这个问题

例如: 如何找到主目录下所有cpp文件并列出详细信息: 
```shell
find ~ -type "*.cpp" -exec ls -l {}\; #{}表示参数, 而\;表示命令结束
find ~ -type "*.cpp" | xargs ls -l
```

