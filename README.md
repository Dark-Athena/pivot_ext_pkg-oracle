# pivot_ext_pkg-oracle

## 项目地址:
https://github.com/Dark-Athena/pivot_ext_pkg-oracle

## 背景
众所周知,oracle的原生pivot功能,无法在"for ... in ()" 的括号中使用子查询,
比如,如果执行

```sql
select *
  from HR.COUNTRIES
pivot(MAX(COUNTRY_NAME)
   for COUNTRY_ID in (select distinct COUNTRY_ID from HR.COUNTRIES))
```

那么会收到报错 **ORA-00936** ,必须将对应的值手动枚举出来,比如

```sql
select *
  from HR.COUNTRIES
pivot(MAX(COUNTRY_NAME)
   for COUNTRY_ID in ('AR','AU','BE','BR','CA','CH','CN','DE','DK',
   'EG','FR','IL','IN','IT','JP','KW','ML','MX','NG','NL',
   'SG','UK','US','ZM','ZW'))
```

如果枚举值是动态变化的,那么sql将无法固定,这对数据报表的导出是个很麻烦的问题。
一般稍微聪明一点的开发者,会选择在程序中使用动态sql来拼接sql,但很难做出通用的拼接程序,并且就算通用,也要传入很多参数,还必须先创建视图,再查询视图。

所以我写了这个pivot的增强包,你可以直接在pivot"for ... in ()"的括号中写子查询,以这个"错误的"sql作为参数,查询此增强包中的函数,即可直接获得数据结果的展现。

## 程序功能用例
### 例1:最简单的用法

```sql
SELECT pivot_ext_pkg.get_cursor(Q'{select * from  HR.COUNTRIES
                               pivot(MAX(COUNTRY_NAME) for COUNTRY_ID in(
                              select distinct COUNTRY_ID from  HR.COUNTRIES  ))}')
  FROM DUAL;
```

![a](https://www.darkathena.top/upload/2021/12/image-abf046d9dcfc435298be70115f322f3b.png)


### 例2:将一个oracle 无法执行的 "pivot... in(select )" sql,转换成Oracle可直接执行的sql
此功能方便开发者使用输出的sql发邮件、导出数据等
```sql
SELECT pivot_ext_pkg.convert_sql(Q'{select * from  HR.COUNTRIES
                                   pivot(MAX(COUNTRY_NAME) for COUNTRY_ID in(
                                 select distinct COUNTRY_ID from  HR.COUNTRIES  ))}') a
  FROM DUAL;

```

![a](https://www.darkathena.top/upload/2021/12/image-9dce1724444c46aa9bb23f7c6888275f.png)

### 例3:输出逗号分割的数据(CSV格式)
```sql
 select *
   from pivot_ext_pkg.GET_DATA(i_sql           => Q'{select * from  HR.COUNTRIES
                                          pivot(MAX(COUNTRY_NAME) for COUNTRY_ID in(
                                           select distinct COUNTRY_ID from  HR.COUNTRIES  ))}',
                               format          => 'CSV',
                               field_delimiter => ',',
                               skip_header     => 'N');
```

![a](https://www.darkathena.top/upload/2021/12/image-d5ef7925b23b4a63bca482c2ee41d6ef.png)

### 例4:输出json格式数据
```sql
 select *
   from pivot_ext_pkg.GET_DATA(i_sql           => Q'{select * from  HR.COUNTRIES
                                          pivot(MAX(COUNTRY_NAME) for COUNTRY_ID in(
                                           select distinct COUNTRY_ID from  HR.COUNTRIES  ))}',
                               format          => 'JSON');
```

![a](https://www.darkathena.top/upload/2021/12/image-7c2909d1cf074570bb0a3b39f5d62cc4.png)

### 例5:输出xml格式数据
```sql
 select *
   from pivot_ext_pkg.GET_DATA(i_sql           => Q'{select * from  HR.COUNTRIES
                                          pivot(MAX(COUNTRY_NAME) for COUNTRY_ID in(
                                           select distinct COUNTRY_ID from  HR.COUNTRIES  ))}',
                               format          => 'XML');
```

![a](https://www.darkathena.top/upload/2021/12/image-48d52993c8814424a1810a58eb231b78.png)

## 其他
由于oracle的sql语法解析相当复杂,本功能只是对关键字' for '来对字符串进行识别,检索出对应的子查询并将子查询的结果替换掉原有的子查询sql,如果sql中有使用unpivot,或者存在和关键字一样的字符串常量,此功能会报错,以后会再看unpivot的for是否也可以进行类似的转换。
除了上面提到的问题,如有用户测试报错,请联系作者或者在github上发issue
