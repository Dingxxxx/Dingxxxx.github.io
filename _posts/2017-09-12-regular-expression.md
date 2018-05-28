---
layout: post
author:  Ding
title:  "正则表达式简介"
date:  2017-09-12
categories: Python
tags: 
- Linux 
- Python 
- Regular Expression
---

* content
{:toc}

> 正则表达式(可以称为REs，regex，regex pattens)是一个小巧的，高度专业化的编程语言，它内嵌于python开发语言中，可通过re模块使用。





## 模式符号


|    模式    |               含义               |         例子         |
|:----------:|:-------------------------------- |:--------------------:|
|    `\d`    | 匹配一个数字                     |                      |
|    `\D`    | 匹配一个非数字                   |                      |
|    `\w`    | 匹配一个字母或数字               |                      |
|    `\W`    | 匹配一个非字母和数字             |                      |
|    `\s`    | 匹配一个空白符 (空格、Tab、换行) |                      |
|    `\S`    | 匹配一个非空白字符               |                      |
|    `\n`    | 匹配一个换行符                   |                      |
|    `\t`    | 匹配一个制表符                   |                      |
|    `.`     | 匹配一个任意字符 (除了换行)      |                      |
|    `*`     | 前面的匹配项有任意个 (包括0个)   |                      |
|    `+`     | 匹配项至少一个                   |                      |
|    `?`     | 匹配项0个或1个                   |                      |
|   `{n}`    | 匹配n次                          |      `'\d{3}'`       |
|  `{n,m}`   | 匹配n-m个次                      |     `'\d{3,8}'`      |
|  `[0-9]`   | 匹配0-9中的一个                  |                      |
|  `[^0-9]`  | 匹配不是0-9的一个字符            |                      |
| `[0-9a-z]` | 匹配0-9或a-z中的一个             |                      |
|   `A|B`    | 匹配A或者B                       |   `'[P|p]ython '`    |
|    `^`     | 从一行的开头匹配                 |       `'^\d'`        |
|    `$`     | 在一行的结束匹配                 |       `'\d$'`        |
|    `\b`    | 匹配一个单词的便捷               | `'er\b'`匹配 'never' |
|    `\B`    | 匹配一个单词的内部               | `‘er\B’`匹配 'verb'  |
|  `(re) `   | 匹配括号内的表达式为一个组       |                      |

## Python re模块

### re模块的方法

* **re.compile**(pattern, flags=0)

把一个正则表达式pattern编译成正则对象，以便可以用正则对象的match和search方法。
得到的正则对象的行为（也就是模式）可以用flags来指定，值可以由几个下面的值OR得到。
以下两段内容在语法上是等效的：

```python
prog = re.compile(pattern)
result = prog.match(string)

result = re.match(pattern, string)
```

区别是，用了`re.compile`以后，正则对象会得到保留，这样在需要多次运用这个正则对象的时候，效率会有较大的提升。

**Flags**

+ **re.I**
re.IGNORECASE，让正则表达式忽略大小写，这样一来，[A-Z]也可以匹配小写字母了。此特性和locale无关。

+ **re.L**
re.LOCALE，让\w、\W、\b、\B、\s和\S依赖当前的locale。

+ **re.M**
re.MULTILINE，影响'^'和'$'的行为，指定了以后，'^'会增加匹配每行的开始（也就是换行符后的位置）；'$'会增加匹配每行的结束（也就是换行符前的位置）。

+ **re.S**
re.DOTALL，影响'.'的行为，平时'.'匹配除换行符以外的所有字符，指定了本标志以后，也可以匹配换行符。

+ **re.U**
re.UNICODE，让\w、\W、\b、\B、\d、\D、\s和\S依赖Unicode库。

+ **re.X**
re.VERBOSE，运用这个标志，你可以写出可读性更好的正则表达式：除了在方括号内的和被反斜杠转义的以外的所有空白字符，都将被忽略，而且每行中，一个正常的井号后的所有字符也被忽略，这样就可以方便地在正则表达式内部写注释了。也就是说，下面两个正则表达式是等效的：

```python
a = re.compile(r"""\d +  # the integral part
                   \.    # the decimal point
                   \d *  # some fractional digits
                """, re.X)
b = re.compile(r"\d+\.\d*")

```

* **re.match**(pattern, string, flags=0)

