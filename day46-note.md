#### 1. 表单验证

即通过form表单对注册/登录页面提交的信息进行验证(使用django框架提供的用户模型)。该笔记主要概述表单验证功能使用方法，其他基本设置请参考上一笔记。

##### 1.1 注册表单验证

1. 表单验证需要的模块：

   > from django import forms

2. 表单验证的代码：

   ```python
   class UserRegisterForm(forms.Form):
       # 注意这里定义的变量名要与网页中表单提交的变量名一致
       name = forms.CharField(max_length=10, min_length=2, required=True,
                              error_messages={'required': '注册姓名必须填写',
                                              'min_length': '账号长度不能短于两个字符',
                                              'max_length': '账号长度不能超过十个字符'})
       pw = forms.CharField(max_length=30, min_length=1, required=True,
                            error_messages={'required': '密码必须填写',
                                            'min_length': '密码长度不能短于一个字符',
                                            'max_length': '密码长度不能超过三十个字符'})
       pw2 = forms.CharField(required=True, error_messages={'required': '注册确认密码必须填写'})

       def clean(self):
           # 获取用户名，用户名检验该用户是否已经注册
           name = self.cleaned_data.get('name')
           # 校验用户是否注册
           user = User.objects.filter(username=name).first()
           if user:
               raise forms.ValidationError({'name':'该账号已经注册'})
           if self.cleaned_data.get('pw') != self.cleaned_data.get('pw2'):
               raise forms.ValidationError({'pw': '两次输入的密码不一致'})
           return self.cleaned_data
   ```

3. 注册的响应函数：

   ```python
   def register(request):
       if request.method == 'GET':
           return render(request, 'register.html')

       if request.method == 'POST':
           data = request.POST
           form = UserRegisterForm(data)

           if form.is_valid():
               # 验证通过
               # 注册账号，使用create_user可以对密码进行加密
               User.objects.create_user(username=form.cleaned_data.get('name'),
                                        password=form.cleaned_data.get('pw'))
               return HttpResponseRedirect(reverse('user:login'))
           else:
               return render(request, 'register.html', {'errors': form.errors})
   ```


##### 1.2 登录表单验证

1. 表单验证的代码：

   ```python
   class UserLoginForm(forms.Form):
       name = forms.CharField(required=True, error_messages={'required': '请填写登录名'})
       pw = forms.CharField(required=True, error_messages={'required': '请填写登录密码'})

       def clean(self):
           # 获取用户名，用户名检验该用户是否已经注册
           name = self.cleaned_data.get('name')
           password = self.cleaned_data.get('pw')
           # 校验用户是否注册
           user = User.objects.filter(username=name).first()
           if not user:
               raise forms.ValidationError({'name': '该用户没有注册，请先注册'})
           return self.cleaned_data
   ```

2. 登录响应函数的代码：

   ```python
   def login(request):
       if request.method == 'GET':
           return render(request, 'login.html')

       if request.method == 'POST':
           data = request.POST
           form = UserLoginForm(data)
           if form.is_valid():
               # 使用随机标识符，也叫签名token
               # authenticate可以对加密的密码进行解密
               info = auth.authenticate(username=form.cleaned_data.get('name'),
                                        password=form.cleaned_data.get('pw'))
               if info:
                   # 登录，想request.user属性赋值，赋值为登录系统的用户对象
                   # 1. 向页面的cookie中设置sessionid值
                   # 2. 向django_session表中设置对应的标识符
                   auth.login(request, info)
                   return HttpResponseRedirect(reverse('user:index'))
               else:
                   return render(request, 'login.html', {'msg': '密码错误'})
           else:
               return render(request, 'login.html', {'errors': form.errors})
   ```

##### 1.3 django框架自带的登录/登出装饰器函数

1. 函数模块：

   > from django.contrib.auth.decorators import login_required

2. 装饰主页和注销响应函数：

   ```python
   @login_required
   def index(request):
       if request.method == 'GET':
           return render(request, 'index.html')

   @login_required
   def logout(request):
       if request.method == 'GET':
           auth.logout(request)
           return HttpResponseRedirect(reverse('user:login'))
   ```




#### 2. 中间件

##### 2.1 中间件Middleware的介绍：

1. 中间件是一个轻量级的，底层的插件，可以介入Django的请求和响应的过程(面向切面编程)；
2. 中间件本质就是一个Django类。

注意：中间件是帮助我们在视图函数执行之前和执行之后都可以做一些额外的操作，它本质上就是一个自定义类，类中定义了几个方法，Django框架会在请求的特定的时间去执行这些方法。

##### 2.2 中间件图解

image

##### 2.3 中间件的定义

1. 在项目下新建utils路径，创建\_\_init__.py文件和存放中间件的middleware.py文件；

2. 中间件需要用到的模块：

   > from django.utils.deprecation import MiddlewareMixin

3. 中间件的创建

   ```python
   class TestMiddleware(MiddlewareMixin):

       def process_request(self, request):
           print('process_request1')
           # 继续执行对应的视图函数
           return None

       def process_response(self, request, response):
           print('process_response1')
           # 返回响应
           return response
   ```

4. 将创建好的中间件加入Django项目中，在settings.py中可以找到已经定义好的中间件，将自己定义的插入即可：

   ```python
   MIDDLEWARE = [
       'django.middleware.security.SecurityMiddleware',
       'django.contrib.sessions.middleware.SessionMiddleware',
       'django.middleware.common.CommonMiddleware',
       # 'django.middleware.csrf.CsrfViewMiddleware',
       'django.contrib.auth.middleware.AuthenticationMiddleware',
       'django.contrib.messages.middleware.MessageMiddleware',
       'django.middleware.clickjacking.XFrameOptionsMiddleware',
       'utils.middleware.TestMiddleware',
   ]
   ```

   ​