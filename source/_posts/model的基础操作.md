---
title: model 基础操作
date: 2020-11-22 18:20:46
tags: [python,Django,model]
---

## model基础操作

- ### 创建Meta源信息：

```python
from django.db import models

class UserInfo(models.Model):
    username=models.CharField(max_length=32)
    password=models.CharField(max_length=32)

    class Meta:
        #1数据库生成的表名称 默认app名称+下划线+类名
        db_table='table_name'  #自己指定创建表名

        #Django admin 中显示的表名称
        verbose_name='user'  #在Django admin后台显示时表名是users

        #3verbose_name_plural='user'
        verbose_name_plural='user' #若果这个字段也是user那么4中表名才显示user

        #4联系唯一索引
        unique_together=(('name'))

        # 5 联合索引 (缺点是最左前缀，写SQL语句基于password时不能命中索引，查找慢)
        # 如：select * from tb where password = ‘xx’ 就无法命中索引
        index_together=[
            ('name')
        ]
```



- ### 常用字段：

```python
class UserGroup(models.Model):
    uid=models.AutoField(primary_key=True)

    name=models.CharField(max_length=32,null=True,blank=True)
    email=models.EmailField(max_length=32)
    text=models.TextField()
    ctime = models.DateTimeField(auto_now_add=True)  # 只有添加时才会更新时间
    uptime = models.DateTimeField(auto_now=True)  # 只要修改就会更新时间
    ip1 = models.IPAddressField()  # 字符串类型，Django Admin以及 ModelForm中提供验证 IPV4 机制
    ip2 = models.GenericIPAddressField()  # 字符串类型，Django Admin以及 ModelForm中提供验证 Ipv4和Ipv6
    active = models.BooleanField(default=True)
    data01 = models.DateTimeField()  # 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ
    data02 = models.DateField()  # 日期格式 YYYY-MM- DD
    data03 = models.TimeField()  # 时间格式 HH:MM[:ss[.uuuuuu]]
    age = models.PositiveIntegerField()  # 正小整数 0 ～ 32767
    balance = models.SmallIntegerField()  # 小整数 -32768 ～ 32767
    money = models.PositiveIntegerField()  # 正整数 0 ～ 2147483647
    bignum = models.BigIntegerField()  # 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807
    user_type_choices = (
        (1, "超级用户"),
        (2, "普通用户"),
        (3, "普普通用户"),
    )
    user_type_id = models.IntegerField(choices=user_type_choices, default=1)
```



- ### 参数：

```python
    null=True    #数据库字段是否可以为空
    blank=True    #表单验证可以为空
    default=True   #数据库中字段的默认值
    parmary_key=True  #数据库中的字段是为主键
    db_index=True      #数据库中字段是否可以建立索引
    unique=True       #数据库字段是否可以建立唯一属性
```



## django一对多表结构操作

- ### 一对多基本增删改查

>models.py

```python
from django.db import models

class UserInfo(models.Model):
    name=models.CharField(max_length=64,unique=True)
    ut=models.ForeignKey(to='UserType',on_delete=models.CASCADE)

class UserType(models.Model):
    type_name=models.CharField(max_length=64,unique=True)
```

>views.py

```python
from django.shortcuts import HttpResponse
from .models import *

def orm(request):
    1.创建
    创建数据方法一
    UserInfo.objects.create(name='roo',ut_id=1)
    创建数据库方法二
    users=UserInfo(name='root',ut_id=2)
    users.save()
    创建数据库方法三
    dict={'name':'r','ut_id':'1'}
    UserInfo.objects.create(**dict)

    删除
    UserInfo.objects.all().delete()  #删除所有
    UserInfo.objects.filter(name='r').delete()  #删除指定

    修改
    UserInfo.objects.all().update(ut_id=1)
    UserInfo.objects.filter(name='ro').update(ut_id=2)

    查找
    4.1正向查找 
    user_obj.ut.type_name
    print(UserInfo.objects.get(name='ro').ut.type_name)
    print(UserInfo.objects.filter(ut__type_name='学生'))

   4.2反向查找 
    type_obj.user_info_set.all()
    print(UserType.objects.get(type_name='学生').userinfo_set.all())
    print(UserType.objects.get(type_name='老师').userinfo_set.filter(name='ro'))

    return HttpResponse('ok')
```



