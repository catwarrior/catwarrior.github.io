title: setup nodejs on windows as a build server
date: 2015-12-28 10:37:08
categories:
- coding
tags:
- windows
- tips
- dailywork
- nodejs
---

Installing NodeJS
Jenkins NodeJS plugin does not work on Windows, so we’ll do a manual installation.

Download latest stable version (e.g. 0.10.35) from http://nodejs.org/

Don’t install to default directory C:\Program Files\nodejs as it requires administration rights, prefer a simpler path like c:\nodejs.

http://blog.majgis.com/npm-nodejs-fails-when-run-from-jenkins-on-windows/

Edit C:\nodejs\node_modules\npm\npmrc to replace
{% codeblock lang:bash %}
prefix=${APPDATA}\npm
{% endcodeblock %}

by

{% codeblock lang:bash %}
prefix=C:\nodejs\node_modules\npm
{% endcodeblock %}

Add the ‘C:\nodejs\node_modules\npm’ folder to the PATH environment variable, remove the one that was added by the installer: ‘C:\Users<user>\AppData\Roaming\npm’

npm may require git, install it from http://msysgit.github.io/

Add bower and grunt:
{% codeblock lang:bash %}
npm install -g bower grunt-cli
bower --version
grunt --version
{% endcodeblock %}