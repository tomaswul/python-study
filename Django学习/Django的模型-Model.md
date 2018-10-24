---
title: Django的模型(Model)
date: 2018-10-23 17:56:08
tags:
---
### 1.模型的创建和说明

#### 创建项目app

```
(djenv6) D:\Python_Study\workspace\day02>python manage.py startapp app
```

![](C:\Users\吴亮\Desktop\人工智能\千锋培训python\python 第三阶段\DjangoApp文件结构.jpg)



orm：object-relational-mapping 对象关系映射

#### 创建model并映射到数据库
1.在app里的models.py里创建学生类
```python
from django.db import models

# Create your models here.
class Student(models.Model):
	# 定义字符串类型max_length设置最大长度
	s_name = models.CharField(max_length=6, unique=True)
	# 定义s_age字段, int类型
	s_age = models.IntegerField(default=18)
	#定义性别，默认值是true(男)
	s_gender = models.BooleanField(default=True)

	class Meta:
		#定义模型迁移到数据库中时的表名
		db_table = 'student'
```
2.修改整个项目的settings.py里的INSTALLED_APPS,添加app
```python
INSTALLED_APPS= [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app',
]
```
3.迁移数据(两步)
```
python manage.py makemigrations 生成迁移文件
python manage.py migrate 执行迁移
```

4.修改模型添加两个字段

```python
#定义create_time字段，创建时间,自动添加(插入数据的时间)当前时间
#auto_now_add:表示添加数据的时间 null=True表示允许为空(默认是null)
create_time = models.DateTimeField(auto_now_add=True,null=True)

#定义operate_time字段，修改时间,修改数据的时间
#auto_now:表示修改数据的时间,null=True默认是null
operate_time = models.DateTimeField(auto_now=True,null=True)
```

5.重新生成迁移文件

```
pyhton manage.py makemigrations
```

6.更新数据库表结构

```
pyhton manage.py mirgrate
```

7.添加创建学生的url

```
# 127.0.0.1:8080/create_stu/   创建学生对象
url(r'^create_stu/', views.create_stu)
```

8.在app里的views.py里面创建create_stu方法 obj.save()表示向数据库表里添加一条记录

```python
def create_stu(request):
	# 第一种方式:创建学生
	stu = Student()
	stu.s_name = '张三'
	stu.s_age = 20
	# 保存到数据库中的表中
	stu.save()
	'''

	# 第二种方式:创建学生
	Student.objects.create(s_name='张四')
```

### 2. Django的orm常用查询方法
1.all()方法:查询信息
```
格式:
	类.objects.all() #返回值是querySet
students = Student.objects.all()
```
2.filter()方法:过滤查询部分的信息

```
格式:
	类.objects.filter(字段过滤条件) #返回值是querySet
stus = Student.objects.filter(s_name='张三')

# first()获取查询集合的第一个
stu = Student.objects.filter(s_age=20).first() #返回值具体的对象

#last()获取查询集合的最后一个
stus = Student.objects.filter(s_age=20).last() #返回值是具体的对象
```

3.get()方法:获取单条记录(具体的对象)

```
# get只能获取一条记录，超过一条记录以及记录不存在都会报错,不建议使用
stus = Student.objects.get(s_age=20)
```

4.满足多条条件的查询

```
# 查询年龄等于二十并且性别是0的信息
stus = Student.objects.filter(s_age=20).filter(s_gender=0)
#这种写法和上面效果一样stus = Student.objects.filetr(s_age=20, s_gender=0)
```

5.模糊查询

```
# 模糊查询 like
stus = Student.objects.filter(s_name__contains='张')

# 模糊查询，查询以什么开始的
stus = Student.objects.filter(s_name__startswith='李')

# 模糊查询，查询以什么结束的
stus = Student.objects.filter(s_name__endswith='四')

模糊查询:
	字段__contains='xx',查询字段包含xx的记录
	字段__startswith='xx',查询字段以xx开始的记录
	字段__endswith='xx',查询字段以xx结束的记录
```

6.大于(gt),小于(lt),大于等于(gte),小于等于(lte)

```
# 大于(gt)小于(lt),大于等于(gte),小于等于(lte)
stus = Student.objects.filter(s_age__gt=18) #年龄大于18的记录
# stus = Student.objects.filter(s_age__lt=21)  # 龄小于18
格式:
	字段__gt=值
	字段__lt=值
	字段__gte=值
	字段__lte=值
```