- ### 一对多查找更多操作

>models.py

```python
from django.db import models


class UserType(models.Model):
    user_type_name=models.CharField(max_length=32)
    def __str__(self):
        return self.user_type_name   #只有加上这个,Django admin才会显示表名

class User(models.Model):
    username=models.CharField(max_length=32)
    pwd=models.CharField(max_length=64)
    ut=models.ForeignKey(
        to='UserType',
        to_field='id',
        on_delete=models.CASCADE,
        related_query_name = 'a',
    )
# 1、反向操作时，使用的连接前缀，用于替换【表名】
# 如： models.UserGroup.objects.filter(a__字段名=1).values('a__字段名')


        #2丶反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.b_set.all()
        # 使用时查找报错
        # related_name='b',

```

>views.py

```python
from django.shortcuts import render,HttpResponse
from .models import *

def many_orm(request):
    # 1.正向查找
    # 1.1 正向查找user表用户名
    print(User.objects.get(username='xiaowang').username)
    # xiaowang
    # 1.2正向跨表查找用户类型
    print(User.objects.get(username='xiaowang').ut.user_trpe_name)
    # 老师
    print(User.objects.all().values('ut__user_type_name','username'))
    # <QuerySet [{'ut__user_trpe_name': '老师', 'username': 'xiaowang'}, {'ut__user_type_name': '学生', 'username': 'wangsai'}]>

    # 2.反向查找
    # 2.1【表名_set】 反向查找user表中用户类型为老师的所有用户
    print(UserType.objects.get(user_trpe_name='老师').user_set.all())
    # <QuerySet [<User: User object (1)>]>
    # 2.2【a__字段名】反向查找user表中张三在UserType表中的类型:(<[UserType:老师>])
    print(UserType.objects.filter(a__username='xiaowang'))
    # <QuerySet [<UserType: 老师>]>
    2.3双下划线跨表反向查找
    print(UserType.objects.all().values('a__username','user_type_name'))
    # <QuerySet [{'a__username': 'xiaowang', 'user_type_name': '老师'}, {'a__username': 'wangsai', 'user_type_name': '学生'}]>

    #3自动创建User表和UserType表中的数据
    '''

    user_type = [{'user_type_name':'student'},{'user_type_name':'teacher'},]

    for type_dict in user_type:
        UserType.objects.create(**type_dict)

    for user_dic in username:
        User.objects.create(**user_dic)
    '''
    return HttpResponse('yes')
```



- ### *一对多使用values和values_list结合双下划线跨表查询*：

```python
from django.shortcuts import HttpResponse 
from .models import *
#一对多使用values和values_list结合双下划线跨表查询
def get_orm(request):
    # 第一种: values----获取的内部是字典  拿固定列数
    # 1.1正向查找 ： 使用ForeignKey字段名ut结合双下划线查询
    suts=User.objects.filter(username='xiaowang').values('username','ut__user_type_name')
    print(suts)
    # <QuerySet [{'username': 'xiaowang', 'ut__user_type_name': '老师'}]>
    # 1.2反向查找:使用ForeignKey的related_query_name='a',的字段
    obj=UserType.objects.all().values('user_type_name','a__username')
    print(obj)
    # <QuerySet [{'user_type_name': '老师', 'a__username': 'xiaowang'}, {'user_type_name': '学生', 'a__username': 'wangsai'}]>

    # 第二种 : values_list-----获取的是元祖  拿固定列数
    # 1.1 正向查找 ：使用ForeignKey字段名ut结合双下划线查询
    stus=User.objects.filter(username='xiaowang').values_list('username','ut__user_type_name')
    print(stus)
    # <QuerySet [('xiaowang', '老师')]>

    # 1.2反向查找
    utype=UserType.objects.all().values_list('user_type_name','a__username')
    print(utype)
    # <QuerySet [('老师', 'xiaowang'), ('学生', 'wangsai')]>
    return HttpResponse('ok')
```



