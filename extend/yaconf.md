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
在yaconf.c中实现上面这个php_yaconf_update方法, 基本copy了PHP_RINIT_FUNCTION中的逻辑。
检测是否存在check_delay这个字段和是否超过单次时间检测，如果没有超过则返回。

```c
    if (YACONF_G(check_delay) && (time(NULL) - YACONF_G(last_check) < YACONF_G(check_delay))) {
		// 检测时间 
		YACONF_DEBUG("config check delay doesn't execceed, ignore");
		return SUCCESS;
	}
```
如果符合更新条件则继续向下执行。然后检测配置目录的字符串是否为目录。如果检测到之前记录的时间和目录的mtime相同则不向下执行。
然后进行遍历文件夹的文件。注意:这里不会去遍历配置文件夹下的二层文件夹的内容。
对于每个文件进行如下检测：

1. 检测文件名字是否是abc.ini的格式，如果出现其他格式则pass。
```c
    if (!(p = strrchr(namelist[i]->d_name, '.')) || strcmp(p, ".ini")) {
        // 返回 . 首次出现的位置，并且返回从这个开始的字符串
        // 所以 a.b.ini 不符合条件
        free(namelist[i]);
        continue;
    }
```
2. 判断是否是常规文件，不是则pass。
```c
    // 单个文件目录+文件  最多255个字符
    if (VCWD_STAT(ini_file, &sb) || !S_ISREG(sb.st_mode)) {
        free(namelist[i]);
        continue;
    }
```
3. 判断是否是已经加载的配置文件，如果有则比对文件的mtime，相等则pass。
```c
    if ((node = (yaconf_filenode*)zend_hash_str_find_ptr(parsed_ini_files, namelist[i]->d_name, strlen(namelist[i]->d_name))) == NULL) {
        YACONF_DEBUG("new configure file found");
    } else if (node->mtime == sb.st_mtime) {
        free(namelist[i]);
        continue;
    }
```
4. 进行对ini文件进行解析，这里调用了zend_parse_ini_file这个方法。这个方法应该是可以传入解析规则的解析方法，在php源码中有使用的例子。
```c
if (zend_parse_ini_file(&fh, 1, 0 /* ZEND_INI_SCANNER_NORMAL */,
        php_yaconf_ini_parser_cb, (void *)&result) == FAILURE || YACONF_G(parse_err)) {
    YACONF_G(parse_err) = 0;
    php_yaconf_hash_destroy(Z_ARRVAL(result));
    free(namelist[i]);
    continue;
}

```
5. 如果是对已有配置的更新，则先删除之前的配置。然后挂载新的配置。如果是新的则直接更新hashtable ini_containers。
并且同步更新记录加载了文件信息的hashtable parsed_ini_files，如果是update则直接 更新对应节点的mtime, 如果是新增的，则用zend_hash_update_mem添加一条。
```c
    file_key = php_yaconf_str_persistent(namelist[i]->d_name, p - namelist[i]->d_name);
    if ((orig_ht = zend_symtable_find(ini_containers, file_key)) != NULL) {
        php_yaconf_hash_destroy(Z_ARRVAL_P(orig_ht));
        ZVAL_COPY_VALUE(orig_ht, &result);
        free(file_key);
    } else {
        php_yaconf_symtable_update(ini_containers, file_key, &result);
    }

    if (node) {
        node->mtime = sb.st_mtime;
    } else {
        yaconf_filenode n = {0};
        n.filename = zend_string_init(namelist[i]->d_name, strlen(namelist[i]->d_name), 1);
        n.mtime = sb.st_mtime;
        zend_hash_update_mem(parsed_ini_files, n.filename, &n, sizeof(yaconf_filenode));
    }  

```

注意：
1. 最后需要释放掉一些使用的空间，如在过程中记录遍历文件的列表变量namelist。
2. 如果已经加载了a.ini的配置信息，如果a.ini删除，在PHP_RINIT_FUNCTION和上面update方法中不能清除配置，需要进行PHP_MINIT_FUNCTION重新加载，即php-fpm重启。

在yaconf.c中定义类的方法update。调用php_yaconf_update方法：
```c
PHP_METHOD(yaconf, update) {
	
	RETURN_BOOL(php_yaconf_update());
}
```
