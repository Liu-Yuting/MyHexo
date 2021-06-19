---
title: Django 基础
date: 2020-11-23 12:03:21
tags: Django
---

##  一、Web框架本质----socket：

### 1、web框架本质

 　1. 对于所有的Web应用，本质上其实就是一个socket服务端，用户的浏览器其实就是一个socket客户端。

 　2. 真实web框架一般会分为两部分：服务器程序和应用程序。

　　　1）服务器程序负责对socket服务器进行封装，并在请求到来时，对请求的各种数据进行整理

　　　2）应用程序则负责具体的逻辑处理

### 2、WSGI（Web Server Gateway Interface）

   　　　　　　1. 不同的框架有不同的开发方式，但是无论如何，开发出的应用程序都要和服务器程序配合，才能为用户提供服务。
   　　　　　　2. 这样，服务器程序就需要为不同的框架提供不同的支持,只有支持它的服务器才能被开发出的应用使用,这时就需要有一个标准
   　　　　　　3. WSGI是一种规范，它定义了使用python编写的web app与web server之间接口格式，实现web app与web server间的解耦。

### 3、使用socket创建一个简单web服务

```python
import socket

def handle_request(client):
    buf = client.recv(1024)
    client.send("HTTP/1.1 200 OK\r\n\r\n".encode('utf-8'))      # 伪造浏览器请求头
    client.send("Hello, Seven".encode('utf-8'))                  # 返回数据到客户端浏览器

def main():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.bind(('localhost', 8000))
    sock.listen(5)
    while True:
        connection, address = sock.accept()
        handle_request(connection)
        connection.close()

if __name__ == '__main__':
    main()
```

### 4、自定义Web框架

　　**说明：** 通过python标准库提供的wsgiref模块开发一个自己的Web框架

```python
# 自定义web框架
from wsgiref.simple_server import make_server

def handle_index():
    return ['<h1>Hello, Index!</h1>'.encode('utf-8'),]
def handle_date():
    return ['<h1>Hello, date!</h1>'.encode('utf-8'),]

URl_DICT = {
    "/index":handle_index,
    "/date":handle_date,
}

#1 environ客户端发来的所有数据
#2 start_response封装要返回给用户的数据，响应头状态
#3 return返回的是正真用户在浏览器中看到的内容（Hello, web!）
def RunServer(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    current_url = environ['PATH_INFO']                           #用户访问的url目录：/index 或 /date
    func = None
    if current_url in URl_DICT:
        func = URl_DICT[current_url]
        return func()
    else:
        return ['<h1>404</h1>'.encode('utf-8'),]

if __name__ == '__main__':
    httpd = make_server('', 8001, RunServer)
    print("Serving HTTP on port 8000...")
    httpd.serve_forever()
```



## 二、MVC和MTV架构

### 1、MVC架构

  　	1. MVC架构把一个完整的程序或者网站项目分成三个主要的组成部分，分别是Model模型，View视图，Controller控制器

  　	2. 希望一个项目可以让内部数据的储存方式，外部的可见部分以及过程控制逻辑相互配合运行

  　	3. 进一步简化项目复杂度，提高可扩充性，维护性，有助于不同成员之间的分工

### 2、MTV框架（Django）

  　　1. 对于网站而言，网页服务器在接收到远程浏览器的请求的时候，不同的网址做出不同的响应

  　　2. 有不同的链接方式其实就隐含了逻辑控制，因此很难严谨的将其定义为上述三个部分

  　　3. 因此Django另外设计了MTV结构（Model，Template，View）。

### 3、MVC vs MTV

　　　　**MVC**

　　　　　　　　Model    View      Controller

　　　　　　　　数据库    模板文件   业务处理

　　　　**MTV**

　　　　　　　　Model   Template    View

　　　　　　　　数据库    模板文件   业务处理



## 三、Django常用命令

### 1、安装Django

```python
pip3 install django
```

### 2、创建Django工程

```js
D:\> django-admin startproject laoniu  //创建项目名称laoniu

D:\> cd laoniu
D:\laoniu> python manage.py runserver 127.0.0.1:8002 //运行laoniu即可用浏览器访问了
D:\laoniu> python manage.py createsuperuser  //创建Django admin用户
```

