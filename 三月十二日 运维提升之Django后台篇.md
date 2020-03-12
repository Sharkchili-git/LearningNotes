# 运维提升之Django后台篇

- ## 日志模块

  - 日志产生的四个步骤:

    - 产生日志 --> 渲染格式 --> 匹配过滤 --> 持久化

  - 日志模块的配置

    - ```python
      # 日志level级别
      # DEBUG：用于调试目的的低级系统信息
      # INFO：一般系统信息
      # WARNING：描述已发生的小问题的信息。
      # ERROR：描述已发生的主要问题的信息。
      # CRITICAL：描述已发生的严重问题的信息。
      ```

    - 格式器 formatter

      - 配置日志格式 

      - 沿用python语言里面的格式属性

      - ![1584006489081](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584006489081.png)

      - ![1584006640437](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584006640437.png)

      - 全部配置代码:

        ```python
        # 持久化日志存储路径
        LOG_DIR = os.path.join(BASE_DIR, 'log')
        if not os.path.exists(LOG_DIR):
            os.makedirs(LOG_DIR)
            
        LOGGING = {
            'version': 1,
            # 'disable_existing_loggers': False,
             # 日志格式
            'formatters': { 
                'standard': {
                    'format': '%(asctime)s [%(threadName)s:%(thread)d] [%                                (pathname)s:%(funcName)s:%(lineno)d] [%                                  (levelname)s]- %(message)s'}
            },
            # 过滤器 导入过滤条件的类
            'filters': {  
                'test': {
                    '()': 'ops.TestFilter'
                }
            },
            # 处理器 对日志进行处理,比如:写进文件,打印到屏幕等
            'handlers': {
                'null': {
                    'level': 'DEBUG',
                    'class': 'logging.NullHandler',
                },
                # 文件处理器
                'file_handler': {  # 记录到日志文件(需要创建对应的目录，否则会出错)
                    'level': 'DEBUG',
                    'class': 'logging.handlers.RotatingFileHandler',
                    'filename': os.path.join(LOG_DIR, 'service.log'),  # 日志输出文件
                    'maxBytes': 1024 * 1024 * 5,  # 文件大小
                    'backupCount': 5,  # 备份份数
                    'formatter': 'standard',  # 使用哪种formatters日志格式
                    'encoding': 'utf8',
                },
                # 终端处理器 pycharm 打印格式
                'console_handler': {
                    'level': 'DEBUG',
                    'class': 'logging.StreamHandler',
                    'formatter': 'standard',
                },
            },
            'loggers': {  # logging管理器  配置添加进来才能生效
                'django': {
                    'handlers': ['console_handler', 'file_handler'],
                    'filters': ['test'],
                    # 限制全局输出日志级别
                    'level': 'INFO'
                    # 'level': 'WARNING'
                }
            }
        }
        ```

    - 过滤器 filter

      - 过滤器类代码:

        ```python
        # 在 ops 文件夹下的 __init__ 文件中定义
        from logging import Filter
        
        
        class TestFilter(Filter):
            def filter(self, record):
                # 过滤 信息中带有 xxx 的信息
                if 'xxx' in record.msg:
                    return False
                return True
        
        ```

    - 处理器 handle

      - 对日志进行处理,比如:写进文件,打印到屏幕等

    - 日志实例 logger

      - 在python代码里面打印日志的入口点

  - 日志模块的使用

    ![1584009765414](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584009765414.png)

    ```python
    import os
    import django
    import logging
    
    # 配置环境
    os.environ['DJANGO_SETTINGS_MODULE'] = 'HelloDjango.settings'
    django.setup()
    
    
    def logdemo():
        # 获取日志实例 loggers下的 django
        logger = logging.getLogger('django')
        logger.info('info:hello info logging')
        logger.info('info:xxx hello info logging')
        logger.debug('debug:hello debug logging')
        logger.debug('debug:xxxhello debug logging')
        logger.warning('warning:hello warning logging')
        logger.warning('warning:xxxhello warning logging')
    
    
    if __name__ == '__main__':
        logdemo()
    
    ```

