虽然VSCode自带的git插件对我来说已经足够好用了, 但是学习git的原理和命令行使用还是很重要的

我们先贴一张梗图: 

![alt text](figure\git_meme.png)

这充分说明了git十分好用, 以及单纯的git指令有多么难学(逃)

# 先来点底层

(小抄一手讲义)

git 将文件认为是blob, 而目录认为是tree, 维护的只有根目录

git 把数据看作是对小型文件系统的一系列snapshot

git 的每一份snapshot都需要父节点(除了init), 并用DAG(而非线性)管理所有的snapshot

用伪代码就有

```
// 文件认为是blob
type blob = array<byte>

// 目录认为是tree, 注意目录文件提供的是字符串和文件之间的映射
type tree = map<string, tree | blob>

// commit 由DAG上的父节点, 当前节点的快照和对应的元数据组成
type commit = struct {
    array<commit> parents
    tree snapshop
    string author,message //metadata
}

//将上面三种结构认为是object
type object = blob | tree | commit
```

git在存储时采用的是哈希方法: 
```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

在实际存储中, 除了对象库(objects)和一些元数据外, 其余部分主要使用指针和引用. 

SHA-1生成的ID显然没有可读性, 因此git还需要一组映射来提供可读性. 例如, `HEAD`、分支名和标签名都可以作为可读的引用: 

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_with_reference(name_or_id):
    if name_or_id in references:
        return load(read_reference[name_or_id])
    else:
        return load(name_or_id)
```

这样通过可读的记录/不太可读的id均可以引用到我们想要的东西

所有的git命令概括起来都是对objects/references进行操作

# 我们看到的git

git分三个区: 

![alt text](figure\git_area.png)

- Working tree: 你真正能见到的那一部分
- index: 暂存区保存了你想要版本控制的文件修改记录, 会在提交时一并记录到仓库
- repo: 整个东西, 保存了所有的commit

# 一些概念误区

- 分支名(Branch): 分支名并不是给树状结构起名(前面提到, git使用DAG而非树状结构). 分支名实际上是某个快照的动态引用(reference). 
  
  与标签(tag)不同的是, 分支名的引用是动态的(指向最新的提交), 而标签是静态的(固定指向某个提交), commit const* tag;

- git checkout如果不指定文件的话实质上为修改了HEAD对应的引用

- 想创建新分支实质上需要先`git branch <branch_name>`,  但是`git checkout -b <branch_name>`取代了这一点

- 先`git add <file>`跟踪文件然后`git commit`才会有snapshot哦, 即使处理了merge冲突也是如此, 等效的方法是用`git commit -a`直接取代先add再commit

- git不存在回档这一概念, 因为难以从git已经记录的东西中删除记录, 类似的指令通过创造一个抵消某个commit的commit/移动HEAD实现

# git cheat sheet

## 基础  
- `git init`: 创建一个新的 Git 仓库，其数据会存放在一个名为 `.git` 的目录下  
- `git status`: 显示当前的仓库状态  
- `git add <filename>`: 添加文件到暂存区 
- `git commit [-a]`: (添加所有修改后的文件到暂存区)并创建一个新的提交 
- `git log [--all --graph --decorate]`: 显示历史记录(并以DAG进行可视化) 
- `git diff [<revision>(默认值为HEAD指向的版本)] <filename>`: 显示某个文件两个版本之间的差异  
- `git checkout [-b] <revision>`: (创建一个新分支并)更新 HEAD 和当前的分支  

## 分支和合并
- `git branch`: 显示当前分支    
- `git merge <revision>`: 将revision合并到当前分支  
- `git rebase`: 合并, 但通过直接复制commit而非指向两个commit来保证线性

## 远端操作
`git remote`: 列出远端  
`git remote add <name> <url>`: 添加一个远端  
`git push <remote> <local branch>:<remote branch>`: 将对象传送至远端并更新远端引用  
`git branch --set-upstream-to=<remote>/<remote branch>`: 创建本地和远端分支的关联关系  
`git fetch`: 从远端获取对象/索引  
`git pull`: 相当于 `git fetch` + `git merge`  
`git clone`: 从远端下载仓库  

## 撤销
- `git commit --amend`: 编辑提交的内容或信息  
- `git reset HEAD <file>`: 恢复暂存的文件  
- `git checkout -- <file>`: 丢弃工作区的修改  
  
## Git 高级操作
- `git config`: Git 是一个高度可定制的工具  
- `git clone --depth=1`: 浅克隆（shallow clone），不包括完整的版本历史信息  
- `git bisect`: 通过二分查找搜索历史记录  
- `.gitignore`: 指定故意不追踪的文件