### 3、创建app

```python
python manage.py startapp app01
```

### 4、创建表

```python
python manage.py makemigrations  # 迁移
python manage.py migrate  # 映射
```

### 5、执行python manage.py migrate无法创建表

```python
# 执行时报：
No migrations to apply

# 1. 进入数据库，找到django_migrations的表，删除该app名字对应的所有记录
python manage.py dbshell
delete from django_migrations where app='app01';

# 2. 删除该app名字下的migrations下的除了__init__.py之外的文件。

# 3. 执行下面这两条命令：（在项目目录下）
python manage.py makemigrations
python manage.py migrate
"""
注：如果个别app表依然无法建立可以在后面添加app名称：
"""
python manage.py makemigrations/migrate app01

```



### 6、“Column 'last_login' cannot be null”报错

```mysql
mysql> SELECT * FROM django_migrations;

mysql> TRUNCATE TABLE django_migrations;

$ python manage.py migrate --fake-initial
Make sure this message appears: 0005_alter_user_last_login_null - [OK]

```

### 7、windows中杀死指定端口号进程

```shell
D:\>netstat -aon|findstr "8000"
D:\>taskkill /PID 12516 -t -f
```



## 四、配置settings.py文件

### 1、配置模板的路径

```python
TEMPLATES = [
    {
       'DIRS': [os.path.join(BASE_DIR,'templates')],
    },
]
```



### 2、 配置静态目录

```python
#像ccs和js这些静态文件如果想要使用必须在这里配置路径
STATICFILES_DIRS = (
    os.path.join(BASE_DIR,'static'),
)
```



### 3、注释CSRF

```python
MIDDLEWARE = [
    # 'django.middleware.csrf.CsrfViewMiddleware',
]
```



### 4、修改settings.py中时区

```python
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'zh-hans'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Shanghai'
```



## 五、Django各种url写法

### 1、无正则匹配url（http://127.0.0.1:8000/index/?nid=1&pid=2）

>urls.py

```python
urlpatterns = [
    url(r'^index/', views.index),
]
```

>views.py

```python
from django.shortcuts import render

def index(request):
    print(request.GET.get('nid'))             # 1
    print(request.GET.get('pid'))             # 2
    return render(request,'index.html')
```

>index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <a href="/index/?nid=1&pid=2"> url.py中不必用正则匹配 </a>
</body>
</html>
```



### 2、基于(\d+)正则的url

>urls.py

```python
urlpatterns = [
    url(r'^index/(\d+)/(\d+)/', views.index),
]
```

>views.py

```python
def index(request,nid,pid):
    return render(request,'index.html')
```

>index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <a href="/index/1/2/"> 使用分组匹配，不必关心传入参数顺序 </a>
</body>
</html>
```



### 3、基于正则分组(?P<nid>\d+)，可以不考虑接收参数顺序 (推荐)

>urls.py

```python
urlpatterns = [
    url(r'^index/(?P<nid>\d+)/(?P<pid>\d+)/', views.index),
]
```

>views.py

```python
def index(request,nid,pid):
    return render(request,'index.html')
```

>index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <a href="/index/1/2/"> 使用分组匹配，不必关心传入参数顺序 </a>
</body>
</html>
```



### 4、使用name构建自己想要的url

#### 		使用name在前端和后端分别构建自己想要的url

>urls.py定义路由系统

```python
urlpatterns = [
    url(r'^index1/', views.index,name='indexname1'),
    url(r'^index2/(\d+)/(\d+)/', views.index,name='indexname2'),
    url(r'^index2/(?P<pid>\d+)/(?P<nid>\d+)/', views.index,name='indexname3'),
]
```

>views.py根据reverse模块构建url

```python
from django.shortcuts import render
from django.core.urlresolvers import reverse

def index(request,*args,**kwargs):
    url1 = reverse('indexname1')                                      # /index1/
    url2 = reverse('indexname2', args=(1,2,))                         # /index2/1/2/
    url3 = reverse('indexname3', kwargs={'pid': 11, "nid":22})        # /index2/11/22/
    return render(request,'index.html')