7.排序order_by('字段')

```python
#排序 order_by()
	# 升序
	stus = Student.objects.order_by('id')

	# 降序
	stus = Student.objects.order_by('-id')
```

8.查询不满足条件的记录exclude(字段=值)

```python
# 查询不满足条件的数据exclude()
stus = Student.objects.exclude(s_age=18) #查询年纪不等于18的
```

9.统计查询个数count()

```pyhton
# 统计查询个数count()和len()等效
stus_count = stus.count()
print(stus_count)
```

10.获取每条记录的每个字段values()返回值是字典

```pyhton
# 获取每条记录的每个字段values('字段1','字段2',...)返回值是字典，
# 如果没有指定字段的话那么就会获取全部字段(字典)

stu_dic = stus.values()
print(stu_dic)
```

11.查询主键等于特定的值

```python
# 查询id等于某个值,pk表示主键，也是id
stus = Student.objects.filter(pk=2)
```

12.或(|),且(&),非(~)条件查询 Q()

```python
from django.db.models import Q
#或查询 |
stus = Student.objects.filter(Q(s_age=20) | Q(s_gender=0))

#且条件查询 &
stus = Student.objects.filter(Q(s_age=20) & Q(s_gender=0))

#非条件查询 ~
stus = Student.objects.filter(~Q(s_age=20))
```

13.大于等于和小于等于F()

```python
#查询语文成绩比数学成绩大
stus = Student.objects.filter(chinese__gt=F('math'))

#查询语文成绩比数学成绩大10分
stus = Student.objects.filter(chinese__gt=F('math')+10)

#查询语文成绩比数学成绩小10分
stus = Student.objects.filter(chinese__lt=F('math')-10)
```

### 3.Django的orm框架删除和修改

1.删除

```python
def del_stu(request):
	Student.objects.filter(s_name='张三').delete()
	return HttpResponse('删除成功')
```

2.修改(更新数据)

```python

def update_stu(request):

	#修改 第一种
	 stu = Student.objects.filter(s_name='李四').first()
	 stu.s_name = '李思'
	 stu.save()

	#修改 第二种
	 Student.objects.filter(s_name='李思').update('栗子')
	 return HttpResponse('修改成功')
```

### 4.Django中的Model(模型之间的关系)(MVT中的M)

用例子说明这个Model的作用，Model中的每一个类其实就是数据库对应的每一张表，通过面向对象的思想，操作对象就可以操作数据库表中的对应的数据，数据库表中的每一条记录在查询之后会转换成相应的Model中的类的对象，这儿用学生，课程，班级和学生补充信息的例子来说明

```python
from django.db import models

# Create your models here.

class Grade(models.Model):
	g_name = models.CharField(max_length=10,unique=True)
	class Meta:
		db_table = 'grade'

class Student(models.Model):
	# 定义字符串类型max_length设置最大长度
	s_name = models.CharField(max_length=6, unique=True)
	# 定义s_age字段, int类型
	s_age = models.IntegerField(default=18)
	#定义性别，默认值是true(男)
	s_gender = models.BooleanField(default=True)
	#定义create_time字段，创建时间,自动添加(插入数据的时间)当前时间
	#auto_now_add:表示添加数据的时间
	create_time = models.DateTimeField(auto_now_add=True,null=True)

	#定义operate_time字段，修改时间,修改数据的时间
	#auto_now:表示修改数据的时间,null=True默认是True
	operate_time = models.DateTimeField(auto_now=True,null=True)
	#定义语文
	chinese = models.DecimalField(decimal_places=1,max_digits=4,null=True)
	#定义数学
	math = models.DecimalField(decimal_places=1,max_digits=4,null=True)

	#添加学生的外键(班级)
	grade = models.ForeignKey(Grade,null=True)



	class Meta:
		#定义模型迁移到数据库中时的表名
		db_table = 'student'

class StudentInfo(models.Model):
	phone = models.CharField(max_length=11, null=True)
	address = models.CharField(max_length=100)

	#OneToOneField指定一对一关联关系
	#stu在数据库中是stu_id,放的是数据库中学生表的id
	stu = models.OneToOneField(Student)

	class Meta:
		db_table = 'student_info'
```



