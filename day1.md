## Ⅰ、hello world

### 1.1 创建django项目

```powershell
[root@ezail ~]# django-admin startproject Pro

项目目录结构
[root@ezail ~]# tree Pro
Pro
├── manage.py			用于项目交互的命令行工具
└── Pro
    ├── __init__.py
    ├── settings.py		配置文件
    ├── urls.py			url声明
    └── wsgi.py			web服务入口

1 directory, 5
```

### 1.2 启动服务

```powershell
启动服务
cd Pro
python manage.py runserver
指定端口启动服务
python manage.py runserver 80
指定监听地址与端口启动服务
python manage.py runserver 0.0.0.0:80

[root@ezail Pro]# python manage.py runserver 0.0.0.0:80
Performing system checks...

System check identified no issues (0 silenced).

You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

July 22, 2019 - 14:13:05
Django version 1.11, using settings 'Pro.settings'
Starting development server at http://0.0.0.0:80/
Quit the server with CONTROL-C.

开新session请求该服务：
[root@ezail ~]# curl -I 127.0.0.1:80
HTTP/1.0 200 OK
Date: Mon, 22 Jul 2019 14:14:14 GMT
Server: WSGIServer/0.2 CPython/3.6.5
Content-Type: text/html
X-Frame-Options: SAMEORIGIN
Content-Length: 1716

原session日志：
[22/Jul/2019 14:14:14] "HEAD / HTTP/1.1" 200 1716
```

### 1.3 新建django app

```powershell
[root@ezail Pro]# python manage.py startapp app
[root@ezail Pro]# tree app
app
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py

1 directory, 7 files
```

### 1.4 配置Pro中url

```python
vim Pro/urls.py
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^app/', include("app.urls")),
]
```

1.5 激活app

```python
vim Pro/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app',
]
```

### 1.6 编写视图函数

```python
vim app/views.py
from django.http import HttpResponse

def index(request):
	return HttpResponse("hello world!!!")
```

### 1.7 配置app的url

```python
vim app/urls.py
from django.conf.urls import url
from .views import index
urlpatterns = [
	url(r'^hello/', index, name='index')
]
```

### 1.8 请求验证

```powershell
新开session：
[root@ezail ~]# curl 127.0.0.1:80/app/hello/
hello world!!!

原session日志：
[22/Jul/2019 14:33:33] "GET /app/hello/ HTTP/1.1" 200 14
```



## Ⅱ、HttpRequest对象

```python
函数视图接受一个request对象作为参数，返回一个HttpResponse对象
request对象是在客户端发送请求后命中urls中hello后由django创建的，然后调用index函数，将request对象传给index函数(这里先理解成只传了这个参数)

def index(request):
    print(request)
    print(dir(request))
    return HttpResponse("hello world!!!")

添加两个print，再请求服务，服务端打印出如下内容：
<WSGIRequest: GET '/app/hello/'>	//wsgi的request对象，请求方式是GET，请求地址是/app/hello/
//request属性
['COOKIES', 'FILES', 'GET', 'META', 'POST', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_encoding', '_get_post', '_get_raw_host', '_get_scheme', '_initialize_handlers', '_load_post_and_files', '_mark_post_parse_error', '_messages', '_post_parse_error', '_read_started', '_set_post', '_stream', '_upload_handlers', 'body', 'build_absolute_uri', 'close', 'content_params', 'content_type', 'csrf_processing_done', 'encoding', 'environ', 'get_full_path', 'get_host', 'get_port', 'get_raw_uri', 'get_signed_cookie', 'is_ajax', 'is_secure', 'method', 'parse_file_upload', 'path', 'path_info', 'read', 'readline', 'readlines', 'resolver_match', 'scheme', 'session', 'upload_handlers', 'user', 'xreadlines']
[22/Jul/2019 15:52:51] "GET /app/hello/ HTTP/1.1" 200 14

常用属性：schema body path method encoding GET POST META
常用方法：get_host() get_port() get_full_path() is_secure() is_ajax()
```



## Ⅲ、HttpResponse对象

传递一个字符串作为页面的内容到HttpResponse构造函数

```python
from django.http import HttpResponse
response = HttpResponse("Here is the text of the web page")
response = HttpResponse("Text only, pls.", content_type="text/plain")

常见属性：content charset status_code reason_phrase

完整实例化方法：
HttpResponse.__init__(content='', content_type=None, status=200, reason=None, charset=None)
```

看一下HttpResponse源码

