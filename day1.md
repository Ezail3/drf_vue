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
