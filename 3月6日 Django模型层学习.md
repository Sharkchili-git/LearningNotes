# 3月6日 Django模型层学习

## 1.Django MTV中的M:model 也即模型

## 2.Django ORM框架:

- Django在数据用户之间构造了 orm框架
- orm框架: 向下屏蔽了多种数据库的差异,向上给开发者提供了很多便捷的功能
- ![1583477666375](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583477666375.png)
- #### 总结

  - 屏蔽了数据库差异
  - 提供了迁移工具
  - 简化开发流程

## 3. 数据库迁移:

- ### 迁移的必要性  sqlite3--> MySQL

  - #### sqlite3

    - 优点: sqlite3在项目开发初期便利,不需要配置数据库环境
    - 缺点: 当项目后期,项目越来越大的时候,sqlite3是文件数据库,性能就跟不上
  - #### MySQL

    - MySQL是工业界常用数据库,性能卓越、服务稳定，很少出现异常宕机
    - 优点:
      - MySQL性能卓越、服务稳定，很少出现异常宕机。
      - MySQL开放源代码且无版权制约，自主性及使用成本低。
      - MySQL历史悠久，用户使用活跃，遇到问题可以寻求帮助。
      - MySQL体积小，安装方便，易于维护。
      - MySQL口碑效应好，是的企业无需考虑就用之，LAMP、LNMP 流行架构。
      - .MySQL支持多种操作系统，提供多种API接口，支持多种开发语言。

- ### 迁移过程:

  - #### 数据库中最重要的是:

    - 数据

    - 表结构

      #迁移数据库实际上就是迁移数据库中的数据和表结构

  - #### 三步骤:

    - ##### 数据备份:

      - python manage.py  dumpdata 应用名 > 应用名.json
      - json 文件一般命名为 对应的应用名
      - 运行完之后会在项目的根目录下多一个 应用名.json文件

    - ##### 表结构同步:

      - 创建MySQL数据库并更新配置

        - 创建一个新的数据库用于存放 (用以前创建好的也行)

        - 更新Django settings.py中的 DATABASES  配置MySQL数据库

          ```python
          # 项目初始 sqlite3 数据库
          DATABASES = {
              'default': {
                  'ENGINE': 'django.db.backends.sqlite3',
                  'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
              }
           }
           
          # 配置MySQL数据库 因为还没有正式启用,并且与原先的名字相同所以先注释了
          # DATABASES = {
          #     'default': {
          #         'ENGINE': 'django.db.backends.mysql',
          #         'NAME': '数据库',
          #         'USER': '账户',
          #         'PASSWORD': '密码',
          #         'HOST': '127.0.0.1',
          #         'PORT': '3306'
          #     }
          # }
          ```

      - 在默认的数据库配置中 创建slave数据库

        - ```python
          DATABASES = {
              'default': {
                  'ENGINE': 'django.db.backends.sqlite3',
                  'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
              },
              'slave': {
                  'ENGINE': 'django.db.backends.mysql',
                  'NAME': '数据库',
                  'USER': '账户',
                  'PASSWORD': '密码',
                  'HOST': '127.0.0.1',
                  'PORT': '3306'
              }
          }
          ```

          dafault 是默认数据库  slave 是备份数据库,数据可以同步

      - 迁移数据库表

        - python manage.py migrate --run-syncdb --database slave
        - 这条命令就实现了数据库表结构同步
        - 现在打开相关数据库就可以看到 同步过来的表了,但是还没有数据

    - ##### 数据迁移:

      - 在Django settings.py文件中启用MySQL数据库配置 注释或删除原先的sqlite3数据库

        ```python
        # DATABASES = {
        #     'default': {
        #         'ENGINE': 'django.db.backends.sqlite3',
        #         'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        #     },
        #     'slave': {
        #         'ENGINE': 'django.db.backends.mysql',
        #         'NAME': '数据库',
        #         'USER': '账户',
        #         'PASSWORD': '密码',
        #         'HOST': '127.0.0.1',
        #         'PORT': '3306'
        #     }
        # }
        
        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': '数据库',
                'USER': '账户',
                'PASSWORD': '密码',
                'HOST': '127.0.0.1',
                'PORT': '3306'
            }
        }
        ```

      - python manage.py loaddata 应用名.json

         运行这条命令就将sqlite3中导出来的数据存入了MySQL数据库中了

         现在再去MySQL中看 表中就有数据了

## 4.数据库索引

- ### 索引的概念

  - 例子:书的目录

  - 数据库存储数据

    - 乱序 无索引:这样如果想找到 9 的话就比较麻烦 (从头开始找,找到9为止)

      ![1583925478904](E:\md文件\django模型层.assets\1583925478904.png)

    - 添加上索引 (可以从乱序的数据中,快速找到数据)

    - B+树![1583925493812](E:\md文件\django模型层.assets\1583925493812.png)

    - 索引的优缺点:

      - 优点:加快检索数据的速度
      - 缺点:降低了插入,删除,更新的速度

  - 

- ### 应该被索引的字段

  - 需要排序操作的字段(order_by)
  - 需要比较操作的字段(>,<,>=,<=)
  - 需要过滤操作的字段(filter,exclude)

