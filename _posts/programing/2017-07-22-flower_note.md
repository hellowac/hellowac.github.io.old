---
nav: blog
layout: post
title: "Celery - Flower 源码备注"
author: "wangchao"
tags:
  - python
  - Celery
  - Flower
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[Celery](http://celery.readthedocs.io/en/latest/index.html) 和 [Flower](http://flower.readthedocs.io/en/latest/index.html?highlight=source#)

- **入口**:
	- `flower.command.FlowerCommand` 类. 继承自 [celery.bin.base.Command](http://celery.readthedocs.io/en/latest/_modules/celery/bin/base.html#Command)
	- 执行流程: `flower = FlowerCommand(); flower.execute_from_commandline()`
		- [execute_from_commandline](http://celery.readthedocs.io/en/latest/_modules/celery/bin/base.html#Command.execute_from_commandline) 函数 <- `Command`
			- `if argv is None: argv = list(sys.argv)` 语句 <- `Command` (没有传入 `argv` 参数, 则取 `sys.argv` )
			- `self.maybe_patch_concurrency(argv)`  语句 <- `Command`  (Should we load any special concurrency environment?, 在 `flower` 中没用, 不关心)
			- `self.on_concurrency_setup()` 语句 <- `Command` (在 `flower` 中没用, 不关心)
			- `self.early_version(argv)` 语句 <- `Command` (不支持的 `celery` 版本则退出.)
			- `argv = self.setup_app_from_commandline(argv)` 语句 <- `Command` (**重要**, 创建指定`broker`的`celery`实例.)
                - 设置工作目录
                - 读取 `celery` 实例 `app` 地址.  (`flower` 不需要.)
                - 读取加载器 ( `loader` 参数, `flower` 不需要.) 相关 [Issue:#1066](https://github.com/celery/celery/issues/1066)
                - 读取 消息中间件地址. ( `broker` 参数)
                - 读取 配置文件地址. (`config` 参数)
                - 读取或创建 `Celery` 实例, 赋值给 `app` 属性
                - 读取用户自定义配置.
			- `self.prog_name = os.path.basename(argv[0])` 语句 <- `Command`  (赋值当前进程名字)
			- `return self.handle_argv(self.prog_name, argv[1:])` 语句 <- `Command` (处理其他参数.)
		- `handle_argv` 函数 (重写) <- `FlowerCommand` <- `Command`
			- `run_from_argv` 语句 (调用自身定义的函数.)
		- `run_from_argv` 函数 (重写) <- `FlowerCommand` <- `Command`
			- `apply_env_options` 函数 <- `FlowerCommand`  (设置 `flower` 环境变量.)
			- `apply_options` 函数 <- `FlowerCommand`
			- `extract_settings` 函数 <- `FlowerCommand`
			- `setup_logging` 函数 <- `FlowerCommand`
			- `self.app.loader.import_default_modules()` 语句 (`Command`类有2个函数 `get_app` 和 `find_app`)
			- `flower = Flower(capp=self.app, options=options, **settings)` 语句 (创建 `flower` 实例.)
			- `atexit.register(dcs_flower.stop)` 语句 (注册退出函数. 参考:[register](http://python.usyiyi.cn/translate/python_352/library/atexit.html#atexit.register))
			- `signal.signal(signal.SIGTERM, sigterm_handler)` 语句 (注册信号处理函数, 参考:[signal](http://python.usyiyi.cn/documents/python_352/library/signal.html#signal.signal))
			- `dcs_flower.start()` 语句 (启动 `flower` )

- **相关函数源码:**

```python
# http://docs.celeryproject.org/en/v4.0.0/_modules/celery/bin/base.html#Command.execute_from_commandline
def execute_from_commandline(self, argv=None):
    """Execute application from command-line.

    Arguments:
        argv (List[str]): The list of command-line arguments.
            Defaults to ``sys.argv``.
    """
    if argv is None:
        argv = list(sys.argv)
    # Should we load any special concurrency environment?
    self.maybe_patch_concurrency(argv)  # 根据参数选择并发矿建
    self.on_concurrency_setup()  # 并发

    # Dump version and exit if '--version' arg set.
    self.early_version(argv)  # 判断celery版本并退出.
    argv = self.setup_app_from_commandline(argv)  # 创建 Celery 实例 以及 必要的环境变量设置.
    self.prog_name = os.path.basename(argv[0])  # 脚本名称就是当前进程的名称.
    return self.handle_argv(self.prog_name, argv[1:])   # FlowerCommand 实现了.

# http://docs.celeryproject.org/en/v4.0.0/_modules/celery/bin/base.html#Command.setup_app_from_commandline
def setup_app_from_commandline(self, argv):

    # 这里返回一个 字典
    preload_options = self.parse_preload_options(argv)  # 解析命令行参数. 过滤了 -h, --help 参数. 这里使用 argparse.ArgumentParser
    quiet = preload_options.get('quiet')
    if quiet is not None:
        self.quiet = quiet
    try:
        self.no_color = preload_options['no_color']
    except KeyError:
        pass
    workdir = preload_options.get('workdir')  # 工作目录.
    if workdir:
        os.chdir(workdir)
    app = (preload_options.get('app') or  # 项目celery实例地址.
           os.environ.get('CELERY_APP') or
           self.app)
    preload_loader = preload_options.get('loader')  # 命令行配置的加载器，比如 djcelery.loaders.DjangoLoader
    if preload_loader:
        # Default app takes loader from this env (Issue #1066).
        os.environ['CELERY_LOADER'] = preload_loader
    loader = (preload_loader,
              os.environ.get('CELERY_LOADER') or
              'default')
    broker = preload_options.get('broker', None)   # 消息中间件, redis? rabbit mq ?
    if broker:
        os.environ['CELERY_BROKER_URL'] = broker
    config = preload_options.get('config')      # celery 配置地址.
    if config:
        os.environ['CELERY_CONFIG_MODULE'] = config
    if self.respects_app_option:   # 寻找配置的 Celery 实例.
        if app:
            self.app = self.find_app(app)
        elif self.app is None:
            self.app = self.get_app(loader=loader)
        if self.enable_config_from_cmdline:
            argv = self.process_cmdline_config(argv)
    else:
        self.app = Celery(fixups=[])  # 创建一个空的Celery实例.

    self._handle_user_preload_options(argv)  # 收集用户命令行配置.

    return argv

# flower.command.FlowerCommand.handle_argv
def handle_argv(self, prog_name, argv=None):
    return self.run_from_argv(prog_name, argv)

# flower.command.FlowerCommand.run_from_argv  重写父类的该方法.
def run_from_argv(self, prog_name, argv=None, **_kwargs):
    self.apply_env_options()  # 从环境变量读取配置
    self.apply_options(prog_name, argv)

    self.extract_settings()   # 从 命令行参数列表读取配置并配置到web端.
    self.setup_logging()  # 安装日志.

    self.app.loader.import_default_modules()  # 默认Loader -> celery.loaders.app:AppLoader(celery.loaders.base.BaseLoader) , 加载 定义任务的 module
    dcs_flower = Flower(capp=self.app, options=options, **settings)
    atexit.register(dcs_flower.stop)

    def sigterm_handler(signal, frame):
        logger.info('SIGTERM detected, shutting down')
        sys.exit(0)
    signal.signal(signal.SIGTERM, sigterm_handler)

    self.print_banner('ssl_options' in settings)

    try:
        dcs_flower.start()
    except (KeyboardInterrupt, SystemExit):
        pass

# flower.command.FlowerCommand.apply_env_options
def apply_env_options(self):
    "apply options passed through environment variables"
    env_options = filter(self.is_flower_envvar, os.environ)  # 遍历环境变量并判断是否有flower环境变量.
    for env_var_name in env_options:
        name = env_var_name.replace(self.ENV_VAR_PREFIX, '', 1).lower()
        value = os.environ[env_var_name]
        try:
            option = options._options[name]
        except:
            option = options._options[name.replace('_', '-')]
        if option.multiple:
            value = [option.type(i) for i in value.split(',')]
        else:
            value = option.type(value)
        setattr(options, name, value)  # 这里的options是指 tornado.options.options
```

## 最终找到 使用 django 操纵 celery 的 4个 相关包:

- [django-celery](https://pypi.python.org/pypi/django-celery)
    - 介绍:
        - 仅针对 celery 版本小于 4.0 的 在 django 中的使用。
    - source : [github](https://github.com/celery/django-celery)
    - 针对 Celery 版本大于等于 4.0 的 在 django 中的使用官网参考：[celery-django](http://docs.celeryproject.org/en/latest/django/)
- [django-celery-beat](https://pypi.python.org/pypi/django_celery_beat)
    - 介绍:
        - This extension enables you to store the periodic task schedule in the database.
        - The periodic tasks can be managed from the Django Admin interface, where you can create, edit and delete periodic tasks and how often they should run.
    - source: [github](http://github.com/celery/django-celery-beat)
- [django-celery-results](https://pypi.python.org/pypi/django_celery_results)
    - 介绍:
        - This extension enables you to store Celery task results using the Django ORM.
        - It defines a single model (django_celery_results.models.TaskResult) used to store task results, and you can query this database table like any other Django model.
    - source: [github](http://github.com/celery/django-celery-results)
- [django-celery-monitor](https://pypi.python.org/pypi/django_celery_monitor)
    - 介绍:
        - This extension enables you to monitor Celery tasks and workers.
        - It defines two models (django_celery_monitor.models.WorkerState and django_celery_monitor.models.TaskState) used to store worker and task states and you can query this database table like any other Django model. It provides a Camera class (django_celery_monitor.camera.Camera) to be used with the Celery events command line tool to automatically populate the two models with the current state of the Celery workers and tasks.
    - source: [github](https://github.com/jezdez/django-celery-monitor)
