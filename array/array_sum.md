## array_sum
----
对array内的所有元素加合
整体代码在ext/standard/array.c，代码如下：
```c
    /* {{{ proto mixed array_sum(array input)
       Returns the sum of the array entries */
    PHP_FUNCTION(array_sum)
    {
    	zval *input,
    		 *entry,
    		 entry_n;
    
    	if (zend_parse_parameters(ZEND_NUM_ARGS(), "a", &input) == FAILURE) {
    		return;
    	}
    
    	ZVAL_LONG(return_value, 0);
    
    	ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(input), entry) {
    		if (Z_TYPE_P(entry) == IS_ARRAY || Z_TYPE_P(entry) == IS_OBJECT) {
    			continue;
    		}
    		ZVAL_COPY(&entry_n, entry);
    		convert_scalar_to_number(&entry_n);
    		fast_add_function(return_value, return_value, &entry_n);
    	} ZEND_HASH_FOREACH_END();
    }
    /* }}} */

```
首先声明变量类型为zval指针类型input，接收参数。设置返回值为0。
```c
   zval *input
   ...
   if (zend_parse_parameters(ZEND_NUM_ARGS(), "a", &input) == FAILURE) {
   		return;
   	}
   	ZVAL_LONG(return_value, 0);
```
循环数组，如果值是array,object类型则pass.
```c
ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(input), entry) {
    if (Z_TYPE_P(entry) == IS_ARRAY || Z_TYPE_P(entry) == IS_OBJECT) {
        continue;
    }

} ZEND_HASH_FOREACH_END();

```
拷贝内容，强制类型转换成number:
```c
// entry_n类型 zval *
ZVAL_COPY(&entry_n, entry);
convert_scalar_to_number(&entry_n);
// convert_scalar_to_number 是底层的系统函数
```
进行相加。
```c
fast_add_function(return_value, return_value, &entry_n);
```
fast_add_function具体底层实现(扩展以下)：
```c
static zend_always_inline int fast_add_function(zval *result, zval *op1, zval *op2)
{
	if (EXPECTED(Z_TYPE_P(op1) == IS_LONG)) {
		if (EXPECTED(Z_TYPE_P(op2) == IS_LONG)) {
			fast_long_add_function(result, op1, op2);
			return SUCCESS;
		} else if (EXPECTED(Z_TYPE_P(op2) == IS_DOUBLE)) {
			ZVAL_DOUBLE(result, ((double)Z_LVAL_P(op1)) + Z_DVAL_P(op2));
			return SUCCESS;
		}
	} else if (EXPECTED(Z_TYPE_P(op1) == IS_DOUBLE)) {
		if (EXPECTED(Z_TYPE_P(op2) == IS_DOUBLE)) {
			ZVAL_DOUBLE(result, Z_DVAL_P(op1) + Z_DVAL_P(op2));
			return SUCCESS;
		} else if (EXPECTED(Z_TYPE_P(op2) == IS_LONG)) {
			ZVAL_DOUBLE(result, Z_DVAL_P(op1) + ((double)Z_LVAL_P(op2)));
			return SUCCESS;
		}
	}
	return add_function(result, op1, op2);
}


```
