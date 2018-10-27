#### 1. 多对多关系

1. 创建课程与学生的多对多关系，在models.py文件中添加课程模型：

   ~~~python
   class Course(models.Model):
       c_name = models.CharField(max_length=10)
       # 多对多关联字段
       stu = models.ManyToManyField(Student)

       class Meta:
           db_table = 'course'
   ~~~

2. 修改templates目录下stus.html文件，通过a标签增加学生选课项：

   ~~~python
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
               <th>电话</th>
               <th>课程</th>
               <th>操作</th>
           </thead>
           <tbody>
               {% for stu in students %}
                   <tr>
                       <td>{{ stu.s_name }}</td>
                       <td>{{ stu.s_age }}</td>
                       <td>{{ stu.studentsinfo.phone }}</td>
                       <td>
                           {% for cou in stu.course_set.all %}
                               {{ cou.c_name }}
                           {% endfor %}
                       </td>
                       <td>
                           <a href="/add_info/?stu_id={{ stu.id }}">添加扩展信息</a>
                           |
                           <a href="/add_stu_course/?stu_id={{ stu.id }}">添加课程</a>
                       </td>
                   </tr>
               {% endfor %}
           </tbody>
       </table>
   </body>
   </html>
   ~~~

3. 在urls.py文件中添加创建课程路由

   ~~~python
   from django.conf.urls import url

   from app import views

   urlpatterns = [
       url(r'add_course/', views.add_course),
       url(r'add_stu_course/', views.add_stu_course),
   ]
   ~~~

