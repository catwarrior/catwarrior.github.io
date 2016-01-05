title: upgrade npm on windows
date: 2015-11-05 16:18:04
categories:
- coding
tags:
- tips
- windows
- npm
- nodejs
---

Powershell Run as Administrator
{% codeblock lang:powershell %}
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
{% endcodeblock %}

{% codeblock lang:powershell %}
npm install -g npm-windows-upgrade
npm-windows-upgrade
{% endcodeblock %}

or Want to upgrade to specific version?

{% codeblock lang:powershell %}
npm-windows-upgrade --version:3.1.0
{% endcodeblock %}

[Reference from npmjs.](https://www.npmjs.com/package/npm-windows-upgrade)