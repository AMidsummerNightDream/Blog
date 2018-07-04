# git命令

### 1 查看远程分支

``` bash
# 查看远程分支
git branch -a

//看到在master下的分支
blog user$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/nnvm
  remotes/origin/piiswrong-patch-1
  remotes/origin/v0.9rc1

```
### 2 查看本地分支

``` bash
git branch

```

### 3 切换分支
``` bash
git checkout -b v0.9 origin/v0.9

```
### git stash

``` bash
git stash
```

git stash 可用来暂存当前正在进行的工作， 比如想pull 最新代码， 又不想加新commit， 或者另外一种情况，为了fix 一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。

基础命令：
$git stash
$do some work
$git stash pop

git stash，将本地更改的代码存放git栈中（也可能有别的叫法），然后git pull,会将代码拉取下，此时你本地的代码和代码库是一样的，你的代码还在git栈中，此时你可以查看一下git status，然后从栈中将你的代码取出来 git stash pop，这时候你的代码会把你放在栈中的代码合并到你本地。 

git stash save "work in progress for foo feature"

当你多次使用’git stash’命令后，你的栈里将充满了未提交的代码，这时候你会对将哪个版本应用回来有些困惑，

’git stash list’ 命令可以将当前的Git栈信息打印出来，你只需要将找到对应的版本号，例如使用’git stash apply stash@{1}’就可以将你指定版本号为stash@{1}的工作取出来，当你将所有的栈都应用回来的时候，可以使用’git stash clear’来将栈清空。

``` bash
git stash          # save uncommitted changes
# pull, edit, etc.
git stash list     # list stashed changes in this git
git show stash@{0} # see the last stash 
git stash pop      # apply last stash and remove it from the list

git stash --help   # for more info
```

先说git stash：

git stash 命令可以将在当前分支修改的内容放到缓存区中，并会自动建立一个缓存的list集合，方便管理。

如果想将修改的内容重新释放出来，git stash apply 和 git stash pop 都可以达到这个目的。

但是两者有什么区别呢。

刚才说过，git stash 可以形成list 集合。通过git stash list 可以看到list下的suoy

使用git stash apply @{x} ，可以将编号x的缓存释放出来，但是该缓存还存在于list中

而 git stash apply，会将当前分支的最后一次缓存的内容释放出来，但是刚才的记录还存在list中

而 git stash pop，也会将当前分支的最后一次缓存的内容释放出来，但是刚才的记录不存在list中