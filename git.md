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

让远端仓库分支回滚到某个commit

![image-20200229204747498](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200229204747498.png)

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

##### 合并冲突

```
当有冲突时，merge 处理好后
git status 查看状况
然后 git commit 提交
```

##### git搜索

```
in:readme
stars:>1000
filename:文件名
```

##### github分支合并

第一种

![image-20200229203403291](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200229203403291.png)

第二种，三个合一个

![image-20200229203328855](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200229203328855.png)

第三种，三个分别合上去

![image-20200229203451261](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200229203451261.png)





git拒绝合并无关的历史，解决方法是在 git pull 时加上–allow-unrelated-histories，

git pull origin master --allow-unrelated-histories