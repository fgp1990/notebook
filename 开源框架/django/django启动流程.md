# Django 启动流程
## 一、manage.py
整个django启动从这个文件开始。
### 第一步、设置配置文件地址
```
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'spug.settings')
```
这是源码，实际使用中，为了能够自动切换不同环境（开发、测试、生产），在这里增加if判断，根据环境变量的不同，使用不同的配置。
例如：
```
if os.environ.get('DJANGO_DEPLOY_ENV') == 'prod':
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'task.settings_prod')
else:
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'task.settings')
```
同时要设置环境变量。比方在/etc/profile里面添加相应环境变量参数。
```
export DJANGO_DEPLOY_ENV=prod
```
对应了if的判断条件
### 第二步、校验是否安装了Django
```
try:
    from django.core.management import execute_from_command_line
except ImportError as exc:
    raise ImportError(
        "Couldn't import Django. Are you sure it's installed and "
        "available on your PYTHONPATH environment variable? Did you "
        "forget to activate a virtual environment?"
    ) from exc
```
这里其实做了两件事情：
1. 检验是否安装了Django
2. import Django的启动库
做的挺巧妙的，直接import，如果失败，那就是没装，通过except把错误报出来
### 第三步、执行Django
```
execute_from_command_line(sys.argv)
```
这里sys.argv，是一个列表。第一个参数是文件本身，即manager.py。从第二个参数开始，就是manage.py后面跟着的一系列参数。
```
(splug) [root@localhost splug]# python manage.py version
启动入参：['manage.py', 'version']
```
这里面就能看得很清楚了。
## 二、execute_from_command_line
这是Django的实际执行函数，里面首先实例化管理组件，然后执行execute这个方法
```
def execute_from_command_line(argv=None):
    """Run a ManagementUtility."""
    utility = ManagementUtility(argv)
    utility.execute()
```
实例化：
实例化时，就做了一件事，赋值：
```
def __init__(self, argv=None):
    self.argv = argv or sys.argv[:]
    self.prog_name = os.path.basename(self.argv[0])
    if self.prog_name == '__main__.py':
        self.prog_name = 'python -m django'
    self.settings_exception = None
```
这里先拿到了manage.py后面跟着的参数，以空格分隔换成列表。self.argv这个参数很重要，后面一直要用。
```
execute：
try:
    subcommand = self.argv[1]
    print("execute 参数：subcommand(self.argv[1]): %s" % subcommand)
except IndexError:
    subcommand = 'help'  # Display help if no arguments were given.
```
获取次级参数，也就是manage.py后面跟着的第一个参数。这个参数是django的一级参数，决定了具体的行为。比方runserver，启动端口监听；makemigrations，生成表迁移数据；migrate，向数据库同步表结构。如果没有，就赋值“help”，打印帮助信息。
```
if settings.configured:
    # Start the auto-reloading dev server even if the code is broken.
    # The hardcoded condition is a code smell but we can't rely on a
    # flag on the command class because we haven't located it yet.
    if subcommand == 'runserver' and '--noreload' not in self.argv:
        try:
            autoreload.check_errors(django.setup)()
        except Exception:
            # The exception will be raised later in the child process
            # started by the autoreloader. Pretend it didn't happen by
            # loading an empty list of applications.
            apps.all_models = defaultdict(dict)
            apps.app_configs = {}
            apps.apps_ready = apps.models_ready = apps.ready = True

            # Remove options not compatible with the built-in runserver
            # (e.g. options for the contrib.staticfiles' runserver).
            # Changes here require manually testing as described in
            # #27522.
            _parser = self.fetch_command('runserver').create_parser('django', 'runserver')
            _options, _args = _parser.parse_known_args(self.argv[2:])
            for _arg in _args:
                self.argv.remove(_arg)

    # In all other cases, django.setup() is required to succeed.
    else:
        django.setup()
```
中间有一段对参数的处理就不要了。这一段是根据次级参数的不同进行了不同的处理，其实就是runserver下是否对项目文件进行变更监控。就是日常使用到的，改了个啥，自动重启Django。最后都要落脚到django.setup()。不过具体怎么实现的，没有看懂。
```
if subcommand == 'help':
    if '--commands' in args:
        sys.stdout.write(self.main_help_text(commands_only=True) + '\n')
    elif not options.args:
        sys.stdout.write(self.main_help_text() + '\n')
    else:
        self.fetch_command(options.args[0]).print_help(self.prog_name, options.args[0])
# Special-cases: We want 'django-admin --version' and
# 'django-admin --help' to work, for backwards compatibility.
elif subcommand == 'version' or self.argv[1:] == ['--version']:
    sys.stdout.write(django.get_version() + '\n')
elif self.argv[1:] in (['--help'], ['-h']):
    sys.stdout.write(self.main_help_text() + '\n')
else:
    self.fetch_command(subcommand).run_from_argv(self.argv)
```
根据参数不同，进行不同的操作。其实就分了4种情况。
help：
1. args是次级参数后面所有带“-”或者“--”的参数的集合（是个列表）
2. options.args是具体的参数值
3. 这个解释不准确，比方python manage.py runserver -h，就能准确的将帮助信息打出来
    但是如果你输入的内容是随便编造的，并不一定是这种规则。例如：
```
(splug) [root@localhost splug]# python manage.py version -h111 123 456 789 -huhuh 9999 -1231231 adfadsfasdf
启动入参：['manage.py', 'version', '-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
类实例化时的参数：
	argv: ['manage.py', 'version', '-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
	prog_name: manage.py
execute 参数：subcommand(self.argv[1]): version
execute 参数：['-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
处理后参数:
	options.args: ['123', '456', '789']
	args: ['-h111', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
```
```
(splug) [root@localhost splug]# python manage.py version qqqq -h111 123 456 789 -huhuh 9999 -1231231 adfadsfasdf
启动入参：['manage.py', 'version', 'qqqq', '-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
类实例化时的参数：
	argv: ['manage.py', 'version', 'qqqq', '-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
	prog_name: manage.py
execute 参数：subcommand(self.argv[1]): version
execute 参数：['qqqq', '-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
处理后参数:
	options.args: ['qqqq']
	args: ['-h111', '123', '456', '789', '-huhuh', '9999', '-1231231', 'adfadsfasdf']
```
这两个的区别在于version后面的参数是否有“-”。可以看出optios.args是不一样的。
<font style="color:red">具体的规则，看了下源码。太复杂了，看不进去。我觉得只要记住这里是读取参数就行</font>
最后一个，才是真正开始执行代码，中间哪两个一个是版本，另一个还是帮助

### execute里最后一步，真正执行代码：
self.fetch_command(subcommand).run_from_argv(self.argv)