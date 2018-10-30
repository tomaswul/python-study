---
title: Django文件上传 
date: 2018-10-30 16:31:04
tags:
---

Django在实现文件上传的时候，需要创建一个media文件夹，这个文件夹主要是用来存放上传的文件(图片，视频，文件)等媒体文件，media文件夹和应用程序文件夹是同级文件夹。在进行文件上传前需要在setting.py里面配置进行MEDIA的配置。如下

```python
#配置media路径(media文件里面主要放媒体(图片视频)文件)
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

在进行文件上传的时候同样需要一个上传表单对应的Model和实现表单验证的类。在model里面文件的字段为FileField(update_to='文件夹名')，或者是ImageField(update_to='文件夹名')。在存入数据库的时候，不是存入的文件的二进制数据，而是文件的路径，真正的文件则存放在/media/指定文件夹/上传文件 里。

因此这里的model设计为:

```python
from django.db import models

# Create your models here.
class Article(models.Model):
	title = models.CharField(max_length=20)
	desc = models.CharField(max_length=150)
    # 指定图片文件的存放文件夹
	img = models.ImageField(upload_to='article')
	create_time = models.DateTimeField(auto_now_add=True)

	class Meta:
		db_table = 'article'
```

进行了上述文件的编写以后要进行前端页面文件的配置，因为要上传文件，因此form表单里面的enctype="multipart/form-data" 文件上传的输入框的类型则选择file即可。

```django
{% extends 'base.html' %}
{% block title %}
    添加文章
{% endblock %}

{% block content %}
    <form action="" method="post" enctype="multipart/form-data">
       文章标题:<input name="title" type="text"><br>
       文章描述:<textarea name="desc" rows="10" cols="20">

        </textarea><br>

       文章图片:<input type="file" name="img"><br>
        <input type="submit" value="添加">

    </form>
{% endblock %}
```

之后进行路由配置(urls.py)

```python
url(r'^add_article/', views.add_article, name='add_article')
```

进行完上述配置后，可以进行view(views.py)的编写了，视图主要负责数据的交互和控制转发

```python
@login_required   # 这儿是判断用户是否是已经登录过的用户的装饰器
def add_article(request):
	if request.method == 'GET':
		return render(request, 'articles.html')

	if request.method == 'POST':
		# 获取数据
        # FILE获取的是具体的文件内容(二进制形式)
		img = request.FILES.get('img')
		title = request.POST.get('title')
		desc = request.POST.get('desc')

		# 创建文章，图片保存到数据库中是路径，这儿会自动进行文件保存目录为/media/article/xxxx.jpg
		Article.objects.create(img=img,title=title,desc=desc)
```

上传后的文件如果要展示在前台去需要在项目对应的urls.py进行如下配置，以声明media文件夹是静态文件夹，从而能够前台页面能够显示这个文件夹下的内容。

```python
# 将media文件夹解析为静态文件夹
# django在debug为True的情况下，就可以访问media文件夹的内容
urlpatterns += static(MEDIA_URL, document_root=MEDIA_ROOT)
```

而在前台显示的时候需要在要显示的文件前加上/media,具体内容如下:

```html
<img src="/media/{{ article.img }}">
```

