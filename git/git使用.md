# git 分支的正确使用流程
- 建立main分支后，git branch ub22，建立ub22分支
- git checkout ub22，切换到ub22分支，进行文件的增加、修改，如修改了README.md
- git add README.md  以下三步都在ub22分支执行
- git commit -m "Update README.md"
- git push
- git checkout main  再切换回main分支
- git merge ub22  将ub22分支合并到main分支
- git push origin main  将更改提交到github.com的main分支，结束，完美！