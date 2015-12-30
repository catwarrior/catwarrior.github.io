title: git-tips-for-daily-work
date: 2015-10-30 10:02:53
categories:
- coding
tags:
- git
- tips
- dailywork
---

## Realize you forgot to add the changes from xxx.ext
{% codeblock lang:git %}
git add xxx.ext
git commit --amend --no-edit
{% endcodeblock %}

## The difference between git reset and git revert
git revert 是撤销某次操作，此次操作之前的commit都会被保留
git reset 是撤销某次提交，但是此次之后的修改都会被退回到暂存区
具体一个例子，假设有三个commit， git st:
commit3: add test3.c
commit2: add test2.c
commit1: add test1.c
当执行git revert HEAD~1时， commit2被撤销了
git log可以看到：
commit1：add test1.c
commit3：add test3.c
git status 没有任何变化
如果换做执行git reset --soft(默认) HEAD~1后，运行git log
commit2: add test2.c
commit1: add test1.c
运行git status， 则test3.c处于暂存区，准备提交。
如果换做执行git reset --hard HEAD~1后，
显示：HEAD is now at commit2，运行git log
commit2: add test2.c
commit1: add test1.c
运行git st， 没有任何变化
另外：
git revert <commit log string>是撤消该commit，作为一个新的commit。

## Change bash indicator.
go to git install dir.
ex: C:\Program Files\Git\etc
open bash.bashrc
find PS1 section.
it should be something like
{% codeblock lang:bash %}
# Set a default prompt of: user@host, MSYSTEM variable, and current_directory
#PS1='\[\e]0;\w\a\]\n\[\e[32m\]\u@\h \[\e[35m\]$MSYSTEM\[\e[0m\] \[\e[33m\]\w\[\e[0m\]\n \e[33m\]>>\e[0m\] '
{% endcodeblock%}