```python
class HttpResponse(HttpResponseBase):
    """
    An HTTP response class with a string as content.

    This content that can be read, appended to, or replaced.
    """

    streaming = False

    def __init__(self, content=b'', *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Content is a bytestring. See the `content` property methods.
        self.content = content
        
这里看只提供了一个content参数，也就是我们传的"hello world!!!"，也可以写成(content="hello world!!!")，默认为空

可以看到这个类继承了HttpResponseBase这个类
class HttpResponseBase:
    """
    An HTTP response base class with dictionary-accessed headers.

    This class doesn't handle content. It should not be used directly.
    Use the HttpResponse and StreamingHttpResponse subclasses instead.
    """

    status_code = 200

    def __init__(self, content_type=None, status=None, reason=None, charset=None):
        # _headers is a mapping of the lowercase name to the original case of
        # the header (required for working with legacy systems) and the header
        # value. Both the name of the header and its value are ASCII strings.
        self._headers = {}
        self._closable_objects = []
        # This parameter is set by the handler. It's necessary to preserve the
        # historical behavior of request_finished.
        self._handler_class = None
        self.cookies = SimpleCookie()
        self.closed = False
        if status is not None:
            try:
                self.status_code = int(status)
            except (ValueError, TypeError):
                raise TypeError('HTTP status code must be an integer.')

            if not 100 <= self.status_code <= 599:
                raise ValueError('HTTP status code must be an integer from 100 to 599.')
        self._reason_phrase = reason
        self._charset = charset
        if content_type is None:
            content_type = '%s; charset=%s' % (settings.DEFAULT_CONTENT_TYPE,
                                               self.charset)
        self['Content-Type'] = content_type
        
这样就可以很清楚看到这个类实例化的时候怎么传参了
```



## Ⅳ、JsonResponse

```python
import json

def index(request):
    data = {
        "name": "ezail",
        "age": "26"
    }
    data_li = ["flask","django"]
    
    //return HttpResponse(data) 这是错误的，刚才已经看了源码，只能传字符串
    //return HttpResponse(json.dumps(data)) 此时将data转为json，但其实返回是个长的像json的字符串
    //return HttpResponse(json.dumps(data), content_type="application/json") 这样指定content_type才会返回真正json，这个用法对data_li这种列表也能用，但是这种方法不专业
```

用JsonReponse替代上述操作

看下JsonResponse源码

```python
class JsonResponse(HttpResponse):
    """
    An HTTP response class that consumes data to be serialized to JSON.

    :param data: Data to be dumped into json. By default only ``dict`` objects
      are allowed to be passed due to a security flaw before EcmaScript 5. See
      the ``safe`` parameter for more information.
    :param encoder: Should be a json encoder class. Defaults to
      ``django.core.serializers.json.DjangoJSONEncoder``.
    :param safe: Controls if only ``dict`` objects may be serialized. Defaults
      to ``True``.
    :param json_dumps_params: A dictionary of kwargs passed to json.dumps().
    """

    def __init__(self, data, encoder=DjangoJSONEncoder, safe=True,
                 json_dumps_params=None, **kwargs):
        if safe and not isinstance(data, dict):
            raise TypeError(
                'In order to allow non-dict objects to be serialized set the '
                'safe parameter to False.'
            )
        if json_dumps_params is None:
            json_dumps_params = {}
        kwargs.setdefault('content_type', 'application/json')
        data = json.dumps(data, cls=encoder, **json_dumps_params)
        super().__init__(content=data, **kwargs)
        
JsonResponse继承于HttpResponse
data参数可以为dict或者list，safe参数默认为True，如果data为list则必须将safe置为False，否则会抛出异常
强行指定content_type为application/json，将data转为json最后交给HttpResponse


from django.http import HttpResponse, JsonResponse
return JsonResponse(data)
return JsonResponse(data_li, safe=False)
```



## Ⅴ、template

### 5.1 加载模板

django.template.loader

get_template(template_name, using=None)

```python
vim app/views.py
from django.template import Context, loader

def index_template(request):
    //get_template会在app中templates文件中寻找传进来的参数test.html
    t = loader.get_template("test.html")
    //print(t) t是一个temlate对象
    //<django.template.backends.django.Template object at 0x0000011AE04374E0>
    context = {"name": "hello template!!!"}
	
    //t.render做一个渲染的工作，将模板中的变量替换为字符串，最后传给HttpResponse
    return HttpResponse(t.render(context, request))

template的render方法源码：
def render(self, context):
    "Display stage -- can be called many times"
    with context.render_context.push_state(self):
        if context.template is None:
            with context.bind_template(self):
                context.template_name = self.name
                return self._render(context)
        else:
            return self._render(context)
```

