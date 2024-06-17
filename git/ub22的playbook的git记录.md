# 2024-06-04 21:55
```sh
root@ub22:~/playbook# git status 
On branch ub22
Your branch is up to date with 'origin/ub22'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   .gitignore
	modified:   README.md
	modified:   docker.yml
	modified:   group_vars/main.yml
	modified:   roles/docker/tasks/main.yml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	setup-kylinV10.md

no changes added to commit (use "git add" and/or "git commit -a")
#  Untracked files：今天新增了setup-kylinV10.md，所以我理解Untracked files就是新增的从未被git操作的文件
######################################################
root@ub22:~/playbook# git add .
root@ub22:~/playbook# git status 
On branch ub22
Your branch is up to date with 'origin/ub22'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   .gitignore
	modified:   README.md
	modified:   docker.yml
	modified:   group_vars/main.yml
	modified:   roles/docker/tasks/main.yml
	new file:   setup-kylinV10.md

root@ub22:~/playbook# 
root@ub22:~/playbook# git diff
root@ub22:~/playbook# 
#  git add .后执行git diff，没任何输出，所以我理解git diff作用是：输出当前工作区做的哪些更新还没有暂存，都暂存了就没有输出了
###########################################################
root@ub22:~/playbook# git commit -m '新增在RHEL8和 Kylin-V10上rpm安装docker'
[ub22 5732e03] 新增在RHEL8和 Kylin-V10上rpm安装docker
 6 files changed, 780 insertions(+), 29 deletions(-)
 create mode 100644 setup-kylinV10.md
root@ub22:~/playbook# 
root@ub22:~/playbook# git status 
On branch ub22
Your branch is ahead of 'origin/ub22' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
############################################################
root@ub22:~/playbook# git push
Enumerating objects: 22, done.
Counting objects: 100% (22/22), done.
Delta compression using up to 4 threads
Compressing objects: 100% (12/12), done.
Writing objects: 100% (12/12), 7.35 KiB | 7.35 MiB/s, done.
Total 12 (delta 8), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (8/8), completed with 8 local objects.
To github.com:cfzxcn/ansible.git
   c11108d..5732e03  ub22 -> ub22
root@ub22:~/playbook# 
root@ub22:~/playbook# git status 
On branch ub22
Your branch is up to date with 'origin/ub22'.

nothing to commit, working tree clean
###################################################
root@ub22:~/playbook# git branch 
* ub22
root@ub22:~/playbook# git branch main
root@ub22:~/playbook# git branch 
  main
* ub22
#  建立main分支，以前的main分支被我删除了
root@ub22:~/playbook# git checkout main 
Switched to branch 'main'
root@ub22:~/playbook# git branch 
* main
  ub22
root@ub22:~/playbook# git merge ub22
Already up to date.
# 将ub22分支合并到main分支
root@ub22:~/playbook# git status 
On branch main
nothing to commit, working tree clean
root@ub22:~/playbook# git branch 
* main
  ub22
##############################################################
root@ub22:~/playbook# git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

root@ub22:~/playbook# git push --set-upstream origin main
To github.com:cfzxcn/ansible.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:cfzxcn/ansible.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
#  现在切换到main分支无法push，懒的解决了，先用ub22分支吧！
#############################################################
root@ub22:~/playbook# git checkout ub22
Switched to branch 'ub22'
Your branch is up to date with 'origin/ub22'.
root@ub22:~/playbook# git status 
On branch ub22
Your branch is up to date with 'origin/ub22'.

nothing to commit, working tree clean
```
# 24/06/09
因为在github上手工修改了README.md，所以：先新建一个备份分支bak
```sh
root@ub22:~/playbook# git checkout -b bak   #  建立bak分支并切换到该分支下
Switched to a new branch 'bak'
root@ub22:~/playbook# git checkout ub22  # 切换回ub22分支
root@ub22:~/playbook# git branch 
  bak
  main
* ub22

root@ub22:~/playbook# git add .
root@ub22:~/playbook# git commit -m 'After modify README.MD on github'
[ub22 fb10fbf] After modify README.MD on github
 35 files changed, 131 insertions(+), 37 deletions(-)
......

root@ub22:~/playbook# git push
To github.com:cfzxcn/ansible.git
 ! [rejected]        ub22 -> ub22 (fetch first)
error: failed to push some refs to 'github.com:cfzxcn/ansible.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
#  果然和我的判断一样，就是在github上手工修改了文件，但本地未修改此文件，现在git push就不被允许了，现在还有个问题，就是我本地ub22分支上在github修改文件前有一些文件也被修改了。所以git pull也有问题了
root@ub22:~/playbook# git pull
remote: Enumerating objects: 434, done.
remote: Counting objects: 100% (433/433), done.
remote: Compressing objects: 100% (165/165), done.
remote: Total 299 (delta 142), reused 238 (delta 106), pack-reused 0
Receiving objects: 100% (299/299), 34.33 MiB | 3.36 MiB/s, done.
Resolving deltas: 100% (142/142), completed with 71 local objects.
From github.com:cfzxcn/ansible
   5732e03..f85e240  ub22       -> origin/ub22
 * [new branch]      dev        -> origin/dev
 * [new branch]      k8         -> origin/k8
 * [new branch]      main       -> origin/main
 * [new branch]      master     -> origin/master
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint: 
hint:   git config pull.rebase false  # merge (the default strategy)
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint: 
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.
#  以上问题现在无法解决，所以新建bak分支并切换，并准备pull，但查看文件发现是ub22分支上次提交的文件，新修改、增加的文件并未包含在内，所以这种方法不可行
root@ub22:~/playbook# git checkout bak 
Switched to branch 'bak'
root@ub22:~/playbook# git branch 
* bak
  main
  ub22
root@ub22:~/playbook# git push
fatal: The current branch bak has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin bak

nothing to commit, working tree clean
root@ub22:~/playbook# git push --set-upstream origin bak
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: 
remote: Create a pull request for 'bak' on GitHub by visiting:
remote:      https://github.com/cfzxcn/ansible/pull/new/bak
remote: 
To github.com:cfzxcn/ansible.git
 * [new branch]      bak -> bak
Branch 'bak' set up to track remote branch 'bak' from 'origin'.

#  为了稳妥起见，先备份到本地：G:\linuxbak\playbook2

root@ub22:~/playbook# git config pull.rebase true
#  按照建议执行此条：重新基础合并策略，前提条件：本地分支上有一系列提交(git commit)，远程分支上也有一系列提交
root@ub22:~/playbook# 
root@ub22:~/playbook# git status 
On branch ub22
Your branch and 'origin/ub22' have diverged,
and have 1 and 1 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

nothing to commit, working tree clean
root@ub22:~/playbook# git pull
Successfully rebased and updated refs/heads/ub22.
root@ub22:~/playbook# 
# 此时查看本地文件，上次成功提交后的修改被保留了，且下载更新了在github.com上的手工修改文件，完美，但上次本地成功提交后的修改还未提交到github.com的ub分支，所以：
root@ub22:~/playbook# git push  # 成功了，本地和github.com的ub分支完全一样了
root@ub22:~/playbook# git status 
On branch ub22
Your branch is up to date with 'origin/ub22'.

nothing to commit, working tree clean


#  以下操作试图将ub22分支合并到main分支，并在main分支上向github进行push
root@ub22:~/playbook# git checkout main
Switched to branch 'main'
root@ub22:~/playbook# 
root@ub22:~/playbook# git branch 
  bak
* main
  ub22
root@ub22:~/playbook# git merge ub22
Already up to date.
#  解决方法：
root@ub22:~/playbook# git reset --hard ub22 
HEAD is now at 39266f2 modify readme.md
root@ub22:~/playbook# git push --force origin main
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:cfzxcn/ansible.git
 + 3913aa5...39266f2 main -> main (forced update)
root@ub22:~/playbook# git status 
On branch main
nothing to commit, working tree clean
#  经查看文件，和ub22分支上一样了，成功！



```

