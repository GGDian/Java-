```xml
//all 全部分支
//n4 最近4条
//graph 视图好看点
git log --oneline --all -n4 --graph
    
    
//查看所有分支
git branch -v
    
//在分离头指针情况下为创建分支
// git branch temp 9ae853f
git branch 新分支名 commit的id
```

##### 基于某个分支或者commit 创建一个新分支

```
git checkout -b 新分支名 旧分支名或者是某个commit
```

##### 区别

```
//两个 commit
git diff HEAD HEAD^(HEAD~1)/HEAD~2(HEAD^^)
//暂存区和Head
git diff --cached 
//工作区和暂存区
git diff
```

##### 回滚

```
//把暂存区恢复成HEAD
git reset HEAD
git reset HEAD -- file名字
//把工作区变暂存
git checkout -- index.html
```

```
//最近的commit不要了
git reset --hard commitHash值
```



##### 删除

```
git branch -D 分支名
```

##### 修改commit的message

```
git commit --amend
```

```
git rebase -i 需要修改的commitId的上一个commitId
```

##### 把多个commit合并到一个 用s

下面三个合并到最上面的

![image-20200227153208843](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200227153208843.png)

##### stash

```
git stash list

git stash pop //弹出后list没有了
git stash apply //弹出了还有
```

##### SSH

```
cd ~/.ssh
```

