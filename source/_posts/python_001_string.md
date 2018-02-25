
---
title: python之字符串篇
date: 2017-10-08 20:12:45
updated: 2017-10-09 20:40:56
tags: [python]
---
python自学之路
<!--more-->

capitaliz() 首字母大写
```
>>> str = 'magicyou'
>>> print(str.capitalize())
Magicyou
```
count() 统计指定元素在字符串中出现次数
```
>>> str = 'my blog \'name is magicyou'
>>> print(str.count('m'))
3
```
center() 以指定的字符占位，返回制定长度的字符串，原字符串居中
```
>>> str = 'my blog \'name is magicyou'
>>> print(str.center(50,'-'))
------------my blog 'name is magicyou-------------
```
endswith() 判断是否以某个字符串结尾，返回布尔值
```
>>> str = 'my blog \'name is magicyou'
>>> print(str.endswith('you'))
True
>>> print(str.endswith('me'))
False
```
expandtabs() 将字符串中的tab替换成指定长度的空格
```
>>> str = 'magic\tyou'
>>> print(str.expandtabs(tabsize=30))
magic                         you
>>>
```
find() 查找某字符在已知字符串中的位置
```
>>> str = 'my blog \'name is magicyou'
>>> print(str.find('name'))
9
```
format() 字符串格式化
```
# 1、使用位置参数
>>> str = 'my name is {},age {}'
>>> print(str.format('magicYou',23))
my name is magicYou,age 23
# 2、使用关键字参数
>>> str = 'my name is {name},age {age}'
>>> print(str.format(name='magicYou',age=23))
my name is magicYou,age 23
# 3、填充与格式化
:[填充字符][对齐方式 <^>][宽度]
>>> '{0:*>10}'.format('abc')
'*******abc'
>>> '{0:*<10}'.format('abc')
'abc*******'
>>> '{0:*^10}'.format('abc')
'***abc****'
# 4、精度与进制
>>> '{0:.2f}'.format(0.66666)
'0.67'
>>> '{0:b}'.format(10)
'1010'
>>> '{0:o}'.format(10)
'12'
>>> '{0:x}'.format(10)
'a'
>>> '{:,}'.format(123456789)
'123,456,789'
# 5、使用索引
>>> info = ['magicYou',23]
>>> 'name is {0[0]} age is {0[1]}'.format(info)
'name is magicYou age is 23'
```
format_map() 格式化字符串
```
>>> hash = {'name':'magicYou','age':23}
>>> str = 'i am {name}, i am {age}'
>>> print(str.format_map(hash))
i am magicYou, i am 23
```
isalnum() 检测字符串是否由字母或数字组成
```
>>> str = 'magicYou'
>>> print(str.isalnum())
True
>>> str = '56ppp'
>>> print(str.isalnum())
True
>>> str = '*&^'
>>> print(str.isalnum())
False
```
isalpha() 检测字符串是否只有字母
```
>>> str = 'magicYou'
>>> print(str.isalpha())
True
>>> str = 'magicYou4'
>>> print(str.isalpha())
False
```
isdecimal() 检查字符串是否只包含十进制字符。这种方法只存在于unicode对象
注意:定义一个十进制字符串，只需要在字符串前添加 'u' 前缀即可
```
>>> str = u'123456'
>>> print(str.isdecimal())
True
>>> str = u'123456qq'
>>> print(str.isdecimal())
False
```
isdigit() 检测字符串是否只由数字组成。
```
>>> str = '123456'
>>> print(str.isdigit())
True
>>> str = '123456qq'
>>> print(str.isdigit())
False
```
isidentifier() 检测是否是一个合法的标识符
注意：合法的标识符就是符合变量名的就是合法标识符（只能以字母或下划线开头）
```
>>> str = '123456'
>>> print(str.isidentifier())
False
>>> str = 'name123'
>>> print(str.isidentifier())
True
>>> str = '_123456'
>>> print(str.isidentifier())
True
```
isnumeric() 检测是否为全为数字（小数也不算）
```
>>> str = '123456'
>>> print(str.isnumeric())
True
>>> str = '1234.56'
>>> print(str.isnumeric())
False
```
isspace() 检测是否只有空格组成
```
>>> str = '123   456'
>>> print(str.isspace())
False
>>> str = '       '
>>> print(str.isspace())
True
```
istitle() 检测字符串中所有的单词拼写首字母是否为大写，且其他字母为小写
```
>>> str = 'My Name Is MagicYou'
>>> print(str.istitle())
False
>>> str = 'My Name Is Magicyou'
>>> print(str.istitle())
True
>>> str = 'My name is magicYou'
>>> print(str.istitle())
False
```
isupper() 检测是否全市大写字母
```
>>> str = 'MAGICYOU'
>>> print(str.isupper())
True
>>> str = 'MagicYou'
>>> print(str.isupper())
False
```

