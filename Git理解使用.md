# Git理解使用

参考：[git pro](https://git-scm.com/book/en/v2)，[git-scm.com/blog](https://git-scm.com/blog)

# 1. 分布式

集中式版本管理（如svn）：    
![](https://git-scm.com/book/en/v2/images/centralized.png)

分布式版本管理：  
![](https://git-scm.com/book/en/v2/images/distributed.png)

演示iSource和两个个linux主机间同步

```
git clone xyc@100.120.252.146:/local/workspace/gittest
```

集中式服务器宕机和磁盘损坏都是很严重的问题，分布式每台机器都拥有完整的仓库信息，所有操作基本都在本地进行。

# 2. 基于快照

增量式版本控记录（svn）：  
![](https://git-scm.com/book/en/v2/images/deltas.png)  

快照式版本记录：  
![](https://git-scm.com/book/en/v2/images/snapshots.png)

基于快照让git像一个文件系统，很多地方采用了引用，相对于增量式，更加简单，也因此功能更强大，分支操作也更高效。

基于快照存在的问题：  
对于大文件的修改，即使修改很小，也会同时记录两份文件，浪费空间。

![](https://www.git-tower.com/learn/content/01-git/01-ebook/en/02-desktop-gui/05-advanced-topics/07-git-lfs/01-large-file-adds-up.png)

为此对于大文件需要采用另外一种机制，Git LFS，参考：[Learn Version Control with Git](https://www.git-tower.com/learn/git/ebook/en/desktop-gui/advanced-topics/git-lfs)和[Git LFS-Tutorials](https://www.atlassian.com/git/tutorials/git-lfs)。

![](https://www.git-tower.com/learn/content/01-git/01-ebook/en/02-desktop-gui/05-advanced-topics/07-git-lfs/03-setup-git-lfs.png)

# 3. 三种状态、三个区域、三棵树

三种状态：
- committed：已经提交到数据库的修改
- modified：没有记录的修改
- staged：已经记录将要提交到数据库的修改，相当于缓存

![](https://git-scm.com/book/en/v2/images/areas.png)

演示流程

三棵树：
- HEAD：last commit snapshot, next parent
- Index：proposed next commit snapshot
- Working Dir：sandbox

![](https://git-scm.com/images/reset/workflow.png)

# 4. 文件的生命周期

![](https://git-scm.com/book/en/v2/images/lifecycle.png)

# 5. 基本操作：clone、record、view、undo、remote

clone主要有三种协议：`http(s)://path/to/repo.git`，`git://path/to/repo.git`， `user@server:path/to/repo.git`

- `http(s)://path/to/repo.git` 零配置，手动输入用户名密码
- `git://path/to/repo.git` 配置sshkey，基于ssh
- `user@server:path/to/repo.git` 基于ssh，主机间

record常用操作:

- **git add**  
  Add file contents to the index
- **git status**  
  Show the working tree status   
  ```
  $ git status -s
  ?? file1.txt              // aren't tracked
  A  file2.txt              // new file added to staging area
  AM file3.txt              // new file in staged area modified 
   M file4.txt              // file modified in workspace
  MM file5.txt              // file modified in staging area and workspace 
  D  file6.txt              // delete with git rm in both area
  R  file1.txt -> file.txt  // rename file in both area
  ```
  左侧状态相对于HEAD，右侧状态相对于INDEX
- **git commit**  
  Record changes to the repository
- **git rm**  
  Remove files from the working tree and from the index
  ```
  git rm --cached file1.txt // delete file just from staging area
  ```
- **git mv**  
  Remove files from the working tree and from the index

view用来查看版本历史：

- **git log**
  ```
  git log --graph
  ```

undo:

- **git commit --amend**  
  撤销一次commit，在过早提交时有效

- **`git reset HEAD <file>...`**
  unstaging a staged file

- **`git checkout -- <file>`**
  Unmodifying a Modified File

remote：

- **git remote**  
  查看当前的远程分支

- **git fetch [remote-name]**  
  同步远程分支

- **git push [remote-name]**  
  推送本地修改

- **git pull [remote-name]**  
  等同于fetch和merge

# 5. 分支、合并

一次commit的数据模型：
![](https://git-scm.com/book/en/v2/images/commit-and-tree.png)

commit之间的关系：
![](https://git-scm.com/book/en/v2/images/commits-and-parents.png)

新建分支：
![](https://git-scm.com/book/en/v2/images/branch-and-history.png)

- **git branch -b xxx**  
  新建分支

- **git merge xxx**  
  合并分支

- **git checkout xxx**  
  切换分支

正因为git的分支模型如此，分支操作才不会像svn是通过复制文件夹来实现。

# 6. reset

基本操作：  

![](https://git-scm.com/images/reset/reset-soft.png)

![](https://git-scm.com/images/reset/reset-mixed.png)

![](https://git-scm.com/images/reset/reset-hard.png)

![](https://git-scm.com/images/reset/reset-path1.png)

![](https://git-scm.com/images/reset/reset-path3.png)

合并一些提交：

![](https://git-scm.com/images/reset/squash-r1.png)

![](https://git-scm.com/images/reset/squash-r2.png)

![](https://git-scm.com/images/reset/squash-r3.png)

reset branch:

![](https://git-scm.com/images/reset/reset-checkout.png)

# FAQ

#### 合并git记录

- [Squash my last X commits together using Git](https://stackoverflow.com/questions/5189560/squash-my-last-x-commits-together-using-git)
- [How can I merge two commits into one?](https://stackoverflow.com/questions/2563632/how-can-i-merge-two-commits-into-one)

#### [What's the difference between HEAD^ and HEAD~ in Git?](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git)

#### There is no tracking information for the current branch.

```
git branch --set-upstream-to=origin/<branch> master
```

#### warning: LF will be replaced by CRLF in bootstrap/css/bootstrap-theme.css

Unix采用LF形式，可以通过input选项将windows checkout保持在crlf，而unix还是lf，参考[stackoverflow](https://stackoverflow.com/questions/5834014/lf-will-be-replaced-by-crlf-in-git-what-is-that-and-is-it-important)

```
git config --global core.autocrlf input
```

#### server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none

```
git config --global http.sslverify false
```

# 其他

* 找回没有引用的commit，参考reflog

* merge和rebase的区别：
  * merge如实记录版本过程
  * rebase简单化版本过程

* 理解diff输出，参考[How to read the output from git diff?](https://stackoverflow.com/questions/2529441/how-to-read-the-output-from-git-diff)

