## Ⅰ、URL配置

URLconf(URL configuration)模块，包含了URL模式(简单正则表达式)到Python函数(视图)的简单映射

### 1.1 Django如何处理一个请求

- django加载ROOT_URLCONF指定的模块，寻找可用urlpatterns(django.conf.urls.url()实例类型的列表)
- django一次匹配每个url，在与请求的url匹配的第一个url处停住
- 一旦其中一个正则表达式匹配上，Django将导入并调用给出的视图(一个简单函数或基于类的视图)
- 视图将获得如下参数
  - 一个HttpRequest实例
  - *args   匹配的正则表达式返回了没有命名的组，匹配内容将作为位置参数传给视图
  - **kwargs   由正则表达式匹配的命名组组成，可以被django.conf.urls.url()的可选参数kwargs覆盖
- 如果没有匹配到正则表达式，或者过程中抛出一个异常，Django将调用一个适当的错误处理视图

urlpatterns中每个正则表达式在第一次被访问时候被编译

### 1.2 url函数

```python
url(regex, views, kwargs=None, name=None)

regex：一个字符串(原始字符串)或简单正则表达式
views：一个视图函数或as_view()结果(基于类的视图)
kwargs：传递额外的参数给视图
name：url名称
```

### 1.3 include

```python
include(module, namespace=None, app_name=None)
include(pattern_list)
include((pattern_list, app_namespace), namespace=None)
include((pattern_list, app_namespace), instance_namespace)

module：URLconf模块
namespace：URL的命名空间
app_name：app的命名空间
pattern_list：可迭代的django.conf.urls.url()实例
app_namespace：应用命名空间
instance_namespace：实例命名空间

from django.conf.urls import url, include
from .views import index, index_template, user_login, user_logout

urlpatterns = [
    url(r'^usr/', include({
        url(r'^login$', user_login),
        url(r'^logout$', user_logout)
    }))
]
```

### 1.4 url参数

**位置参数**

若要从url中捕获一个值，则在它两边放置一对圆括号

```python
vim app/urls.py
from django.conf.urls import url
from .views import index, index_template, user_login, articles

urlpatterns = [
    #url(r'articles/(2018)$', articles, name='articles')
    url(r'^articles/([0-9]{4})$', articles, name='articles')
    #url(r'^articles/([0-9]{4})/([0-9{2}])$', articles, name='articles')
    #url(r'^articles/([0-9]{4})/([0-9{2}])/([0-9]+)$', articles, name='articles')
]

vim app/views.py
def articles(request, *args, **kwargs):
    return HttpResponse(args[0]) 

浏览器会返回最后的四位数字

参数保存在args这个元组中
请求地址：/articles/2003/03/03
调用函数：views.article(request,"2003","03","03")
```

**关键字参数**

```python
(?P<name>pattern)

name是传给视图参数的名字
pattern是一个正则表达式，也是关键字参数的值

urlpatterns = [
    #url(r'articles/(2018)$', articles, name='articles')
    url(r'^articles/(?P<year>[0-9]{4})$', articles, name='articles')
    #url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})$', articles, name='articles')
    #url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]+)$', articles, name='articles')
]

参数保存在kwargs这个字典中
请求地址：/articles/2003/03/03
调用函数：views.article(request, year="2003", month="03", day="03")
```

**额外参数**

URLconfs有一个钩子，让你传递一个Python字段作为额外的参数传递给视图函数

django.conf.urls.url()函数可以接收一个可选的第三个参数，它是一个字典，表示想要传递给视图函数的额外关键字参数

```python
urlpatterns = [
    url(r'^blog/(?p<year>[0-9]{4})$', views.function, {'foo': 'bar'})
]

请求地址：/blog/2015
调用函数：views.function(request, year='2015', foo='bar')
```



## Ⅱ、类视图

视图是一个可调用对象，它接收一个请求然后返回一个响应，这个可调用对象可以不只是函数，Django提供一些可以用作视图的类

基于类的视图使用python对象实现视图，提供函数视图之外的另一种方式

### 2.1 http请求需求

