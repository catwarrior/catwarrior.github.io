title: git-tips-for-daily-work
date: 2015-10-30 10:02:53
categories:
- coding
tags:
- git
- tips
- dailywork
---

# Realize you forgot to add the changes from xxx.ext
{% codeblock lang:git %}
git add xxx.ext
git commit --amend --no-edit
{% endcodeblock %}
