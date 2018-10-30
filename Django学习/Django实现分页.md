---
title: Django实现分页
date: 2018-10-30 16:31:21
tags:
---

Django实现分页主要借助于django.core.paginator.Paginator类。这个类的主要作用是对从数据库取回来的数据进行分割，进而封装成一个一个对象，例如一个表中有七条数据，按每三条进行分割，就可以生成三个page对象，两个page对象有各有三条记录，一个page对象有一条记录。

这儿用一个分页的例子来演示分页的效果

首先是urls.py配置:

```python
url(r'^articles/', views.articles, name='articles')
```

其次是views.py里的方法:

```python
from django.core.paginator import Paginator  #引入分页的类
def articles(request):
	if request.method == 'GET':
		# 查询所有文章对象并进行分页
		articles = Article.objects.all()
		# 分页，按每页三条记录进行分页
		paginator = Paginator(articles, 3)
		# 获取指定页数的数字，如果为None,默认值为1
		page = request.GET.get('page', 1)

		#返回指定页数的数据(本质是返回一个页面的所有数据是一个对象列表)
        arts = paginator.page(page)
		# arts.has_previous() 是否有上一页
		# arts.has_next() 是否有下一页
		# arts.number 当前页数
		# arts.next_page_number() 下一页页码
		# arts.previous_page_number() 上一页页码
		# arts.paginator.page_range() 是一个range对象(遍历的时候有用)
		return render(request, 'arts.html', {'arts':arts})
```

最后是在前端展示并实现分页效果:

```django
{% extends 'base.html' %}

{% block title %}
    分页查询
{% endblock %}

{% block content %}
    {% for art in arts %}
        <h1>{{ art.title }}</h1>
        <p>{{ art.desc }}</p>
        <img src="/media/{{ art.img }}">
    {% endfor %}
    {% if arts.has_previous %}
	{# 这儿判断是否存在上一页，如果存在才显示上一页 #}
    <a href="{% url 'user:articles' %}?page={{ arts.previous_page_number }}">上一页</a>
    {% endif %}
    {% for i in arts.paginator.page_range %}
	{# 这儿是进行遍历取出每一个具体的记录 #}
    <a href="{% url 'user:articles' %}?page={{ i }}">{{ i }}</a>
    {% endfor %}

    {% if arts.has_next %}
	{# 这儿判断是否存在下一页，如果存在才显示下一页 #}
    <a href="{% url 'user:articles' %}?page={{ arts.next_page_number }}">下一页</a>
    {% endif %}


{% endblock %}

```

实现分页这儿用到了前面的文件上传的知识(主要是图片展示的时候)