```tex
用户

	=>	GET
		 1.返回所有用户列表	/users/
		 2.返回指定用户信息	/users/23/
	=>	POST
		 修改用户信息 /users/23
				body: {
					username: "ezail"
				}
	=》	PUT
		 创建一个用户
			/users/23
				{}
	=》 DELETE
		 删除一个用户 /users/23/
```

### 2.2 伪代码

```python
views.py
class Users(object):
    http_method_names = ["get","post","put","delete"]

    def as_view(self):
        def view(request, *args, **kwargs):
	    self.request = request
	    self.args = args
	    self.kwargs = kwargs
	    return self.dispath(request, *args, **kwargs)
	return view

    def dispath(self, request, *args, **kwargs):
	def view(request, *args, **kwargs):
	    if request.method.lower() in http_method_names:
		handle = getattr(self, request.method.lower())
	    else:
		return self.http_method_not_allowed
		
	return handle(request, *args, **kwargs)	
		
    def http_method_not_allowed(self, request, *args, **kwargs):
	return HttpResponse("")
		
    def get(self, request, *args, **kwargs):
	# 返回用户列表
	return HttpResponse("")
		
    def post(self, request, *args, **kwargs):
	# 修改用户信息
	return HttpResponse("")
		
    def put(self, request, *args, **kwargs):
	# 创建用户
	return HttpResponse("")
		
    def delete(self,request, *args, **kwargs):
	# 删除用户		
	return HttpResponse("")
		
		
urls.py
from .views import Users

urlpatterns = [
    url(r'^xxx/', Users.as_view())
]
```

### 2.3 演示

```python
vim app/views.py

from django.views import View	
class MyView(View):
    def get(self, request, *args, **kwargs):
	return HttpResponse("hello view")
		
vim app/urls.py
from .views import Myview

urlpatterns = [
    url(r'^view/', MyView.as_view())
]

新开session验证：
[root@ezail Pro]# curl 127.0.0.1:80/app/view/
hello view
```

### 2.4 看下view的源码

嗯哼？是不是和前面的伪代码很像啊

```python
class View:
    """
    Intentionally simple parent class for all views. Only implements
    dispatch-by-method and simple sanity checking.
    """

    http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']

    def __init__(self, **kwargs):
        """
        Constructor. Called in the URLconf; can contain helpful extra
        keyword arguments, and other things.
        """
        # Go through keyword arguments, and either save their values to our
        # instance, or raise an error.
        for key, value in kwargs.items():
            setattr(self, key, value)

    @classonlymethod
    def as_view(cls, **initkwargs):
        """Main entry point for a request-response process."""
        for key in initkwargs:
            if key in cls.http_method_names:
                raise TypeError("You tried to pass in the %s method name as a "
                                "keyword argument to %s(). Don't do that."
                                % (key, cls.__name__))
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r. as_view "
                                "only accepts arguments that are already "
                                "attributes of the class." % (cls.__name__, key))

        def view(request, *args, **kwargs):
            self = cls(**initkwargs)
            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get
            self.setup(request, *args, **kwargs)
            if not hasattr(self, 'request'):
                raise AttributeError(
                    "%s instance has no 'request' attribute. Did you override "
                    "setup() and forget to call super()?" % cls.__name__
                )
            return self.dispatch(request, *args, **kwargs)
        view.view_class = cls
        view.view_initkwargs = initkwargs

        # take name and docstring from class
        update_wrapper(view, cls, updated=())

        # and possible attributes set by decorators
        # like csrf_exempt from dispatch
        update_wrapper(view, cls.dispatch, assigned=())
        return view

    def setup(self, request, *args, **kwargs):
        """Initialize attributes shared by all view methods."""
        self.request = request
        self.args = args
        self.kwargs = kwargs

    def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn't exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn't on the approved list.
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)

    def http_method_not_allowed(self, request, *args, **kwargs):
        logger.warning(
            'Method Not Allowed (%s): %s', request.method, request.path,
            extra={'status_code': 405, 'request': request}
        )
        return HttpResponseNotAllowed(self._allowed_methods())

    def options(self, request, *args, **kwargs):
        """Handle responding to requests for the OPTIONS HTTP verb."""
        response = HttpResponse()
        response['Allow'] = ', '.join(self._allowed_methods())
        response['Content-Length'] = '0'
        return response

    def _allowed_methods(self):
        return [m.upper() for m in self.http_method_names if hasattr(self, m)]
```

