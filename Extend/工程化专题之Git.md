## Git 常用指令
- git init   //初始化仓库
- git status //查看工作区的状态
- git add read.md //将改动添加到暂存区
- git commit -m "description"  //将提交到暂存区的修改提交到本地仓库
- git diff  //比较工作区文件与暂存区文件的区别
- git log --pretty=oneline   //查看提交历史
- git reset --hard HEAD^ //回退上一个版本
- git reflog  //查看命令历史
> 暂存区就是.git/index下这么个文件。  
那么，刚刚做完git commit之后，.git/index会变为空吗？ 显然不是。  
刚刚做完git commit之后，git diff比较工作区与暂存区，难道是工作区与空比较吗？ 显然不是。  
如果做完git add .之后，把工作区的文件、文件夹全部删除，然后执行git commit, 会怎样？ 当然是一切OK，又提交了一个版本。这说明什么？ 说明增加到暂存区的信息，虽然没有存入版本库，但已经独立于工作区的另一套东西了。  
如果你做了一大堆更改，增加了几个.lib依赖库，一共又好几兆的数据要增加给git，那么你猜git add . 与git commit，哪条命令明显耗时，哪条命令闪电执行完？ 答案是git add很耗时;git commit闪电完成。  
答案是git add命令把发生了变化的文件压缩创制为BLOB型的git object存入了.git\objects，当然很耗时；在.git/index保存了这些文件（BLOB对象）与文件夹（TREE对象）的引用信息。  
结论：刚刚做完git commit之后，.git/index所指向的git objects与HEAD是一样的。

> git diff 比较的是工作区文件与暂存区文件的区别（上次git add 的内容）   
git diff --cached 比较的是暂存区的文件与仓库分支里（上次git commit 后的内容）的区别

> 你在dev分支修改了文件，但是你没有提交到仓库，实际上就是相当于你在本地手动修改了这个文件，仓库并不能保存你做的改动，所以在master分支能看到文件被改动了（相当于你没用dev分支直接修改了这个文件一样），所以你可以用master分支add、commit

> git 切换分支时会把未add或未commit的内容带过去， 这一点值得注意。  
> 为什么呢？  
因为未add的内容不属于任何一个分支， 未commit的内容也不属于任何一个分支。 也就是说，对于所有分支而言， 工作区和暂存区是公共的。  
要想在分支间切换， 又不想又上述影响， 怎么办呢？ git stash搞起。要注意，在当前分支git stash的内容， 在其他分支也可以git stash pop出来，为什么？ 因为：工作区和暂存区是公共的。
