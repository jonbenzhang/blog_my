### revert 合并
[参考文档](https://zhuanlan.zhihu.com/p/443183430)
#### revert后重新合并后diff无法识别到代码修改
解决方案是将revert之后产生的分支，再次进行revert。这个操作能够将本要提交的代码，放置到最新的HEAD，其commitid要比master高，所以会重新diff。

```bash
#1.从master拉出一条分支
git checkout -b revert_tmp
# revert 合并的commit 需要指定reverse的分支
# git show <commit>^1 查看标号为1 的父分支的hash(一般当前分支为1)
#2.在revert_tmp将revert的commitid再次执行revert
# -m 为选择revert到父分支的编号(非合并commit的revert不需要-m)
git revert <commit> -m 1

#3.切换到开发分支
git checkout feature_pzq_status
#4.开发分支合并revert_tmp
git merge revert_tmp
#5.推送远端
git push origin feature_pzq_status
```