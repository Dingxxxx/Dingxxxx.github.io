---
layout: post
author: Ding
title:  "使用sqlalchemy封装数据库"
date:   2018-04-26
categories: python 
tags: python 
---

* content
{:toc}

> ORM：Object Relation Mapping，最初主要描述的是程序中的 Object 对象和关系型数据库中 Rlation 关系 (表) 之间的映射关系，目前来说也是描述程序中对象和数据库中数据记录之间的映射关系的统称，是一种进行程序和数据库之间数据持久化的一种编程思想。
> 
> 常规情况下，软件程序中的 ORM 操作主要有四个操作场景：增、删、改、查.





## 连接数据库
使用 sqlalchemy 进行数据库操作，首先我们需要建立一个指定数据库的连接引擎对象建立引擎对象的方式被封装在了`sqlalchemy.create_engine`函数中，通过指定的数据库连接信息就可以进行创建。

创建数据库连接引擎时的连接字符串

```
dialect[+driver]://user:password@host/dbname[?key=value..]
```

`echo`选项表示会自动打印所有sql语句。

### mysql

```python
from sqlalchemy import create_engine
# 默认情况（即使用mysql-python）
engine = create_engine("mysql://username:password@hostname/database", encoding="utf-8", echo=True)
# 使用mysql-python
engine = create_engine('mysql+mysqldb://username:password@hostname/database')
# 使用MySQL-connector-python
engine = create_engine('mysql+mysqlconnector://username:password@hostname/database')
# 使用OurSQL
engine = create_engine('mysql+oursql://username:password@hostname/database')
```

### postgresql

```python
# 默认情况(即使用psycopg2)
engine = create_engine("postgresql://username:password@hostname/database")
# 使用psycopg2
engine = create_engine('postgresql+psycopg2://username:password@hostname/database')
# 使用pg8000
engine = create_engine('postgresql+pg8000://username:password@hostname/database')
```

### sqlite

```python 
# 基于文件的数据库，URL 形式是 sqlite://<nohostname>/<path>
# 在Unix/Mac
engine = create_engine('sqlite:////absolute/path/to/foo.db')
# 在Windows
engine = create_engine('sqlite:///C:\\path\\to\\foo.db')
# 在Windows 中使用原始字符串
engine = create_engine(r'sqlite:///C:\path\to\foo.db')
# 使用内存
engine = create_engine('sqlite://')
engine = create_engine('sqlite:///:memory:')
```

### oracle

```python
# 默认情况（即使用cx_oracle）
engine = create_engine('oracle://scott:tiger@127.0.0.1:1521/sidname')
# 使用cx_oracle
engine = create_engine('oracle+cx_oracle://scott:tiger@tnsname')
```

## 连接会话

```python
# 引入创建session连接会话需要的处理模块 
from sqlalchemy.orm import sessionmaker 
# 创建一个连接会话对象；需要指定是和那个数据库引擎之间的会话 
Session = sessionmaker()
Session.configure(bind=engine)
session = Session()
```

## 创建对象
我们的程序中的对象要使用 sqlalchemy 的管理，实现对象的 orm 操作，就需要按照框架指定的方式进行类型的创建操作，sqlalchemy 封装了基础类的声明操作和字段属性的定义限制方式，开发人员要做的事情就是引入需要的模块并在创建对象的时候使用它们即可。

基础类封装在 sqlalchemy.ext.declarative.declarative_base 模块中
字段属性的定义封装在 sqlalchemy 模块中，通过 sqlalchemy.Column 定义属性，通过封装的 Integer、String、Float 等定义属性的限制。

### 基础类

```python
# 引入需要的模块 
from sqlalchemy.ext.declarative import declarative_base 
# 创建基础类 
Base = declarative_base()
```

### 创建数据类

数据类继承于基础类，通过`__tablename__`和数据库中的数据表建立关联关系。

```python
# 引入需要的模块 
from sqlalchemy import Column, String, Integer 
# 创建用户类型 
class User(Base): 
    # 定义和指定数据库表之间的关联 
    __tablename__ = "user"
    # 创建字段类型 
    id = Column(Integer, primary_key=True) 
    name = Column(String(50)) 
    age = Column(Integer)
```

### 完成数据映射

检查数据库中是否有需要创建的表，不存在的话创建对应的表。

```python
Base.metadata.create_all(engine)
```

## 操作数据

### 增加和更新

```python
user = User(name="tom", age=18)
session.add(user)
session.commit()
```

### 查询对象 Query
Session 是 sqlalchemy 和数据库交互的桥梁，Session 提供了一个 Query 对象实现数据库中数据的查询操作。

+ 常规查询 

```python
user_list = session.query(User) 
for user in user_list: 
    print(user.name)
```

+ 排序

```python
user_list = session.query(User).order_by(User.id) # 默认顺序 
ser_list = session.query(User).order_by(-User.id) # 指定倒序 
user_list = session.query(User).order_by(-User.id, User.name) # 多个字段
```