- ### 一对多ForeignKey可选参数：

```python
to      要进行关联的表名
to_field-None      # 要关联的表中的字段名称
on_delete=None     # 当删除关联表的数据时，当前表与其无关联的行的行为

models.CASCADE      # 删除关联数据时，与之关联也删除
models.DO_NOTHING   # 删除关联数据，引发错误IntegrityError
models.PROTECT      # 删除关联数据，引发错误ProtectedError
models.SET_NULL     # 删除关联数据，与之关联的值设置为null (前提FK字段需要设置为空)
models.SET_DRFAULT  # 删除关联数据，与之关联的值设置为默认值
models.SET    # 删除关联数据,
4.related_name=None,  # 反向操作是，使用的字段名，用于代替【表名_set】如obj.表名_set.all()在做自关联时必须指定此字段,防止查找冲突
5.delated_query_name=None  # 反向操作是,使用的连接前缀，用于替换【表名】

models.UserGroup.objects.filter(表名__字段名1).values('表名__字段名')
```



## django多对多表结构操作

- ### 第一种：ManyToManyField创建：

```python
from django.db import models
#自己不创建第三张表，有m2m字段：根据queryset对象增删改查(推荐)

class UserInfo(models.Model):
    username=models.CharField(max_length=32)

class UserGroup(models.Model):
    group_name=models.CharField(max_length=64)
    user_info=models.ManyToManyField(to='UserInfo',related_query_name='m2m')
```



- ### 查询：

```python
user_info_obj=UserInfo.objects.get(username='zhangsan')
print(user_info_obj)
    # zhangsan
user_info_objs=UserInfo.objects.all()
print(user_info_objs)
# <QuerySet [<UserInfo: zhangsan>, <UserInfo: lisi>, <UserInfo: wangwu>]>
group_obj=UserGroup.objects.get(group_name='group_python')
print(group_obj)
# group_python
group_objs=UserGroup.objects.all()
print(group_obj)
# <QuerySet [<UserGroup: group_python>, <UserGroup: group_linux>,<UserGroup:group_mysql>]>
# 添加  ：正向
group_obj.user_info.add(user_info_obj)
group_obj.user_info.add(*user_info_objs)
# 删除 ：正向
group_obj.user_info.remove(user_info_obj)
group_obj.user_info.remove(*user_info_objs)
# 添加 ：反向
user_info_obj.usergroup_set.add(group_obj)
user_info_obj.usergroup_set.add(*group_objs)
# 删除 ：反向
user_info_obj.usergroup_set.remove(group_obj)
user_info_obj.usergroup_set.remove(*group_objs)
# 查找正向
print(group_obj.user_info.all())   #查找group_python组中所有的用户
# 查找反向
print(user_info_obj.usergroup_set.all())
# 查找用户张三属于哪个组
# <QuerySet [<UserGroup: UserGroup object (1)>]>
# 双下划线 正向 反向查找
# 正向：从用户组表中查找zhangsan属于哪个用户组：[<UserGroup: group_python>]
print(UserGroup.objects.filter(user_info__username='zhangsan'))
# <QuerySet [<UserGroup: UserGroup object (1)>]>
# 反向 ：从用户表中查询group_python组中有哪些用户：related_query_name='m2m'
print(UserInfo.objects.filter(m2m__group_name='group_python'))
# <QuerySet [<UserInfo: UserInfo object (1)>]>

# 自动创建UserInfo表和UserGroup表中的数据
 '''
      user_list = [{'username': 'zhangsan'},
                     {'username': 'lisi'},
                     {'username': 'wangwu'}, ]

        group_list = [{'group_name': 'group_python'},
                      {'group_name': 'group_linux'},
                      {'group_name': 'group_mysql'}, ]

        for c in user_list:
            UserInfo.objects.create(**c)

        for l in group_list:
            UserGroup.objects.create(**l)


      '''
```



