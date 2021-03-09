### rebase

在 cat 分支上 rebase dog 分支

```
$ git rebase dog
```

大概就是「我，就是 `cat` 分支，我現在要重新定義我的參考基準，並且將使用 `dog` 分支當做我新的參考基準」的意思

Rebase 指令等於是修改歷史，**不應該隨便對已經推出去給別人內容進行 rebase**，因為這很容易造成其它人的困擾

Rebase 合併分支跟一般的合併分支，第一個很明顯的差別，就是使用 Rebase 方式合併分支的話，**Git 不會特別做出一個專門用來合併的 Commit**。



## 【狀況題】怎麼取消 rebase？

如果是一般的合併，也許只要 `git reset HEAD^ --hard` 一行指令，拆掉這個合併的 Commit 大家就會退回合併前的狀態。但是，從上面的結果可得知，**Rebase 並沒有做出那個合併專用的 Commit**，而是整串都串在一起了，就跟一般的 Commit 差不多。所以這時候如果執行 **`git reset HEAD^ --hard`**，**只會拆掉最後一個 Commit**，**但並不會回到 Rebase 前的狀態**。



### 使用 ORIG_HEAD

在 Git 有另一個特別的紀錄點叫做 `ORIG_HEAD`，這個 ORIG_HEAD 會記錄**「危險操作」之前 HEAD 的位置**。例如分支合併或是 Reset 之類的都算是所謂的「危險操作」。

```
$ git rebase dog
```

成功重新計算 2 個 Commit 並接到 `dog` 分支上了，這時候**可使用這個指令輕鬆的跳回 Rebase 前的狀態**：

```
$ git reset ORIG_HEAD --hard
```



