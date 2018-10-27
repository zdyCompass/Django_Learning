#### 1. 页面跳转

1. 在app路径下的urls.py文件中添加跳转路由index_redirect：

   ~~~python
   urlpatterns = [
       url(r'^index_redirect/', views.index_redirect),
   ]
   ~~~

2. 可以将其他路由重命名，方便书写；app目录下的路由使用name重命名，工程目录下的路由使用namespace重命名。

   + app目录下的urls.py文件：

     ~~~python
     urlpatterns = [
         url(r'^index/', views.index, name='ind'),
         url(r'^all_stu/', views.all_stu, name='all'),
         url(r'^index_redirect/', views.index_redirect),
     ]
     ~~~

   + 工程目录下的urls.py文件

     ~~~python
     urlpatterns = [
         url(r'^admin/', admin.site.urls),
         # 127.0.0.1:8080/app/XXXX
         url(r'^app/', include('app.urls', namespace='dy')),
     ]
     ~~~

3. 在app目录下views.py文件中写入index_redirect响应函数：

   ```python
   def index_redirect(request):
       if request.method == 'GET':
           # 实现重定向到index方法上去
           # 第一种重定向，地址是硬编码
           # return HttpResponseRedirect('/app/all_stu/')
           # 第二种重定向，使用反向解析reverse('namespace:name')
           # reverse('dy:all') ====> '/app/all_stu/'
           return HttpResponseRedirect(reverse('dy:all'))
   ```

#### 2. 页面编辑

