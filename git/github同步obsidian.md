### …or create a new repository on the command line
echo "# cfsync" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M master
git remote add github-origin https://github.com/cfzxcn/cfsync.git
git push -u github-origin master
### …or push an existing repository from the command line
git remote add github-origin https://github.com/cfzxcn/cfsync.git
git branch -M master
git push -u github-origin master
# 初始步骤
```sh
cy@cy MINGW64 /
$ cd g:\onedrive\cfsync
cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --global user.name cf

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --global user.email 604174410@qq.com

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git remote add github-origin https://github.com/cfzxcn/cfsync.git

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git remote -v
github-origin   https://github.com/cfzxcn/cfsync.git (fetch)
github-origin   https://github.com/cfzxcn/cfsync.git (push)
origin  https://gitee.com/cfzxcn/obsidian.git (fetch)
origin  https://gitee.com/cfzxcn/obsidian.git (push)

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git branch
* master
```
# 在Github上添加win10生成的ssh 公钥
```sh
https://github.com/，点击：头像---settings---左侧菜单ssh and gpg keys---New SSH key，Title输入框随意，Key输入框填入公钥：C:\Users\cy\.ssh\id_rsa.pub的内容

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ ssh -T git@github.com
Hi cfzxcn! You've successfully authenticated, but GitHub does not provide shell access.
```
# 通过win10的git-bash推送报错
```bash
cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git push -u github-origin master
fatal: unable to access 'https://github.com/cfzxcn/cfsync.git/': schannel: next InitializeSecurity
Context failed: Unknown error (0x80092012) - µõÏú¹¦ÄÜÎ޷¨¼ì²é֤ÊéÊǷñµõÏú¡£

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --system http.sslbackend openssl

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git push -u github-origin master
fatal: unable to access 'https://github.com/cfzxcn/cfsync.git/': SSL certificate problem: unable t
o get local issuer certificate

cy@cy MINGW64 /g/onedrive/cfsync (master)
$ git config --global http.sslVerify false
```
![[Pasted image 20240620095824.png]]
选择：Sign in with your browser，弹出浏览器，输入用户名和密码
![[Pasted image 20240620101114.png]]
再报错：
```bash
remote:       —— Google OAuth Client ID ————————————————————————————
remote:        locations:
remote:          - commit: 8eacba4899916e62b2b8ff24899122358e51ceb3
remote:            path: .obsidian/plugins/remotely-save/main.js:90
......
修改：G:\OneDrive\cfsync\.gitignore，追加一条：.obsidian/plugins/remotely-save/
```
然后就自动push了，成功！

 








