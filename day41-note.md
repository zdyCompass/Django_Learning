#### 1. MVC模式

MVC 是一种使用 MVC（Model View Controller 模型-视图-控制器）设计创建 Web 应用程序的模式：

+ Model：(模型)是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。

+ View：(视图)是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的。

+ Controller：(数据库)是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

  ​

#### 2. Django中的MVT模式

+ Model：负责业务与数据库(ORM)的对象。
+ View：负责业务逻辑并适当调用Mode和Template。
+ Template：负责把页面渲染展示给用户。




#### 3. 安装虚拟环境

##### 3.1 下载virtualenv

pip install virtualenv

##### 3.2 创建虚拟环境

virtualenv --no-site-packages -p D:\Program Files\anaconda\python.exe [环境名称]

##### 3.3 参数说明

> -p 指定版本，如果不指定版本则默认使用环境变量中的python版本
>
> --no-site-packages 指定不需要之前的包
>
> pip list 查看安装的包
>
> pip freeze 查看版本
>
> python -m pip install --upgrade pip 升级pip命令

##### 3.4 虚拟环境操作命令

首先进入虚拟环境路径下的scripts目录： /env/[环境名称]/scripts/

> 激活环境：activate
>
> 退出环境：deactivate



#### 4. 创建Django项目

##### 4.1 创建项目

> django-admin startproject [工程名称]

##### 4.2 项目目录介绍

打开所创建的工程项目目录后，会看到如下文件：

![Django项目文件](https://github.com/zdyCompass/Django_Learning/blob/master/re/DjangoProject.jpg)

它们的作用如下所示：

>_\_int\_\_：指明该目录结构是一个python包，暂无内容，在后期会初始化一些工具会使用到。
>
>settings.py：Django项目的配置文件，其中定义了本项目的引用组件
>
>urls.py：项目的URL路由映射，实现客户端请求url由哪个模块进行响应。
>
>wsgi.py：定义WSGI接口信息，通常本文件生成后无需改动
>
>manage.py：是Django用于管理本项目的管理集工具，之后站点运行，数据库自动生成，数据表的修改等都是通过该文件完成。

##### 4.3 启动服务器运行项目

在Terminal中使用命令：

> python manage.py runserver  [IP]:[PORT] 8080	运行服务

或者创建一个Debug脚本：

![runDebug](https://github.com/zdyCompass/Django_Learning/blob/master/re/runDebug.jpg)

##### 4.4 进入Django管理

输入如下地址进入Django管理登录界面：

> http://127.0.0.1:8080/admin

会看到如下登录界面：

![Djangomanage](https://github.com/zdyCompass/Django_Learning/blob/master/re/Djangomanage.jpg)

但这个登录界面需要进行如下设置才能进入：

1. 安装数据库驱动

   > pip install pymysql


2. 在settings.py中修改DATABASES配置：

   ~~~python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.mysql',
           'NAME': 'dj6',
           'USER': 'root',
           'PASSWORD': '123456',
           'HOST': '127.0.0.1',
           'PORT': 3306,
       }
   }
   ~~~



3. 在__init\_\_.py文件中写入如下代码：

   ~~~python
   import pymysql

   pymysql.install_as_MySQLdb()
   ~~~

4. 在Mysql数据库下新建一个名字为’dj6‘的数据库

5. 使用如下命令映射模型到数据库中：

   > python manage.py migrate

6. 创建超级管理员：

   > python manage.py createsuperuser

   根据提示设置登录名和密码。做完以后就可以在数据库中看到如下表了：

   ![dj6_table](https://github.com/zdyCompass/Django_Learning/blob/master/re/dj6_table.jpg)

完成上述操作之后就可以用设置好的密码和用户名进入Django管理后台了。
