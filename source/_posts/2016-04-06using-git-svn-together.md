title: using git svn together
date: 2016-04-06 12:16:26
categories:
- coding
tags:
- git
- svn
- tips
- dailywork
---

[offical document](https://git-scm.com/book/zh/v1/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-Git-%E4%B8%8E-Subversion)

### clone
``` bash
$ git svn clone  https://10.100.xx.xx/svn/xxx/ -T trunk -b branches -t tags
```
-T trunk -b branches -t tags 告诉 Git 该 Subversion 仓库遵循了基本的分支和标签命名法则。如果你的主干(译注：trunk，相当于非分布式版本控制里的master分支，代表开发的主线），分支或者标签以不同的方式命名，则应做出相应改变。由于该法则的常见性，可以使用 -s 来代替整条命令，它意味着标准布局（s 是 Standard layout 的首字母），也就是前面选项的内容。下面的命令有相同的效果：
``` bash
$ git svn clone file:///tmp/test-svn -s
```

only clone from last serveral revsion
``` bash
 git svn clone -r486:HEAD https://10.100.xx.xx/svn/xxx/
```
### commit
``` bash
$ git commit -am 'Adding git-svn instructions to the README'
```

### rebase
``` bash
git svn rebase
```

### push
``` bash
$ git svn dcommit
```