### 2.5 View小结

**属性**

- http_method_names

**方法**

- as_view()
- dispatch()
- http_method_not_allowed()

### 2.6 了解类视图的登陆验证

```python
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
class FooView(View):
    @method_decorator(login_required)
    def get(request, *args, **kwargs):
    return HttpResponse("hello world")
```



## Ⅲ、数据分页

### 3.1 造数据

```python
python manage.py shell

from django.contrib.auth.models import User

username = 'ezail'

for i in range(1000):
    name = "{}_{}".format(username, i)
    User.objects.create_user(name, "{}@163.com".format(name), "123456")
	
select * from user limit 0,10
select * from user limit 11,20
		
User.objects.all()[0:10]
User.objects.all()[10,20]

>>> queryset = User.objects.all()[0:10]
>>> list(queryset.values("username","email"))
[{'username': 'ezail', 'email': 'ezail@163.com'}, {'username': 'admin', 'email': 'admin@163.com'}, {'username': 'ezail_0', 'email': 'ezail_0@163.com'}, {'username': 'ezail_1', 'email': 'ezail_1@163.com'}, {'username': 'ezail_2', 'email': 'ezail_2@163.com'}, {'username': 'ezail_3', 'email': 'ezail_3@163.com'}, {'username': 'ezail_4', 'email': 'ezail_4@163.com'}, {'username': 'ezail_5', 'email': 'ezail_5@163.com'}, {'username': 'ezail_6', 'email': 'ezail_6@163.com'}, {'username': 'ezail_7', 'email': 'ezail_7@163.com'}]
```

### 3.2 写类视图

```python
vim app/views.py

from django.http import HttpResponse, JsonResponse
from django.contrib.auth.models import User
from django.views import View
import json

class MyView(View):
    def get(self, request, *args, **kwargs):
        try:
            page = int(request.GET.get("page",1))
        except:
            page = 1

        end = page *10
        queryset = User.objects.all()[end-10:end]
        data = list(queryset.values("id","username","email"))

        return JsonResponse(data, safe=False)
    
新开session验证：
curl 127.0.0.1:80/app/view/
[{"id": 1, "username": "ezail", "email": "ezail@163.com"}, {"id": 2, "username": "admin", "email": "admin@163.com"}, {"id": 3, "username": "ezail_0", "email": "ezail_0@163.com"}, {"id": 4, "username": "ezail_1", "email": "ezail_1@163.com"}, {"id": 5, "username": "ezail_2", "email": "ezail_2@163.com"}, {"id": 6, "username": "ezail_3", "email": "ezail_3@163.com"}, {"id": 7, "username": "ezail_4", "email": "ezail_4@163.com"}, {"id": 8, "username": "ezail_5", "email": "ezail_5@163.com"}, {"id": 9, "username": "ezail_6", "email": "ezail_6@163.com"}, {"id": 10, "username": "ezail_7", "email": "ezail_7@163.com"}]
```

### 3.3 paginator和page

**需求**

- 一共有多少条记录
- 一共可以分多少页
- 是否有上一页，上一页的页码
- 是否有下一页，下一页的页码
- 首页
- 尾页

**解决**

- Paginator

```python
class Paginator(object_list, per_page, orphans=0, allow_empty_first_page=True)

requried arguments:	object_list, per_page
option arguments:	orphans, allow_empty_first_page

属性：
Paginator.count			所有页面object总数
Paginator.num_pages 	 页面总数
Paginator.page_range 	 页码的范围

方法：
Paginator.page(number)	 返回一个page对象
```

- Page

```python
class Page(object_list, number, paginator)

属性：
Page.object_list	当前页面的对象列表
Page.number		    当前页的序号
Page.paginator 		Paginator对象

方法：
Page.has_next()		   	    如果有下一页，返回True
Page.has_previous()     	如果有上一页，返回True
Page.has_other_pages()  	如果有上一面或下一页，返回True
Page.next_page_number() 	返回下一页的页码 如果不存在，抛出InvalidPage异常
Page.previous_page_number() 返回上一页的页码如果不存在，抛出InvalidPage异常
Page.start_index() 		    返回当前页上的第一个对象，相对于分页列表的所有对象的序号
Page.end_index() 		    返回当前页上的最后一个对象，相对于分页列表的所有对象的序号
```

