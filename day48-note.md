#### 1. RBAC权限控制

**在Django中会自己动创建下列表：**

+ auth_group 组(角色)
+ auth_group_permissions 组权限
+ auth_permission 控制权限
+ tb_users_groups 用户所属组
+ tb_users 用户
+ tb_users_user_permissions 用户额外权限

**权限控制思想：**

1. 创建角色（组）
2. 角色对应权限
3. 用户分配角色
4. （特殊情况）用户分配权限

**表与表之间的关系：**

用户表和权限表的ManyToManyFiled()为：user_permission

用户表和组表的ManyToManyFiled()为：groups

组表和权限表的ManyToManyFiled()为：permissions

**添加与删除：**

+ add()
+ remove()

##### 1.1 自定义权限

1. 在models.py文件中添加用户模型继承自Django自带的用户类AbstractUser，在Meta元中添加permissions参数，自定义权限的名称（‘codename’，‘name’）即codename为权限名，name为权限的描述。在数据库的auth_permission表中还有一个content_type字段，其表示prmission属于哪个model：

   ```python
   from django.contrib.auth.models import AbstractUser

   class MyUser(AbstractUser):
       # 扩展django自带的auth_user表，可以自定义新增的字段

       class Meta:
           # django默认给每个模型初始化三个权限
           # 默认的change、delete、add权限
           permissions = (
               ('add_my_user', '新增用户权限'),
               ('change_my_user_username', '修改用户名权限'),
               ('change_my_user_password', '修改用户密码权限'),
               ('all_my_user', '查看用户权限'),
           )
   ```

   并在settings.py文件中添加模型路径`AUTH_USER_MODEL = 'app名.模型名'` ，并进行数据迁移:

   ```python
   # 告诉django，User模型修改为自定义的User模型
   AUTH_USER_MODEL = 'permission.MyUser'
   ```

   注意：模型迁移后会在数据库的auth_permission表中，会新增权限，包括自带的对Users管理的权限，和自定义的四个权限。如下所示：

   ![permissions](E:\千锋上课\python课堂代码\第四阶段\day48\permissions.png)

2. 创建用户并给定权限

   + 语法:
     + 添加权限：user对象.user_permission.add(permission对象1, permission对象2)
     + 删除权限：user对象.user_permission.remove(permission对象1, permission对象2)
     + 清空权限：user对象.user_permission.clear()

   ```python
   # 创建用户，并给用户分配权限
   url(r'^add_user_permission/', views.add_user_permission, name='add_user_permission'),
   ```

   ```python
   from django.contrib.auth.models import Permission
   from django.http import HttpResponse

   from permission.models import MyUser

   def add_user_permission(request):
       if request.method == 'GET':
           # 1. 创建用户
           user = MyUser.objects.create_user(username='zdy',
                                             password='123123',)
           # 2. 指定刚刚创建的用户，并分配给它权限（新增用户权限，查看用户权限）
           permissions = Permission.objects.filter(codename__in=['add_my_user', 'all_my_user']).all()
           for permission in permissions:
               # 多对多的添加
               user.user_permissions.add(permission)
           # 3. 删除刚刚创建的用户的新增用户权限
           # user.user_permissions.remove(权限对象)
           return HttpResponse('创建用户权限成功')
   ```

3. 使用装饰器进行权限过滤

   ```python
   from django.http import HttpResponseRedirect, HttpResponse

   from permission.views import MyUser

   # 定义登录验证的装饰器
   # 装饰器(闭包)三个条件：
   # 1. 外层函数套内层函数
   # 2. 内层函数调用外层函数的参数
   # 3. 外层函数返回内层函数
   def check_permissions(func):

       def check(request):
           # zdy用户查看用户列表权限，才能访问index函数。
           user = MyUser.objects.filter(username='zdy').first()
           u_p = user.user_permissions.filter(codename='all_my_user').first()
           if u_p:
               return func(request)
           else:
               return HttpResponse('用户没有权限')
       return check
   ```

   在创建访问路由并添加响应函数：

   ```python
   url(r'^index/', views.index, name='index'),
   ```

   ```python
   from utils.functions import check_permissions

   @check_permissions
   def index(request):
       # zdy用户有查看用户列表权限，才能访问index函数。使用装饰器去写
       if request.method == 'GET':
           return render(request, 'index.html')
   ```

