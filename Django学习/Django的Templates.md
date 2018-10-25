---
title: Django的Templates
date: 2018-10-25 11:51:47
tags:
---

templates是MVT模式中的T，负责展示数据，相当于MVC中的V,但是templates里的html有一些独有的特性，可以实现继承。例如:index.html继承base_main.html。那么index.html就是base_main.html的子模板,而base_main.html则继承base.html,这样base.html是根模板，只需要设置大体框架,而base_main.html则是二级根模板作为对根模板的补充，而index.html则是具体的显示页面。这样就可以建立模板的继承体系，实现模板的体系化管理

base.html

```django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>
{#父模板挖坑block后面的名字随意,最好有意义#}
        {% block title %}
        {% endblock %}
    </title>
</head>
<body>
{% block content %}
{% endblock %}
</body>
</html>
```

base_main.html

```django
{#这样可以个性化定制每个具体页面的效果，根页面只负责大体的框架#}
{#每个根页面下有数个二级根页面，作为对根页面的补充，具体显示的页面则继承二级根页面#}
{#这样设计可以形成页面体系#}

{% extends 'base.html' %}

{% block js %}
     <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
{% endblock %}
```



index.html

```django
 {#继承base_main.html，base_main.html是base.html的子模板#}
{%  extends 'base_main.html' %}

{% block title %}
    我是首页
{% endblock %}

{% block css %}

{#加载静态文件css的两种方式#}
{#方式一#}
{#<link href="/static/css/index.css" rel="stylesheet">#}

{#方式二#}
    {% load static %}
    <link href="{% static 'css/index.css' %}" rel="stylesheet">
{% endblock %}

{% block content %}
    <p>hello world</p>
{% endblock %}

{#重写了一个js的坑就不会调用父模板的坑了#}
{% block js %}
    {#这样就能使用父模板的block js坑里面定义的内容#}
    {{ block.super }}
{% endblock %}

xxxxxxxxxx {#继承base.html，index.html是base.html的子模板#}{%  extends 'base.html' %}{% block title %}    我是首页{% endblock %}{% block content %}    hello world{% endblock %}
```

1.继承

```django
标签:{% tag %} {% endtag %}
变量:{{ var }}
父模板base.html:block块，挖坑
	{% block xxx%}
	{% endblock %}
子模板index.html,可以选择性的填一些坑
	{% extends 'base.html' %}
	
	{% block xxxx %}
	xxxx
	{% endblock%}
```

2.注解

```django
<!--{# #}注解不会在网页源代码显示-->
<!---->这种注解会在源代码中显示，也可能会影响之下
<!--第一种注解:单行注解-->
{# 注解1 #}
{# 注解1 #}
{# 注解1 #}

<!--第二种注解:多行注解-->
{% comment %}
注解内容
{% endcomment %}
```

3.Django过滤器