+ 指定字段

```python
user_list = session.query(User, User.name).all() 
for u in user_list: 
    print(u.User, u.name)
```

+ 指定字段别名

```python
user_list = session.query(Usre.name.label('n')).all() 
for user in user_list: 
    print(user.n)
```

+ 指定数据表别名

```python
from sqlalchemy.orm import aliased 
user_alias = aliased(User, name='u_alias') 
user_list = session.query(u_alias, u_alias.name).all() 
for u in user_list: 
    print(u.u_alias, u.name)
```

+ 切片

```python
user_list = session.query(User).all()[1:3]
```

+ 条件语句 filter
相当于SQL中的where
* 等值条件 equals/not equals

```python
# equals 
session.query(User).filter(User.id == 1) # 相等判断 
# not equals 
session.query(User).filter(User.name != 'tom')# 不等判断
```
* 模糊条件 like

```python
session.query(User).filter(User.name.like('%tom%'))
```
* 范围条件 in/not in

```python
# IN 
session.query(User).filter(User.id.in_([1,2,3,4])) 
session.query(User).filter(User.name.in_([ 
    session.query(User.name).filter(User.id.in_[1,2,3,4]) 
])) 
# NOT IN 
session.query(User).filter(~User.id.in_([1,2,3]))
```
* 空值条件 is null/is not null

```python
# IS NULL s
ession.query(User).filter(User.name == None) 
session.query(User).filter(User.name.is_(None)) # pep8 
# IS NOT NULL 
session.query(User).filter(User.name != None) 
session.query(User).filter(User.name.isnot(None)) # pep8
```
* 与 AND

```python
from sqlalchemy import and_ 
session.query(User).filter(User.name=='tom').filter(User.age==12) 
session.query(User).filter(User.name=='tom', User.age==12) 
session.query(User).filter(and_(User.name=='tom', User.age==12))
```

* 或 OR

```python
from sqlalchemy import or_ 
session.query(User).filter(or_(User.name=='tom', User.name=='jerry'))
```

+ 直接使用SQL语句

```python
from sqlalchemy import text
session.query(User).from_statement( 
    text('select * from users where name=:name and age=:age'))
    .params(name='tom', age=12).all()
```

### 查询结果

+ all() 函数返回查询列表

```python
session.query(User).all()
```

+ filter() 函数返回单项数据的列表生成器

```python
session.query(User).filter(..)
```

+ one()/one_or_none()/scalar() 返回单独的一个数据对象

```python
session.query(User).filter(..).one()/one_or_none()/scalar()
```


## 定义关系

### 一对多关系

一篇文章对应一个用户，一个用户对应多篇文章。

```python
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    username = Column(String(64), nullable=False, index=True)
    password = Column(String(64), nullable=False)
    email = Column(String(64), nullable=False, index=True)
    articles = relationship('Article')

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.username)


class Article(Base):
    __tablename__ = 'articles'

    id = Column(Integer, primary_key=True)
    title = Column(String(255), nullable=False, index=True)
    content = Column(Text)
    user_id = Column(Integer, ForeignKey('users.id'))
    author = relationship('User')

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.title)
```

其中`ForeignKey`函数定义了外键关系，`relationship`描述了用户与文章的双向关系，也可以用如下方式定一双想关系：

```python
articles = relationship('Article', backref='author')
```

### 一对一关系

在 User 中我们只定义了几个必须的字段， 但通常用户还有很多其他信息，但这些信息可能不是必须填写的，我们可以把它们放到另一张 UserInfo 表中，这样User 和 UserInfo 就形成了一对一的关系。你可能会奇怪一对一关系为什么不在一对多关系前面？那是因为一对一关系是基于一对多定义的，只需要是添加`uselist=False`：

```python
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    username = Column(String(64), nullable=False, index=True)
    password = Column(String(64), nullable=False)
    email = Column(String(64), nullable=False, index=True)
    articles = relationship('Article', backref='author')
    userinfo = relationship('UserInfo', backref='user', uselist=False)

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.username)


class UserInfo(Base):
    __tablename__ = 'userinfos'

    id = Column(Integer, primary_key=True)
    name = Column(String(64))
    qq = Column(String(11))
    phone = Column(String(11))
    link = Column(String(64))
    user_id = Column(Integer, ForeignKey('users.id'))
```

### 多对多关系

一遍博客通常有一个分类，好几个标签。标签与博客之间就是一个多对多的关系。多对多关系不能直接定义，需要分解成俩个一对多的关系，为此，需要一张额外的表来协助完成：

```python
article_tag = Table(
    'article_tag', Base.metadata,
    Column('article_id', Integer, ForeignKey('articles.id')),
    Column('tag_id', Integer, ForeignKey('tags.id'))
)


class Tag(Base):
    __tablename__ = 'tags'

    id = Column(Integer, primary_key=True)
    name = Column(String(64), nullable=False, index=True)

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.name)
```