join() 将字符串、元组、列表中的元素以指定的字符(分隔符)连接生成一个新的字符串
```
>>> print('-'.join(['m','a','g','i','c']))
m-a-g-i-c
>>> print('-'.join(('m','a','g','i','c')))
m-a-g-i-c
>>> print('-'.join('magic'))
m-a-g-i-c
```
ljust() 返回一个原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串
rjust() 返回一个原字符串右对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串
```
>>> str = 'magic'
>>> print(str.ljust(50,'*'))
magic*********************************************
>>> print(str.rjust(50,'*'))
*********************************************magic
```
lower() 转换字符串中所有大写字符为小写
```
>>> str = 'MAGICYOU'
>>> print(str.lower())
magicyou
```

upper() 转换字符串中所有小写字符为大写
```
>>> str = 'magic'
>>> print(str.upper())
MAGIC
```
lstrip() 截掉字符串左边的空格或指定字符。
```
>>> str = '   magic'
>>> print(str.lstrip())
magic
>>> str = 'PPmagic'
>>> print(str.lstrip('PP'))
magic
```

strip() 移除字符串头尾指定的字符（默认为空格）
```
>>> str = '   magic     '
>>> print(str.strip())
magic
>>> str = 'PPmagicPP'
>>> print(str.strip('PP'))
magic
```
maketrans(str1,str2) 创建字符映射的转换表，对于接受两个参数的最简单的调用方式，第一个参数是字符串，表示需要转换的字符，第二个参数也是字符串表示转换的目标
```
>>> p = str.maketrans('ace','135')
>>> print('abcdefghi'.translate(p))
1b3d5fghi
```
replace(str1, str2, [max]) 替换字符串中的str1为str2，最多不超过max次
```
>>> str = 'i am magicYou'
>>> print(str.replace('m', 'M'))
i aM MagicYou
>>> print(str.replace('m', 'M', 1))
i aM magicYou
```

find(str, start, end) 检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，如果包含子字符串返回开始的索引值，否则返回-1
```
>>> str = 'i am magicYou'
>>> str.find('m')
3
>>> str.find('m', 3)
3
>>> str.find('m', 8, 10)
-1
```
rfind() 返回字符串最后一次出现的位置(从右向左查询)，如果没有匹配项则返回-1
```
>>> str = 'i am magicYou'
>>> str.rfind('m')
5
>>> str.rfind('m', 6, 8)
-1
>>> str.rfind('m', 5)
5
>>> str.rfind('m', 4)
5
```
split(str, [num]) 通过指定分隔符对字符串进行切片，返回列表，如果参数num 有指定值，则仅分隔 num 个子字符串
```
>>> str = 'i am magicYou'
>>> str.split(' ')
['i', 'am', 'magicYou']
>>> str.split(' ', 2)
['i', 'am', 'magicYou']
>>> str.split(' ', 1)
['i', 'am magicYou']
```
splitlines()  根据换行符('\r', '\r\n', \n')分割成列表
```
>>> print('1+2\n3+5'.splitlines())
['1+2', '3+5']
```
swapcase() 将字符串中的大写换小写，小写换大写
```
>>> str = 'i am magicYou'
>>> str.swapcase()
'I AM MAGICyOU'
```
title() 将字符串中的每个单词的首字母转换为大写
```
>>> str = 'i am magicYou'
>>> str.title()
'I Am Magicyou'
```
zfill() 方法返回指定长度的字符串，原字符串右对齐，前面填充0
```
>>> str.zfill(50)
'000000000000000000000000000000000000000000magicYou'
```

此外，字符串也可以切片(和列表类似)
```
>>> str = 'magicYou'
>>> print(str[2:])
gicYou
>>> print(str[2:3])
g
>>> print(str[:6])
magicY
```