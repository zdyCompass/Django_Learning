#### 1. 中间件补充

##### 1.1 重定向太多问题

在做登录验证的时候，注册和登录页面是不需要进行验证的，因此利用中间件进行登录验证时要跳过这两个页面：

```python
class AuthMiddleware(MiddlewareMixin):

    def process_request(self, request):
        # 设置不需要做检查的url
        not_check = ['/user/login', '/user/register']
        path = request.path
        if path in not_check:
            # 不继续执行登录验证的代码，直接去执行是视图函数
            return None
        # 登录验证
        token = request.COOKIES.get('token')
        if not token:
            # cookie中没有登录的标识符，跳转到登录页面
            return HttpResponseRedirect(reverse('user:login'))
        user_token = UserToken.objects.filter(token=token).first()
        if not user_token:
            # token标识符有误，跳转到登录页面
            return HttpResponseRedirect(reverse('user:login'))
        # 继续执行对应的视图函数
        return None
```

#### 2. 图片上传/查看

1. 首先在modles.py文件中创建Article模型，并迁移到数据库：

   ```python
   class Article(models.Model):
       title = models.CharField(max_length=20)
       desc = models.CharField(max_length=150)
       # 注意图片使用的是ImageFiled类型
       img = models.ImageField(upload_to='article')
       create_time = models.DateTimeField(auto_now_add=True)

       class Meta:
           db_table = 'article'
   ```

2. 在项目目录下创建media文件夹用来存放上传的文件，同时在settings.py文件中添加media路径：

   ```python
   # 上传媒体文件路径
   MEDIA_URL = '/media/'
   MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
   ```

3. 将media路径添加到项目下的urls.py文件中：

   ```python
   from django.contrib.staticfiles.urls import static

   from day46.settings import MEDIA_URL, MEDIA_ROOT

   # 将media文件夹解析为静态文件夹
   # django在debug为True的情况下，就可以访问media文件夹下的内容
   urlpatterns += static(MEDIA_URL, document_root=MEDIA_ROOT)
   ```

4. 创建上传和展示文章的路由：

   ```python
   urlpatterns = [
       # 上传文章
       url(r'^add_article/', views.add_article, name='add_article'),
       # 查看文章
       url(r'^show_article/(\d+)', views.show_article, name='show_article'),
   ]
   ```