### 5.2 创建模板

```html
mkdir app/templates
vim app/templates/test.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ name }}
</body>
</html>
```

### 5.3 配置路由并验证

```python
vim app/urls
from django.conf.urls import url
from .views import index, index_template

urlpatterns = [
    url(r'^hello/', index, name='index'),
    url(r'^hellot/', index_template, name='index_template'),
]

新开session验证：
[root@ezail app]# curl 127.0.0.1:80/app/hellot/
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    hello template!!!
</body>
</html>
```

### 5.4 封装成快捷写法

```python
def index_template(request):
    context = {"name": "hello world!!!"}

    return render(request, 'test.html', context)
```



## Ⅵ、GET与POST请求

### 6.1 看看web请求的内容

```python
def index(request):
    if 'GET' == request.method:
        print(request.GET)
    elif 'POST' == request.method:
        print(request.POST)

    return HttpResponse("")
```

**测试**

```powershell
session 2:
[root@ezail app]# curl 127.0.0.1:80/app/hello/
[root@ezail app]# curl 127.0.0.1:80/app/hello/?name=ezail

session 1:
<QueryDict: {}>
[23/Jul/2019 08:31:00] "GET /app/hello/ HTTP/1.1" 200 0
<QueryDict: {'name': ['ezail']}>
[23/Jul/2019 08:31:20] "GET /app/hello/?name=ezail HTTP/1.1" 200 0
```

发现请求参数保存在一个叫QueryDict的东西里面

### 6.2 QueryDict

在HttpRequest对象中，GET和POST属性是django.http.QueryDict的实例，是一个自定义的类似字典的类，用来处理同一个key带有多个value，这个类的需求来自html表单元素传递多个value给同一个key

request.POST和request.GET的QueryDict在一个正常请求/响应循环中是不可变的，若要获得可变版本，必须使用copy()

```python
def index(request):
    if 'GET' == request.method:
        request.GET['name'] = 'allen'	//这是错的，不能修改
        data = request.GET.copy()
        data['name'] = 'allen'
        
        name = request.GET.get('name')	//get取的是QueryDict中key为name的value列表中的最后一个值，结果是字符串
        name = request.Get.getlist('name')	//取key为name的value整个列表，结果是列表
       
    elif 'POST' == request.method:
        print(request.POST)

    return HttpResponse("")
```

### 6.3 QueryDict实例化

```python
QueryDict.__init__(query_string=None, Mutable=False, encoding=None)
这个string其实就是浏览器请求时url中最后的字符串
示例：
from django.http import QueryDict
QueryDict('a=1&a=2&b=3')
<QueryDict: {'a': ['1', '2'], 'b': ['3']}>
```

1.11新增通过fromkeys实例化QueryDict

```python
classmethod QueryDict.fromkeys(iterable, value='', mutable=False, encoding=None)

QueryDict.fromkeys(['a','a','b'], value='val')
<QueryDict: {'a': ['val', 'val'], 'b': ['val']}>
```

### 6.4 QueryDict方法

```python
QueryDict.get(key, default=None)
QueryDict.setdefault(key, default=None)[source]
QueryDict.update(other_dict)
QueryDict.items()
QueryDict.values()
QueryDict.copy()
QueryDict.getlist(key, default=None)
QueryDict.setlist(key, list_)[source]
QueryDict.appendlist(key, item)
QueryDict.setlistdefault(key, default_list=None)
QueryDict.lists()
QueryDict.pop(key)
QueryDict.popitem()
QueryDict.dict()
QueryDict.urlencode(safe=None)
```



## Ⅶ、数据库同步

### 7.1 编辑配置文件

```python
项目下settings.py文件中key名为DATABASE处
vim Pro/settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
```

### 7.2 同步数据

```powershell
[root@ezail Pro]# python manage.py showmigrations
admin
 [ ] 0001_initial
 [ ] 0002_logentry_remove_auto_add
app
 (no migrations)
auth
 [ ] 0001_initial
 [ ] 0002_alter_permission_name_max_length
 [ ] 0003_alter_user_email_max_length
 [ ] 0004_alter_user_username_opts
 [ ] 0005_alter_user_last_login_null
 [ ] 0006_require_contenttypes_0002
 [ ] 0007_alter_validators_add_error_messages
 [ ] 0008_alter_user_username_max_length
contenttypes
 [ ] 0001_initial
 [ ] 0002_remove_content_type_name
sessions
 [ ] 0001_initial
[root@ezail Pro]# python manage.py sqlmigrate sessions 0001
BEGIN;
--
-- Create model Session
--
CREATE TABLE `django_session` (`session_key` varchar(40) NOT NULL PRIMARY KEY, `session_data` longtext NOT NULL, `expire_date` datetime(6) NOT NULL);
CREATE INDEX `django_session_expire_date_a5c62663` ON `django_session` (`expire_date`);
COMMIT;
[root@ezail Pro]# python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
  
  [root@ezail Pro]# python manage.py dbshell
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 5.7.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

(django@129.211.68.34) [django]> show tables;
+----------------------------+
| Tables_in_django           |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
10 rows in set (0.00 sec)
```



