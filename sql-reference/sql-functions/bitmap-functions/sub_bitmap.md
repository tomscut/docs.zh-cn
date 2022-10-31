# sub_bitmap

## 功能

从 `offset` 指定的位置开始，从 BITMAP 类型的 `src` 中截取 `len` 个元素，返回 `src` 的子集。返回值类型仍为 BITMAP。如果参数非法，则返回 NULL。该函数主要用于分页查询相关的场景。

## 语法

```Haskell
BITMAP SUB_BITMAP(BITMAP src, BIGINT offset, BIGINT len)
```

## 参数说明

`src`: 要截取的值，支持的数据类型为 BITMAP。

`offset`: 指定的偏移量，支持的数据类型为 BIGINT。

`len`: 截取的元素个数，支持的数据类型为 BIGINT。

## 返回值说明

返回值的数据类型为 BITMAP。

## 注意事项

* 偏移量从 0 开始。
* 负偏移量指的是从字符串结尾开始计数的偏移量。
* 输出的子集内的元素不能重复。

## 示例

示例一：从偏移量 0 开始，输出 bitmap 值中 2 个不重复的元素。

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例二：从偏移量 0 开始，输出 bitmap 值中 100 个不重复的元素。由于 bitmap 值中没有 100 个元素，所以输出符合条件的所有元素。

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例三：从偏移量 -3 开始（从右往左第 4 个元素），输出 bitmap 值中 100 个不重复的元素。由于 bitmap 值中没有 100 个元素，所以输出符合条件的所有元素。

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

示例四：从偏移量 -3 开始（从右往左数第 4 个元素），输出 bitmap 值中 2 个不重复的元素。

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

示例四：-10 为不合法输入，返回 NULL。

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```