- ### 第二种：自己创建第三张表：

>自己创建第三张表时，无m2m字段，自己创建链表查询
>
>models.py

```python
#表1：主机表
class Host(models.Model):
    nid = models.AutoField(primary_key=True)
    hostname = models.CharField(max_length=32,db_index=True)
#表2：应用表
class Application(models.Model):
    name = models.CharField(max_length=32)
#表3：自定义第三张关联表
class HostToApp(models.Model):
    hobj = models.ForeignKey(to="Host",to_field="nid")
    aobj = models.ForeignKey(to='Application',to_field='id')
# 向第三张表插入数据，建立多对多外键关联
    HostToApp.objects.create(hobj_id=1,aobj_id=2)
```



- ### 多对多双下划线查找：

```python
from django.shortcuts import HttpResponse
from app01 import models
def orm(request):
    # 第一种：values-----获取的内部是字典,拿固定列数
    # 1.1 正向查找： 使用ManyToManyField字段名user_info结合双下划线查询
	models.UserGroup.objects.filter(group_name='group_python').values('group_name',
    'user_info__username')
    # 1.2 反向查找： 使用ManyToManyField的related_query_name='m2m',的字段
    models.UserInfo.objects.filter(username='zhangsan').values('username',
    'm2m__group_name')
    # 第二种：values_list-----获取的是元组 拿固定列数
    # 2.1 正向查找： 使用ManyToManyField字段名user_info结合双下划线查询
    models.UserGroup.objects.filter(group_name='group_python').values_list('group_na
    me', 'user_info__username')
    # 2.2 反向查找： 使用ManyToManyField的related_query_name='m2m',的字段
    lesson = models.UserInfo.objects.filter(username='zhangsan').values_list('username',   'm2m__group_name')
    # 自动创建UserInfo表和UserGroup表中的数据
'''
    user_info_obj = models.UserInfo.objects.get(username='lisi')
    user_info_objs = models.UserInfo.objects.all()
   
    group_obj = models.UserGroup.objects.get(group_name='group_python')
    group_objs = models.UserGroup.objects.all()
    group_obj.user_info.add(*user_info_objs)
    user_info_obj.usergroup_set.add(*group_objs)
		user_list = [{'username':'zhangsan'},
        {'username':'lisi'},
        {'username':'wangwu'},]
    group_list = [{'group_name':'group_python'},
        {'group_name':'group_linux'},
        {'group_name':'group_mysql'},]
    for c in user_list:
        models.UserInfo.objects.create(**c)
    for l in group_list:
        models.UserGroup.objects.create(**l)
'''
    return HttpResponse('orm')
```



- ### 创建m2m时ManyToManyField可以添加参数:

```python
1、to, # 要进行关联的表名
2、related_name=None, # 反向操作时，使用的字段名，用于代替 【表名_set】如： obj.
表名_set.all()
3、related_query_name=None, # 反向操作时，使用的连接前缀，用于替换【表名】
```



## 一大波model操作

- ### 基础语句查询：

