---
title: Django权限系统
date: 2018-10-31 16:52:30
tags:
---

### 1.权限系统介绍
RBAC（Role-Based Access Control，基于角色的访问控制），就是用户通过角色与权限进行关联。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。这样，就构造成“用户-角色-权限-资源”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，权限与资源之间一般是多对多的关系。 

Django自带了用户-组-权限，以及他们之间的联系。图解为:

![django权限联系图](https://www.qygou.club/images/django%E6%9D%83%E9%99%90%E5%9B%BE%E8%A7%A3.jpg)

django自带了用户表，也可以对用户表进行自定义，只需要继承django.contrib.auth.models.AbstarctUser即可。同时还有Group和Permission表。它们之间的关系实User和Group是多对多的，因此Django在定义User的Model时有一个groups字段来联系Group表，而User和Permission也是多对多的关系。User的Model中则是由user_permissions字段来联系Permission表。由于是多对多的关系，都生成了对应的中间表。Group和Permission表之间则是Group定义了permissons字段来关联Permission表。

![django表之间的联系](https://www.qygou.club/images/%E6%9D%83%E9%99%90%E8%A1%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E8%81%94%E5%85%B3%E7%B3%BB.jpg)

```
权限:
表：用户表、权限表、角色表
思想:
	1.创建角色
	2.角色分配权限
	3.用户分配角色
	4.(特殊情况)用户分配权限
用户表和权限表的ManyToManyField()字段:user_permissions
用户表和组表ManyToManyField()字段:groups
组表和权限表ManyToManyField()字段:permisssions
添加和删除: add(),remove()

查询
1.通过用户查询权限
#自己实现查询用户对应的权限方法
user.user_permissions.all()
user.groups.all()[0].permissions.all()
#django自带的查询方法
user.get_group_permissions()
user.get_all_permissions()

2.权限验证
#自己实现权限验证
user.user_permisssions.filter(codename='xxx')
user,groups.all()[0].permissions.filter(codename='xxx')
#django实现权限验证
user.has_perm('名称.权限名')

3.装饰器
#自己实现
def a(func):
	def b(request):
	return func(request)
return b
#django实现
@permission_required('名称.权限名')

```



### 2.自己实现权限系统

#### 1.自定义User模型，并生成相应的权限

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
# User和Group关联字段是User的groups字段
# User和Permission关联字段是User的user_permissions
# Group和Permission关联字段是Group的permissions字段

class MyUser(AbstractUser):
	# 扩展django自带的auth_user表，可以自定义新增的字段

	class Meta:
		# django默认给每个模型初始化三个权限
		# 默认是change,delete,add权限
		# permissions是设置用户权限，是一个元组，元组里面存放很多元组('权限功能', '权限名称')
		permissions = (
			('add_my_user', '新增用户权限'),
			('change_my_user_username', '修改用户名权限'),
			('change_my_user_password', '修改用户密码权限'),
			('all_my_user', '查看用户权限'),

		)
```

#### 2.生成相应的用户

这里生成了用户名为李四，密码为123123的用户

用户添加权限格式：

​	user.user_permissions.add(权限对象)

用户删除权限格式:

​	user.user_permissions.remove(权限对象)

```python
from django.contrib.auth.hashers import make_password
from django.contrib.auth.models import Permission
from django.http import HttpResponse, HttpResponseRedirect
def add_permission(request):
	if request.method == 'GET':
		# 1.创建用户
		password=make_password('123123')
		user = MyUser.objects.create(username='李四',password=password)
		# 2.指定刚创建的用户，并指定权限(新增用户权限，查看用户权限)
		# 正向添加权限
		# user.user_permissions.add(permission)
		permissions = Permission.objects.filter(codename__in=['add_my_user', 'all_my_user']).all()

		for permission in permissions:
			user.user_permissions.add(permission)
		# 3.删除刚创建的用户的新增用户权限
		# 正向删除权限
		# user.user_permissions(permission)

		for permission in permissions:
			if permission.codename == 'add_my_user':
				user.user_permissions.remove(permission)
		return HttpResponse('修改成功')
```

#### 3.通过装饰器来限制用户的访问

这里的装饰器主要是对登录表单提交的数据进行验证，看是否能在数据库中用户表中找到相对应的记录，如果能找到，再通过找到的对象去获取该对象的权限，看是否有权限访问指定页面，如果有权限就执行被装饰函数，如果没有权限就跳转到登录页面。

```python
from django.shortcuts import render
from user.models import MyUser
def loging_required(func):
	# 1.外层函数内嵌内层函数
	# 2.外层函数返回内层函数
	# 3.内层函数调用外层函数的参数
	def check_permission(request):
		myuser = MyUser.objects.filter(username=request.POST.get('username'),
									   password=request.POST.get('password')).first()
		if not myuser:
			return render(request, 'login.html')
		permission = myuser.user_permissions.filter(codename='all_my_user').first()
		if not permission:
			# 登录的用户没有权限，就返回登录页面
			return render(request, 'login.html')

		return func(request)

	return check_permission
```

被装饰的代码内容: 由于这儿登录表单是post方式提交的，因此在这儿就不做post和get方式判断，反正不论是post还是get方式如果不能通过装饰器都不会执行index函数，只有完整的执行了装饰器内容才会执行index函数的跳转。

```python
@loging_required
def index(request):
	# 用户名为张三的用户,有查看用户列表的权限,的才能访问如下的视图函数
	return render(request, 'index.html')
```

#### 4.生成相应的组，并给组赋予权限

```python
def add_group_permisssion(request):
	if request.method == 'GET':
		# 创建超级管理员(所有权限)、创建普通管理员(修改/查看)
		group1 = Group.objects.create(name='审核组')
		ps = Permission.objects.filter(codename__in= ['change_my_user_password',
												 'change_my_user_username',
												 'all_my_user']).all()
		for permission in ps:
			group1.permissions.add(permission)

		return HttpResponse('创建组权限成功')
```

#### 5.给用户赋予角色(组)

```python
def add_user_group(request):
	if request.method == 'GET':
		# 给指定用户分配审核组
		myuser = MyUser.objects.filter(username='李四').first()
		group = Group.objects.filter(name='审核组').first()

		myuser.groups.add(group)

		return HttpResponse('给用户分配组权限成功')
```

#### 6. 通过用户查询权限

获取user部分

```python
def show_user_permission(request):
	if request.method == 'GET':
		user = MyUser.objects.get(username='李四')


		return render(request, 'permission.html', {'user': user})
```

前端展示部分

```django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户权限信息</title>
</head>
<body>
<!--通过用户查询组,组查询权限 -->
<p>组权限</p>
{# user.groups.all.0   0获取第一个对象，1获取第二个对象 #}
    {% for per1 in user.groups.first.permissions.all %}
        {{ per1.codename }}<br>
    {% endfor %}
<!--通过用户直接查询权限 -->

<p>关联权限</p>

    {% for per2 in user.user_permissions.all %}
        {{ per2.codename }}<br>
    {% endfor %}
</body>
</html>
```

注意:django.middleware.csrf.CsrfViewMiddleware是用来防止会话劫持的中间件，它通过发放一次性的令牌和验证浏览器发送的一次性令牌来防止会话劫持。如果需要去掉403错误就需要在前端页面的表单中添加

```
{% csrf_token %}
```

```django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
</head>
<body>
<form action="{% url 'user:login' %}" method="post">
    {#默认生成input标签,一次性令牌，防止会话劫持#}
    {% csrf_token %}
    用户名:<input type="text" name="username">
    密码:<input type="password" name="password">
    <input type="submit" value="登录">
</form>
</body>
</html>
```

### 3.Django自带的权限系统

#### 1.实现django自带的登录验证

```python
def login(request):
	if request.method == 'GET':
		return render(request, 'login.html')
	if request.method == 'POST':
		form = UserLoginFrom(request.POST)
		if form.is_valid():
			user = auth.authenticate(request, username=form.cleaned_data.get('username'),
									 password=form.cleaned_data.get('password'))
			if user:
				auth.login(request, user)
				return HttpResponseRedirect(reverse('user:my_index'))
			else:
				return render(request, 'login.html')
		else:
			return render(request, 'login.html', {'errors': form.errors})

```

#### 2.登录后django自带的权限查询

```python
def my_index(request):
	if request.method == 'GET':

		# 当前登录系统用户
		user = request.user

		# 获取当前用户对应组的权限
		user.get_group_permisssions()
		# 获取所有权限
		user.get_all_permissions()
		# 判断是否有某个权限
		user.has_perm('应用app名.权限名')

		return render(request, 'my_index.html')
```

#### 3.django的自带的权限装饰器实现权限验证

```python
from django.contrib.auth.decorators import permission_required
# @permission_required('应用app名.权限名')
@permission_required('user.change_myx_user_password')
def new_index(request):
	if request.method == 'GET':
		return HttpResponse('需要权限才能查看')
```