如果字符串string的开头和正则表达式pattern匹配的话，返回一个相应的MatchObject的实例，否则返回None。
注意：要在字符串的任意位置搜索的话，需要使用上面的search()。

* **re.search**(pattern, string, flags=0)

如果字符串string的任意位置和正则表达式pattern匹配的话，返回一个相应的MatchObject的实例，否则返回None。

* **re.split**(pattern, string[, maxsplit=0])

用匹配pattern的子串来分割string，如果pattern里使用了圆括号，那么被pattern匹配到的串也将作为返回值列表的一部分。

如果maxsplit不为0，则最多被分割为maxsplit个子串，剩余部分将整个地被返回。

```python
>>> re.split('\W+', 'Words, words, words.')
['Words', 'words', 'words', '']
>>> re.split('(\W+)', 'Words, words, words.')
['Words', ', ', 'words', ', ', 'words', '.', '']
>>> re.split('\W+', 'Words, words, words.', 1)
['Words', 'words, words.']
```

* **re.findall**(pattern, string, flags=0)

以列表的形式返回string里匹配pattern的不重叠的子串。string会被从左到右依次扫描，返回的列表也是从左到右一次匹配到的。

如果pattern里含有组的话，那么会返回匹配到的组的列表；如果pattern里有多个组，那么各组会先组成一个元组，然后返回值将是一个元组的列表。

由于这个函数不会涉及到MatchObject之类的概念，所以，对新手来说，应该是最好理解也最容易使用的一个函数了。下面就此来举几个简单的例子：

```python
#简单的findall
>>> re.findall('\w+', 'hello, world!')
['hello', 'world']
#这个返回的就是元组的列表
>>> re.findall('(\d+)\.(\d+)\.(\d+)\.(\d+)', 'My IP is 192.168.0.2, and your is 192.168.0.3.')
[('192', '168', '0', '2'), ('192', '168', '0', '3')]
```

* **re.finditer**(pattern, string, flags=0)

和上面的findall()类似，但返回的是MatchObject的实例的迭代器。

```python
>>> for m in re.finditer('\w+', 'hello, world!'):
... print m.group()
...
hello
world
```

* **re.sub**(pattern, repl, string, count=0, flags=0)

将string里匹配pattern的部分，用repl替换掉，最多替换count次（剩余的匹配将不做处理），然后返回替换后的字符串。如果string里没有可以匹配pattern的串，将被原封不动地返回。repl可以是一个字符串，也可以是一个函数（也可以参考我以前的例子）。如果repl是个字符串，则其中的反斜杆会被处理过，比如 \n 会被转成换行符，反斜杆加数字会被替换成相应的组，比如 \6 表示pattern匹配到的第6个组的内容。

```python
>>> re.sub(r'def\s+([a-zA-Z_][a-zA-Z_0-9]*)\s*\(\s*\):',
...        r'static PyObject*\npy_\1(void)\n{',
...        'def myfunc():')
'static PyObject*\npy_myfunc(void)\n{'
```

如果repl是个函数，每次pattern被匹配到的时候，都会被调用一次，传入一个匹配到的MatchObject对象，需要返回一个字符串，在匹配到的位置，就填入返回的字符串。

```python
>>> def dashrepl(matchobj):
...     if matchobj.group(0) == '-': return ' '
...     else: return '-'
>>> re.sub('-{1,2}', dashrepl, 'pro----gram-files')
'pro--gram files'
```

零长度的匹配也会被替换，比如：

```python
>>> re.sub('x*', '-', 'abcxxd')
'-a-b-c-d-'
```

特殊地，在替换字符串里，如果有\g这样的写法，将匹配正则的命名组（前面介绍过的，(?P...)这样定义出来的东西）。\g这样的写法，也是数字的组，也就是说，\g<2>一般和\2是等效的，但是万一你要在\2后面紧接着写上字面意义的0，你就不能写成\20了（因为这代表第20个组），这时候必须写成\g<2>0，另外，\g<0>代表匹配到的整个子串。

```python
>>> re.sub('-(\d+)-', '-\g<1>0\g<0>', 'a-11-b-22-c')
'a-110-11-b-220-22-c'
```

* **re.subn**(pattern, repl, string[, count])

跟上面的sub()函数一样，只是它返回的是一个元组 (新字符串, 匹配到的次数)。

```python
>>> re.subn('-(\d+)-', '-\g<1>0\g<0>', 'a-11-b-22-c')
('a-110-11-b-220-22-c', 2)
```

