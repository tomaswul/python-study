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
```
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
```
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

```
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

```
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
2.filte()方法:过滤查询部分的信息

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

```
#排序 order_by()
	# 升序
	stus = Student.objects.order_by('id')

	# 降序
	stus = Student.objects.order_by('-id')
```

8.查询不满足条件的记录exclude(字段=值)

```
# 查询不满足条件的数据exclude()
stus = Student.objects.exclude(s_age=18) #查询年纪不等于18的
```

9.统计查询个数count()

```
# 统计查询个数count()和len()等效
stus_count = stus.count()
print(stus_count)
```

10.获取每条记录的每个字段value()返回值是字典

```
# 获取每条记录的每个字段values('字段1','字段2',...)返回值是字典，
# 如果没有指定字段的话那么就会获取全部字段(字典)

stu_dic = stus.values()
print(stu_dic)
```

11.查询主键等于特定的值

```
# 查询id等于某个值,pk表示主键，也是id
stus = Student.objects.filter(pk=2)
```
