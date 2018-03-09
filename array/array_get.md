## array_get

----
先来看看php的实现：
```php
    function array_get($array, $key, $default = null)
    {
        if (is_null($key)) {
            return $array; 
        }

        if (isset($array[$key])) {
            return $array[$key];
        }

        foreach (explode('.', $key) as $segment) {
            if (! is_array($array) || ! array_key_exists($segment, $array)) {
                return value($default);
            }

            $array = $array[$segment];
        }

        return $array;
    }

```

用扩展实现如下(没有考虑key为空的情况):
```c
    PHP_FUNCTION(array_get)
    {
    	zval *arr; // array
    	zend_string* strkey; // key
    	zval *defaultval = NULL; // default value
    	zval *retval;
    	HashTable *arrHashTable;
    	zval *dest_entry;
    	if (zend_parse_parameters(ZEND_NUM_ARGS(), "zS|z", &arr, &strkey, &defaultval) == FAILURE) {
    	    return;
    	}
    	if ((retval = zend_hash_find(Z_ARRVAL_P(arr), strkey)) != NULL){
    	    RETURN_ZVAL(retval, 1, 0);
    	} 
    	// foreach
    	if (zend_memrchr(ZSTR_VAL(strkey), '.', ZSTR_LEN(strkey))) {
            char *entry, *ptr, *seg;
            HashTable *target = Z_ARRVAL_P(arr);
            entry = estrndup(ZSTR_VAL(strkey), ZSTR_LEN(strkey));
            if ((seg = php_strtok_r(entry, ".", &ptr))) {
                do {
                    if (target == NULL || (retval = zend_symtable_str_find(target, seg, strlen(seg))) == NULL) {
                        break;
                    }
            
                    if (Z_TYPE_P(retval) == IS_ARRAY) {
                        target = Z_ARRVAL_P(retval);
                    } else {
                        target = NULL;
                    }
                } while ((seg = php_strtok_r(NULL, ".", &ptr)));
            }
    		efree(entry);
    		if (retval) {
    			RETURN_ZVAL(retval, 1, 0);
    		}
    	}
    	// end foreach
    	if (defaultval) {
    	    // 如果有默认值，则返回默认值
    		RETURN_ZVAL(defaultval, 1, 0);
    	} else {
    		RETURN_NULL();
    	}
    }

```