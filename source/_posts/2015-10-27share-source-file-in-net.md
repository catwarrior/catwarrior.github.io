title: Share source file in .Net
date: 2015-10-27 10:51:00
category: programing
tags:
- .net
---

{% codeblock lang:c# http://stackoverflow.com/questions/1116465/how-do-you-share-code-between-projects-solutions-in-visual-studio %}
<Compile Include="..\MySisterProject\**\*.cs">
  <Link>_Inlined\MySisterProject\%(RecursiveDir)%(Filename)%(Extension)</Link>
  <Visible>false</Visible>
</Compile>
{% endcodeblock %}

## linq samples
[101 linq sample](http://blog.csdn.net/smartsmile2012/article/details/44873251)
