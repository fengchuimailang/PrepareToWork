# 这才是真正的GIT —— GIT原理及其实用技巧

## 学习时间 2019.12.10

## 问题
- 命令容易混淆
- git文档晦涩难懂（如果不懂内部原理，会变得复杂）

## 小调查
- 知不知道Git Object blob tree commit 等
- 知道git reset --soft git reset -- hard
- 听说过 git reflog

## 第一个要点：git是怎样存储信息的
- 执行git init
- 添加两个文件 a.txt b.txt
-  在 .git/object 出现两个文件
-  看不见文件内容 因为做了二进制压缩
-  可以实用 git cat-file -t
-  存的是增加文件的具体内容
-  git object 是存储git类型的最小单元 
-  git obeject分类
    + blob object 存储的是文件的具体内容
    + tree obejct 目录结构、文件权限、文件名
    + commit object 上一个commit、 对应快照、作者、提交信息


- blob 会实用 SHA1哈希算法
- git object 是 键值对
> git cat-file -p 4caaa1
> 100644 blob 58c9..... a.txt
> 权限   类型 内容SHA1 文件名
- 结构如下
> 一个commit object 指向一个tree object
> 一个tree  object 指向两个blob object
- 分支tag储存在哪里呢
> cat .git/HEAD
> ref: refs/heads/master
> cat .git/refs/heads/master
- HEAD、分支、普通的Tag可以简单的理解成是一指针，指向对应的commit的SHA1值
-  git object 一旦创建了就不允许变更
-  refs 是一个指针，可以改变
-  问题： 为什么文件的权限和文件名 要存在Tree object 而不是Blob object呢？
> 回答：因为不用重新创建文件


## GIT三个分区以及变更历史的形成
- 三个分区
    + 工作分区 操作系统上的文件，所有的代码开发编辑够在这上面完成
    + 索引 一个暂存区域 会在下一次commit被提交到git仓库
    + git仓库 由Git object 记录着灭一次提交的快照，以及链式结构的记录变更历史

- example
    + 第一步 修改文件
    + 第二步 git add a.txt 
        - 将修改之后的文件新建一个 blob 
        - 更新索引到新建的blob
    + 第三步 git commit
        - 将索引里的东西在仓库中新建一个tree object
        - 新建一个 commit object
        - 第三步 将指针指引到新的commit object上面

> tips <br>
> 1. git的指令就是在操作这三个分区以及这条链
> 2. 思考一下git的各种命令，尝试在上图将它们“可视化”出来

- 问题：每次commit ，git存储的是全新的文件快照还是存储文件的变更部分
> 全新的文件快照，因为时间复杂度更低 <br>
> 在网络传输的时候，会将相似的压缩

- 问题:git怎样保证历史记录不被篡改？
> git和区块链的数据结构是非常相似的，两者都是基于哈希树和分布式 <br>
> 就该之后 哈希值就变动了

## GIT的使用技巧
- 问题： 误操作导致分支不见了，如何回复？
    + eg: git reset --hard  / git rebase
> git reflog 时光机 将每一次ref 变更都保存起来了

- 问题： 如何获得一个干净的工作空间
    + 现在工作空间太乱了； 工作一般，改需求
> - 方法1:
>   + git reset --hard HEAD 或者git checkout -f
>   + git clean -df
> - 方法2:
>   + git stash push [ -u | --include-untracked]
    + (还原) git stash pop

- 问题： 从git历史汇总删除一个文件
    + 敏感信息（私钥 内网ip等等）
    + 不需要控制版本控制的超大文件
> git filter-branch --tree-filter 'rm -f password.txt' HEAD <br>
> git是不可篡改的，运行之前一定沟通好

- git commit -amend
    + commi信息写错了，可以修改commit信息
    + 少改了信息，可以增加一下修改
    + 使用前沟通
- git rebase -i origin/master
    + 推到远端之前可以修改
- git show-branch
    + 查看两个分支区别
- git blame
    + 查看每一行代码的最后一次变更
- git bisect
    + 二分法
## 相关资料
https://www.lzane.com/talk/