## Ⅷ、创建django用户

**法1：django shell**

```powershell
[root@ezail Pro]# python manage.py shell
Python 3.6.5 (default, Jul 22 2019, 17:38:36) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from django.contrib.auth.models import User
>>> User.objects.create_user('ezail','ezail@163.com','123456')
<User: ezail>
```



**法2：命令行**

```powershell
[root@ezail Pro]# python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@163.com
Password: 
Password (again): 
Superuser created successfully.
```

Django不会在user模型上存储原始密码，而是保存为一个hash，所以不能直接操作user的password属性修改密码，这也是创建一个user时要使用辅助函数的原因

如下方法修改密码：

```powershell
>>> from django.contrib.auth.models import User
>>> u = User.objects.get(username='ezail')
>>> u.set_password('654321')
>>> u.save()
```



## Ⅸ、用户登陆

### 9.1 登陆验证过程

相当于一段伪代码

```python
from django.contrib.auth.models import User
from django.contrib.auth import login, logout, authenticate

def user_login(request):
    print(request.GET)
    # 1.获取提交过来的用户名
    username = request.GET.get('username')
    password = request.GET.get('password')
    try:
        u = User.objects.get(username=username)
    except User.DoesNotExist:
        pass
    except User,MultipleObjectsReturned:
        pass
    except Exception as e:
        print(e.args)

    # 2.根据用户名从数据库里取出这条记录
    # 2.1不存在
    # 2.2用户存在
    # 3.存在，判断密码是否一致
    u.check_password(password)

    return render(request, 'login.html')
```

### 9.2 标准写法

```python
vim app/views.py 
from django.contrib.auth.models import User
from django.contrib.auth import login, logout, authenticate

def user_login(request):
    username = request.GET.get('username')
    password = request.GET.get('password')
    user_obj = authenticate(username=username, password=password)

    if user_obj:
        login(request, user_obj)
        print("登陆成功")
    else:
        print("登陆失败")

    return render(request, 'login.html')

vim app/templates/login.html 
<ul>
    <form method='get' action='#'>
        <li>用户名：<input type='text' name='username'></li>
        <li>密码：<input type='password' name='password'></li>
        <li><input type='submit'></li>
    </form>
</ul>

vim app/urls.py
from django.conf.urls import url
from .views import index, index_template, user_login

urlpatterns = [
    url(r'^hello/', index, name='index'),
    url(r'^hellot/', index_template, name='index_template'),
    url(r'^user_login/', user_login, name='user_login')
]

浏览器验证请求
在浏览器分别输入错误和正确的账号密码，服务端日志如下：
登陆失败
[24/Jul/2019 08:50:56] "GET /app/user_login/?username=Ezail&password=123 HTTP/1.1" 200 217
登陆成功
[24/Jul/2019 08:51:08] "GET /app/user_login/?username=ezail&password=654321 HTTP/1.1" 200 217

浏览器cookie中的session
vnapqg0dxw7pcle73hxm2l9yuldy8qab
和下面django数据库中当前会话session对比是一致的
[root@ezail Pro]# python manage.py dbshell   
(django@129.211.68.34) [django]> select session_key from django_session;
+----------------------------------+
| session_key                      |
+----------------------------------+
| vnapqg0dxw7pcle73hxm2l9yuldy8qab |
+----------------------------------+
1 row in set (0.00 sec)
```

### 9.3 换post请求

request.POST = QueryDict(request.body)

```python
login.html中将get改为post

def user_login(request):
    if "GET" == request.method:
        return render(request, 'login.html')
    elif "POST" == request.method:
    	username = request.POST.get('username')
    	password = request.POST.get('password')
    	user_obj = authenticate(username=username, password=password)

    	if user_obj:
        	login(request, user_obj)
        	print("登陆成功")
    	else:
        	print("登陆失败")

    return render("")
```