```

>index.html构建url路径前端

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p><a href="{% url 'indexname1' %}">  http://127.0.0.1:8000/index/  </a></p>
    <p><a href="{% url 'indexname2' 1 2 %}">  http://127.0.0.1:8000/index2/1/2/  </a></p>
    <p><a href="{% url 'indexname3' 11 22 %}">  http://127.0.0.1:8000/index2/11/22/  </a></p>
</body>
</html>
```

#### 		

#### 		根据request.path中的绝对路径反解出url中的name名字

```python
resolve_url_obj = resolve(request.path)    #request.path路径： /student/homework_detail/52
resolve_url_obj.url_name                   #从path中解析出url名字 url_name = homework_detail
```



### 5、Django路由分发   并使用name构建url路径

#### 		作用：对URL路由关系进行命名，以后可以根据此名称生成自己想要的URL 

>@/project/urls.py

```python
from django.conf.urls import url,include
from app01 import urls

urlpatterns = [
    url(r'^app01/', include("app01.urls", app_name='app01', namespace='app01')),
]
```

>@/app/urls.py

```python
from django.conf.urls import url
from app01 import views

urlpatterns = [
    url(r'^index/$', views.index, name='index'),
]
```

>views.py

```python
from django.shortcuts import render

def index(request):
    return render(request,'index.html')
```

>index.http

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p> <a href="{% url "app01:index" %}">  http://127.0.0.1:8000/app01/index/</a></p>
</body>
</html>
```



## 六、Django的CBV和FBV

### 1、FBV（function base view）:在views.py文件中使用函数

```python
def index(request):
    return render(request, 'index.html')
```



### 2、CBV（class base view）：在views.py文件中使用类

​		1、 dispatch是父类中用来反射的函数，找对应的函数（比对应函数先执行）

　　2、 比如你发送post请求就可以通过dispatch找到对应的post函数进行处理，get就会找到get函数处理

>urls.py

```python
from django.conf.urls import url
from app01 import views

urlpatterns = [
    url(r'^home/',views.Home.as_view()), #as_view是指定调用view的as_view这个方法固定写法
]
```

>views.py

```python
from django.shortcuts import HttpResponse

from django.views import View
class Home(View):
    '''使用CBV时必须要继承view父类'''
    def dispatch(self, request, *args, **kwargs):
        # 调用父类中的dispatch
        result = super(Home,self).dispatch(request, *args, **kwargs)
        # 使用result主动继承view父类，然后return就可以重写父类的dispath方法
        return result
    # 在这里使用get发来请求就会调用get方法，使用post发来请求就会调用post方法
    def get(self,request):
        print(request.method)
        return HttpResponse('get')

    def post(self,request):
        print(request.method,'POST')
        return HttpResponse('post')
```

>CBV：指定某个视图函数不使用csrf验证view.py

```python
from django.shortcuts import HttpResponse
from django.views import View
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

class Home(View):
    '''使用CBV时必须要继承view父类'''
    @method_decorator(csrf_exempt)        # 指定Home这个视图函数不使用csrf
    def dispatch(self, request, *args, **kwargs):
        # 调用父类中的dispatch
        result = super(Home,self).dispatch(request, *args, **kwargs)
        # 使用result主动继承view父类，然后return就可以重写父类的dispath方法
        return result
    # 在这里使用get发来请求就会调用get方法，使用post发来请求就会调用post方法
    def get(self,request):
        print(request.method)
        return HttpResponse('get')

    def post(self,request):
        print(request.method,'POST')
        return HttpResponse('post')
```



## 七、 模板渲染

### 1、for循环

```html
<body>
    {% for k in user_dic.keys %}
        <p>{{ k }}</p>
    {% endfor %}

    {% for v in user_dic.values %}
        <p>{{ v }}</p>
    {% endfor %}

    {% for k,v in user_dic.items %}
        <p>{{ k }}--{{ v }}</p>
    {% endfor %}
</body>
```

```html
<body>
    {% for k in l %}
        <p>{{ k }}</p>
    {% endfor %}
