# sub_bitmap

## 功能

将bitmap src从offset的位置开始，截取len个元素，返回src的子集，类型仍为bitmap。如果参数非法，则返回NULL。该函数主要用于分页查询相关的场景。

## 语法

```Haskell
BITMAP SUB_BITMAP(BITMAP src, BIGINT offset, BIGINT len)
```

## 参数说明

`src`: 支持的数据类型为 BITMAP。

`offset`: 支持的数据类型为 BIGINT。

`len`: 支持的数据类型为 BIGINT。

## 返回值说明

返回值的数据类型为 BITMAP。

## 示例

```Plain Text
mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+

mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+

mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+

mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+

mysql> select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```