#### 4.1一对一关系

一对一关系其实就是一张表的补充表，之所以不把这个补充表里的字段，放到一张表里，是为了给最开始那张表添加扩展性，之后的如果要对原始表添加或者修改字段，就不用修改原始表，给补充表添加字段就可以了。

一对一关系，在Django中用models.OneToOneField(<关联的类>),一对一的说明再在两张表中都可以声明，但是一般声明再补充表中:

通过学生表获取学生信息补充表:
格式:
	原始表对象.补充表类小写

```python
def sel_info_by_stu(request):
	#方式一
	#先获取stu对象
    stu = Student.objects.filter(s_name='张三').first()
    info = stu.studentinfo
   
    # 方式二
    info = StudentInfo.objects.filter(stu_id=stu.id)
    # 面向对象的方式
    info = StudentInfo.objects.filter(stu=stu)
```

通过学生信息补充表获取学生:

格式:

​	学生信息补充对象.关联字段

```python
def sel_stu_by_info(request):
    info = StudentInfo.objects.get(phone='110')
    stu = info.stu
    
```

#### 4.2 一对多关系

一对多的关系主要是要在多的那一边加上一个那边的外键，就比如说一个班级会有多个学生，在django设计model是需要在多的表使用ForeignKeyField(<一的表的对象>,null=True)

```python
grade = models.ForeignKey(Grade,null=True)
```

通过多的一方获取一的一方的格式:

​	多的一方的对象.字段

通过一个一方获取多的一方的格式:

​	一的对象,多的类(xiaoxie)_set()   -- 这种方式获取的是关系，如果还要获取具体的对象，需要对这个关系进行filter()或者all()

通过学生(多)获取班级(一):

```python
def sel_stu_grade(request):
	if request.method == 'GET':
		# 通过学生查找班级
		stu = Student.objects.filter(s_name='张三').first()
		grade = stu.grade
```

通过班级(一)获取学生(多):

```python
def sel_garde_stu(request):
    # 通过班级查找学生   一的对象.多的模型小写__set() 获取班级的学生关系
	# 查询出来的关系可以通过filter等方法经过筛选
	grade = Grade.objects.filter(g_name='软件工程2班').first()
	stus = grade.student_set.all()
	for stu in stus:
		print(stu.s_name)
```



#### 4.3 多对多关系

### 5.Django中的view(MVT中的V)

Django中view是存在于每个应用程序的，而不是一个项目只有一个view,它位于应用程序目录下的views.py里，通过在这里创建方法，然后在项目的setting.py里面配置应用程序，以及在urls.py配置执行路径，就可以通过views.py里的方法操作Model，在把操作的结果返回template里的指定的html文件进行显示。

显示学生信息的例子:

views.py部分:

```python
def all_stu(request):
	#获取所有学生信息
	stus = Student.objects.all()
	#返回页面
	return render(request, 'stus.html', {'students':stus})
```

setting.py部分:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app', #这个是该应用程序的名字
]


TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        #配置项目模板路径
		#D:\\python学习\python第二阶段\项目名\templates
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
urls.py部分
```python
urlpatterns = [
	url(r'^all_stu/', views.all_stu),
]
```



### 6. Django的templates(MVT中的T)

templates是和应用程序同级的文件夹，里面存放的是html模板，用来显示通过view拿到model中的数据，templates里的获取从views传来的数据通过

```
{% 数据%}和{{}}
```

例子:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>学生信息</title>
</head>
<body>
<h2>学生信息</h2>
    <table style="border: 1px solid black">
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>性别</th>
            <th>操作</th>
        </tr>
        {% for stu in students %}
        <tr>
            <td>{{ stu.s_name }}</td>
            <td>{{ stu.s_age }}</td>
            {% if stu.s_gender %}
            <td>
                男
            </td>
            {% else %}
            <td>
                女
            </td>
            {% endif %}
            <td> <a href="/add_info/?stu_id={{ stu.id }}">添加扩展信息</a>
            </td>
            <td><a href="/add_rel_grade/?stu_id={{ stu.id }}">添加所属班级</a></td>

        <tr>
        {% endfor %}
    </table>

</body>
</html>
```



