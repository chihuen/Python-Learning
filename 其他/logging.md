



~~~
import logging
logging.warning('Watch out!')  # will print a message to the console
logging.info('I told you so')  # will not print anything
~~~

以上的打印是这样的：

~~~
WARNING:root:Watch out!
~~~

只有`Watch out!`被打印的原因是:The default level is `WARNING`, which means that only events of this level and above will be tracked, unless the logging package is configured to do otherwise.如下是等级表，越往下等级越高

| Level      | When it’s used                                               |
| ---------- | ------------------------------------------------------------ |
| `DEBUG`    | Detailed information, typically of interest only when diagnosing problems. |
| `INFO`     | Confirmation that things are working as expected.            |
| `WARNING`  | An indication that something unexpected happened, or indicative of some problem in the near future (e.g. ‘disk space low’). The software is still working as expected. |
| `ERROR`    | Due to a more serious problem, the software has not been able to perform some function. |
| `CRITICAL` | A serious error, indicating that the program itself may be unable to continue running. |





#Logging to a file

 这一章讲的是输出到文本的logging等级和打印是一样的，都是等级高的才会被track。

如果想要通过变量loglevel(string)设置等级，那么可以通过

~~~
# assuming loglevel is bound to the string value obtained from the
# command line argument. Convert to upper case to allow the user to
# specify --log=DEBUG or --log=debug
numeric_level = getattr(logging, loglevel.upper(), None)
if not isinstance(numeric_level, int):
    raise ValueError('Invalid log level: %s' % loglevel)
logging.basicConfig(level=numeric_level, ...)
~~~

需要注意的是，使用`basicConfig`设置等级，需要在调用`debug()`, `info()`这些函数之前，否则不生效：The call to `basicConfig()` should come before any calls to `debug()`, `info()` etc. As it’s intended as a one-off simple configuration facility, only the first call will actually do anything: subsequent calls are effectively no-ops.



# Logging from multiple modules





# Changing the format of displayed messages

如何格式化输出

~~~
import logging
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug('This message should appear on the console')
logging.info('So should this')
logging.warning('And this, too')
~~~

结果

~~~
DEBUG:This message should appear on the console
INFO:So should this
WARNING:And this, too
~~~