```python
class UserInfo(models.Model):
    username=models.CharField(max_length=32)
    password=models.CharField(max_length=32)

def orms(request):
    # 创建
    # 第一种创建数据方法
    UserInfo.objects.create(username='zhangsan',password='123456')
    # 第二种创建数据方法
    obj=UserInfo(username='wangwu',password='123456')
    obj.save()
    # 第三种创建数据方法
    dic={'username':'liu','password':'123456'}
    UserInfo.objects.create(**dic)

    # 查
    # 查询所有数据
    result=UserInfo.objects.all()
    print(result)
    result=UserInfo.objects.filter(username='zhangsan',password='123456')
    for row in result:
        print(row.id,row.username,row.password)
    # 1 zhangsan 123456
    # 3 zhangsan 123456

    # 删
    UserInfo.objects.all().delete()  #删除全部数据
    UserInfo.objects.filter(username='zhangsan').delete()  #删除指定

    # 改
    UserInfo.objects.all().update(password='12345')
    UserInfo.objects.filter(id=4).update(password='15')

    # 获取个数
    num=UserInfo.objects.filter(username='wangwu').count()
    print(num)
    # 3执行原生sql
    new=UserInfo.objects.raw('select * from user_info')

    # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名

    # 指定数据库
    models.UserInfo.objects.raw('select * from userinfo', using="default")
    # 每访问一次数据库中zhangsan的年纪就会自动增加1
    UserInfo.objects.filter(name='wangwu').update(age=F('age')+1)

    return HttpResponse('yes')
```



- ### 进阶操作：牛掰的双下划线：

```python
1.大于
# models.Tb1.objects.filter(id__gt=1) # 获取id大于1的值 
# models.Tb1.objects.filter(id__gte=1) # 获取id大于等于1的值
# models.Tb1.objects.filter(id__lt=10) # 获取id小于10的值 
# models.Tb1.objects.filter(id__lte=10) # 获取id小于等于10的 值 
# models.Tb1.objects.filter(id__lt=10, id__gt=1) # 获取id大于1 且 小于 10的值

2.in
# models.Tb1.objects.filter(id__in=[11, 22, 33]) # 获取id等于11、22、33的数据
# models.Tb1.objects.exclude(id__in=[11, 22, 33]) # not in

3.isnull
# Entry.objects.filter(pub_date__isnull=True) #双下划线isnull，查找
pub_date是null的数据

4、contains #就是原生sql的like操
作:模糊匹配
# models.Tb1.objects.filter(name__contains="ven")
# models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏
感
# models.Tb1.objects.exclude(name__icontains="ven")

5.range 
# models.Tb1.objects.filter(id__range=[1, 2]) # 范围bettwen and

6.order by
# models.Tb1.objects.filter(name='seven').order_by('id') # asc
没有减号升续排列
# models.Tb1.objects.filter(name='seven').order_by('-id') # desc
有减号升续排列

7.group by 
# from django.db.models import Count, Min, Max, Sum
# models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
#根据id列进行分组
# SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM
"app01_tb1"
# WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"

8.limit offset  #分页
models.Tb1.objects.all[10:20]

9.regex正则匹配，iregex 不区分大小写
# Entry.objects.get(title__regex=r'^(An?|The) +')
# Entry.objects.get(title__iregex=r'^(an?|the) +')

10.date
#Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
#__data表示日期查找，2005-01-01
# Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
# 2005-01-01以后创建的数据
11、year
# Entry.objects.filter(pub_date__year=2005)
#__year根据年查找
# Entry.objects.filter(pub_date__year__gte=2005)
12、month
# Entry.objects.filter(pub_date__month=12)
# Entry.objects.filter(pub_date__month__gte=6)
13、day
# Entry.objects.filter(pub_date__day=3)
# Entry.objects.filter(pub_date__day__gte=3)
14、week_day
# Entry.objects.filter(pub_date__week_day=2)
# Entry.objects.filter(pub_date__week_day__gte=2)
15、hour
# Event.objects.filter(timestamp__hour=23)
# Event.objects.filter(time__hour=5)
# Event.objects.filter(timestamp__hour__gte=12)
16、minute
# Event.objects.filter(timestamp__minute=29)
# Event.objects.filter(time__minute=46)
# Event.objects.filter(timestamp__minute__gte=29)
17、second
# Event.objects.filter(timestamp__second=31)
# Event.objects.filter(time__second=2)
# Event.objects.filter(timestamp__second__gte=31)
```



- ### 时间查询：

```python
from django.utils import timezone
from report.models import *
now = timezone.now()
# start_time = now - timezone.timedelta(days=7)
start_time = now - timezone.timedelta(hours=240) # 查询10天前的数据
end_time = now
qs = AllClarm.objects.filter(start_tm__range=(start_time, end_time))
```



