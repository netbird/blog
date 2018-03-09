## yaconf

----
读了一遍yaconf的源码，发现大部分可以理解。在这个项目上进行练习，进行一些小改动是个比较有趣的事情。

### 问题描述
>* 虽然yaconf是为web请求模式开发的，但是在CLI模式下，一个使用了Yaconf的正在运行的程序，如何更新yaconf的内容。必须每次停止程序，重启一次么？

>* 同一台机器，是否可以根据host和port加载的配置文件？是否可以增加一种测试模式？
 
### 解决方案
对于以上两个问题，现在提供如下解决方案。
>* 增加update方法，用于更新已经加载的配置信息，逻辑同RINIT的逻辑。

>* 先实现基本的内容。增加测试模式，在测试模式下, 先遵循原有的规则，加载原来的配置信息。在RINIT方法中，开辟一段空间，根据配置加载一个测试目录。当用Yaconf::get Yaconf::has方法时，优先查找测试目录对应的内容，如果没有则查找原来的配置信息。 添加RSHUTDOWN方法，当请求结束的时候，释放这段空间。

### 添加update方法

在php_yaconf.h中添加php_yaconf_update函数定义.
```c
BEGIN_EXTERN_C() 
...
PHP_YACONF_API int php_yaconf_update();
...
END_EXTERN_C()
```