5. 在views.py文件中添加对应的响应函数：

   ```python
   from user.models import Article

   def add_article(request):
       if request.method == 'GET':
           return render(request, 'articles.html')

       if request.method == 'POST':
           # 获取数据
           # 注意图片的获取方式：FILES.get()
           img = request.FILES.get('img')
           title = request.POST.get('title')
           desc = request.POST.get('desc')
           # 创建文章
           Article.objects.create(img=img,
                                 title=title,
                                 desc=desc)
           return HttpResponse('创建图片成功')
   ```


   def show_article(request, id):
       if request.method == 'GET':
           article = Article.objects.get(pk=id)
           return render(request, 'show_articles.html', {'article': article})
   ```

6. 上传图片的HTML页面：

   ```html
   {% extends 'base_main.html' %}

   {% block title %}
       添加文章页面
   {% endblock %}

   {% block content %}
       <form action="" method="post" enctype="multipart/form-data">
           标题：<input type="text" name="title">
           <br>
           描述：<input type="text" name="desc">
           <br>
           图片：<input type="file" name="img">
           <br>
           <input type="submit" value="提交">
       </form>
   {% endblock %}
   ```

   注意：这里form标签中一定要加上`enctype="multipart/form-data"`，将文件以二进制的形式上传，这样可以实现多种类型的文件上传。

7. 展示图片的HTML页面：

   ```html
   {% extends 'base_main.html' %}

   {% block title %}
       展示文章
   {% endblock %}

   {% block content %}
       标题：{{ article.title }}
       <br>
       描述：{{ article.desc }}
       <br>
       图片：<img src="/media/{{ article.img }}" alt="">
       <br>
   {% endblock %}
   ```

#### 3. 分页

1. 主要用到的模块：

   > from django.core.paginator import Paginator

2. 在urls.py文件中添加分页路由：

   ```python
   urlpatterns = [
       # 分页查看
       url(r'^articles/', views.articles, name='articles'),
   ]
   ```

3. 在views.py文件中添加响应函数：

   ```python
   def articles(request):
       if request.method == 'GET':
           # 获取访问的页码，用GET方法
           page = request.GET.get('page', 1)
           # 查询所有文章对象，并进行分页
           articles = Article.objects.all()
           # 将所有文章进行分页，每页最多三条数据
           paginator = Paginator(articles, 3)
           # 获取哪一页的文章信息
           arts = paginator.page(page)
           return render(request, 'arts.html', {'arts': arts})
   ```

4. 在templates目录下添加对应的分页查看页面：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       {% for art in arts %}
           标题：{{ art.title }}
           <br>
           描述：{{ art.desc }}
           <br>
           图片：<img src="/media/{{ art.img }}" alt="">
           <br>
       {% endfor %}
   <br>

   <p>
       {% if arts.has_previous %}
           <a href="{% url 'user:articles' %}?page={{ arts.previous_page_number }}">上一页</a>
       {% endif %}
       {% for i in arts.paginator.page_range %}
           <a href="{% url 'user:articles' %}?page={{ i }}">{{ i }}</a>
       {% endfor %}
       {% if arts.has_next %}
           <a href="{% url 'user:articles' %}?page={{ arts.next_page_number }}">下一页</a>
       {% endif %}
   </p>
   {% endblock %}
   ```

#### 4. 配置日志

1. 在工程下新建logs文件夹用来存放日志。

2. 在settings.py文件中配置：

   ```python
   # 日志文件夹的路径
   LOG_PATH = os.path.join(BASE_DIR, 'logs')
   # 如果日志文件夹地址不存在，则自动创建
   if not LOG_PATH:
       os.mkdir(LOG_PATH)
   # 配置日志
   LOGGING = {
       # 必须是1
       'version': 1,
       # 该参数True 表示禁用日志
       'disable_existing_loggers': False,
       # 指定写入到日志文件中的日志格式
       'formatters':{
           # 格式一
           'default': {
               'format': '%(name)s %(asctime)s %(message)s'
           },
           # 格式二
       },
       'handlers': {
           'console': {
               'level': 'INFO',
               # 指定日志文件存在路径
               'filename': '%s/log.txt' % os.path.join(BASE_DIR, 'logs'),
               # 指定写日志的格式
               'formatter': 'default',
               # 当日志大于指定值时就自动做备份，这里是5M
               'class': 'logging.handlers.RotatingFileHandler',
               'maxBytes': 5 * 1024 * 1024,
           }
       },
       'loggers': {
           '': {
               'handlers': ['console'],
               'level': 'INFO',
           }
       }
   }
   ```

3. 定义日志处理的中间件，进行日志文件的输出处理：

   ```python
   import time
   import logging

   class LoggingMiddleware(MiddlewareMixin):
       def process_request(self, request):
           # 记录当前请求访问服务器的时间，请求参数，请求内容....
           request.init_time = time.time()
           request.init_body = request.body
           return None

       def process_response(self, request, response):
           try:
               # 记录返回响应的时间和访问服务器的时间的差，记录返回状态码...
               times = time.time() - request.init_time
               # 响应状态码
               code = response.status_code
               # 响应内容
               res_body = response.content
               # 请求内容
               req_body = request.init_body

               # 日志信息
               msg = '%s %s %s %s' % (times, code, res_body, req_body)
               # 写入日志
               logging.info(msg)
           except Exception as e:
               logging.critical('log error, Exception: %s' % e)
           return response
   ```