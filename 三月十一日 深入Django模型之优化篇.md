# 深入Django模型之优化篇

![1583929701216](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583929701216.png)

## 一.理解模型变更与迁移:

- 模型迁移的操作

  ![1583930168896](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583930168896.png)

-  迁移相关命令:

  - makemigrations: 将模型变更用Django的形式记录下来 (在 应用下的migrations文件夹下增加文件)

  - migrate: 执行模型变更,同步到数据库 (执行 应用下migrations文件夹下新增的文件)

  - sqlmigrate:显示每次迁移执行的实际sql语句

    - python manage.py sqlmigrate 应用名 迁移文件的前缀编号

  - showmigrations: 显示某个应用的模型变更和迁移的历史记录

    - python manage.py showmigrations 应用名

      [ ]中的 x 代表文件是否被迁移,有代表是

-  迁移文件详解:

  - dependencies字段
    - 数组中每一个元素都是一个二元组
    - 元素:(应用名,迁移文件名)
    - 意义:这次的迁移依赖于 元素中这个应用名下的这个迁移文件
  - operations字段
    - migrations.Addfield(字段信息): 添加一个字段
    - migrations.RemoveIndex(字段信息):删除一个字段
  - 可以自己编写迁移文件
    - 不推荐,这个功能只是为我们开发者去debug,以及查看每次迁移具体的操作

## 二.懒加载与预加载

- 懒加载的问题根源

  - ORM的不透明

    - ORM框架向下屏蔽了不同数据库的差异,向上提供了统一的操作接口,因为有ORM框架的存在我们对数据库的细节了解更少了
    - 这样也就引发了两个问题:
      - 一次查询经过ORM会不会变成两次SQL语句
      - 两次查询通过ORM能不能变成一次

  - 懒加载是什么

    - 存在于外键和多对多关系中
    - 不检索关联对象的数据
    - 调用关联对象会再次查询数据库

  - 查看Django ORM的数据加载

    ![1583933156392](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583933156392.png)

- 预加载的方法

  - 预加载单个关联对象 --> select_related (外键)
  - 预加载多个关联对象 --> prefetch_related (多对多关系)

- 性能数据对比

  - 预加载比懒加载块
  - 快多少和模型定义有关
  - 预加载一个对象的关联对象,在进行相关操作,这样对性能提升很多

  ```python
  import os
  import django
  
  # 配置环境
  os.environ['DJANGO_SETTINGS_MODULE'] = 'HelloDjango.settings'
  # os.environ.setdefault('DJANGO_SETTING_MODULE', 'HelloDjango.settings')
  django.setup()
  
  from jhapp.models import User, App
  from timeit import Timer
  
  
  # 懒加载 (慢)
  # 存在于外键和多对多关系
  # 不检索关联对象的数据
  # 调用关联对象会再次查询数据库
  # 查看django orm的数据加载,两次. 查询user,查询menu
  # 第一次查询 用户表  第二次查询 app表
  def lazy_load():
      for user in User.objects.all():
          print(user.menu.all())
          # print(user.query)
          # res = user.menu.all()
          # print(res.query)
  
  
  # 查询一条数据时 懒加载块
  
  # 预加载  (快)
  # 预加载单个关联对象--select_related (ForeignKey)
  # 预加载多个关联对象--prefetch_related
  def pre_load():
      for user in User.objects.prefetch_related('menu'):
          print(user.menu.all())
          # print(user.query)
          # res = user.menu.all()
          # print(res.query)
  
  
  def add():
      App.objects.create(name='微信')
  
  
  if __name__ == '__main__':
      # lazy_load()
      # pre_load()
      # add()
      t1 = Timer("lazy_load()", "from __main__ import lazy_load")
      t2 = Timer("pre_load()", "from __main__ import pre_load")
      print('懒加载运行时间为:', t1.timeit(1000), '预加载运行时间为:', t2.timeit(1000))
  ```

  ![1583933579450](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583933579450.png)

## 三.长连接

- 数据库连接的处理逻辑

  - 使用CONN_MAX_AGE配置限制DB连接寿命
  - CONN_MAX_AGE的默认值是0
  - 每个DB连接的寿命保持到该次请求结束
  - ![1583934201577](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583934201577.png)
  - 弊端:
    - 每个请求都将重复连接数据库
    - 处理高并发请求给服务器带来巨大压力
    - 无法承载更高并发的服务

- 避免负优化

  - 存储数据库连接的位置:线程局部变量
  - 数据库支持的最大连接数
    - 进入MySQL命令行模式
    -  show variables like "%max_connection%";  # 查看数据库最大连接数
    - max_connections : 151   # 表示MySQL数据库的最大连接数是151
    - set global max_connections = 200;   # 将数据库最大连接数修改为200
  - CONN_MAX_AGE的配置指南:
    - 部署线程数 < 数据库最大连接数  # 否则的话会导致其他服务无法连接数据库
    - 不建议开发模式下使用CONN_MAX_AGE

- 理论效果

  ![1583936023733](E:\md文件\三月十一日 深入Django模型之优化篇.assets\1583936023733.png)

## 四.数据库操作规范

- 使用正确的优化策略优化查询
  - 正确使用索引
  - 索引该被索引的列
  - 不索引不该被索引的列
- 使用iterator迭代器迭代QuerySet
  - QuerySet非常大时,迭代器迭代节省内存
- 理解对象的属性缓存
  - 不可调用的属性会被ORM框架缓存
    - 这个属性不可以按照函数的方法调用
  - 可调用的属性不会被ORM框架缓存
    - 这个属性可以按照函数的方法调用
- 数据库的工作留给数据库做
  - 过滤: 使用filter,exclude等属性
  - 聚合: 使用annotate函数进行聚合
  - 使用原生SQL进行查询
- 正确检索行数据 (合理使用索引)
  - 使用被索引的列字段检索
  - 使用被unique修饰的列字段检索
- 不要进行不必要的检索
  - QuerySet使用values(),value_list()函数返回python结构容器
  - 查询结果长度使用QuerySet.count() 而不是len(QuerySet)
  - 判断是否为空使用QuerySet.exists() 而不是if QuerySet
  - 不要进行不必要的排序
- 批量操作
  - 当同时操作大量数据时使用批量操作









































