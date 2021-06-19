---
title: Django 进阶
date: 2020-11-24 08:30:49
tags: Django
---

## 一、模板继承

### 模板继承的使用：

- 在master.html中定义模板：

```html
{% block css %}{% endblock %}
```

- 在子类中引入要继承的模板：

```html
{% extends "master.html" %}
```

### 模板导入：

- 使用时直接导入即可：

  ```html
  {% include "tag.html" %}
  ```

>模板：templates/master.html

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">

    {#1、子类标题#}
        <title>{% block title %}{% endblock %}</title>

        <link rel="stylesheet" href="/static/commons.css">
    {#2、子类css样式#}
        {% block css %} {% endblock %}

    </head>
    <body>
        <div class="pg-header">小男孩管理</div>

    {#3、子类页面主要内容#}
        {% block content %}{% endblock %}

        <script src="/static/jquery.js"></script>
    {#4、子类js样式#}
        {% block js %} {% endblock %}

    </body>
</html>

```



>在子类中继承模板中的内容

```html
{#1 extends关键字引入要继承的模板#}
{% extends 'master.html' %}

{#2 有些时候我们继承的时候想要有一些自己的东西，比如：标题#}
{% block title %} 用户管理 {% endblock %}

{#3 我们可以将每个页面自己的内容放到这里#}
{% block content %}
    <h1>用户管理</h1>
    <ul>
        {% for i in u %}
            <li>{{ i }}</li>
        {% endfor %}
    </ul>

{#模板导入 这里的”tag.html”在右边#}
{% include "tag.html" %}

{% endblock %}

{#4 本网页自己的css样式#}
{% block css %}
    <style>
        body{
            background-color: red;
        }
    </style>
{% endblock %}

{#5 本网页自己的js样式#}
{% block js %}
    <script>
        alert(1234)
    </script>
{% endblock %}
```



## 二、前后端交互：提交数据、提交文件

>index.html提交数据

```html
<form action="/login/" method="post" enctype="multipart/form-data">
    <p>
        <input type="text" name="user" placeholder="用户名">
    </p>
    
    {# 1、单选框，返回单条数据的列表 #}
    <p>     
        男：<input type="radio" name="gender" value="1">
        女：<input type="radio" name="gender" value="2">
        张扬：<input type="radio" name="gender" value="3">
    </p>

    {# 2、多选框、返回多条数据列表 #}
    <p>     
        男：<input type="checkbox" name="favor" value="11">
        女：<input type="checkbox" name="favor" value="22">
        张扬：<input type="checkbox" name="favor" value="33">
    </p>

    {# 3、多选，返回多条数据的列表 #}
    <p>     
        <select name="city" multiple>
            <option value="bj">北京</option>
            <option value="sh">上海</option>
            <option value="gz">广州</option>
        </select>
    </p>

    {# 4、提交文件 #}
    <p><input type="file" name="fff"></p>
    <input type="submit" value="提交">
</form>
```

>views.py获取数据

```python
def login(request):
    if request.method == 'GET':
        return render(request,'login.html')
    elif request.method == 'POST':
        v = request.POST.get('gender')      #1 获取单选框的value值
        v = request.POST.getlist('favor')   #2 获取多选框value：['11', '22', '33']
        v = request.POST.getlist('city')    #3 获取多选下拉菜单：['bj', 'sh', 'gz']
        print(v.name,type(v))

        #当服务器端取客户端发送来的数据，不会将数据一下拿过来而是一点点取(chunks就是文件分成的块)
        
        obj = request.FILES.get('fff')      #4 下面是获取客户端上传的文件（如：图片）
        import os
        file_path = os.path.join('upload',obj.name)
        f = open(file_path,mode='wb')
        for i in obj.chunks():
            f.write(i)
        f.close()

        return render(request,'login.html')
```

>request获取

```python
# 1、request.POST
# 2、request.GET
# 3、request.FILES
# 4、request.getlist
# 5、request.method
# 6、request.path_info                   #获取当前url
# 7、request.body                        #自己获取数据
# 8、a = ‘中国’
    response = HttpResponse(a)
    response[“name”] = ‘alex’        #HttpResponse不仅可以在响应头中传字符串，也可以传键值对
    response.set_cookie()                #设置cookie实质也是向请求头中传入键值对
```



## 三、django的生命周期和中间件

- ### 中间件的处理过程

1. 首现由客户端发起请求，会将请求交给settings.py中排在最前面的中间件
2. 中间件收到请求后回调用类中的process_request方法处理，然后交给下一个中间件的process_request函数
3. 到达最后一个中间件的process_request函数处理后会到达路由系统
4. 然后经过路由系统直接跳转到第一个中间件的process_view函数，依次向后面中间件的process_view传递最后到达view.py处理函数，获取网页中的数据
5. 获取的数据会交给最后一个中间件的process_response方法处理，然后依次向前面的中间件process_response方法提交请求的内容，最后最前面的中间件将请求数据返回到客户端
6. 在任意中间件的process_request和process_view方法中有返回值就会直接返回给process_response

- ### 生命周期请求过程

  - #### 客户端访问

    ```python
    # 客户端在浏览器中输入url路径访问指定网页
    ```

  - #### 请求发送给django程序

    ```python
    # 1、首先会交给中间件，中间件处理后交给路由系统
    # 2、路由系统
    	# django程序会到urls.py文件中找到对应请求的处理函数（视图函数）
    # 3、视图函数
    	# 1：视图函数会找到对应的HTML模板文件
        # 2：然后到数据库中获取数据替换HTML模板中的内容
        # 3：使用static的js和css文件结合对HTML渲染
        # 4：最后django将最终渲染后的HTML文件返回给中间件
    # 4、中间件再调用process_response方法处理，最后给客户
    ```



## 四、中间件使用举例

### 1、创建存放中间件的文件夹

- 在工程目录下创建任意目录，这里创建为：project/middle/m1.py

### 2、在project/settings.py文件中注册自己的中间件

```python
MIDDLEWARE = [
    'middle.m1.Row1',
    'middle.m1.Row2',
    'middle.m1.Row3',
]
```

### 3、在view.py文件中写处理函数test

```python
def test(request):
    # int('fds')    #当views函数出现异常，中间件中的process_exception执行
    print('没带钱|')
    return HttpResponse('ok')
```

### 4、在project/middle/m1.py文件中定义中间件

```python
from django.utils.deprecation import MiddlewareMixin
class Row1(MiddlewareMixin):
    def process_request(self,request):
        print('process_request_1')

    def process_view(self,request, view_func, view_func_args, view_func_kwargs):
        #view_func_args:   url中传递的非字典的值会用这个变量接收
        #view_func_kwargs: url中传递的字典会传递到这个变量接收（如：nid=1）
        print('process_view_1')

    def process_response(self,request, response):    #response就是拿到的返回信息
        print('response_1')
        return response

    def process_exception(self, request, exception):
        '''只有当views函数中异常这个函数才会执行'''
        if isinstance(exception, ValueError):
            return HttpResponse('>>出现异常了')

class Row2(MiddlewareMixin):
    def process_request(self,request):
        print('process_request_2')
        #1 如果在Row2中的process_request中有返回值，那么就不会到达Row3
        #2 Row2直接将返回值交给自己的process_response再交给Row1的process_response
        #3 最后客户端页面显示的就是‘走’请求没机会到达views函数，不会打印‘没带钱’
        # return HttpResponse('走')

    def process_view(self,request, view_func, view_func_args, view_func_kwargs):
        print('process_view_2')

    def process_response(self,request, response):
        print('response_2')
        return response

class Row3(MiddlewareMixin):
    def process_request(self,request):
        print('process_request_3')

    def process_view(self,request, view_func, view_func_args, view_func_kwargs):
        print('process_view_3')

    def process_response(self,request, response):
        print('response_3')
        return response
```



## 五、cookie

### 1、cookie简介

1. cookie实质是客户端硬盘存放的键值对，利用这个特性可以用来做用户验证
2. 比如：{"username" : "Liuxingxing"}  # 再次访问url就会携带这些信息过来

### 2、前端操作cookie

​	说明：使用下面的方法操作cookie必须先引入jquery.cookie.js

1. 前端获取cookie值：

   ```js
   var v = $.cookie('per_page_count');
   ```

2. 前端设置cookie值：

   ```js
   $.cookie('per_page_count', v);
   ```

### 3、后端操作cookie

​	说明：response = HttpResponse(...) 或 response = render(request, ...)

1. 后端设置cookie值：

   ```python
   response.set_cookie('username', 'liuxingxing')
   ```

2. 后端获取cookie值：

   ```python
   response.COOKIES.get('username')
   ```

### 4、设置cookie时常用的参数

```python
def cookie(request):
    #1 获取cookie中username111得值
    request.COOKIES.get('username111')

    #2 设置cookie的值，关闭浏览器失效
    response.set_cookie('key',"value")
    # 设置cookie, N秒只后失效
    response.set_cookie('username111',"value",max_age=10)

    #3 设置cookie, 截止时间失效（expires后面指定那个时间点失效）
    import datetime
    current_date = datetime.datetime.utcnow()
    exp_date = current_date + datetime.timedelta(seconds=5)         #seconds指定再过多少秒过期
    response.set_cookie('username111',"value",expires=exp_date)

    #4 设置cookie是可以使用关键字salt对cookie加密（加密解密的salt中值必须相同）
    obj = HttpResponse('s')

    obj.set_signed_cookie('username',"kangbazi",salt="asdfasdf")
    request.get_signed_cookie('username',salt="asdfasdf")

    #5 设置cookie生效路径
    path = '/'

    #6 删除cookie中is_login的值
    response.delete_cookie('is_login')
    return response
```

### 5、使用cookie实现用户登录、注销

>views.py

```python
from django.shortcuts import render,HttpResponse,redirect

def index(request):
    username = request.COOKIES.get('username')        # 获取cookie
    if not username:
        return redirect('/login/')
    return HttpResponse(username)

def login(request):
    if request.method == "GET":
        return render(request,'login.html')
    if request.method == "POST":
        u = request.POST.get('username')
        p = request.POST.get('pwd')
        if u == 'tom' and p == '123':
            res = redirect('/index/')
            res.set_cookie('username',u,max_age=500)        # 设置500s免登陆
            return res
        else:
            return render(request,'login.html')

def logout(req):
    response = redirect('/login/')
    #清理cookie里保存username
    response.delete_cookie('username')
    return response
```

>login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="POST">
        <input type="text" name="username" placeholder="用户名">
        <input type="text" name="pwd" placeholder="密码">
        <input type="submit" value="提交">
    </form>
</body>
</html>
```

### 6、前端设置、获取cookie

>urls.py

```python
from app01 import views
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index),
    url(r'^get_ck/', views.get_ck),
]
```

>views.py

```python
from django.shortcuts import render,HttpResponse

def index(request):
    return render(request, 'index.html', )

def get_ck(request):
    val = request.COOKIES.get('per_page_count')
    print('get_ck',val)
    return HttpResponse(val)
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
    {# 选择每页显示多少条的select单选框 #}
    <div>
        <select id="ps" onchange="changePageSize(this)">
            <option value="10">10</option>
            <option value="20">20</option>
            <option value="50">50</option>
            <option value="100">100</option>
        </select>
    </div>

    <script src="/static/jquery-1.12.4.js"></script>
    <script src="/static/jquery.cookie.js"></script>

    <script>
        //当框架加载完成后获取cookie的值，并设置到select中
        $(function(){
           var v = $.cookie('per_page_count');     //前端获取cookie值
            console.log(v);
            $('#ps').val(v);
        });

        function changePageSize(ths){
            //获取select单选框选择的值（10,20,50,100）这些选项
            var v = $(ths).val();
            //使用cookie将v的值传递到后台
            $.cookie('per_page_count',v, { expires: 7 });        //前端设置cookie值
            $.cookie('per_page_count',v, {'path':'/'});       // 将这个路径设置为网站的根目录
        }
    </script>
</body>
</html>
```



## 六、session

### 1、session作用及原理

​	注释：session操作依赖cookie

1. 基于cookie做用户验证时：敏感信息不适合放在cookie中
2. 用户成功登录后服务端会成为一个随机字符串并将这个字符串作为字典的key，将用户登录信息作为value
3. 当用户再次登录时就会带着这个随机字符串过来，就不用再一次输入用户名和密码了
4. 用户使用cookie将这个随机字符串保存到客户端本地，当用户再来时携带这个随机字符串，服务器根据这个字符串查找对应的session中的值，这样就避免敏感信息泄露

### 2、cookie和session对比

1. cookie是保存在浏览器端的键值对
2. session是保存在服务器端的键值对

### 3、django中支持下面五种session，可以在settings.py中配置

>5种session在settings.py中公用配置参数
>
>```python
># 1、SESSION_COOKIE_NAME ＝ "sessionid"          # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
># 2、SESSION_COOKIE_PATH ＝ "/"                  # Session的cookie保存的路径（默认）
># 3、SESSION_COOKIE_DOMAIN = None                # Session的cookie保存的域名（默认）
># 4、SESSION_COOKIE_SECURE = False               # 是否Https传输cookie（默认）
># 5、SESSION_COOKIE_HTTPONLY = True              # 是否Session的cookie只支持http传输（默认）
># 6、SESSION_COOKIE_AGE = 1209600                # Session的cookie失效日期（2周）（默认）
># 7、SESSION_EXPIRE_AT_BROWSER_CLOSE = False     # 是否关闭浏览器使得Session过期（默认）
># 8、SESSION_SAVE_EVERY_REQUEST = False          # 是否每次请求都保存Session，默认修改之后才保存（默认）
>                                            # 10s 免登陆时，这里必须配置成True
>```
>
>settings.py中配置使用session5种方法
>
>```python
>def index11(request):                #request.session中存放了所有用户信息
>#1 获取   Session中数据
>request.session['k1']
>request.session.get('k1',None)
>
>#2 设置
>request.session['k1'] = 123
>request.session.setdefault('k1',123)    # 存在则不设置
>
>#3 删除
>del request.session['k1']               #删除某个键值对
>#删除某用户登录时在数据库生成的随机字符串的整条数据全部删除
>request.session.delete("session_key")
>
>#4 注销  当用户注销时使用request.session.clear()
>request.session.clear()         #这个短句相当于下面这个长句
>request.session.delete(request.session.session_key)
>
>#5 设置超时时间（默认超时时间两周）
>request.session.set_expiry("value")
># 如果value是个整数，session会在些秒数后失效。
># 如果value是个datatime或timedelta，session就会在这个时间后失效。
># 如果value是0,用户关闭浏览器session就会失效。
># 如果value是None,session会依赖全局session失效策略。
>
># 所有 键、值、键值对
>request.session.keys()
>request.session.values()
>request.session.items()
>request.session.iterkeys()
>request.session.itervalues()
>request.session.iteritems()
>
># 获取某个用户session的随机字符串（不常用）
>request.session.session_key
>
># 将所有Session失效日期小于当前日期的数据删除（数据库中可以设置超时时间）
>request.session.clear_expired()
>
># 检查 用户session的随机字符串 在数据库中是否存在（不常用）
>request.session.exists("session_key")
>```

### 5、session实现用户十秒免登陆，以及注销功能

1. session默认使用数据库session，使用前必须先执行下面命令

   ```python
   python manage.py makemigrations
   python manage.py migrate
   ```

2. settings.py中配置每次用户访问都会推迟更新时间

   ```django
   SESSION_SAVE_EVERY_REQUEST = True
   ```

3. 实现10s免登陆关键步骤

   ```python
   # 1、设置session：request.session['is_login'] = True
   # 2、设置10s超时：request.session.set_expiry(10)
   # 3、获取session：request.session.get('is_login')
   ```

   >views.py
   >
   >```python
   >from django.shortcuts import render,HttpResponse,redirect
   >
   >def index(request):
   >if request.session.get('is_login'):
   >   return render(request,'index.html',{'username':request.session.get('username')})
   >else:
   >   return HttpResponse('滚')
   >
   >def login(request):
   >if request.method == 'GET':
   >   return render(request,'login.html')
   >elif request.method == 'POST':
   >   user = request.POST.get('user')
   >   pwd = request.POST.get('pwd')
   >   if user == 'tom' and pwd == '123':
   >       #1 生成随机字符串
   >       #2 写到用户浏览器cookie
   >       #3 保存到session中
   >       #4 在随机字符串对应的字典中设置相关内容
   >       #5 有等号设置session键值对，没有等号获取键的值
   >
   >       request.session['username']=user       #1 用户登录成功设置用户名
   >
   >       request.session['is_login'] = True     #2 登陆成功时可以设置一个标志
   >
   >       if request.POST.get('rmb') == '1':     #3 当勾选checkbox框后设置10秒超时
   >           request.session.set_expiry(10)     #4 设置10s后超时，需要重新登录
   >       return redirect('/index')
   >   else:
   >       return render(request,'login.html')
   >
   >def logout(request):
   >request.session.clear()       #5 访问'/logout/'时注销登陆
   >return redirect('/login/')
   >```
   >
   >login.html
   >
   >```html
   ><!DOCTYPE html>
   ><html lang="en">
   ><head>
   ><meta charset="UTF-8">
   ><title>Title</title>
   ></head>
   ><body>
   ><form action="/login/" method="POST">
   >   <input type="text" name="user">
   >   <input type="text" name="pwd">
   >   <input type="checkbox" name="rmb" value="1">十秒免登陆
   >   <input type="submit" value="提交">
   ></form>
   ></body>
   ></html>
   >```
   >
   >index.html
   >
   >```html
   ><!DOCTYPE html>
   ><html lang="en">
   ><head>
   ><meta charset="UTF-8">
   ><title>Title</title>
   ></head>
   ><body>
   ><h1>欢饮登陆：{{ username }},{{ request.session.username }}</h1>
   ><a href="/logout/">注销</a>
   ></body>
   ></html>
   >```
   >
   >settings.py
   >
   >```python
   >SESSION_SAVE_EVERY_REQUEST = True
   >```





## 七、django序列化操作

### 1、为什么需要序列化操作

1. python中很多格式的数据类型不能通过简单的json直接序列化转换成字符串格式传到前端
2. 如：form提交数据出现错误，返回的是django的error.dict对象，不能直接用python的json序列化(必须先as_json)

### 2、django序列化（方法一：前端两次序列化得到错误信息）

1. 首现后端先用as_json将error.dict对象转换成字符串，然后通过json传递到前端
2. 前端第一次序列化：将整个错误信息大字典变成对象，提取其中的error小字典
3. 前端第二次序列化：由于获取的小字典是字符串，还需要将error序列化成对象，才能获取到错误信息

>views.py
>
>```python
>from django.shortcuts import render,HttpResponse
>from django import forms
>from django.forms import fields         #fields字段专门用于验证
>from django.forms import widgets        #widgets专门用于生成html标签
>import json
>
>#这里为了方便没有将创建form放到forms.py中
>class LoginForm(forms.Form):
>username = fields.CharField()
>password = fields.CharField(
>   max_length=12,
>   min_length=4,
>   error_messages={
>       'required':'密码不能为空',
>       'min_length':'密码长度不能小于4',
>       'max_length':'密码长度不能大于12'
>   }
>)
>
>def login(request):
>if request.method == 'GET':
>   return render(request,'login.html')
>elif request.method == 'POST':
>   ret = {'status':True,'error':None,'data':None}
>   obj = LoginForm(request.POST)
>   if obj.is_valid():
>       print(obj.cleaned_data)
>       #正确信息：{'password': '1234567343434343434', 'username': 'tom'}
>   else:
>       # obj.errors中封装的数据必须变成字符串才能json操作
>       ret['error'] = obj.errors.as_json()    # as_json() 返回的是字符串
>       # print(obj.errors.as_data())
>   return HttpResponse(json.dumps(ret))
>```
>
>login.html
>
>```html
><!DOCTYPE html>
><html lang="en">
><head>
><meta charset="UTF-8">
><title>Title</title>
></head>
><body>
>	<form id="fm">
>   {% csrf_token %}
>   <p><input type="text" name="username"></p>
>   <p><input type="password" name="password">
>       <span id="pwd-err" style="color: red"></span></p>
>   <a id="submit">ajax提交</a>
></form>
>
><script src="/static/jquery-1.12.4.js"></script>
><script>
>   /*  #### 当不填密码提交后返回的错误信息时这样的  ####
>   {'status': True,
>   'error': '{"password": [{"code": "required", "message": "密码不能为空"}]}',
>   'data': None}
>    */
>
>$(function(){
>    $('#submit').click(function(){
>        $.ajax({
>            url: '/login/',
>            type:'POST',
>            data:$('#fm').serialize(),
>            success:function(arg){  // 服务端返回的arg是字符串格式
>
>              arg = JSON.parse(arg); // 第一次json转换：将arg字符串转换成对象
>
>              error = arg.error;  // 获取到arg对象中的error(error此时是字符串)
>
>              error = JSON.parse(error);  // 第二次json转换：将error字符串转换成对象
>
>              pwd_err = error.password[0].message; // 转换成对象后才能获取到最终的错误信息
>              console.log(pwd_err);  //pwd_err打印结果：密码不能为空
>              $('#pwd-err').text(pwd_err);
>         },
>         error:function(){
>         }
>       })
>     })
>   })
></script>
></body>
></html>
>```

### 3、django序列化（方法二：前端一次序列化得到错误信息）

1. 首现后端as_data()将获取到错误数据
2. 然后将数据放到JsonCustomEncoder类中转换成指定格式的字典，前端一次即可得到错误提示信息

>views.py
>
>```python
>from django.shortcuts import render,HttpResponse
>from django import forms
>from django.forms import fields        #fields字段专门用于验证
>from django.forms import widgets       #widgets专门用于生成html标签
>import json
>
>from django.core.exceptions import ValidationError
>class JsonCustomEncoder(json.JSONEncoder):
>def default(self, field):
>   if isinstance(field, ValidationError):
>       return {'code':field.code,'messages':field.messages}
>   else:
>       return json.JSONEncoder.default(self, field)
>
>#这里为了方便没有将创建form放到forms.py中
>class LoginForm(forms.Form):
>username = fields.CharField()
>password = fields.CharField(
>   max_length=12,
>   min_length=4,
>   error_messages={
>       'required':'密码不能为空',
>       'min_length':'密码长度不能小于4',
>       'max_length':'密码长度不能大于12'
>   }
>)
>
>def login(request):
>if request.method == 'GET':
>   return render(request,'login.html')
>elif request.method == 'POST':
>   ret = {'status':True,'error':None,'data':None}
>   obj = LoginForm(request.POST)
>   if obj.is_valid():
>       print(obj.cleaned_data)
>       #正确信息：{'password': '1234567343434343434', 'username': 'tom'}
>   else:
>       # obj.errors中封装的数据必须变成字符串才能json操作
>       ret['error'] = obj.errors.as_data()  # as_json() 返回的是字符串
>       # print(obj.errors.as_data())  # {'password': [ValidationError(['密码不能为空'])]}
>       print(ret)
>   request = json.dumps(ret,cls=JsonCustomEncoder)
>   print(request)
>   return HttpResponse(request)
>```
>
>index.html
>
>```html
><!DOCTYPE html>
><html lang="en">
><head>
><meta charset="UTF-8">
><title>Title</title>
></head>
><body>
><form id="fm">
>   {% csrf_token %}
>   <p><input type="text" name="username"></p>
>   <p><input type="password" name="password">
>       <span id="pwd-err" style="color: red"></span></p>
>   <a id="submit">ajax提交</a>
></form>
>
><script src="/static/jquery-1.12.4.js"></script>
><script>
>   $(function(){
>       $('#submit').click(function(){
>           $.ajax({
>               url: '/login/',
>               type:'POST',
>               data:$('#fm').serialize(),
>               success:function(arg){  //服务端返回的是字符串格式
>
>                   arg = JSON.parse(arg);  //这里只需要一次序列化就可以得到error字典格式
>
>                   pwd_err = arg.error.password[0].messages;
>
>                   console.log(pwd_err);  //pwd_err打印结果：密码不能为空
>                   $('#pwd-err').text(pwd_err);
>               },
>               error:function(){
>               }
>           })
>       })
>   })
></script>
></body>
></html>
>```




## 八、CSRF跨站请求伪造

### 1、CSRF原理

1. 当用户第一次发送get请求时，服务端不仅给客户返回get内容，而且中间包含一个随机字符串
2. 这个字符串是加密的，只有服务端自己可以反解
3. 当客户端发送post请求提交数据时，服务端会验证客户端是否携带这个随机字符串，没有就会引发csrf错误
4. 如果没有csrf，那么黑客可以通过任意表单向我们的后台提交数据，不安全

### 2、form和ajax提交数据，解决CSRF方法

1. $.cookie('csrftoken')可以获取到那个随机字符串
2. headers{}可以将指定信息添加到请求头部发送给服务端
3. 通过cookie和通过{{ csrf_token }}获取的随机字符串不同

>form和ajax提交数据，解决CSRF方法
>
>```html
><form action="/login/" method="POST">
>
>{#1 form方式提交仅需要在form表单中添加csrf_token生成随机字符串即可 #}
>{% csrf_token %}
>
><input type="submit" name="提交">
><input id="btn" type="button" value="按钮">
></form>
><script src="/static/jquery-1.12.4.js"></script>
><script src="/static/jquery.cookie.js"></script>
>
>{#2 使用ajax提交CSRF字符串的两种方法 #}
><script>
>$(function(){
>
>{# 方法一： 仅用定义一个ajaxSetup实现对所有页面ajax提交数据自动发送CSRF字符串 #}
>   //xhr是ajax内部封装的一个方法
>   $.ajaxSetup({
>      beforeSend:function(xhr,settings){
>           xhr.setRequestHeader('X-CSRFtoken',$.cookie('csrftoken'))
>      }
>   });
>
>{# 方法二： 必须在每个ajax提交的函数中都指定要发送CSRF字符串 #}
>   $('#btn').click(function(){
>       $.ajax({
>           url: '/login/',
>           type: 'POST',
>           data: {'user':'root','pwd':'123'},
>           headers:{'X-CSRFtoken':$.cookie('csrftoken')},
>           success:function(data){
>           }
>       })
>   })
>})
></script>
>```

### 3、ajax提交过滤

​	说明：对以GET、HEAD、OPTIONS、TRACE这四种提交数据方法不提交CSRF字符串

>ajax提交过滤
>
>```html
><script>
>var csrftoken = $.cookie('csrftoken');
>function csrfSafeMethod(method) {
>   // these HTTP methods do not require CSRF protection
>   return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
>}
>
>$.ajaxSetup({
>   beforeSend: function(xhr, settings) {
>       if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
>           xhr.setRequestHeader("X-CSRFToken", csrftoken);
>       }
>   }
>});
></script>
>```

### 4、指定哪些函数启用CSRF验证

>方法一：启用CSRF验证指，仅指定某些页面不需要验证
>
>```python
>#1 启用settings文件中的CSRF
>
>MIDDLEWARE = [
>'django.middleware.csrf.CsrfViewMiddleware',
>]
>
>#2 对不需要验证的函数添加@csrf_exempt
>
>from django.views.decorators.csrf import csrf_exempt,csrf_protect
>@csrf_exempt
>def login(request):
>'处理函数内容'
>```
>
>方法二：关闭CSRF验证指，仅指定某些页面需要验证
>
>```python
>#1 关闭settings文件中的CSRF
>
>MIDDLEWARE = [
># 'django.middleware.csrf.CsrfViewMiddleware',
>]
>
>#2 对需要验证的函数添加@csrf_protect
>
>from django.views.decorators.csrf import csrf_exempt,csrf_protect
>@csrf_protect
>def login(request):
>'处理函数内容'
>```



## 九、信号

### 1、信号的作用

1. django中提供了"信号调度"，用于在框架执行操作时解耦
2. 通俗来讲：就是一些动作发生的时候，信号允许特定的发送者去提醒一些接受者
3. 比如，我们在删除数据的时候需要记录删除了什么就可以使用信号了

### 2、django内置信号

​	作用：django中提供了"信号调度"，用于在框架执行操作时解耦；通俗来讲：就是一些动作发生的时候，信号允许特定的发送者去提醒一些接受者

1. django内置信号

   >django内置信号
   >
   >```python
   >1、    Model signals
   >#1  obj = models.UserInfo(user='root') django的modal执行其构造方法
   >#2  obj.save()    django的modal对象保存
   >pre_init                 # django的modal执行其构造方法前，自动触发
   >post_init                # django的modal执行其构造方法后，自动触发
   >pre_save                 # django的modal对象保存前，自动触发
   >post_save                # django的modal对象保存后，自动触发
   >
   >pre_delete               # django的modal对象删除前，自动触发
   >post_delete              # django的modal对象删除后，自动触发
   >m2m_changed              # django的modal中使用m2m字段操作第三张表（add,remove,clear）前后，自动触发
   >class_prepared           # 程序启动时，检测已注册的app中modal类，对于每一个类，自动触发
   >
   >2、Management signals
   >pre_migrate              # 执行migrate命令前，自动触发
   >post_migrate             # 执行migrate命令后，自动触发
   >
   >3、Request/response signals
   >request_started          # 请求到来前，自动触发
   >request_finished         # 请求结束后，自动触发
   >got_request_exception    # 请求异常后，自动触发
   >
   >4、Test signals
   >setting_changed          # 使用test测试修改配置文件时，自动触发
   >template_rendered        # 使用test测试渲染模板时，自动触发
   >
   >5、Database Wrappers
   >connection_created       # 创建数据库连接时，自动触发
   >```

2. 使用django内置信号（当django新建数据时自动触发某函数）

   1. 首现在django工程目录下新建sg.py文件，将django中所有内置信号全部导入到里面
   2. 在project/__ init __.py中导入sg.py（作用：当django一运行就自动导入所有信号）
   3. 在views.py中定义single函数，执行obj = model.UserInfo(user="root")测试

>project/sg.py自定义信号
>
>```python
>#1  pizza_done是自己定义的信号名称
>#2 ["toppings", "size"]指定要想触发这个信号必须传递这两个参数
>import django.dispatch
>pizza_done = django.dispatch.Signal(providing_args=["toppings", "size"])
>
>#3 定义一个函数
>def callback(sender, **kwargs):
>print("callback")
>print(sender,kwargs)
>
>#4 将上面定义的函数注册到我们自定义的信号中
>pizza_done.connect(callback)
>```
>
>project/__ init __.py导入sy.py模板
>
>```python
>import sg
>```
>
>project/views.py中触发信号
>
>```python
>def signal(request):
>from sg import pizza_done
>pizza_done.send(sender='suifasongde',toppings='suibianxie',size='suibianxie')
>
>return HttpResponse('111')
>```



## 十、django中的缓存

### 1、django缓存的作用

1. 由于django是动态网站，所以每次请求均会去数据库进行相应的操作，当程序访问量增大时，耗时必然会更加明显
2. 缓存讲一个某个views的返回值保存至内存或memcache中，5分钟内再有人来访问时，则不再去执行views中的操作
3. 而是直接从内存或者Redis中之前缓存的内容拿到，并返回

### 2、django中提供了六种缓存方式

>1：开发调试缓存
>
>```python
># 开发调试缓存（虽然配置上，但实际没有缓存，还是到数据库取）
>
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.dummy.DummyCache',  # 引擎
>
>   #注： 下面这些参数时公用的，五种缓存都可以使用
>   'TIMEOUT': 300,           # 缓存超时时间（默认300，None表示永不过期，0表示立即过期）
>   'OPTIONS':{
>       'MAX_ENTRIES': 300,   # 最大缓存个数（默认300）
>       'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例(3就是1/3)
>   },
>}
>}
>```
>
>2：内存缓存
>
>```python
># 注：内存缓存本质上就是在内存中维护一个字典，所存储的形式就是字典的键值对组合
>
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
>   'LOCATION': 'unique-snowflake',     #这个参数指定变量名必须唯一
>}
>}
>```
>
>3：文件缓存
>
>```python
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
>   'LOCATION': os.path.join(BASE_DIR,'cache'),  #缓存内容存放的文件夹路径
>}
>}
>```
>
>4：数据库缓存
>
>```python
># 注：执行创建表命令 python manage.py createcachetable
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
>   'LOCATION': 'my_cache_table', # 数据库表名（名字是自己取的）
>}
>}
>```
>
>5：Memcache缓存（两种）
>
>```python
># 注：Memcache缓存有两个模块：python-memcached模块、 pylibmc模块
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
>   'LOCATION': '127.0.0.1:11211',          #使用ip加端口连接memcached
>}
>}
>
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
>   'LOCATION': 'unix:/tmp/memcached.sock',     #以文件的形式连本地memcached
>}
>}
>
>CACHES = {
>'default': {
>   'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
>   'LOCATION': [               # memcached天生支持集群
>       #1 均衡分配
>       '172.19.26.240:11211',
>       '172.19.26.242:11211',
>       #2 调整权重(权重和请求比例成正比)
>       ('172.19.26.240:11211',1),
>       ('172.19.26.242:11211',10),
>   ]
>}
>}
>
># 注： pylibmc模块只改变上面'BACKEND'配置为下面样式即可
># 'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache'
>```

### 3、django中缓存三种应用

1. 页面级别缓存（需要views.py文件中引入cache_page）

   >页面级别缓存
   >
   >```python
   ># 1、views.py文件中的处理函数
   >
   >from django.views.decorators.cache import cache_page
   >@cache_page(6)                 #6秒后缓存失效
   >def cache(request):
   >import time
   >ctime = time.time()
   >return render(request,'cache.html',{'ctime':ctime})
   >
   ># 2、cache.html文件中内容都会被缓存，5s后才会失效
   >
   ><body>
   ><h1>{{ ctime }}</h1>      #页面刷新时时间不会时刻变化，5s过后才会变一次
   ></body>
   >```

2. 模板级别缓存（直接在cache.html模板文件中指定某个值放入缓存）

   >模板级别缓存
   >
   >```html
   >{#1 在文件最顶部引入TemplateTag#}
   >{% load cache %}
   >
   ><body>
   >{#2 使用缓存   c1是缓存的key值 #}
   >{% cache 5 c1 %}        {# 将数据缓存5秒 #}
   >   {{ ctime }}
   >{% endcache %}
   ></body>
   >```

3. 全局缓存（只需要在settings.py中间配置的首尾各加一条）

   注：设置全局缓存后所有页面的模板数据都会应用

   >全局缓存
   >
   >```python
   >MIDDLEWARE = [
   >   'django.middleware.cache.UpdateCacheMiddleware',
   >   # 其他中间件...
   >   'django.middleware.cache.FetchFromCacheMiddleware',
   >]
   >```