**演示**

```python
>>> from django.contrib.auth.models import User
>>> from django.core.paginator import Paginator
>>> queryset = User.objects.all()
>>> paginator = Paginator(queryset, 10)
/usr/local/python/lib/python3.6/site-packages/django/core/paginator.py:112: UnorderedObjectListWarning: Pagination may yield inconsistent results with an unordered object_list: <QuerySet [<User: ezail>, <User: admin>, <User: ezail_0>, <User: ezail_1>, <User: ezail_2>, <User: ezail_3>, <User: ezail_4>, <User: ezail_5>, <User: ezail_6>, <User: ezail_7>, <User: ezail_8>, <User: ezail_9>, <User: ezail_10>, <User: ezail_11>, <User: ezail_12>, <User: ezail_13>, <User: ezail_14>, <User: ezail_15>, <User: ezail_16>, <User: ezail_17>, '...(remaining elements truncated)...']>
  UnorderedObjectListWarning
>>> paginator.count
1002
>>> paginator.num_pages
101
>>> paginator.page_range
range(1, 102)

>>> page = paginator.page(10)
>>> page.object_list
<QuerySet [<User: ezail_88>, <User: ezail_89>, <User: ezail_90>, <User: ezail_91>, <User: ezail_92>, <User: ezail_93>, <User: ezail_94>, <User: ezail_95>, <User: ezail_96>, <User: ezail_97>]>
>>> page.number
10
>>> page.has_next()
True
>>> page.next_page_number()
11
```



## Ⅳ、Django日志

### 4.1 概念

Django使用Python内建的logging模块打印日志，Python的logging配置由四部分组成

- 记录器——Logger

  - 日志系统入口，每个logger命名都是bucket，可以向这个bucket写入需要处理的小细

  - 每个logger都有一个日志级别，日志级别表示logger将要处理的小细的严重性

    ```tex
    DEBUG		用于调试目的的底层系统该信息
    INFO		普通的系统该信息
    WARNING		较小问题
    ERROR		较大问题
    CRITICAL	致命问题
    ```

  - 写入looger的每条消息都是一条日志，每条日志具有一个日志级别，表示对应消息的严重性，每个日志记录还可以包含描述正在打印的事件的元信息

  - 挡一条消息传送给logger时，消息的日志级别将与logger的日志级别进行比较。如果消息的日志级别大于等于logger的日志级别，该消息将会继续处理，如果小于，该消息将被忽略

  - Logger一旦决定消息需要处理，它将该消息传给一个Handler

  - Logger配置

    logger对应的值是个字典，每个键都是logger的名字，每个值有事一个字典，描述了如何配置对应的Logger实例

    - level(可选)	logger级别
    - propagate(可选)    logger的传播设置
    - filters(可选)    logger的filter标识符列表
    - handlers(可选)    logger的handler标识符列表

    ```python
    vim Pro/settings.py
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'loggers': {
            "log_name": {
                "level": "DEBUG",
                "handlers": []
            }
        }
    }
    ```

- 处理程序——Handler

  - 决定如何处理logger中的每条消息，表示一个特定的日志行为，例如将消息写到屏幕、文件或网络socket
  - 与logger一样，handler也有一个日志级别，如果消息的日志级别小于handler的级别，handler将忽略该消息
  - Logger可以有多个handler，但每个handler可以有不同的日志级别，里用这种方式，可以根据消息的重要性提供不同形式的处理

  ```python
  vim Pro/settings.py
  LOGGING = {
      'handlers': {
          "handler_name": {
              "level": "DEBUG",
              "class": "logging.StreamHandler",
              "formatter": "simple"
          }
      }
  }
  ```

