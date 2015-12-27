title: notes of powershell
date: 2015-12-27 16:12:00
categories:
- coding
tags:
- powershell
- tips
- dailywork
---

## using where in pipeline
{% codeblock lang:powershell %}
Clear-Host
$objNetwork = Get-WmiObject -List | Where {$_.name -Match "Network"}
$objNetwork | Format-Table name
{% endcodeblock %}