1. 在页面中创建编辑项，即添加编辑a标签：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       <table>
           <thead>
               <th>序号</th>
               <th>id</th>
               <th>姓名</th>
               <th>年龄</th>
               <th>创建时间</th>
               <th>编辑</th>
           </thead>
           <tbody>

               {{ content_h2 | safe }}

               {% for stu in stus %}
                   <tr>
                       <!--forloop.counter从1开始打印序号-->
                       <!--forloop.counter0从0开始打印序号-->
                       <!--forloop.revcounter0 倒序开始打印到0结束-->
                       <!--forloop.revcounter 倒序开始打印到1结束-->
                       <!--forloop.first 循环第一次打印True-->
                       <!--forloop.last 循环最后一次打印True-->
                       <td>{{ forloop.counter }}</td>
                       <td>{{ stu.id }}</td>
                       <td>
                           {# {% if forloop.counter == 1 %} #}
                           {% ifequal forloop.counter 1  %}
                               <em style="color:red;">{{ stu.s_name }}</em>
                           {% else %}
                               {{ stu.s_name }}
                           {% endifequal %}
                       </td>
                       <td>{{ stu.s_age | add:1 }}</td>
                       <td>{{ stu.create_time | date:'Y年m月s日 H:m:s' }}</td>
                       <td>
                           <!--<a href="/app/edit_stu/{{ stu.id }}/">编辑</a>-->
                           <a href="{% url 'dy:edit' stu.id %}"></a>
                       </td>
                   </tr>
               {% endfor %}
           </tbody>
       </table>
   {% endblock %}
   ```

2. 在templates目录下创建编辑页面edit.html：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       <form action="" method="post">
           <table>
               <thead>
                   <th>姓名</th>
                   <th>年龄</th>
               </thead>
               <tbody>
                   <tr>
                       <td>
                           <input type="text" name="s_name" value="{{ stu.s_name }}">
                       </td>
                       <td>
                           <input type="text" name="s_age" value="{{ stu.s_age }}">
                       </td>
                   </tr>
               </tbody>
           </table>
           <input type="submit" value="提交">
       </form>
   {% endblock %}
   ```

3. 在app目录下urls.py文件中添加编辑路由：

   ~~~python
   urlpatterns = [
       # 其中(\d+)正则表达式用来获取需要编辑对象的id，传入edit_stu响应函数
       url(r'^edit_stu/(\d+)/', views.edit_stu, name='edit'),
   ]
   ~~~

4. 在app目录下的views.py文件中添加edit_stu响应函数：

   ```python
   def edit_stu(request, id):
       if request.method == 'GET':
           stu = Student.objects.get(pk=id)
           return render(request, 'edit.html', {'stu': stu})

       if request.method == 'POST':
           # ss = request.POST
           # 获取姓名和年龄
           stu_n = request.POST.get('s_name')
           stu_a = request.POST.get('s_age')
           # 修改当前编辑的学生的信息
           Student.objects.filter(pk=id).update(s_name=stu_n, s_age=stu_a)
           return HttpResponseRedirect(reverse('dy:all'))
   ```

#### 3. 用户注册/登录功能实现

##### 3.1 用户注册功能的实现

1. 使用一个设置好的新项目工程，并创建名为user的app应用：

   > python manage.py startapp user

2. 在user目录下创建urls.py文件用来存放需要用到的链接，同时修改主目录下的urls.py文件，链接到user目录下的urls.py文件：

   主目录下的urls.py文件：

   ```python
   urlpatterns = [
       url(r'^admin/', admin.site.urls),
       url(r'^user/', include('user.urls', namespace='user')),
   ]
   ```

   user目录下的urls.py文件：

   ```python
   from django.conf.urls import url

   from user import views


   urlpatterns = [
       # 注册路由
       url(r'^register/', views.register, name='register'),
   ]
   ```

3. 在views.py文件中添加注册响应函数register：

   ```python
   from django.urls import reverse

   from django.http import HttpResponseRedirect
   from django.shortcuts import render
   from django.urls import reverse

   from user.models import User, UserToken

   def register(request):
       if request.method == 'GET':
           return render(request, 'register.html')

       if request.method == 'POST':
           # 用于创建用户
           # 1. 获取参数
           name = request.POST.get('name')
           password = request.POST.get('pw')
           password2 = request.POST.get('pw2')
           # 2.校验参数是否完整
           if not all([name, password, password2]):
               msg = '请填写完整的参数'
               return render(request, 'register.html', {'msg': msg})
           # 3.先判断数据库中是否存在该用户
           if User.objects.filter(name=name).first():
              msg = '该账号已注册，请去登录'
              return render(request, 'register.html', {'msg': msg})
           # 4.校验密码是否一致
           if password != password2:
               msg = '两次输入的密码不一致'
               return render(request, 'register.html', {'msg': msg})
           # 注册，成功后返回登录页面
           User.objects.create(name=name, password=password)
           return HttpResponseRedirect(reverse('user:login'))
   ```

4. 创建templates路径，修改settings.py文件下templates的路径：

   ```python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
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

5. 在templates路径下创建模板和注册html文件：

   父模板base.html：

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>
           {% block title %}
           {% endblock %}
       </title>
       {% block css %}
       {% endblock %}

       {% block js %}
       {% endblock %}
   </head>
   <body>
       {% block content %}
       {% endblock %}
   </body>
   </html>
   ```

   css和js存放模板base_main(继承自父模板):

   ```html
   {% extends 'base.html' %}

   {% block js %}
       　<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
   {% endblock %}
   ```

   注册页面register.html：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       <form action="" method="post">
           <table>
               姓名：<input type="text" name="name">
               密码：<input type="password" name="pw">
               确认密码：<input type="password" name="pw2">
               <input type="submit" value="提交">
           </table>
       </form>
   {% endblock %}
   ```

##### 3.2 用户登录功能的实现

1. 在user路径下的urls.py文件中添加登录路由：

   ```python
   urlpatterns = [
       # 登录路由
       url(r'^login/', views.login, name='login'),
   ]
   ```

2. 在views.py文件中添加登录响应函数login：

   ```python
   import random

   def login(request):
       if request.method == 'GET':
           return render(request, 'login.html')

       if request.method == 'POST':
           # 1. 获取参数
           name = request.POST.get('name')
           password = request.POST.get('pw')
           # 2. 数据完整性
           if not all([name, password]):
               msg = '请填写完整的登录信息'
               return render(request, 'login.html', {'msg': msg})
           # 3. 验证用户是否存在
           user = User.objects.filter(name=name).first()
           if not user:
               msg = '该用户没有注册，请先注册'
               return render(request, 'login.html', {'msg': msg})
           # 4. 校验密码
           if password != user.password:
               msg = '密码不正确'
               return render(request, 'login.html', {'msg': msg})
           # 重点 与响应
           # 请求：请求是从浏览器发送请求的时候，传递给后端的。
           # 响应：后端返回给浏览器的
           res = HttpResponseRedirect(reverse('user:index'))
           # set_cookie(key, vaule, max_age)
           token = ''
           s = '1234567890qwertyuiopasdfghjklzxcvbnm'
           for i in range(25):
               token += random.choice(s)
           res.set_cookie('token', token, max_age=6000)

           # 存token值
           user_token = UserToken.objects.filter(user=user).first()
           if not user_token:
               UserToken.objects.create(token=token, user=user)
           else:
               user_token.token = token
               user_token.save
           return res
   ```

3. 在templates目录下创建登录页面login.html：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       <form action="" method="post">
           <p>登录</p>
           用户名：<input type="text" name="name">
           密码：<input type="password" name="pw">
           <input type="submit" value="提交">
       </form>
   {% endblock %}
   ```

##### 3.3 首页和注销

1. 在user路径下的urls.py文件中添加首页和注销路由：

   ```python
   urlpatterns = [
       # 首页
       url(r'^index/', views.index, name='index'),
       # 注销
       url(r'^logout/', views.logout, name='logout'),
   ]
   ```

2. 创建登录需求的装饰器。新建名为utils的包，并在其目录下创建functions.py文件，写入登录验证的装饰器login_required：

   ```python
   from django.http import HttpResponseRedirect
   from django.urls import reverse

   from user.models import User,UserToken
   # 定义登录验证的装饰器
   # 装饰器(闭包)三个条件：
   # 1. 外层函数套内层函数
   # 2. 内层函数调用外层函数的参数
   # 3. 外层函数返回内层函数


   def login_required(func):

       def check_login(request):
           # func是被login_required装饰的函数
           token = request.COOKIES.get('token')
           if not token:
               # cookie中没有登录的标识符，跳转到登录页面
               return HttpResponseRedirect(reverse('user:login'))
           user_token = UserToken.objects.filter(token=token).first()
           if not user_token:
               # token标识符有误，跳转到登录页面
               return HttpResponseRedirect(reverse('user:login'))
           return func(request)

       return check_login
   ```

3. 在views.py文件中添加主页和注销的响应函数index、logout：

   ```python
   @login_required
   def index(request):
       if request.method == 'GET':
           # token = request.COOKIES.get('token')
           # # 查询标识符是否有效
           # user_token = UserToken.objects.filter(token=token).first()
           # if not user_token:
           #     # 查询不到信息，说明用户没有登录
           #     return HttpResponseRedirect(reverse('user:login'))
           return render(request, 'index.html')


   @login_required
   def logout(request):
       if request.method == 'GET':
           # 1. 删除浏览器cookie中的token参数
           res = HttpResponseRedirect(reverse('user:login'))
           res.delete_cookie('token')
           # 2. 删除UserToken中的数据
           token = request.COOKIES.get('token')
           UserToken.objects.filter(token=token).delete()
           return res
   ```

4. 在templates目录下创建登录页面login.html和注销a标签：

   ```python
   {% extends 'base_main.html' %}


   {% block content %}
       欢迎来到主页
       <a href="{% url 'user:logout' %}">注销</a>
   {% endblock %}
   ```

   ​