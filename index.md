## 初篇介绍

----
毕业十年了，不再浑浑噩噩，没有了当初的舒适气氛，处处感到压力，而今我们只有砥砺前行，才能等到彼岸花开。
愿开篇顺利，有些经验的东西，分享给大家，和大家一起进步。

本博客包括：

>* 一些经验总结
>* 一起分析一些扩展开发

目录:

### extend function add by fuyuan
>* [ self_concats ](https://netbird.github.io/array/self_concats)
>* [ array_get ](https://netbird.github.io/array/array_get)

### base function (ext/standard/array.c)
>* [ array_sum ](https://netbird.github.io/array/array_sum)
```
PHP_FUNCTION(ksort);
PHP_FUNCTION(krsort);
PHP_FUNCTION(natsort);
PHP_FUNCTION(natcasesort);
PHP_FUNCTION(asort);
PHP_FUNCTION(arsort);
PHP_FUNCTION(sort);
PHP_FUNCTION(rsort);
PHP_FUNCTION(usort);
PHP_FUNCTION(uasort);
PHP_FUNCTION(uksort);
PHP_FUNCTION(array_walk);
PHP_FUNCTION(array_walk_recursive);
PHP_FUNCTION(count);
PHP_FUNCTION(end);
PHP_FUNCTION(prev);
PHP_FUNCTION(next);
PHP_FUNCTION(reset);
PHP_FUNCTION(current);
PHP_FUNCTION(key);
PHP_FUNCTION(min);
PHP_FUNCTION(max);
PHP_FUNCTION(in_array);
PHP_FUNCTION(array_search);
PHP_FUNCTION(extract);
PHP_FUNCTION(compact);
PHP_FUNCTION(array_fill);
PHP_FUNCTION(array_fill_keys);
PHP_FUNCTION(range);
PHP_FUNCTION(shuffle);
PHP_FUNCTION(array_multisort);
PHP_FUNCTION(array_push);
PHP_FUNCTION(array_pop);
PHP_FUNCTION(array_shift);
PHP_FUNCTION(array_unshift);
PHP_FUNCTION(array_splice);
PHP_FUNCTION(array_slice);
PHP_FUNCTION(array_merge);
PHP_FUNCTION(array_merge_recursive);
PHP_FUNCTION(array_replace);
PHP_FUNCTION(array_replace_recursive);
PHP_FUNCTION(array_keys);
PHP_FUNCTION(array_values);
PHP_FUNCTION(array_count_values);
PHP_FUNCTION(array_column);
PHP_FUNCTION(array_reverse);
PHP_FUNCTION(array_reduce);
PHP_FUNCTION(array_pad);
PHP_FUNCTION(array_flip);
PHP_FUNCTION(array_change_key_case);
PHP_FUNCTION(array_rand);
PHP_FUNCTION(array_unique);
PHP_FUNCTION(array_intersect);
PHP_FUNCTION(array_intersect_key);
PHP_FUNCTION(array_intersect_ukey);
PHP_FUNCTION(array_uintersect);
PHP_FUNCTION(array_intersect_assoc);
PHP_FUNCTION(array_uintersect_assoc);
PHP_FUNCTION(array_intersect_uassoc);
PHP_FUNCTION(array_uintersect_uassoc);
PHP_FUNCTION(array_diff);
PHP_FUNCTION(array_diff_key);
PHP_FUNCTION(array_diff_ukey);
PHP_FUNCTION(array_udiff);
PHP_FUNCTION(array_diff_assoc);
PHP_FUNCTION(array_udiff_assoc);
PHP_FUNCTION(array_diff_uassoc);
PHP_FUNCTION(array_udiff_uassoc);

PHP_FUNCTION(array_product);
PHP_FUNCTION(array_filter);
PHP_FUNCTION(array_map);
PHP_FUNCTION(array_key_exists);
PHP_FUNCTION(array_chunk);
PHP_FUNCTION(array_combine);
```

### extension tes
>* [ yaconf测试及二次编辑 ](https://netbird.github.io/extend/yaconf)

---
#### 微博：[@福愿_飞鸟](https://weibo.com/teacherbird/home?wvr=5)

---

三十而立，刚刚起步，我就已经碰到了很多的困难，期间得到了许多帮助，聊表感谢～