- ## Admin模块

  - 为什么Django提供admin模块

    - 管理界面是基础设施非常重要的一部分

    - 管理者可以添加,编辑,查看和删除网站内容

    - 功能类似但开发繁琐,所以由Django提供

      ![1584010693945](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584010693945.png)

  - admin模块是什么

    - Django的自动管理界面
    - 管理Django用户权限,模型数据,信息发布
    - 拓展性强,基于此定制很多功能

  - admin模块怎么使用

    - 确认添加admin模块和路由说明

      - 在settings.py文件中确认 INSTALLED_APPS配置中 'django.contrib.admin',是否已经配置上
      - 项目 url 总配置中的 urls.py文件中 urlpatterns是否配置 path('admin/', admin.site.urls),

    - 创建超级管理员

      - python manage.py createsuperuser

    - 注册模型到admin模块 

      - 在相关应用的admin.py中注册

      - 模型类中重写 _ str _函数

      - 不显示不可编辑字段:fields列表或exclude列表

      - 重写save_mode函数实现保存逻辑

        ```python
        from django.contrib import admin
        from jhapp.models import App, User
        import time
        
        # Register your models here.
        
        # 注册的两种方式
        # 一:直接注册
        admin.site.register(App)
        
        
        # 二:装饰器
        @admin.register(User)
        class UserAdmin(admin.ModelAdmin):
            # 在admin添加博客界面不显示 openid
            exclude = ['openid']
            """
                因为 openid 是微信小程序自动生成发送过来的 所以不能自己添加,
            但是如果再在admin页面下添加的话 第一次 openid 是空 可以添加不报错,
            第二次添加的话 openid 还是空 就会报错了 因为 openid 这个字段定义的时候
            设置为了 unique 唯一 ,所以不能添加第二遍了,解决方法如下
            """
        
            def save_model(self, request, obj, form, change):
                """
                :param request:请求方式和请求路由
                :param obj: 就是 User 对象
                :param form:HTML form表单
                :param change:
                :return:
                """
                # 将nickname+时间戳 设置为 openid 这样就能不显示 openid 还能添加数据了,还不报错
                obj.openid = obj.nickname + str(time.time())
                # print('request', request)  # <WSGIRequest: POST '/admin/jhapp/user/add/'>
                # print('obj', obj)  # User对象
                # print('form', form)  # admin添加数据页面的HTML form 表单
                # print('change', change)  # False
                return super(UserAdmin, self).save_model(request, obj, form, change)
        ```

- ## 缓存模块

  - 缓存概述

    - 缓存是(高速)缓存的简称,英文:cache

    - 根本目的是为了加快数据访问速度,提高性能

    - 协调两者数据传输速度的差异

      ![1584012602860](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584012602860.png)

      ![1584012688464](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584012688464.png)

      ![1584012764522](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584012764522.png)

    - 把所有的数据放在缓存可以吗:

      - 不可以,成本太高

    - 缓存的置换策略

    - LFU,FIFO,LRU等算法

  - Django缓存模块

    - 基于缓存框架的缓存

    - 基于数据库的缓存

    - 基于文件系统的缓存

    - 基于内存的缓存

      ```python
      # 缓存
      # BACKEND:Django提供支持的一个兼容模块
      # LOCATION:1.IP:端口 / 2.表名 / 3.文件路径 / 4.backend-cache
      CACHES = {
          'default': {
              # 1. MemCache /框架缓存
              # 'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
              # 'LOCATION': '127.0.0.1:11211',
      
              # 2. DB Cache  /数据库缓存
              # 'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
              # 'LOCATION': 'my_cache_table',
      
              # 3. Filesystem Cache  /文件系统缓存
              # 'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
              # 'LOCATION': '/var/tmp/django_cache',
      
              # 4. Local Mem Cache /本地内存缓存
              'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
              'LOCATION': 'backend-cache'
          }
      }
      ```

    - 实现代码:

      ```python
      import os
      import django
      from django.core.cache import cache
      
      # 配置环境
      os.environ['DJANGO_SETTINGS_MODULE'] = 'HelloDjango.settings'
      # os.environ.setdefault('DJANGO_SETTING_MODULE', 'HelloDjango.settings')
      django.setup()
      
      
      def get_cache(k):
          return cache.get(k)
      
      
      def set_cache(k, v='百事可乐'):
          cache.set(k, v, 100)
      
      
      def main(data):
          res = get_cache(data)
          print(res)
          if res is None:
              print('保存数据')
              set_cache(k=data)
              res = get_cache(data)
          return res
      
      
      if __name__ == '__main__':
          print(main('可乐'))
          # print(cache.get('可乐'))
          # cache.set('可乐', '百事可乐',100)
          # print(cache.get('可乐'))
      ```

