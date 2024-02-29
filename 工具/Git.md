#### 练习网站
https://github.com/pcottle/learnGitBranching
#### 常用分支指令

1. git branch命令创建一个新的分支：`git branch <新分支>`
2. git checkout命令切换到新创建的分支：`git checkout <新分支>`
3. git merge命令将要合并的分支合并到目标分支（所在的分支）上：`git merge <要合并的分支>` 合并过程中可能发生冲突，如果发生，需要手动编辑冲突文件，解决冲突后使用`git add <冲突文件>`，之后再commit
4. 合并完成后，git branch -d命令删除已合并的分支：`git branch -d <要删除的分支>`
5. git cherry-pick命令将某个分支的某次提交应用到当前分支。对于将特定的代码改动从一个分支移动到另一个分支非常有用。：`git cherry-pick <commit>`
