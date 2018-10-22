---
title: Django安装与初始化部署
date: 2018-10-22 16:50:02
tags:
---
Django最初被用来开发CMS(内容管理系统)软件

### 1.MVC模式
即模型(model),视图(view),控制器(controller)的缩写，一种软件设计典范。
![mvc模型](https://img-blog.csdn.net/20161018141129316?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
Model:即数据模型(java中的javabean),负责和数据库进行交互
Controller:即业务逻辑层,负责业务的控制与分发
View:数据展示层,复制数据展示

mvt:

Model:负责业务和数据库(ORM)的对象
View:负责业务逻辑并适当调用Model和Template
Template：负责把页面渲染展示给用户(html)

### 2.virtualenv虚拟环境创建指南
虚拟环境可以解决版本冲突问题
pip安装的库在python安装目录的lib下的site-packages下

#### 1.安装虚拟环境方式一

1.安装virtualenv

```
pip install vitualenv
```
2.创建虚拟环境
```
virtualenv --no-site-packages <虚拟环境目录>
	参数:
		--no-site-packages 表示不使用site-packeages里的库
		-p <python工作路径>	表示指定python版本
```
3.激活虚拟环境
```
进入djenv6/Scripts
activate
```
4.pip参数
```
pip 
	list:查看所有库
	freeze:查看python版本和虚拟环境
	--help:查看所有帮助文档
```

#### 2.创建虚拟环境方式二

1.创建虚拟环境

```
python -m venv djenv6
	-m 表示模式
	venv 表示虚拟环境
	djenv6 表示创建的虚拟环境目录
```
2.激活python虚拟环境
```
windows使用
djenv6/Scripts/actiavte.bat 启动这个文件
```
3.安装django1.11版本
```
>pip install django==1.11
```
4.检查django版本

```
(venv)$ python
>>> import django
>>> django.get_version()
```
5.安装PyMySQL

```
pip install PyMySQL
```



6.退出当前虚拟环境

```
deactivate
```

### 3.使用django创建项目

```
django-admin startproject <项目名>
```

> 注意：上面的命令最后的那个点，它表示在当前路径下创建项目。

执行上面的命令后看看生成的文件和文件夹，它们的作用如下所示：

- `manage.py`： 一个让你用各种方式管理 Django 项目的命令行工具。

- `oa/__init__.py`：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包。

- `oa/settings.py`：Django 项目的配置文件。

- `oa/urls.py`：Django 项目的 URL 声明，就像你网站的“目录”。

- `oa/wsgi.py`：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。

启动服务器运行项目
```
python manage.py runserver
```
在浏览器中输入http://127.0.0.1:8000访问我们的服务器，效果如下图所示。

![效果图](C:\Users\吴亮\Downloads\01.png)

> 说明1：刚刚启动的是Django自带的用于开发和测试的服务器，它是一个用纯Python编写的轻量级Web服务器，但它并不是真正意义上的生产级别的服务器，千万不要将这个服务器用于和生产环境相关的任何地方。

> 说明2：用于开发的服务器在需要的情况下会对每一次的访问请求重新载入一遍Python代码。所以你不需要为了让修改的代码生效而频繁的重新启动服务器。然而，一些动作，比如添加新文件，将不会触发自动重新加载，这时你得自己手动重启服务器。

> 说明3：可以通过`python manage.py help`命令查看可用命令列表；在启动服务器时，也可以通过`python manage.py runserver 1.2.3.4:56789`来指定绑定的IP地址和端口。

> 说明4：可以通过Ctrl+C来终止服务器的运行。

接下来我们进入项目目录oa并修改配置文件settings.py，Django是一个支持国际化和本地化的框架，因此刚才我们看到的默认首页也是支持国际化的，我们将默认语言修改为中文，时区设置为东八区。

```

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/chongqing'
```

回到manage.py所在的目录，刷新刚才的页面

![图片](C:\Users\吴亮\Downloads\02.png)

安装数据库驱动:

```
pip install pymysql
```

```
在该项目的__init__.py的文件中
import pymysql
pymysql.install_as_MySQLdb()
```



创建auth_user表

```
python manage.py migrate
```

添加用户

```
python manage.py createsuperuser
```

添加用户需要创建用户名等信息

后台登录

```
http:127.0.0.1:8080/admin
```