```
1、add ：将value的值增加2。使用形式为：{{ value | add:2}}。
2、addslashes：在value中的引号前增加反斜线。使用形式为：{{ value | addslashes }}。
3、capfirst：value的第一个字符转化成大写形式。使用形式为：{{ value | capfirst }}。
4、cut：从给定value中删除所有arg的值。使用形式为：{{ value | cut:arg}}。
5、date: 格式化时间格式。使用形式为：{{ value | date:"Y-M-d h:m:s" }}
6、default：如果value是False，那么输出使用缺省值。使用形式：{{ value | default: "nothing" }}。例如，如果value是“”，那么输出将是nothing
7、default_if_none：如果value是None，那么输出将使用缺省值。使用形式：{{ value | default_if_none:"nothing" }}，例如，如果value是None，那么输出将是nothing
8、dictsort：如果value的值是一个字典，那么返回值是按照关键字排序的结果
使用形式：{{ value | dictsort:"name"}}，例如，
如果value是：
[{'name': 'python'},{'name': 'java'},{'name': 'c++'},]
那么，输出是：
[{'name': 'c++'},{'name': 'java'},{'name': 'python'}, ]
9、dictsortreversed：如果value的值是一个字典，那么返回值是按照关键字排序的结果的反序。使用形式：与dictsort过滤器相同。
10、divisibleby：如果value能够被arg整除，那么返回值将是True。使用形式：{{ value | divisibleby:arg}}，如果value是9，arg是3，那么输出将是True
11、escape：替换value中的某些字符，以适应HTML格式。使用形式：{{ value | escape}}。例如，< 转化为 &lt;> 转化为 &gt;' 转化为  &#39;" 转化为  &quot;
13、filesizeformat：格式化value，使其成为易读的文件大小。使用形式：{{ value | filesizeformat }}。例如：13KB，4.1MB等。
14、first：返回列表/字符串中的第一个元素。使用形式：{{ value | first }}
16、iriencode：如果value中有非ASCII字符，那么将其进行转化成URL中适合的编码，如果value已经进行过URLENCODE，改操作就不会再起作用。使用形式：{{value | iriencode}}
17、join：使用指定的字符串连接一个list，作用如同python的str.join(list)。使用形式：{{ value | join:"arg"}}，如果value是['a','b','c']，arg是'//'那么输出是a//b//c
18、last：返回列表/字符串中的最后一个元素。使用形式：{{ value | last }}
19、length：返回value的长度。使用形式：{{ value | length }}
20、length_is：如果value的长度等于arg的时候返回True。使用形式：{{ value | length_is:"arg"}}。例如：如果value是['a','b','c']，arg是3，那么返回True
21、linebreaks：value中的"\n"将被<br/>替代，并且整个value使用</p>包围起来。使用形式：{{value|linebreaks}}
22、linebreaksbr：value中的"\n"将被<br/>替代。使用形式：{{value |linebreaksbr}}
23、linenumbers：显示的文本，带有行数。使用形式：{{value | linenumbers}}
24、ljust：在一个给定宽度的字段中，左对齐显示value。使用形式：{{value | ljust}}
25、center：在一个给定宽度的字段中，中心对齐显示value。使用形式：{{value | center}}
26、rjust：：在一个给定宽度的字段中，右对齐显示value。使用形式：{{value | rjust}}
27、lower：将一个字符串转换成小写形式。使用形式：{{value | lower}}
30、random：从给定的list中返回一个任意的Item。使用形式：{{value | random}}
31、removetags：删除value中tag1,tag2....的标签。使用形式：{{value | removetags:"tag1 tag2 tag3..."}}
32、safe：当系统设置autoescaping打开的时候，该过滤器使得输出不进行escape转换。使用形式：{{value | safe}}
33、safeseq：与safe基本相同，但有一点不同的就是：safe是针对字符串，而safeseq是针对多个字符串组成的sequence
34、slice：与python语法中的slice相同。使用形式：{{some_list | slice:"2"}}
37、striptags：删除value中的所有HTML标签.使用形式：{{value | striptags}}
38、time：格式化时间输出。使用形式：{{value | time:"H:i"}}或者{{value | time}}
39、title：转换一个字符串成为title格式。
40、truncatewords：将value切成truncatewords指定的单词数目。使用形式：{{value | truncatewords:2}}。例如，如果value是Joel is a slug 那么输出将是：Joel is ...
42、upper：转换一个字符串为大写形式
43、urlencode：将一个字符串进行URLEncode
46、wordcount：返回字符串中单词的数目
```

5.templates的循环计数

```django
            <!--
                forloop.counter是序号，是循环的计数
                forloop.counter0是从0开始的序号
                forloop.revcounter0，是倒序打印序号，以0结束
                forloop.revcounter,倒序打印到1结束
                forloop.first  第一个序列号(True)转换成数值就是1，其他就是False
                forloop.last  最后一个序列号(True)，转成数值就是1，其他就是False

                注解写在！里面依然会加载，会影响真正的需要执行的代码
            -->
```

6.常用标签

```django
1.{% block <名称> %}
  		(父)挖坑(子)填坑
  		相当于java中的方法
  {% endblock %>
  
2.{% extends <html页面> %}
  继承:相当于java中的继承，可以对继承的坑进行重写，也可以通过{% block.super %}执行继承的父模板的坑里面的内容
  
3.{% for <值> in <序列> %}
  	{{值}}
  	这儿是执行循环的内容
  {% empty %}
  	如果序列是空的话，就执行empty里的内容
  {% endfor %} 
  
4.{% if <条件表达式> %}
  		表达式一
  {% else %}/{% elif %}
  		表达式二
  {% endif %}
  
5.{% ifequal 值1 值2 %}   {#判断值1是否等于值2#}
  		表达式一
  {% else %}
  		表达式二
  {% endifequal %}
```



