#### 1. 创建app应用

在Django中的app就是一个功能模块, 与其他的web框架可能有很大的区别, 将不同功能放在不同的app中, 方便代码的复用。在Terminal中执行以下命令：

> python manage.py startapp [应用名]app

[应用名]可以自定义，这里我就把它叫做app，执行后会在工程下创建一个名为app的文件夹。

![appfiles](https://github.com/zdyCompass/Django_Learning/blob/master/re/appfiles.jpg)

各py文件作用如下：

+ migrations:  用于在之后定义引用迁移功能。
+ admin.py：管理站点模型的声明文件，默认为空。
+ apps.py：应用信息定义文件。在其中生成了类Appconfig，类用于定义应用名等Meta数据。
+ models.py:  添加模型层数据类的文件。
+ test.py：测试代码文件。
+ views.py：定义URL响应函数。

#### 2.创建一个hello页面

1. 首先新建一个路由，在urls.py中写入‘hello’路由：

   ~~~python
   from app import views

   urlpatterns = [
       # 127.0.0.1:8080/admin/
       url(r'^admin/', admin.site.urls),
       # 127.0.0.1:8080/hello/
       url(r'^hello/', views.hello),
   ]
   ~~~


2. 然后在app/views.py中导入HttpResponse模块，并写出对应的hello函数：

   ~~~python
   from django.http import HttpResponse


   def hello(request):
          return HttpResponse('你好，百度')
   ~~~

3. 在Terminal中输入命令`python manage.py runserver` ，或运行已创建好的脚本打开服务。在浏览器中输入 `http://127.0.0.1:8080/hello/`可以看到网页返回的‘你好，百度’语句。


#### 3. 创建模型

##### 3.1 模型类到数据库的实现

1. 在创建好的app目录下打开models.py，定义一个Student类：

   ~~~python
   from django.db import models


      class Student(models.Model):
      # 定义s_name字段，varchar类型，最长不超过6字符，唯一
      s_name = models.CharField(max_length=6, unique=True)
      # 定义s_age字段，ini类型
      s_age = models.IntegerField(default=18)
      # 定义s_gender字段，int类型
      s_gender = models.BooleanField(default=1)
      # 定义create_time字段，创建时间
      create_time = models.DateTimeField(auto_now_add=True, null=True)
      # 定义operate_time字段，修改时间
      operate_time = models.DateTimeField(auto_now=True, null=True)

      class Meta:
          # 定义模型迁移到数据库中的表名
          db_table = 'student'
   ~~~



2. 在settings.py中的应用列表中，加入刚刚创建好的app

   ~~~python
   # Application definition

   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'app',
   ]
   ~~~



3. 在Terminal中输入命令`python manage.py makemigrations`,该命令会在app/migrations/目录下生成一个迁移文件。然后输入`python manage.py migrate`命令，将模型映射到数据库dj6中，这样就可以在数据库dj6中看到多了一张名字为student的表。

   > 注：每次在修改类的字段后，都需要通过上面两个命令将数据迁移到数据库。

##### 3.2 向数据库中插入数据

1. 在urls.py文件中插入路由create_stu：

~~~python
urlpatterns = [
    # 127.0.0.1:8080/admin/
    url(r'^admin/', admin.site.urls),
    # 127.0.0.1:8080/hello/
    url(r'^hello/', views.hello),
    url(r'create_stu', views.create_stu),
]
~~~

2. 在views.py文件中导入Student模块，并创建学生：

~~~python
from app.models import Student


def create_stu(request):
    # 创建学生
    # 方法一：
    '''
    stu = Student()
    stu.s_name = '乔峰'
    stu.s_age = 20
    stu.save()
    ''''
    # 方法二：
    Student.objects.create(
        s_name='虚竹',
    )
    return HttpResponse('创建成功')
~~~

3. 在Terminal中输入命令`python manage.py runserver` ，或运行已创建好的脚本打开服务。在浏览器中输入 `http://127.0.0.1:8080/create_stu/`就可以以看到dj6数据库的student表中多了一行刚刚添加的数据。

#### 4. 创建查询 

1. 在urls.py文件中插入路由sel_stu

   ~~~python
   urlpatterns = [
       # 127.0.0.1:8080/admin/
       url(r'^admin/', admin.site.urls),
       # 127.0.0.1:8080/hello/
       url(r'^hello/', views.hello),
       url(r'create_stu', views.create_stu),
       url(r'sel_stu', views.sel_stu),
   ]
   ~~~

2. 在views.py文件中写入查询函数

   ~~~python
   def sel_stu(request):
       # 实现查询
       # all()查询所有对象信息
       # stus = Student.objects.all()

       # 模糊查询 like
       stus = Student.objects.filter(s_name__contains='灯')
       stu_names = [ob.s_name for ob in stus]
       print(stu_names)
       return HttpResponse('查询成功')
   ~~~

3. 其他查询命令

   + filter()：过滤不符合查询条件的对象，可以做多条件查询

     >stus =Student.objects.**filter**(s_name='虚竹')
     >
     >或	stus = Student.objects.**filter**(s_age=20).**filter**(s_gender=0)
     >
     >或	stus = Student.objects.**filter**(s_age=20, s_gender=0)

   + first()和last()：获取符合查询条件的第一个和最后一个对象

     > stus =Student.objects.filter(s_age=20).**first()**
     >
     > stus =Student.objects.filter(s_age=20).**last()**

   + get()：获取符合条件的对象，但要求符合条件的对象有且只有一个，否则会报错

     > stus = Student.objects.**get**(s_age=20)

   + __contains：模糊查询，like '%X%'

     > stus = Student.objects.filter(s_name**__contains**='灯')

   + startswith/endswith：模糊查询，like 'X%' / like '%X'

     > stus = Student.objects.filter(s_name__**startswith**='段')
     >
     > stus = Student.objects.filter(s_name__**endswith**=''嫣)

   + gt / gte / lt / lte：大于 / 大于等于 / 小于 / 小于等于

     > stus = Student.objects.filter(s_age**__gt**=18)

   + order_by()：排序，升序和逆序的区别在于字段前面的负号

     > 升序    stus = Student.objects.**order_by**('s_age')
     >
     > 降序    stus = Student.objects.**order_by**('**-**s_age')

   + exclude()：查询不满足条件的数据

     > stus = Student.objects.**exclude**(s_age=21)

   + count()：统计查询到的对象个数，与len()类似

     > stu_count = stus.**count**()
     >
     > 或    stu_count = **len**(stus)

     ​

   ​
