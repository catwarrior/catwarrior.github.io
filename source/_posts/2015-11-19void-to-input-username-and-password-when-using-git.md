title: void to input username and password when using git
date: 2015-11-19 16:10:28
categories:
- coding
tags:
- tips
- git
- tools
---

using git bash.
vim ~/.get-credentials
`https://myusername:mypassword@git.oschina.net`

`git config credential.https://git.oschina.net.username myusername [--global]`
`git config credential.helper store --file=~/.git-credentials [--global]`