4. 在views.py中向数据库中添加课程信息，并提交学生选课信息：

   ~~~python
   def add_course(request):
       # 添加课程
       if request.method == 'GET':
           names = ['高数', '英语', '政治', 'c语言', 'Python基础']
           for name in names:
               Course.objects.create(c_name=name)
       return HttpResponse('创建课程成功')
       

   def add_stu_course(request):

   ​```python
   if request.method == 'GET':
       cous = Course.objects.all()
       return render(request, 'course.html', {'cous':cous})

   if request.method == 'POST':
       # 获取课程id和学生id
       c_id = request.POST.get('cous_id')
       s_id = request.GET.get('stu_id')

       stu = Student.objects.get(pk=s_id)
       # 设置学生和课程的关联关系
       couse = Course.objects.get(pk=c_id)
       # stu.course_set.add(couse)
       couse.stu.add(stu)
       return HttpResponseRedirect('/all_stu/')
   ~~~

#### 2. 网页展示

1. 新建工程，做好基本配置。

2. 在项目下创建static目录，在static目录下再创建css和js目录用来存放css和js文件，并修改settings.py文件：

   ```python
   # Static files (CSS, JavaScript, Images)
   # https://docs.djangoproject.com/en/1.11/howto/static-files/

   # 定义静态的路由
   STATIC_URL = '/static/'
   # 指定静态目录static的地址
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, 'static')
   ]
   ```

3. 创建templates目录，并创建base.html、base_main和index.html文件，base文件作为base_main文件的父模板，base_main用来存放css和js，并作为index的父模板：

   对于base.html：

   ~~~html
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
       <!--父模板-->
       {% block content %}
       {% endblock %}
   </body>
   </html>
   ~~~

   对于base_main.html：

   ~~~html
   {% extends 'base.html' %}

   {% block js %}
       　<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
   {% endblock %}
   ~~~

   对于index.html：

   ~~~html
   <!--引用父类模板-->
   {% extends 'base_main.html' %}

   <!--填补父类模板的对应的block-->
   {% block title %}
       我是首页
   {% endblock %}

   {% block css %}
       <!--加载静态文件CSS的两种方式-->
       <!--<link rel="stylesheet" href="/static/css/index.css">-->
       {% load static %}
       <link rel="stylesheet" href="{% static 'css/index.css' %}">
   {% endblock %}

   {% block js %}
       <!--引入父模板的block js中定义的内容-->
       {# 第一种注解，单行注解 #}
       {% comment %}
           第二种注解，多行注解
           <p>好厉害</p>
       {% endcomment %}
       {# super,继承父类 #}
       {{ block.super }}
   {% endblock %}

   {% block content %}
       <!--<p style="color:red;">千峰lalala</p>-->
       <p>千峰lalala</p>
   {% endblock %}
   ~~~


#### 3. forloop、ifequal方法的实现

1. 在app目录下的urls.py文件中创建一个显示所有学生信息的路由：

   ```python
   urlpatterns = [
       url(r'^all_stu/', views.all_stu, name='all'),
   ]
   ```

2. 在views.py文件中创建相应的响应函数all_stu：

   ```python
   def all_stu(request):
       if request.method == "GET":
           # 获取所有学生信息
           stus = Student.objects.all()
           content_h2 = '<h2>今天天气真好</h2>'
           return render(request, 'stus.html', {'stus': stus, 'content_h2': content_h2})
   ```

3. 在templates目录下创建stus.html，用来展示所有学生信息。包含知识点（）：

   ```html
   {% extends 'base_main.html' %}

   {% block content %}
       <table>
           <thead>
               <th>序号</th>
               <th>id</th>
               <th>姓名</th>
               <th>年龄</th>
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
                   </tr>
               {% endfor %}
           </tbody>
       </table>
   {% endblock %}
   ```

#### 4. 过滤器

使用方法：{{ var | 过滤器 }}；作用：在变量显示前做修改。

1. 加法

   > {{ stu.s_age| add:1 }}
   >
   > 过滤器没有减法，但是可以通过加负数实现

2. 修改大小

   > lower / upper : {{ stu.s_name | lower }}

3. 定制日期格式

   > {{ stu.create_time | date:'Y年m月s日 H : m : s' }} 
   >
   > {{ stu.create_time | date:'Y-m-s  H : m : s' }} 

   如果过滤器定义为小写的y，则返回的数据是两位的年，如果写的是大写的Y，则返回的是四位的年。

   定义小写的m，返回数字的月份，如果是大写的M，则返回英文。

   定义小写的h，返回12小时制度的时，如果是大写的H，返回24小时制度的时。

4. 是否转义

   HTML转义: 将接收到的数据当成普通字符串处理还是当成HTML代码来渲染的一个问题 渲染成html：

   > {{ code | safe }} 

5. 其它过滤器

   ```
   1、add ：将value的值增加2。使用形式为：{{ value | add: "2"}}。
   2、addslashes：在value中的引号前增加反斜线。使用形式为：{{ value | addslashes }}。
   3、capfirst：value的第一个字符转化成大写形式。使用形式为：{{ value | capfirst }}。
   4、cut：从给定value中删除所有arg的值。使用形式为：{{ value | cut:arg}}。
   5、date: 格式化时间格式。使用形式为：{{ value | date:"Y-m-d H:M:S" }}
   6、default：如果value是False，那么输出使用缺省值。使用形式：{{ value | default: "nothing" }}。例如，如果value是“”，那么输出将是nothing
   7、default_if_none：如果value是None，那么输出将使用缺省值。使用形式：{{ value | default_if_none:"nothing" }}，例如，如果value是None，那么输出将是nothing
   8、dictsort：如果value的值是一个字典，那么返回值是按照关键字排序的结果
   使用形式：{{ value | dictsort:"name"}}，例如，
   如果value是：
   [{'name': 'python'},{'name': 'java'},{'name': 'c++'},]
   那么，输出是：
   [{'name': 'c++'},{'name': 'java'},{'name': 'python'}, ]
   9、dictsortreversed：如果value的值是一个字典，那么返回值是按照关键字排序的结果的反序。使用形式：与dictsort过滤器相同。
   10、divisibleby：如果value能够被arg整除，那么返回值将是True。使用形式：{{ value | divisibleby:arg}}，如果value是9，arg是3，那么输出将是True
   11、escape：替换value中的某些字符，以适应HTML格式。使用形式：{{ value | escape}}。例如，< 转化为 &lt;> 转化为 &gt;' 转化为  &#39;" 转化为  &quot;
   13、filesizeformat：格式化value，使其成为易读的文件大小。使用形式：{{ value | filesizeformat }}。例如：13KB，4.1MB等。
   14、first：返回列表/字符串中的第一个元素。使用形式：{{ value | first }}
   16、iriencode：如果value中有非ASCII字符，那么将其进行转化成URL中适合的编码，如果value已经进行过URLENCODE，改操作就不会再起作用。使用形式：{{value | iriencode}}
   17、join：使用指定的字符串连接一个list，作用如同python的str.join(list)。使用形式：{{ value | join:"arg"}}，如果value是['a','b','c']，arg是'//'那么输出是a//b//c
   18、last：返回列表/字符串中的最后一个元素。使用形式：{{ value | last }}
   19、length：返回value的长度。使用形式：{{ value | length }}
   20、length_is：如果value的长度等于arg的时候返回True。使用形式：{{ value | length_is:"arg"}}。例如：如果value是['a','b','c']，arg是3，那么返回True
   21、linebreaks：value中的"\n"将被<br/>替代，并且整个value使用</p>包围起来。使用形式：{{value|linebreaks}}
   22、linebreaksbr：value中的"\n"将被<br/>替代。使用形式：{{value |linebreaksbr}}
   23、linenumbers：显示的文本，带有行数。使用形式：{{value | linenumbers}}
   24、ljust：在一个给定宽度的字段中，左对齐显示value。使用形式：{{value | ljust}}
   25、center：在一个给定宽度的字段中，中心对齐显示value。使用形式：{{value | center}}
   26、rjust：：在一个给定宽度的字段中，右对齐显示value。使用形式：{{value | rjust}}
   27、lower：将一个字符串转换成小写形式。使用形式：{{value | lower}}
   30、random：从给定的list中返回一个任意的Item。使用形式：{{value | random}}
   31、removetags：删除value中tag1,tag2....的标签。使用形式：{{value | removetags:"tag1 tag2 tag3..."}}
   32、safe：当系统设置autoescaping打开的时候，该过滤器使得输出不进行escape转换。使用形式：{{value | safe}}
   33、safeseq：与safe基本相同，但有一点不同的就是：safe是针对字符串，而safeseq是针对多个字符串组成的sequence
   34、slice：与python语法中的slice相同。使用形式：{{some_list | slice:"2"}}
   37、striptags：删除value中的所有HTML标签.使用形式：{{value | striptags}}
   38、time：格式化时间输出。使用形式：{{value | time:"H:i"}}或者{{value | time}}
   39、title：转换一个字符串成为title格式。
   40、truncatewords：将value切成truncatewords指定的单词数目。使用形式：{{value | truncatewords:2}}。例如，如果value是Joel is a slug 那么输出将是：Joel is ...
   42、upper：转换一个字符串为大写形式
   43、urlencode：将一个字符串进行URLEncode
   46、wordcount：返回字符串中单词的数目
   ```

   ​

   ​

