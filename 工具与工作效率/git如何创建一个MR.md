https://zmcdbp.com/gitlab-merge-request-simple-use/



在 imaygou.ai 的流程：

1.  git clone source
2.  git checkout -b jemmy
3.  finish coding and commit 
4.  git push -u origin jemmy(此时会创建一个 MR )
5.  等待leader review代码，同意MR
6.  同意之后去看 Jenkins是否正确
7.  正确之后，自己分支打tag: git tag -a jemmy001 -m “finish helloworld”
8.  提交tag：git push origin jemmy001
9.  再次去看Jenkins





git checkout -b newbrabch -t origin/newbranch  与远端保持同步



**修改最近几次的提交信息：**

git rebase -i HEAD~3

将需要修改的某一次提交前面的 pick 改为 edit；

git commit —amend 然后修改

git rebase —continue

