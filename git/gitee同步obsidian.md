https://gitee.com/cfzxcn/obsidian
# gitee新建仓库
```sh
#### 简易的命令行入门教程:
Git 全局设置:
git config --global user.name "cfzxcn"
git config --global user.email "cfzxcn@126.com"

创建 git 仓库:
mkdir obsidian
cd obsidian
git init 
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com:cfzxcn/obsidian.git
或
git remote add origin https://gitee.com/cfzxcn/obsidian.git
git push -u origin "master"

已有仓库?
cd existing_git_repo
git remote add origin git@gitee.com:cfzxcn/obsidian.git
或
git remote add origin https://gitee.com/cfzxcn/obsidian.git
git push -u origin "master"
```
# 在OneDrive目录新建子目录cfsync
```bash
考虑以后也能通过onedrive同步，所以将原仓库：d:\Users\cy\Documents\Obsidian Vault下的所有内容复制到：G:\OneDrive\cfsync

运行E:\tool-cf171013\ide\PortableGit2.42\git-bash.exe

cd G:\OneDrive\cfsync
cy@cy MINGW64 /g/onedrive/cfsync
$ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
Initialized empty Git repository in G:/OneDrive/cfsync/.git/

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .obsidian/
        .trash/
        Elasticsearch/
        Linux/
        Welcome.md
        attachment/
        clickhouse/
        git/
        nacos/
        operater system install/
        pxe/
        "\345\210\206\345\270\203\345\274\217\357\274\210\345\255\230\345\202\250
\357\274\211\346\226\207\344\273\266\347\263\273\347\273\237/"
        "\346\234\252\345\221\275\345\220\215.canvas"

nothing added to commit but untracked files present (use "git add" to track)

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git add .
warning: in the working copy of '.obsidian/app.json', LF will be replaced by CRLF
 the next time Git touches it
 ......
 
cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git commit -m "obsidian notes first commit"
[master (root-commit) 8eacba4] obsidian notes first commit
 745 files changed, 268013 insertions(+)
......

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --global user.name "cfzxcn"

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --global user.email "cfzxcn@126.com"

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git remote add origin https://gitee.com/cfzxcn/obsidian.git

git push -u origin "master"
```
![[Pasted image 20240618050515.png]]
![[Pasted image 20240618050455.png]]
首次提交成功！










