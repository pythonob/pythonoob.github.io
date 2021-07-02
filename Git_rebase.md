假设自己用dev, 总的为master

rebase 操作：
1. 先切换到master,拉取最新代码
2. 切换到dev, 然后git rebase master
3. 如果有冲突则解决冲突， 解决完冲突git add *** ,  git rebase -- continue
4. 切换到master分支，git merge dev



rebase 相比merge的优点：
1. 可以合并多次commit
   比如一个问题，修改了n次才最终修改好，可以使用git rebase --i [startpoint] [endpoint]将多次提交合并为1次，这样最终合并到master的分支上只会看到一次commit,避免commit信息太多，太混乱
2. 提交记录清晰
   如果使用merge, 会自动生成一个新的commit记录，而如果使用rebase, 会将dev的修改暂存，然后拉取master的最新代码，然后将dev的修改再合并到新的代码上。（这都是在dev操作的，完事还要在master上合并dev）
   这样不会生成新的commit记录。