- ### 添加索引的两种方法

  - (任何表结构的改变都需要迁移数据库)

  - 模型的字段中定义:

    -  对于模型字段添加  db_index=True 

  - 模型的Meta属性中定义:

    - 定义:每个模型类下都有一个子类Meta，这个子类就是定义元数据的地方。Meta类封装了一些数据库的信息，称之为模型的元数据。

    - Meta 中的属性参考如下网址:

      https://docs.djangoproject.com/en/dev/ref/models/options/ 

      https://www.jianshu.com/p/774a8f16d624 

      https://blog.csdn.net/bbwangj/article/details/79967858 

    - ```python
      db_table = 'xxx'  #修改表名为xxx
      indexes = [
                  # models.Index(fields=['nikename']),#将nickname字段设置为索引
                  models.Index(fields=['nickname','openid'], name='name'),#将nickname,openid两个字段设置为联合索引 并且命名为 name
              ]
      ```

## 5.数据库的关系映射

- 三种关系映射

  - 一对一

    - 例子: 父亲 <----> 母亲

  - 一对多(多对一)

    - 例子:父亲 <----> 大女儿,二女儿,三女儿

  - 多对多

    - 例子: 哥哥有多个弟弟,弟弟有多个哥哥 

    ![1583927937296](E:\md文件\3月6日 Django模型层学习.assets\1583927937296.png)

- Django表达三种关系映射

  - 一对一的关系
    - OneToOneField
  - 一对多(多对一)的关系
    - Foreignkey
  - 多对多的关系
    - ManyToManyField 

- 关系映射实战

  ![1583928900378](E:\md文件\3月6日 Django模型层学习.assets\1583928900378.png)



## 6.数据库操作

需要单独运行,并且需要用到django的模型,我们需要先建立django的环境

```
import django
import os
# myfirstproj为你自己的项目名
os.environ['DJANGO_SETTINGS_MODULE'] = 'myfirstproj.settings'
django.setup()
```

```python
def ranstr(length):
    CHS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
    salt = ''
    for i in range(length):
        salt += random.choice(CHS)
    return salt

# 添加一个用户
def add_one():
    # 1 需要 save保存  
    user = User(open_id = 'test_open_id', nickname='test_nickname')
    user.save()

    # 2 自动保存 create
    User.objects.create(open_id = 'test_open_id2', nickname='test_nickname2')

# 增：批量  bulk_create
def add_batch():
    new_user_list = []
    for i in range(10):
        open_id = ranstr(32)
        nickname = ranstr(10)
        user = User(open_id=open_id, nickname=nickname)
        new_user_list.append(user)
    User.objects.bulk_create(new_user_list)

# 查询 get
def get_one():
    user = User.objects.get(open_id='test_open_id')
    print(user)

# 数据过滤 filter
def get_filter():
    users = User.objects.filter(open_id__contains='test_')
    # open_id__startswith
    # 大于: open_id__gt(greater than)
    # 小于: open_id__lt(little than)
    # 大于等于：open_id__gte(greater than equal)
    # 小于等于：open_id__lte(little than equal)
    print(users)

# 数据排序 order_by
def get_order():
    users = User.objects.order_by('open_id')
    print(users)

# 连锁查询
# 和管道符类似
def get_chain():
    users = User.objects.filter(open_id__contains='test_').order_by('open_id')
    print(users)

# 改一个 
def modify_one():
    user = User.objects.get(open_id = 'test_open_id')
    user.nickname = 'modify_username'
    user.save()

# 批量改
def modify_batch():
    User.objects.filter(open_id__contains='test_').update(nickname='modify_uname')

def delete_one():
    User.objects.get(open_id='test_open_id').delete()

# 批量删除
def delete_batch():
    User.objects.filter(open_id__contains='test_').delete()

# 全部删除
def delete_all():
    User.objects.all().delete()
    # User.objects.delete()


```

```python
# ---------数据库函数----------------
# 数据库函数
# 字符串拼接：Concat

from django.db.models import Value
from django.db.models.functions import Concat
# annotate创建对象的一个属性, Value,如果不是对象中原有属性
def concat_function():
    user = User.objects.filter(open_id='test_open_id').annotate(
        # open_id=(open_id), nickname=(nickname)
        screen_name = Concat(
            Value('open_id='),
            'open_id',
            Value(', '),
            Value('nickname='),
            'nickname')
        )[0]
    print('screen_name = ', user.screen_name)

# 字符串长度： Length
from django.db.models.functions import Length

def length_function():
    user = User.objects.filter(open_id='test_open_id').annotate(
        open_id_length = Length('open_id'))[0]

    print(user.open_id_length)

# 大小写函数
from django.db.models.functions import Upper, Lower

def case_function():
    user = User.objects.filter(open_id='test_open_id').annotate(
        upper_open_id=Upper('open_id'),
        lower_open_id=Lower('open_id')
    )[0]
    print('upper_open_id:', user.upper_open_id, ', lower_open_id:', user.lower_open_id)
    pass

# 日期处理函数
# Now()

from apis.models import App
from django.db.models.functions import Now

def now_function():
    # 当前日期之前发布的所有应用
    apps = App.objects.filter(publish_date__lte=Now())
    for app in apps:
        print(app)


# 时间截断函数
# Trunc
from django.db.models import Count
from django.db.models.functions import Trunc


def trunc_function():
    # 打印每一天发布的应用数量
    app_per_day = App.objects.annotate(publish_day=Trunc('publish_date', 'month'))\
        .values('publish_day')\
        .annotate(publish_num=Count('appid'))

    for app in app_per_day:
        print('date:', app['publish_day'], ', publish num:', app['publish_num'])

    pass



```



















