需要注意的是，这里不再出现root，原因可以在 [LogRecord attributes](https://docs.python.org/3.5/library/logging.html#logrecord-attributes)这里找到



## 格式化：输出时间

如何输出时间

~~~
import logging
logging.basicConfig(format='%(asctime)s %(message)s')
logging.warning('is when this event was logged.')
~~~

结果

~~~
2010-12-12 11:41:42,612 is when this event was logged.
~~~

要精准化，需要使用`basicConfig`的`datefmt`参数:

~~~
mport logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')
logging.warning('is when this event was logged.')
~~~

结果

~~~
12/12/2010 11:46:36 AM is when this event was logged.
~~~



# Loggers

Logger负责三个工作：

1. 提供api用于记录messages：
   1. `Logger.debug()`, `Logger.info()`, `Logger.warning()`, `Logger.error()`, and `Logger.critical()`

   2. `Logger.exception()`用法与`Logger.error()`类似，区别：The difference is that Logger.exception() dumps a stack trace along with it. Call this method only from an exception handler.

   3. `Logger.log()`和上面的方法区别在于，它会多一个参数用于设置log level，用于自定义消息等级

   4. 以下是在logging/\_\_init\_\_.py中的等级

   5. ~~~
      CRITICAL = 50
      FATAL = CRITICAL
      ERROR = 40
      WARNING = 30
      WARN = WARNING
      INFO = 20
      DEBUG = 10
      NOTSET = 0
      ~~~

2. 根据消息等级决定哪些messages会被track，使用`Logger.setLevel()`来指定最低级

3. 将messages发送到相关的handlers，使用`Logger.addHandler()` and`Logger.removeHandler()`添加handlers

`getLogger()`会根据名字返回相应的logger实例，可以根据名字获知logger的父子关系，For example, given a logger with a name of foo, loggers with names of foo.bar, foo.bar.baz, and foo.bam are all descendants of foo.

父子关系会影响Logger的有效等级（Loggers have a concept of *effective level*），如果当前logger没有设置等级，那么就会继承父级的等级，如果父级也没有设置等级，那么继承再上一级的等级，最上级为root的等级（默认为`WARNING`）

logger要发送给哪个handllers，默认是可以直接继承上级的handlers的，因此无需对所有子logger都设置handlers（`addHandler（）`）



关于logger object：

> Note that Loggers are never instantiated directly, but always through the module-level function `logging.getLogger(name)`. Multiple calls to [`getLogger()`](https://docs.python.org/3.5/library/logging.html#logging.getLogger) with the same name will always return a reference to the same Logger object.

在创建logger对象时，应该使用`logging.getLogger(name)`方法来创建，



## Handlers

Handlers会将从logger传送过来的信息发送到相应的地方，例如console，email，files and etc. Handler objects are responsible for dispatching the appropriate log messages (based on the log messages’ severity) to the handler’s specified destination.

1. Handlers也需要设置消息等级限制，低于设置值的消息将不会发送到desination
2. 可以设置输出格式，使用`setFormatter()`创建Formatter Object来设置
3. 可以设置过滤，使用`addFilter()` and `removeFilter()`来设置

使用的时候需要注意不需要实例化Handler类

> Application code should not directly instantiate and use instances of Handler. Instead, the Handler class is a base class that defines the interface that all handlers should have and establishes some default behavior that child classes can use (or override).



## Formatters









# Configuring Logging

配置logging有三种方法：

1. 在代码中通过调用api逐个配置logger、handler、formatter
2. 在配置文件中写好参数，使用`logging.config.fileConfig（）`进行直接配置
3. 在dict中写好参数，使用`dictConfig()`

需要注意的是，使用2、3方法默认会将代码中原有的1方法配置屏蔽掉：

>Warning
>
>The [`fileConfig()`](https://docs.python.org/3.5/library/logging.config.html#logging.config.fileConfig) function takes a default parameter, `disable_existing_loggers`, which defaults to`True` for reasons of backward compatibility. This may or may not be what you want, since it will cause any loggers existing before the [`fileConfig()`](https://docs.python.org/3.5/library/logging.config.html#logging.config.fileConfig) call to be disabled unless they (or an ancestor) are explicitly named in the configuration. Please refer to the reference documentation for more information, and specify `False` for this parameter if you wish.
>
>The dictionary passed to [`dictConfig()`](https://docs.python.org/3.5/library/logging.config.html#logging.config.dictConfig) can also specify a Boolean value with key `disable_existing_loggers`, which if not specified explicitly in the dictionary also defaults to being interpreted as `True`. This leads to the logger-disabling behaviour described above, which may not be what you want - in which case, provide the key explicitly with a value of `False`.



## 使用配置文件配置logging



>Note that the class names referenced in config files need to be either relative to the logging module, or absolute values which can be resolved using normal import mechanisms. Thus, you could use either WatchedFileHandler (relative to the logging module) or mypackage.mymodule.MyHandler (for a class defined in package mypackage and module mymodule, where mypackage is available on the Python import path).





## 不使用任何配置

对于3.5版本来说，不配置handlers、formatter的话，那么就会将message发到console（acts like a StreamHandler）

>### What happens if no configuration is provided
>
>If no logging configuration is provided, it is possible to have a situation where a logging event needs to be output, but no handlers can be found to output the event. The behaviour of the logging package in these circumstances is dependent on the Python version.
>
>For versions of Python prior to 3.2, the behaviour is as follows:
>
>- If *logging.raiseExceptions* is `False` (production mode), the event is silently dropped.
>- If *logging.raiseExceptions* is `True` (development mode), a message ‘No handlers could be found for logger X.Y.Z’ is printed once.
>
>In Python 3.2 and later, the behaviour is as follows:
>
>- The event is output using a ‘handler of last resort’, stored in `logging.lastResort`. This internal handler is not associated with any logger, and acts like a [`StreamHandler`](https://docs.python.org/3.5/library/logging.handlers.html#logging.StreamHandler) which writes the event description message to the current value of `sys.stderr` (therefore respecting any redirections which may be in effect). No formatting is done on the message - just the bare event description message is printed. The handler’s level is set to `WARNING`, so all events at this and greater severities will be output.
>
>To obtain the pre-3.2 behaviour, `logging.lastResort` can be set to `None`.
>
>



# Configuring Logging for a Library

这一章讲的是，如果Library中用了logging模块，那么最好在document中说明logging配置。同时，最好设置的level是WARNING

>If the using application does not use logging, and library code makes logging calls, then (as described in the previous section) events of severity `WARNING` and greater will be printed to `sys.stderr`. This is regarded as the best default behaviour.

如果不想让库打印这些信息怎么办？

> you can attach a do-nothing handler to the top-level logger for your library

也就是说，在父级logger中添加一个`NullHandler`，那么作为子级logger会自动继承这个handler，从而达到子级logger不打印任何信息的目的

例如：已知一个库foo使用的logging模块配置中，logger name是‘foo.x’, ‘foo.x.y’，那么

~~~python
import logging
logging.getLogger('foo').addHandler(logging.NullHandler())
~~~

就可以实现不打印的效果





## 关于logging报错

如果logging过程中发生错误，除了`SystemExit` 和 `KeyboardInterrupt`，其他的错误不会导致appliaction崩溃，是因为The logging package is designed to swallow exceptions which occur while logging in production，这些错误会被passed to its `handleError()` method。

logging的raiseExceptions默认是True，表示会将错误信息打印出来

> The default implementation of [`handleError()`](https://docs.python.org/3.5/library/logging.html#logging.Handler.handleError) in `Handler` checks to see if a module-level variable, `raiseExceptions`, is set. If set, a traceback is printed to [`sys.stderr`](https://docs.python.org/3.5/library/sys.html#sys.stderr). If not set, the exception is swallowed.
>
> Note
>
> The default value of `raiseExceptions` is `True`. This is because during development, you typically want to be notified of any exceptions that occur. It’s advised that you set `raiseExceptions` to `False` for production usage.







# Handlers

学习一些使用的handler

## RotatingFileHandler实现记录基于内容大小切分

> *class* `logging`.`handlers`.`RotatingFileHandler`(*filename*, *mode='a'*, *maxBytes=0*, *backupCount=0*, *encoding=None*, *delay=False*)

maxBytes：写入内容大小超过当前值时，当前文件将会关闭，然后打开新文件继续保存

backupCount：决定保存文件的最大数量。每次另存新文件时，旧文件都会被修改扩展名，如第一次app.log被修改为app.log.1，第二次app.log.1会被改成app.log.2，而将新的app.log修改为app.log.1，也就是说，新文件永远是app.log

**maxBytes和backupCount任意一个为0，都将不会执行rollover**

delay:文件在第一次写入的时候才会打开，If *delay* is true, then file opening is deferred until the first call to [`emit()`](https://docs.python.org/3.5/library/logging.handlers.html#logging.handlers.RotatingFileHandler.emit). By default, the file grows indefinitely



## TimedRotatingFileHandler实现记录基于时间切分

> *class* `logging.handlers.``TimedRotatingFileHandler`(*filename*, *when='h'*, *interval=1*, *backupCount=0*, *encoding=None*, *delay=False*, *utc=False*, *atTime=None*)

when:指定保存的周期单位

| Value        | Type of interval                                |
| ------------ | ----------------------------------------------- |
| `'S'`        | Seconds                                         |
| `'M'`        | Minutes                                         |
| `'H'`        | Hours                                           |
| `'D'`        | Days                                            |
| `'W0'-'W6'`  | Weekday (0=Monday) 选择该项时interval参数不生效 |
| `'midnight'` | Roll over at midnight                           |

interval：指定周期

backupCount：指定最大保存文件数，保存数达到最大值时，将会删除最旧的文件并创建新文件。文件名后缀格式为：`%Y-%m-%d_%H-%M-%S`

delay：文件在第一次写入的时候才会打开

utc：If the *utc* argument is true, times in UTC will be used; otherwise local time is used.

atTime：需要`datetime.time` instance which specifies the time of day when rollover occurs