* **re.escape**(string)

把string中，除了字母和数字以外的字符，都加上反斜杆。

```python
>>> print re.escape('abc123_@#$')
abc123\_\@\#\$
```

### MatchObject

`re.MatchObject`被用于布尔判断的时候，始终返回True，所以你用 if 语句来判断某个 match() 是否成功是安全的。

它有以下方法和属性：

* **expand**(template)

用template做为模板，将MatchObject展开，就像sub()里的行为一样。

```python
>>> m = re.match('a=(\d+)', 'a=100')
>>> m.expand('above a is \g<1>')
'above a is 100'
>>> m.expand(r'above a is \1')
'above a is 100'
```

* **group**([group1, ...])

返回一个或多个子组。如果参数为一个，就返回一个子串；如果参数有多个，就返回多个子串注册的元组。如果不传任何参数，效果和传入一个0一样，将返回整个匹配。如果某个groupN未匹配到，相应位置会返回None。如果某个groupN是负数或者大于group的总数，则会抛出IndexError异常。

```python
>>> m = re.match(r"(\w+) (\w+)", "Isaac Newton, physicist")
>>> m.group(0)       # 整个匹配
'Isaac Newton'
>>> m.group(1)       # 第一个子串
'Isaac'
>>> m.group(2)       # 第二个子串
'Newton'
>>> m.group(1, 2)    # 多个子串组成的元组
('Isaac', 'Newton')
```

如果有其中有用(?P...)这种语法命名过的子串的话，相应的groupN也可以是名字字符串。例如：

```python
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Malcolm Reynolds")
>>> m.group('first_name')
'Malcolm'
>>> m.group('last_name')
'Reynolds'
```

如果某个组被匹配到多次，那么只有最后一次的数据，可以被提取到：

```python
>>> m = re.match(r"(..)+", "a1b2c3")  # 匹配到3次
>>> m.group(1)                        # 返回的是最后一次
'c3'
```

* **groups**([default])

返回一个由所有匹配到的子串组成的元组。default参数，用于给那些没有匹配到的组做默认值，它的默认值是None。

```python
>>> m = re.match(r"(\d+)\.(\d+)", "24.1632")
>>> m.groups()
('24', '1632')
```

default的作用：

```python
>>> m = re.match(r"(\d+)\.?(\d+)?", "24")
>>> m.groups()      # 第二个默认是None
('24', None)
>>> m.groups('0')   # 现在默认是0了
('24', '0')
```

* **groupdict**([default])

返回一个包含所有命名组的名字和子串的字典，default参数，用于给那些没有匹配到的组做默认值，它的默认值是None。

```python
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Malcolm Reynolds")
>>> m.groupdict()
{'first_name': 'Malcolm', 'last_name': 'Reynolds'}
```

*　**start**([group]); end([group])

返回被组group匹配到的子串在原字符串中的位置。如果不指定group或group指定为0，则代表整个匹配。如果group未匹配到，则返回 -1。

对于指定的m和g，m.group(g)和m.string[m.start(g):m.end(g)]等效。

注意：如果group匹配到空字符串，m.start(group)和m.end(group)将相等。

```python
>>> m = re.search('b(c?)', 'cba')
>>> m.start(0)
1
>>> m.end(0)
2
>>> m.start(1)
2
>>> m.end(1)
2
```

下面是一个把email地址里的“remove_this”去掉的例子：

```python
>>> email = "tony@tiremove_thisger.net"
>>> m = re.search("remove_this", email)
>>> email[:m.start()] + email[m.end():]
'tony@tiger.net'
```

* **span**([group])

返回一个元组： (m.start(group), m.end(group))

* **pos**

就是传给RE对象的search()或match()方法的参数pos，代表RE开始搜索字符串的位置。

* **endpos**

就是传给RE对象的search()或match()方法的参数endpos，代表RE搜索字符串的结束位置。

* **lastindex**

最后一次匹配到的组的数字序号，如果没有匹配到，将得到None。

例如：(a)b、((a)(b))和((ab))正则去匹配'ab'的话，得到的lastindex为1。而用(a)(b)去匹配'ab'的话，得到的lastindex为2。

* **lastgroup**

最后一次匹配到的组的名字，如果没有匹配到或者最后的组没有名字，将得到None。

* **re**

得到本Match对象的正则表达式对象，也就是执行search()或match()的对象。

* **string**
传给search()或match()的字符串。