- ## 后台服务部署

  - 同步代码
    - 把本地的代码同步到Git上
    - 服务器把代码从Git上拉取下来
  - 同步环境
    - 在本地: pip freeze > requirements.txt
    - 上传Git
    - 在服务器端 拉取Git
    - 载入环境：`pip install -r /path/requirements.txt`
  - 在服务器安装MySQL
    - 卸载MySQL残留文件 (依次执行)
      - dpkg -l | grep mysql  # 查询有哪些残留文件
      - sudo apt-get purge 查询出来的所有文件
      - sudo /etc/init.d/mysql stop
      - sudo rm/etc/mysql/ -R
      - sudo rm/var/lib/mysql -R
    - 安装MySQL
      - sudo apt-get install mysql-client mysql-server
    - 同步数据库
      - <https://blog.csdn.net/kkfd1002/article/details/80247882>
    - 更改Django文件中的数据库配置

- ## Crontab 定时任务

  - ![1584019155643](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584019155643.png)

  - Linux下的crontab命令

    - 用于提交和管理用户的周期性任务

    - crond进程每分钟定时检查

    - 时间间隔

      ![1584017091712](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584017091712.png)

  - django-crontab 插件的使用 (django-crontab只能在 Linux上运行)

    - 在Linux上使用 pip进行安装

      - pip install django-crontab

    - 配置 django-crontab

      - 先在 Linux 项目settings.py文件中的 INSTALLED_APPS 配置 'django_crontab'

      - ```python
        # 定时任务配置
        # 每隔一个钟执行一次
        CRONJOBS = [
            # ('*/1 * * * *', 'echo "Hello World > /dev/null"')
            ('*/1 * * * *', 'cron.jobs.demo')
        ]
        ```

        创建cron 文件夹 以及 jobs.py文件

        ```python
        import logging
        import datetime
        
        logger = logging.getLogger('django')
        
        
        def demo():
            message = 'Job log in crontab, now time is :' + str(datetime.datetime.now())
            print(message)
            logger.info(message)
        
        ```

      - 在服务器上 运行 python manage.py crontab show 查看有哪些定时任务 (现在是没有的)

      - python manage.py crontab add  # 通过这条命令把刚刚创建的任务添加上

      - 再通过 python manage.py crontab show 这条命令就可以看见定时任务了

      - python manage.py crontab remove 删除定时任务

      - python manage.py crontab run 立即运行定时任务

- ## Middleware 中间件

  - 什么是Django中间件

    - Django中间件就是一个类
    - 请求前后在合适的时机执行相应方法
    - 配置MIDDLEWARE_CLASSES属性
    - 有哪些:
      - SecurityMiddleware: 安全中间件
      - GzipMiddleware: Gzip压缩中间件
      - SessionMiddleware: 状态服务中间件
      - CsrfViewMiddleware: 防止跨站伪造请求的中间件
      - AuthorizationMiddleware: Django自带的认证体系中间件
      - MessageMiddleware: 消息中间件
      - 等等

  - Django中间件的执行逻辑

    - 每个请求都会两次经过配置的中间件

      ![1584020088602](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584020088602.png)

    - 多个中间件之间存在执行顺序

  - 实现自定义中间件

    - 在每个请求前后打印 log(验证是否每次请求都经两次中间件)

      ```python
      # 自己定义中间件
      import logging
      
      logger = logging.getLogger('django')
      
      
      class TestMiddleWare():
          def __init__(self, get_response):
              self.get_response = get_response
              logger.info('Build TestMiddleWare')
      	# 重写 __call__方法
          def __call__(self, request):
              logger.info('TestMiddleWare before request')
              response = self.get_response(request)
              logger.info('TestMiddleWare after request')
              return response
      
      ```

    - 运行结果

      证明 每次请求都经两次中间件

      ![1584021216849](E:\md文件\三月十二日 运维提升之Django后台篇.assets\1584021216849.png)

- ## 邮件模块

- ## 综合实践:基于邮件通知的服务监控和告警系统



