## F()函数和Q()函数查询

- ### F()----专门取对象中某列值的操作

>作用：F()允许django在未实际连接数据的情况下具有对数据库字段值的引用

```python
from django.shortcuts import HttpResponse
from app01 import models
from django.db.models import F,Q
def orm(request):
    # 每访问一次数据库中zhangsan的年纪就会自动增加1
    models.Student.objects.filter(name='zhangsan').update(age=F("age") + 1)
    # 自动生成Student表中数据
    '''
    stu_list = [{'name':'zhangsan','age':11},
    {'name': 'lisi', 'age': 22},
    {'name': 'wangwu', 'age': 33},]
    for u in stu_list:
    models.Student.objects.create(**u)
    '''
    return HttpResponse('orm')
```



- ### Q()----复杂查询（用法1）

>1、Q对象(django.db.model.Q)可以对关键字参数进行封装，从而更好地应用多个查询
>
>2、可以使用&（and-与）；|（or-或）；~（not-非）操作符，当一个操作符用于两个对象时，它能够产生一个新对象
>
>3、如：Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')

```python
from django.shortcuts import HttpResponse
from app01 import models
from django.db.models import F,Q
def orm(request):
    # 查找学生表中年级大于1小于30姓zhang的所有学生
    stus = models.Student.objects.filter(
        Q(age__gt=1) & Q(age__lt=30),
        Q(name__startswith='zhang')
    )
    print('stu',stus) #运行结果：[<Student: zhangsan>]
    # 自动生成Student表中数据
    '''
    stu_list = [{'name':'zhangsan','age':11},
        {'name': 'lisi', 'age': 22},
        {'name': 'wangwu', 'age': 33},]
    for u in stu_list:
        models.Student.objects.create(**u)
    '''
    return HttpResponse('orm')
```



## aggregate和annotate聚合函数：求平均值、最大值、最小值等

- ### aggregate聚合函数：

>作用：从数据库中取出一个汇总的集合

```python
from django.db.models import Count,Avg,Max,Sum
def orm(request):
stus = models.Student.objects.aggregate(
stu_num=Count('age'), #计算学生表中有多少条age条目
stu_avg=Avg('age'), #计算学生的平均年纪
stu_max=Max('age'), #找到年纪最大的学生
stu_sum=Sum('age')) #将表中的所有年纪相加
print('stu',stus)
return HttpResponse('ok')
#运行结果：{'stu_sum': 69, 'stu_max': 24, 'stu_avg': 23.0, 'stu_num': 3}
```



- ### annotate实现聚合group by查询：

>作用：对查询结果进行分组，比如：分组求出各年龄段的人数
>
>注释：annotate后面加filter过滤相当于原生SQL语句中的HAVING

```python
from django.db.models import Count, Avg, Max, Min, Sum
def orm(request):
    #1 按年纪分组查找学生表中各个年龄段学生人数：（22岁两人，24岁一人）
    # 查询结果：[{'stu_num': 2, 'age': 22}, {'stu_num': 1, 'age': 24}]
    stus1 = models.Student.objects.values('age').annotate(stu_num=Count('age'))
    #2 按年纪分组查找学生表中各个年龄段学生人数，并过滤出年纪大于22的：
    # 查询结果：[{'stu_num': 1, 'age': 24}] （年级大于22岁的仅一人，年级为24岁）
    stus2 =
    models.Student.objects.values('age').annotate(stu_num=Count('age')).filter(age__g
    t=22)
    #3 先按年纪分组，然后查询出各个年龄段学生的平均成绩
    # 查询结果：[{'stu_Avg': 86.5, 'age': 22}, {'stu_Avg': 99.0, 'age': 24}]
    # 22岁平均成绩：86.5 24岁平均成绩：99
    stus3 = models.Student.objects.values('age').annotate(stu_Avg=Avg('grade'))
    return HttpResponse('ok')
```
