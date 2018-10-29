---
title: Django的登录验证以及自带的session生成和注销
date: 2018-10-29 15:56:59
tags:
---

### 1.Form表单验证

之所以会有form表单验证，是因为不可能每张表单每个输入项都要一个一个if结构去验证，这样效率太低，并且错误信息不够具体。表单验证的主体思想是为每一个form表单创建一个基类是forms.Form类的form表单类，这个类的字段就是form表单里每个输入项的name值(这个一定要一致)。这样就可以把form表单提交的数据通过对象进行封装，并且封装的同时还要对每个字段的要求进行验证。并且设置特定的错误提示信息，同时在封装对象之后的返回值是一个form表单对象，通过这个对象的is_vaild()方法，可以判断form表单除格式外是否满足要求，该方法默认会调用form表单类的clean(self)方法，而这个方法是forms.Form类的方法，是通过继承的方法。在这个方法里需要返回的是表单数据self.clean_data。并且可以在这个方法里通过raise.ValidationError(字典)抛出错误信息。注册和登录的form表单类设计:

```python
#引入form表单的模块
from django import forms
#引入django自带的用户Model
from django.contrib.auth.models import User

class UserRegisterForm(forms.Form):
    '''
    用户注册表单类
    '''
	username = forms.CharField(max_length=16,min_length=2,required=True,
						   error_messages={
							   'required': '用户名不能为空',
							   'min_length': '最小长度不能小于2字符',
							   'max_length': '最大长度不能超过16字符'
						   })
	password = forms.CharField(max_length=16,min_length=2,required=True,
						   error_messages={
							   'required': '密码不能为空',
							   'min_length': '最小长度不能小于2字符',
							   'max_length': '最大长度不能超过16字符'
						   })
	
	password2 = forms.CharField(max_length=16,min_length=2,required=True,
						   error_messages={
							   'required': '确认密码不能为空',
							   'min_length': '最小长度不能小于2字符',
							   'max_length': '最大长度不能超过16字符'
						   })
	def clean(self):
        #这个方法是验证表单项和字段是否满足条件
		if User.objects.filter(username=self.cleaned_data.get('username')).first():
			raise forms.ValidationError({'username':'账号已注册，请去登录'})
		if self.cleaned_data.get('password') != self.cleaned_data.get('password2'):
			raise forms.ValidationError({'password':'密码不一致'})

		return self.cleaned_data


class UserLoginForm(forms.Form):
    '''
    用户登录表单类
    '''
	username = forms.CharField (max_length=16, min_length=2, required=True,
								error_messages={
									'required': '用户名不能为空',
									'min_length': '最小长度不能小于2字符',
									'max_length': '最大长度不能超过16字符'
								})
	password = forms.CharField (max_length=16, min_length=2, required=True,
								error_messages={
									'required': '密码不能为空',
									'min_length': '最小长度不能小于2字符',
									'max_length': '最大长度不能超过16字符'
								})

	def clean(self):
		user = User.objects.filter(username=self.cleaned_data.get('username'))
		if not user:
			raise forms.ValidationError({'username': '该账号未注册，请去注册'})
		return self.cleaned_data
```



### 2.Django自带的登录验证和注销

django自带了实现session和cookie的方法，这个session对应的是数据库中django_session,同样django自带的User(django.contlib.auth.models.User)类对应的是数据库表中的auth_user表

#### 1.实现注册表单数据的验证和注册

```python
def register(request):
	if request.method == 'GET':
		return render(request, 'register.html')
	if request.method == 'POST':
		data = request.POST
		form = UserRegisterForm(data)
		if form.is_valid():
			# 验证通过

			# 密码加密
			password = make_password(form.cleaned_data.get('password'))
			# 将数据存入django自建的user表中
			# User.objects.create()创建普通用户
			# User.objects.create_superuser()创建超级管理员
			User.objects.create(username=form.cleaned_data.get('username'),
							password=password)
			return HttpResponseRedirect(reverse('user:login'))
		else:
			return render(request, 'register.html', {'errors':form.errors})

```

#### 2.django登录(session的产生，发布与保存)

```python
def login(request):
	if request.method == 'GET':
		return render(request, 'login.html')
	if request.method == 'POST':
		data = request.POST
		form = UserLoginForm(data)
		if form.is_valid():  ==>会默认调用clean()方法进行验证
			#用户名密码不为空
			# django自带的auth  auth是django.contrib.auth
            # 通过auth模块可以实现用户的验证
            # auth.authenticate(username='xxx',password='xxx')返回值是一个User对象或者是None.
            # 在这儿为None只有一种情况，就是密码错误，因为其他信息都经过了验证，而程序还执行到了这儿
            # 肯定就是这儿有问题导致无法查询出User对象

			user = auth.authenticate(username=form.cleaned_data.get('username'),
							  password=form.cleaned_data.get('password'))
			if user:
				# 登录,向request.user属性赋值，赋值为登录系统的用户对象
				# 1.向页面的cookie中设置sessionid值，(标识符)
				# 2.向django-session表中设置对应的标识符
				auth.login(request,user)

				return HttpResponseRedirect(reverse('user:index'))
			else:
				return render(request, 'login.html',{'msg': '密码错误'})


		else:
			# 用户名面为空
			return render(request, 'login.html', {'errors':form.errors})
```