- 过滤器——Filter

  - Filter用于对从logger传递给handler的日志记录进行额外的控制
  - 默认情况，满足日志级别的任何消息都将被处理，通过安装一个filter，你可以对日志处理添加额外的条件，例如只允许处理来自特定源的ERROR
  - Filters还可以用于修改将要处理的日志记录的优先级。例如，如果日志记录满足特定条件，可以编写一个filter将日志记录从ERROR降为WARNING
  - Filters可以安装在logger上或者handler上，多个filter可以串联起来实现多层filter

- 格式化——Formatter

  日志记录需要转换成文本，formatter表示文本的格式，formatter通常由包含日志记录属性的Python格式字符串组成，可以编写自定义的formatter来实现自己的格式

  ```python
  vim Pro/settings.py
  LOGGING = {
      'formatters': {
          "formatter_name": {
              "format": "%(asctime)s - %(pathname)s:%(lineno)d[%(levelname)s] - %(message)s"
          },
          "simple": {
              "format": "%(asctime)s %(levelname)s %(message)s"
          }
      }
  }
  ```

  ### 4.2 演示

  ```python
  vim app/views.py
  import logging
  logger = logging.getLogger("my_log")
  
  class MyView(View):
      def get(self, request, *args, **kwargs):
          logger.debug("这是第一条日志")
          try:
              page = int(request.GET.get("page",1))
          except:
              page = 1
  
          end = page *10
          queryset = User.objects.all()[end-10:end]
          data = list(queryset.values("id","username","email"))
  
          return JsonResponse(data, safe=False)
  
  vim Pro/settings.py
  LOGGING = {
      "version": 1,
      "disable_existing_loggers": False,
      "formatters": {
          "verbose": {
  			"format": "%(asctime)s - %(pathname)s:%(lineno)d[%(levelname)s] - %(message)s",
  			},
              "simple": {
                  "format": "%(asctime)s %(levelname)s %(message)s",
              },
  			"default": {
  				"format": "%(asctime)s %(levelname)s %(pathname)s[%(lineno)s] %(message)s",
  				"datefmt": "%Y-%m-%d %H:%M:%S",
  			},		
          },
      "handlers": {
          "console": {
              "level": "DEBUG",
              "class": "logging.StreamHandler",
              "formatter": "default",
          },
          "file": {
              "level": "DEBUG",
              "class": "logging.FileHandler",
              "filename": "/tmp/debug.log",
              "formatter": "verbose",
          },
      },
      "loggers": {
          "my_log": {
              "level": "DEBUG",
              "handlers": ["console", "file"],
              "propagate": False,
          },
      },
  }
  
  启动服务，请求view视图验证
  控制台输出如下：
  [root@ezail Pro]# python manage.py runserver 0.0.0.0:80 
  Performing system checks...
  
  System check identified no issues (0 silenced).
  August 06, 2019 - 03:09:51
  Django version 1.11, using settings 'Pro.settings'
  Starting development server at http://0.0.0.0:80/
  Quit the server with CONTROL-C.
  2019-08-06 03:09:55 DEBUG /root/Pro/app/views.py[46] 这是第一条日志
  2019-08-06 03:09:55 WARNING /root/Pro/app/views.py[55] 这是第二条日志
  [06/Aug/2019 03:09:55] "GET /app/view/ HTTP/1.1" 200 613
  
  日志输出如下：
  [root@ezail Pro]# tailf /tmp/debug.log 
  2019-08-06 03:09:55,076 - /root/Pro/app/views.py:46[DEBUG] - 这是第一条日志
  2019-08-06 03:09:55,081 - /root/Pro/app/views.py:55[WARNING] - 这是第二条日志
  ```

  ### 4.3 django内置日志

  ```python
  有个层级关系
  django
  django.request
  diango.db.backends
  django.security.*
  dijango.db.backends.schema
  
  propagate属性		向上传播
  如果关闭，则自己管自己，打开子的日志会传给父
  
  默认像这样写，代表当前模块
  logger = logging.getLogger(__name__)
  
  日志切割：
  "file": {
      "level": "DEBUG",
  	"class": "logging.handlers.TimeRotatingFileHandler",
  	"filename": "/tmp/debug.log",
  	"when": "D",
  	"interval": 7,
  	"formatter": "default"
  }
  ```