</body>
```



### 2、if语句

>views.py

```python
def index(request):
    return render(request,'index.html',{
        'current_user':"alex",
        'user_list':['alex','eric'],
        'user_dict':{'k1':'v1', 'k2':'v2'}
    })
```

>index.html

```html
<div>{{current_user}}</div>

<a> {{ user_list.1 }} </a>          #在模板语言中获取列表中的元素
<a> {{ user_dict.k1 }} </a>         #在模板语言中获取字典中元素

{% if age %}
   <a>有年龄</a>
   {% if age > 16 %}
      <a>老男人</a>
   {% else %}
      <a>小鲜肉</a>
   {% endif %}
{% else %}
   <a>无年龄</a>
{% endif %}
```


### 3、ifequal判断值相等

>判断两个值是否相等

```html
{% if '讲师' in roles %}
<ul class="nav nav-sidebar">
  <li><a data-url="/webapp/staff/attendance/">发起签到</a></li>
  <li><a data-url="/webapp/staff/score/">学员成绩</a></li>
</ul>
{% endif %}
```

### 4、in在列表中

>in判断是否在列表中

```html
{% if '讲师' in roles %}
<ul class="nav nav-sidebar">
  <li><a data-url="/webapp/staff/attendance/">发起签到</a></li>
  <li><a data-url="/webapp/staff/score/">学员成绩</a></li>
</ul>
{% endif %}
```

### 5、HTML转换时间格式

>时间格式化

```html
# 1、使用下面方法转换
    <td>{{ c.start_time|date:'Y-m-d' }}</td>

# 2、转换前显示的格式
    July 2, 2017, 10:27 a.m.

# 3、转换后的格式
    2017-07-02
```

### 6、forloop显示数据序号

>forloop模板语言中显示数据序号

```python
# 1、<td>{{ forloop.counter }}</td>
# 2、<td>{{ forloop.counter0 }}</td>
# 3、<td>{{ forloop.revcounter }}</td>
# 4、<td>{{ forloop.revcounter0 }}</td>
# 5、<td>{{ forloop.last }}</td>
# 6、<td>{{ forloop.first }}</td>
```



### 7、for循环时 json化

>for循环时 json化

```html
@register.filter(name='json_to_obj')
def json_to_obj(obj):
    return json.loads(obj)
<!-- 最主要的语句 -->
{% for detail in parameter|safe|json_to_obj %}    

{% endfor %}
```



## 八、自定义simple_tag和自定义filter

​		**作用：在模板语言中对数据加工后返回**

### 1、自定义simple_tag步骤

　　1、app下创建templatetags目录，并在templatetags目录下新建任意python文件（比如：xxoo.py）

　　2、settings中注册APP

　　3、在templates/下的html文件中引用

### 2、在templatetags/xxoo.py文件中写处理函数

>在templatetags/xxoo.py文件中写处理函数

```python
from django import template
from django.utils.safestring import mark_safe

register = template.Library()      # 对象名register不可变

@register.simple_tag               #1 simple_tag用法
def houyafan(a1,a2):
    return a1 + a2

@register.filter                  #2 filter用法
def jiajingze(a1,a2):
    return a1 + a2
```

>在templates/index.html文件中使用

```html
<!-- 1、 在html文件最顶部引用xxoo.py文件  -->
{% load xxoo %}
<html>
<head lang="en">
    <meta charset="UTF-8">
</head>
<body>
<!-- 2、在html文件最顶部引用xxoo.py文件  -->
<!-- houyafan是xxoo.py文件中函数名，2 4是传入的参数 显示到页面的结果是2+4是6 -->

    {% houyafan 2 4 %}


<!-- 3、filter参数格式  参数1|函数名:参数2  -->

    {{ "maliya"|jiajingze:"LS" }}

    {% if "maliya"|jiajingze:"LS" %}

    {% endif %}
</body>
</html>
```



### 3、自定义simple_tag和自定义filter优缺点比较

　　1、自定simple_tag

　　　　　**缺点：**  不能作为if条件

　　　　　**优点：**  参数任意个

　　2、自定义filter

　　　　　**缺点：**  最多两个参数，不能加空格

　　　　　**优点：**  能作为if条件