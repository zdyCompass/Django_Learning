#### 1. API接口的使用

​	API（Application Programming Interface,应用程序编程接口）是一些预先定义的函数，目的是提供应用程序与开发人员基于某软件或硬件得以访问一组例程的能力，而又无需访问源码，或理解内部工作机制的细节。

1. API定义规范

   > http://xxx.com/api/

2. HTTP的请求方式

   + GET：获取数据
   + POST：创建数据
   + PUT：用于修改数据（修改全部属性）
   + PATCH：用于修改数据（修改部分属性）
   + DELETE：删除数据
#### 2. Django中使用restful

##### 2.1 基本配置

1. 安装包

   > pip install djangorestframework==3.4.6
   >
   > pip install django-filter  # Filtering support 

2. 在settings.py文件中添加rest_framework

   ```python
   INSTALLED_APPS = [
       'back_management',
   ]
   ```

3. 在应用app中定义路由：

   ```python
   from rest_framework.routers import SimpleRouter

   from django.conf.urls import url

   from app import views

   # 获取路由对象
   router = SimpleRouter()

   # 127.0.0.1:8080/article/
   router.register('article', views.ArticleView)

   urlpatterns = [

   ]
   urlpatterns += router.urls
   ```

4. 在views.py文件中定义对应的views.ArticleView类，其中继承GenericViewSet类是为了获得`get_object` and `get_queryset` 方法，ListModelMixin可以实现所有数据的查看，DestroyModelMixin可以实现数据的删除，UpdateModelMixin可以实现数据的修改，RetrieveModelMixin可以实现单个数据的查看，CreateModelMixin可以实现创建数据：

   ```python
   class ArticleView(viewsets.GenericViewSet,
                     mixins.ListModelMixin,
                     mixins.DestroyModelMixin,
                     mixins.UpdateModelMixin,
                     mixins.RetrieveModelMixin,
                     mixins.CreateModelMixin,):

       # 查询数据
       # queryset = Article.objects.all()
       queryset = Article.objects.filter(is_delete=0)
       # 序列化
       serializer_class = ArticleSerializer
   ```

5. 定义序列化类

   在应用目录下新建serializers.py文件，序列化类需要继承ModelSerializer，使用ModelSerializer表明序列化整个Student模型，并且可以指定虚拟化哪些字段。

   ```python
   from rest_framework import serializers

   from app.models import Article


   class ArticleSerializer(serializers.ModelSerializer):

       class Meta:
           # 指定序列化的模型
           model = Article
           # 序列化字段
           fields = ['id', 'title', 'desc']
   ```

6. 通过以上的代码，可以通过以下的URL和HTTP请求方式，完成对资源Article的CRUD操作：

   CRUD对应的增删改查：

   > 增： POST http://127.0.0.1:8080/article
   >
   > 删： DELETE http://127.0.0.1:8080/article/1/
   >
   > 改： UPDATE http://127.0.0.1:8080/article/1/
   >
   > 查： GET http://127.0.0.1:8080/stu/article/1/

##### 2.2 方法的补充

1. 在views.py中定义的ArticleView类中可以实现方法的重写（以删除为例）：

   ```python
   # 重构删除功能，DestroyModelMixin，这里的删除并不是真正的删除
   def perform_destroy(self, instance):
       instance.is_delete = 1
       instance.save()
   ```

2. 在页面中实现数据的修改

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>文章列表页面</title>
       <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
       <script type="text/javascript">
           $.ajax({
               url:'/app/article/',
               type:'GET',
               dataType:'json',
               success:function(data){
                   var table_html = '<table><thead><th>id</th><th>title</th><th>操作</th></thaed><tbody>'
                   for(var i=0; i<data.length; i+=1){
                       table_html += '<tr><td>' + data[i].id
                       table_html += '</td><td>' + data[i].title
                       table_html += '</td><td><a onclick="add_article();">创建</a>'
                       table_html += '  <a onclick="delete_article(' + data[i].id + ');">删除</a>'
                       table_html += '</td></tr>'
                       data[i].title
                   }
                   table_html += '</tbody></table>'
                   $('.art_class').html(table_html)
               },
               error:function(data){
                   alert('请求失败')
               }
           })

           function delete_article(id){
               $.ajax({
                   url: '/app/article/' + id + '/',
                   type: 'DELETE',
                   dataType: 'json',
                   success: function(data){
                       alert('删除成功')
                   },
                   error: function(data){
                       alert('删除失败')
                   }
               })
           }

           function add_article(){
               var form_html = '<form action="">'
               form_html += "{% csrf_token %}"
               form_html += '标题：<input type="text" name="title">'
               form_html += '描述：<input type="text" name="desc">'
               form_html += '<input type="button" value="提交" onclick="add();">'
               form_html += '</form>'
               $('.add_class').html(form_html)
           }

           function add(){
               var title = $('input[name="title"]').val()
               var desc = $('input[name="desc"]').val()
               var csrf = $('input[name="csrfmiddlewaretoken"]').val()
               $.ajax({
                   url: '/app/article/',
                   type: 'POST',
                   data: {'title': title, 'desc': desc},
                   headers: {'X-CSRFToken': csrf},
                   dataType: 'json',
                   success: function(data){
                       alert('创建成功')
                   },
                   error: function(data){
                       alert('创建失败')
                   }
               })
           }
       </script>
   </head>
   <body>
       <div class="art_class"></div>
       <div class="add_class"></div>
   </body>
   </html>
   ```