auth.authenticate()这个函数的作用是通过登录表单传递的值，来和数据库进行比对，如果数据库有符合这个的记录，就返回对应的对象，否则返回的是None，然后在通过auth.login(request,user)函数来实现登录后的一些操作用户第一次登录，后端用session保留登录状态，并标记了一个sessonid在请求中；  

1.如果没有新用户提交登录，就将原保存在request中的user返回，保持登录状态；    

2.如果有新用户提交登录，对user中密码进行加密,通过hash中的md5，传入参数有：加密盐，密码，settings.py中设置SECRET_KEY；    

To avoid reusing another user's session, create a new, empty    

session if the existing session corresponds to a different    authenticated user.    

如果新的用户登录，为了避免存在的session与其他认证通过的用户一致，需要清除原来的session， 并且重新创建一个session来存放新用户；

#### 3.登录之后通过session实现无密登录

```python
@login_required
def index(request):
	if request.method == 'GET':
		return render(request, 'index.html')
```

这儿的@login_required(django.contrib.auth.decorators.login_required)是django自带的装饰器，作用是实现session(标识符)的验证，以确定用户是否能通过get方式直接访问一些特殊页面(需登录后的页面)。

#### 4.登录后的注销

```python
@login_required
def logout(request):
	if request.method == 'GET':
		# 1.删掉了session_id的值，
		# 2.并且通过session_id去django-session表中删除了session_id等于session_key的值
		auth.logout(request)
		return HttpResponseRedirect(reverse('user:login'))
```

注意:以上的函数的具体实现方式和过程，可以通过断点的模式一步一步执行。这样可以清楚的知道整个流程到底是怎样实现的。说的简单一点就是封装。

### 3.中间件

#### 1.中间件的介绍

中间件类似java中的过滤器，不过更智能一点，java中的过滤器处理之后需要手动放行，否则就会在过滤器那儿堵住，python的中间件会自动放行。

python中的中间件就是一个类，如果要使用中间件需要在setting.py文件的MIDDLEWARE里面配置中间件。

```python
# 中间件
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'utils.middleware.TestMiddleware1',
    'utils.middleware.TestMiddleware',
]
```



多个中间去的执行顺序如图所示:

![中间件调用顺序](https://files.jb51.net/file_images/article/201807/2018071609004710.png)

#### 2.中间件的设计

并且中间件的类需要如下的方式进行设计:创建一个 工具包，中间件是这个包的一个模块，这个模块里包含了多个中间件的类。中间件需要继承的类因为版本的问题，不同的版本可能不一样。这儿的django版本是1.11，需要其他的版本的时候再查再用。

```python
from django.utils.deprecation import MiddlewareMixin


class TestMiddleware(MiddlewareMixin):


	def process_request(self, request):
		print('process_request')

		# 继续执行对应的视图函数
		return None

	def process_response(self,request,response):
		print('process_response')

		# 返回相应
		return response

# 中间件是双向的
# 当有两个中间件或者多个中间件的时候,request请求的时候中间件的执行顺序依赖于setting.py文件里
# 的MIDDLEWARE中中间件配置的先后顺序，先配置的中间件先执行，后配置的中间件后执行
# response响应的时候执行顺序则是从下向上执行，中间件的顺序是双向的(用于写日志中间件)
class TestMiddleware1(MiddlewareMixin):


	def process_request(self, request):
		print('process_request1')

		# 继续执行对应的视图函数
		return None

	def process_response(self,request,response):
		print('process_response1')

		# 返回相应
		return response
```

#### 3.常用的中间件方法

``` 
1. process_request(self, request)
	执行时机在django接收到request之后, 但仍未解析出url以确定运行哪个视图函数view之前,就是在用户提交请求数据之后，但是服务器还未解析提交的信息，只知道用户提交的请求数据的时候会执行这个方法里面内容

2. process_view(self, request, view_func, view_args, view_kwargs)
	执行时机在django执行完request预处理函数并确定待执行的view之后, 但在视图函数view之前
	request: HttpRequest对象
	view_fun: 是django将要调用的视图函数, 是真实的函数对象本身
	view_args: 将传入view的位置参数列表, 不包括request参数
	view_kwargs: 将传入view的字典参数

3. process_response(self, request, response)
	该方法必须返回HttpResponse对象, 可以是原来的, 也可以是修改后的

	调用时机在django执行完view函数并生成response之后, 该中间件能修改response的内容, 常见用途比如压缩内容(加密后传输)
	request是request对象
	response是从view中返回的response对象

4. process_exception(self, request, exception)
	默认不主动调用，该方法只有在request处理过程中出了问题并且view函数抛出了一个未捕获的异常才会被调用, 可以用来发送错误通知, 将相关信息输出到日志文件, 或者甚至尝试从错误中自动恢复
	参数包括request对象, 还有view函数抛出的异常对象exception
	必须返回None或HttpResponse对象

5. process_template_response(self, request, response)
	默认不主动调用，在视图执行render()返回后进行调用，必须返回None或HttpResponse对象
	
	
在不发生异常和执行render函数的时候具体的执行顺序是:process_request()  ==>   process_view()  ==>  process_response()

如果发生了异常且有异常未捕获的时候直接执行process_exception(),

process_template_resposonse()则是很少用，因为他是在reponse之后执行的，并没有多大的实用价值。
```

