---
title: '[代码借鉴]PHP循环遍历方法'
date: 2019-07-21 11:40:00
tags: wordpress
---

> 优秀代码值得反复阅读，深入理解！

```
/**
 * Maps a function to all non-iterable elements of an array or an object.
 * 将函数映射到数组或对象的所有不可迭代元素。
 * (个人理解，遍历对象或数组并通过一个回调方法处理值)
 *
 * This is similar to `array_walk_recursive()` but acts upon objects too.
 * 虽然和`array_walk_recursive()`类似但是这个在对象上也起作用。
 *
 * @since 4.4.0
 *
 * @param mixed    $value    The array, object, or scalar.
 * @param callable $callback The function to map onto $value.
 * @return mixed The value with the callback applied to all non-arrays and non-objects inside it.
 */
function map_deep( $value, $callback ) {
	if ( is_array( $value ) ) {
		foreach ( $value as $index => $item ) {
			$value[ $index ] = map_deep( $item, $callback );
		}
	} elseif ( is_object( $value ) ) {
		$object_vars = get_object_vars( $value );
		foreach ( $object_vars as $property_name => $property_value ) {
			$value->$property_name = map_deep( $property_value, $callback );
		}
	} else {
		$value = call_user_func( $callback, $value );
	}

	return $value;
}
```

> 该方法来自于WordPress项目中`wp-includes/formatting.php`