4. 创建组并分配权限：

   + 给组添加权限语法:
     + 添加权限：group对象.permissions.add(permission对象1, permission对象2)
     + 删除权限：group对象.permissions.remove(permission对象1, permission对象2)
     + 清空权限：group对象.permissions.clear()
   + 给用户添加权限语法:
     + 添加权限：user对象.groups.add(groups对象1, groups对象2)
     + 删除权限：user对象.groups.remove(groups对象1, groups对象2)
     + 清空权限：user对象.groups.clear()

   ```python
   # 创建组，分配组的权限
   url(r'^add_group_permission/', views.add_group_permission, name='add_group_permission'),
   # 给人分配组
   url(r'^add_user_group/', views.add_user_group, name='add_user_group'),
   ```

   ```python
   def add_group_permission(request):
       if request.method == 'GET':
           # 创建超级管理员(所有权限)、创建普通管理员(修改/查找权限)
           group = Group.objects.create(name='审核组')

           ps = Permission.objects.filter(codename__in=['change_my_user_username',
                                                        'change_my_user_password',
                                                        'all_my_user'])
           for permission in ps:
               group.permissions.add(permission)
           return HttpResponse('创建组权限成功')
       

   def add_user_group(request):
       if request.method == 'GET':
           user = MyUser.objects.get(username='zdy')
           group = Group.objects.get(name='审核组')

           # 分配租
           user.groups.add(group)

           return HttpResponse('用户分配组成功')
   ```

5. 检测用户是否有某权限,和所有权限，组权限

   + 语法：用户对象.has_perm('模型名.权限codename')
     + 查询用户所有的权限：user.get_all_permissions()方法列出用户的所有权限，返回值是permission name
     + 查询用户的组权限：user.get_group_permissions()方法列出用户所属group的权限，返回值是permission name

   ```python
   # 查看用户权限
   url(r'^show_user_permission/', views.show_user_permission, name='show_user_permission'),
   ```

   ```python
   def show_user_permission(request):
       if request.method == 'GET':
           user = MyUser.objects.get(username='zdy')
           return render(request, 'permission.html', {'user': user})
   ```

   对应的permission.html代码：

   ```python
   {% block content %}
       <!--通过用户查询组，组查询权限-->
       {% for permission in user.groups.all.0.permissions.all %}
           {{ permission.codename }}
           <br>
       {% endfor %}
       <!--通过用户直接查找权限-->
       {% for permission in user.user_permissions.all %}
           {{ permission }}
           <br>
       {% endfor %}
   {% endblock %}
   ```

##### 1.2 Django自带的权限

1. 表单验证：

   ```python
   from django import forms
   from permission.models import MyUser


   class UserLoginForm(forms.Form):
       username = forms.CharField(max_length=10,min_length=2, required=True,
                                  error_messages={'required': '注册姓名必须填写', })
       password = forms.CharField(max_length=30, min_length=1, required=True,
                            error_messages={'required': '密码必须填写', })

       def clean(self):
           # 获取用户名，用户名检验该用户是否已经注册
           name = self.cleaned_data.get('username')
           password = self.cleaned_data.get('password')
           # 校验用户是否注册
           user = MyUser.objects.filter(username=name).first()
           if not user:
               raise forms.ValidationError({'username': '该用户没有注册，请先注册'})
           return self.cleaned_data
   ```

2. Django自带的登录功能

   ```python
   # 登录
   url(r'^login/', views.login, name='login'),
   ```

   ```python
   def login(request):
       if request.method == 'GET':
           return render(request, 'login.html')

       if request.method == 'POST':
           form = UserLoginForm(request.POST)

           if form.is_valid():
               # 验证通过
               # 检验账号
               user = auth.authenticate(username=form.cleaned_data['username'],
                                        password=form.cleaned_data['password'])
               if user:
                   # request.user赋值，赋值为用户登录对象
                   auth.login(request, user)
                   return HttpResponseRedirect(reverse('permission:my_index'))
               else:
                   return render(request, 'login.html')
           else:
               return render(request, 'login.html', {'errors': form.errors})
   ```

3. Django自带的权限查看：

   ```python
   url(r'^my_index/', views.my_index, name='my_index'),
   url(r'^new_index/', views.new_index, name='new_index'),
   ```

   ```python
   def my_index(request):
       if request.method == 'GET':
           # 当前登录系统用户
           user = request.user
           # 获取组权限
           user.get_group_permissions()
           # 获取当前用户的所有权限
           user.get_all_permissions()
           # 判断是否有某个权限
           # user.has_perm('应用app名.权限名')

           return render(request, 'my_index.html')
   ```

   ```python
   from django.contrib.auth.decorators import permission_required

   @permission_required('permission.all_my_user')
   def new_index(request):
       if request.method == 'GET':
           return HttpResponse('需要权限才能查看')
   ```

   ​

