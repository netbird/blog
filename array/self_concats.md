## self_concats

----
参照网上的说明自己亲手敲下代码, 作为php扩展的入门吧。

```php
    // php函数
    function self_concat($string, $n){
        $result = "";
        for($i = 0; $i < $n; $i++){
            $result .= $string;
        }
        return $result;
    }

```

用扩展方式实现：

```c
PHP_FUNCTION(self_concats)
{
    char *str = NULL;
    size_t str_len;
    zend_long n;

    char *result; /* point to result*/
    char *ptr;
    size_t result_len;
    
    /** 接收变量 */
    if (zend_parse_parameters(ZEND_NUM_ARGS(), "sl", &str, &str_len, &n) == FAILURE) {
        return;
    }
    
    result_len = str_len * n;
    result = (char *)emalloc(result_len + 1);  // 分配空间
    ptr = result;
    
    while(n--) {
        memcpy(ptr, str, str_len);
        ptr += str_len;
    }
    // add end flag
    *ptr = '\0';
    RETURN_STRINGL(result, result_len);
}

```
