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

```coffeescript
    if (YACONF_G(check_delay) && (time(NULL) - YACONF_G(last_check) < YACONF_G(check_delay))) {
		// 检测时间 
		YACONF_DEBUG("config check delay doesn't execceed, ignore");
		return SUCCESS;
	}
```
如果符合条件则继续向下执行。

```php
    char *dirname;
    struct zend_stat dir_sb = {0};

    YACONF_G(last_check) = time(NULL);
    // 更新最后检测时间 为现在

    if ((dirname = YACONF_G(directory)) && !VCWD_STAT(dirname, &dir_sb) && S_ISDIR(dir_sb.st_mode)) {
        if (dir_sb.st_mtime == YACONF_G(directory_mtime)) {
            // 检测目标配置文件的路径是否是常规路径和检测
            YACONF_DEBUG("config directory is not modefied");
            return SUCCESS;
        } else {
            zval result;
            int i, ndir;
            struct dirent **namelist;
            char *p, ini_file[MAXPATHLEN]; // MAXPATHLEN = 256

            YACONF_G(directory_mtime) = dir_sb.st_mtime;

            if ((ndir = php_scandir(dirname, &namelist, 0, php_alphasort)) > 0) {
                // 检索文件夹里的有效文件
                // ndir 是返回值
                zend_string *file_key;
                struct zend_stat sb;
                zend_file_handle fh = {0};
                yaconf_filenode *node = NULL;

                for (i = 0; i < ndir; i++) {
                    zval *orig_ht = NULL;
                    if (!(p = strrchr(namelist[i]->d_name, '.')) || strcmp(p, ".ini")) {
                        // 返回 . 首次出现的位置，并且返回从这个开始的字符串
                        // 所以 a.b.ini 不符合条件
                        free(namelist[i]);
                        continue;
                    }

                    snprintf(ini_file, MAXPATHLEN, "%s%c%s", dirname, DEFAULT_SLASH, namelist[i]->d_name);

                    // 单个文件目录+文件  最多255个字符
                    if (VCWD_STAT(ini_file, &sb) || !S_ISREG(sb.st_mode)) {
                        free(namelist[i]);
                        continue;
                    }

                    // 从ini 文件记录表中 找到文件的mtime和文件的mtime进行比对 如果相等则pass
                    if ((node = (yaconf_filenode*)zend_hash_str_find_ptr(parsed_ini_files, namelist[i]->d_name, strlen(namelist[i]->d_name))) == NULL) {
                        YACONF_DEBUG("new configure file found");
                    } else if (node->mtime == sb.st_mtime) {
                        free(namelist[i]);
                        continue;
                    }

                    if ((fh.handle.fp = VCWD_FOPEN(ini_file, "r"))) {
                        fh.filename = ini_file;
                        fh.type = ZEND_HANDLE_FP;
                        ZVAL_UNDEF(&active_ini_file_section);
                        YACONF_G(parse_err) = 0;
                        php_yaconf_hash_init(&result, 128);

                        if (zend_parse_ini_file(&fh, 1, 0 /* ZEND_INI_SCANNER_NORMAL */,
                                php_yaconf_ini_parser_cb, (void *)&result) == FAILURE || YACONF_G(parse_err)) {
                            YACONF_G(parse_err) = 0;
                            php_yaconf_hash_destroy(Z_ARRVAL(result));
                            free(namelist[i]);
                            continue;
                        }
                    }


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
                    free(namelist[i]);
                }
                free(namelist);
            }
            return SUCCESS;
        }
    } 
```



在yaconf.c中定义类的方法update。调用php_yaconf_update方法：
```c
PHP_METHOD(yaconf, update) {
	
	RETURN_BOOL(php_yaconf_update());
}
```
