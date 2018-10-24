#### 1. 模型数据的删除

1.  在urls.py中创建删除路由：

   ~~~python
   urlpatterns = [
       url(r'del_stu/', views.del_stu),
   ]
   ~~~

2.  在views.py中创建响应函数：

   ~~~python
   def del_stu(request):
       # 实现删除
       Student.objects.filter(s_name='慕容复').delete()
       return HttpResponse('删除成功')
   ~~~


#### 2.  模型数据的更新

1. 在urls.py中创建更新路由：

   ~~~python
   urlpatterns = [
       url(r'update_stu/', views.update_stu)
   ]
   ~~~

2. 在views中创建响应函数：

   ~~~python
   def update_stu(request):
       # 实现更新，第一种方法：
       # stu = Student.objects.filter(s_name='乔峰').first()
       # stu.s_name = '乔老大'
       # stu.save()
       # 第二种方法：
       Student.objects.filter(s_name='云中鹤').update(s_name='鹤老二')
       return HttpResponse('修改成功')
   ~~~

#### 3. 在页面显示查询结果

1. 在项目下创建一个templates目录，并创建一个HTML文件(stus.html)：

   ![templates](E:\千锋上课\python课堂代码\第四阶段\day43\templates.jpg)

2. 在settings.py文件中，修改templates路径：

   ~~~python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           # 修改DIRS路径，其它保持不变
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
   ~~~

3. 在urls.py中添加查询路由all_stu：

   ~~~python
   urlpatterns = [
       url(r'all_stu/', views.all_stu),
   ]
   ~~~

4. 在views.py中添加响应函数：

   ```python
   def all_stu(request):
       # 获取所有学生信息
       stus = Student.objects.all()
       # 返回页面
       return render(request, 'stus.html', {'students':stus})
   ```

5. 打开之前创建的templates目录下的stus.html文件进行修改：

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       {% for stu in students %}
           <p>姓名：{{ stu.s_name }}, 年龄：{{ stu.s_age }}, 性别：{{ stu.s_gender }} <p>
       {% endfor %}
   </body>
   </html>
   ```

#### 4. 在页面中添加拓展信息

1. 修改上面stus.html文件，增加一个a标签作为添加链接：

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <table>
           <thead>
               <th>姓名</th>
               <th>年龄</th>
               <th>操作</th>
           </thead>
           <tbody>
               {% for stu in students %}
                   <tr>
                       <td>{{ stu.s_name }}</td>
                       <td>{{ stu.s_age }}</td>
                       <td><a href="">添加扩展信息</a></td>
                   </tr>
               {% endfor %}
           </tbody>
       </table>
   </body>
   </html>
   ```

2. 在urls中添加路由'add_info/'：

   ```python
   urlpatterns = [
       url(r'add_info/', views.add_info),
   ]
   ```

3. 在views.py中添加响应函数：

   ```python
   def add_info(request):
       return render(request, 'info.html')
   ```

4. 修改stus.html文件中a标签的herf值：

   ~~~html
   <a href="/add_info/">添加扩展信息</a>
   ~~~

5. 在templates目录下添加info.html文件：

   ~~~html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <form action="" method="post">
           电话号码：<input type="text" name="phone">
           地址：<input type="text" name="address">
           <input type="submit" value="提交">
       </form>
   </body>
   </html>
   ~~~

6. 修改stus.html文件中a标签的herf值：

   ```html
   <a href="/add_info/?stu_id={{ stu.id }}">添加扩展信息</a>
   ```

7. 修改views.py中的add_info响应函数：

   ```python
   def add_info(request):
       # method 获取请求HTTP方式
       if request.method == 'GET':
           return render(request, 'info.html')

       if request.method == 'POST':
           # 获取页面中提交的手机号码和地址，并保存
           phone = request.POST.get('phone')
           address = request.POST.get('address')
           stu_id = request.GET.get('stu_id')
           # 保存
           StudentsInfo.objects.create(
               phone=phone,
               address=address,
               stu_id=stu_id
           )
           return HttpResponse('创建扩展表信息成功')
   ```

#### 5. 通过页面查询信息

1. 创建查询路由，学生表查扩展表和扩展表查学生表两个路由：

   ```python
   urlpatterns = [
       url(r'sel_info_by_stu/', views.sel_info_by_stu),
       url(r'sel_stu_by_info/', views.sel_stu_by_info),
   ]
   ```

2. 添加响应函数：

   ```python
   def sel_info_by_stu(request):
       if request.method == 'GET':
           # 通过学生查询扩展表信息
           stu = Student.objects.get(s_name='王语嫣')
           # 第一种，获取关联表对象
           # info = StudentsInfo.objects.filter(stu_id=stu.id)
           # 同等于
           # info = StudentsInfo.objects.filter(stu=stu)
           # 第二种, 学生对象，关联的模型名的小写
           info = stu.studentsinfo
           return HttpResponse('通过学生查找扩展表信息')


   def sel_stu_by_info(request):
       if request.method == 'GET':
           info = StudentsInfo.objects.get(phone='13512345678')
           student = info.stu
           print(student.s_name)
           return HttpResponse('通过扩展表查找学生信息')
   ```

#### 6. 关联关系

##### 6.1 一对一关系

两表中选择任一表，通过`OneToOneField`，指定一对一的关联。例如：

~~~python
class TempleA:
    id = xxxx
    b = OneToOneField(TempleB)
    

class TempleB:
    id = xxxx
~~~

> 通过a对象，查找b对象：a.b
>
> 通过b对象，查找a对象：b.templeb （即类名的小写）

##### 6.2 一对多关系

两表选择"多"的表，通过``，指定一对多的关联。例如（TempleA为"多"的表）：

~~~python
class TempleA:
    id = xxxx
    b = OneToOneField(TempleB)
    

class TempleB:
    id = xxxx
~~~

> 通过a对象，查找b对象：a.b
>
> 通过b对象，查找a对象：b.templeb_set