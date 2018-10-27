---
title: Cookie和Session
date: 2018-10-26 11:23:23
tags:
---

HTTP无状态协议:HTTP协议本身是没有状态的，要保持状态需要使用Cookie+Session

### Cookie:

Cookie意为“甜饼”，是**由W3C组织提出**，最早由Netscape社区发展的一种机制。目前Cookie已经成为标准，所有的主流浏览器如IE、Netscape、Firefox、Opera等都支持Cookie。

由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。怎么办呢？就**给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理**。

Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客 户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务 器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。





### 使用cookie实现登录后的状态保留：

#### 1.注册

```python
def register(request):
	if request.method == 'GET':
		return render(request, 'register.html')

	if request.method == 'POST':
		#用于创建用户
		# 1.获取参数
		username = request.POST.get('username')
		password = request.POST.get('password')
		password2 = request.POST.get('password2')
		email = request.POST.get('email')

		# 2.校验参数是否完整,all函数的作用是里面的参数有任何一个None，返回值就为False
		# all函数里面全部不为空，返回值才为True,可以用来做非空校验
		flag = all([username, password, password2, email])


		if not flag:
			msg = '请填写完整信息'
			return render(request, 'register.html', {'msg': msg})

		# 3.先判断数据库中是否存在name用户
		if User.objects.filter(username=username).first():
			msg = '该账号已注册，请去登录'
			return render(request, 'register.html', {'msg': msg})

		# 4.校验密码是否一致
		if password != password2:
			msg = '两次密码不一次,请重新确认'
			return render(request, 'register.html', {'msg': msg})
		# 5.注册

		User.objects.create(username=username,
							password=password,
							email=email)

		# 6.跳转到登录页面
		return HttpResponseRedirect(reverse('user:login'))
```

#### 2.登录成功(给一个标识符)

​	登录:

```python
def login(request):
	if request.method == 'GET':
		return render(request,'login.html')
	if request.method == 'POST':
		# 1.获取参数
		username = request.POST.get('username')
		password = request.POST.get('password')
		# 2.校验数据完整性
		flag = all([username,password])

		if not flag:
			msg = '用户名密码不能为空'
			return render(request, 'login.html', {'msg': msg})
		# 3.验证用户是否注册
		user = User.objects.filter(username=username).first()
		if not user:
			msg = '该账户未注册，请去注册'
			return render(request, 'login.html', {'msg': msg})
		# 4.校验密码
		if user.password != password:
			msg = '密码不正确'
			return render(request, 'login.html', {'msg': msg})
				# 重点
		# 请求：请求是从浏览器发送请求的时候传递给后端的
		# 响应:后端返回给浏览器的
		res = HttpResponseRedirect(reverse('user:index'))
		# 添加标识符cookie
		# set_cookie(key,value,max_age)
		# 生成随机的字符串token
		s = '1234567890abcdefghijklmnopqrstuvwxyz'
		token = ''
		for i in range(25):
			token += random.choice(s)
		res.set_cookie('token', token, max_age=6000)

		# 存token值

		user_token = UserToken.objects.filter(user=user).first()
		if not user_token:
			UserToken.objects.create(token=token, user=user)

		return res
```

​	1).登录成功后向页面的cookie中添加标识符:set_cookie(key,value,max_age)
```python
		# 重点
		# 请求：请求是从浏览器发送请求的时候传递给后端的
		# 响应:后端返回给浏览器的
		res = HttpResponseRedirect(reverse('user:index'))
		# 添加标识符cookie
		# set_cookie(key,value,max_age)
		# 生成随机的字符串token
		s = '1234567890abcdefghijklmnopqrstuvwxyz'
		token = ''
		for i in range(25):
			token += random.choice(s)
		res.set_cookie('token', token, max_age=6000)
```
​	2).向后端的tb_usertoken表中存入这个标识符和登录的用户
```python
# 存token值

		user_token = UserToken.objects.filter(user=user).first()
		if not user_token:
			UserToken.objects.create(token=token, user=user)

		return res
```

#### 3.访问任何路由，先校验你的标识符是否正确，如果正确则放行。如果标识符不正确，则不让访问。

​	使用的是装饰器(装饰器)

​	装饰器:

```python
# 1.外层函数套内层函数
# 2.内层函数调用外层函数的参数
# 3.外层函数返回内层函数
from django.http import HttpResponseRedirect
from django.urls import reverse

from user.models import UserToken


def login_required(func):

	def check_login(request):
		# func是被login_required装饰的函数
		token = request.COOKIES.get('token')
		if not token:
			# cookie中没有登录的标识符，跳转到登录页面
			return HttpResponseRedirect(reverse('user:login'))
		user_token = UserToken.objects.filter(token=token).first()

		if not user_token:
		# 	标识符有误，跳转到登录页面
			return HttpResponseRedirect(reverse('user:login'))
		else:
			user_token.token = token
			user_token.save()
		return func(request)


	return check_login
```
验证是否登录成功的view
```python
@login_required
def index(request):
	if request.method == 'GET':
		return render(request, 'index.html')
```

#### 4.注销

​	删除页面同样需要先用装饰器判断是否有该cookie的值,总体来说可以分为以下两步
​	1).删除页面cookie中的标识符:delete_cookie(key)

​	2).删除后端tb_usertoken表中标识符对应的哪一条数据
```python
@login_required
def logout(request):
	if request.method == 'GET':
		# 1.删除cookie
		res = HttpResponseRedirect(reverse('user:login'))

		res.delete_cookie('token')
		# 2.删除数据库中的token的值
		token =  request.COOKIES.get('token')
		UserToken.objects.filter(token=token).delete()

		return res
```