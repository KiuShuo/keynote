# 0基础入门
[Learn Version Control with Git
](https://www.git-tower.com/learn/git/ebook/cn/command-line/introduction#start)  
[廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
[git-recipes](https://github.com/geeeeeeeeek/git-recipes/wiki)  

# SourceTree用法
[Git可视化工具SourceTree的使用](http://blog.csdn.net/chenyufeng1991/article/details/51347083)  

[sourceTree回退撤销commit](http://blog.csdn.net/gang544043963/article/details/71511958)  

# git常用操作

[git pull和git fetch](https://oschina.net/translate/git-fetch-and-merge)  

###### ...merge和rebase 


# git工作流
[git工作流指南](https://github.com/xirong/my-git/blob/master/git-workflow-tutorial.md)  
[Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)

#### 1. 集中式工作流
像用svn一样使用git
远端只有一个分支，每个人从远端clone代码，每个人写完代码之后都提交到master上进行代码交换合并。(每次push之前都进行pull操作(merge或者rebase))

#### 2. 功能分支工作流

功能分支 + 集中式工作流
按照功能打分支，同一功能分支的开发者在一个分支上进行工作和代码合并。功能开发结束后将功能分支合并到master上。

#### 3. Gitflow工作流

master分支 + dev分支 (+ 功能分支)  
master分支用来记录每一次的发布，即只有发布的代码可以合并到master上。  
dev分支来源于master。  

如果不加功能分支，每个人都把自己的代码合并到dev分支上，大家进行代码的合并和交流。  
如果加功能分支，就按照功能从dev上打不通的功能分支，同一功能的人在同一功能分支上进行开发合并，功能开发结束后再将改功能分支的代码合并到dev分支上。  

一个版本开发结束后，此时dev上已经包含了最新的代码，将该dev的代码合并到master上。  
当然，也可以在预发布的时候从dev上打一个分支，预发布分支用来测试和修改bug，当完全测试通过后，将预发布分支合并到master和dev上。



##### 4. Forking工作流

master分支 + 开发者分支
  
每个开发者都从master上fork一个分支，每个开发者都clone自己的远端分支到本地进行开发，开发结束后push到自己的远端代码，等开发结束后在自己的远端发起pull request到master，由master的维护者来检查合并。

##### Rebase和merge
