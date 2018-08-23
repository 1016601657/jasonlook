---
title: 经典算法-冒泡排序
date: 2016-07-04 10:06:40
tags: [算法,php,排序]
categories: 算法
---
>冒泡的原理就是相邻的两个数字进行比较满足条件交替位置,将最大或者最小的数字依次排到尾部.

<!-- more -->
### PHP-实现方式

``` bash
  $arr = array(1,43,54,62,21,66,32,78,36,76,39);
  $num = count($arr);
  for ($a = 1; $a < $num; $a++){
    for($b = 0; $b < $num - $a; $b++){
  	  if($arr[$b] < $arr[$b + 1]){
                $tmp = $arr[$b];
                $arr[$b] = $arr[$b + 1];
                $arr[$b + 1] = $tmp;
  	  }
  	}
  }
  var_dump($arr);

```