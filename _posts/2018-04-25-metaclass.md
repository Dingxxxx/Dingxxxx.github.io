---
layout: post
author: Ding
title:  "学习python metaclass 元类"
date:   2018-04-25
categories: python 
tags: python 
---

* content
{:toc}

> 元类在很多框架中有用到，是python中一个高级概念.





使用type()可以动态创建一个类。所有的python类也是对象(类对象)，由type类派生。

> type --> 元类 metaclass --> 类对象 class --> 实例 instance

当创建一个类时，会在内存中创建一个类对象:

+ Foo 中有__metaclass__这个属性吗？如果是，Python 会在内存中通过__metaclass__创建一个名字为 Foo 的类对象
+ 如果 Python 没有找到__metaclass__，它会继续在父类中寻找__metaclass__属性，并尝试做和前面同样的操作
+ 如果 Python 在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作
+ 如果还是找不到__metaclass__,Python 就会用内置的 type 来创建这个类对象


```
class Field(object):

    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type

    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)
    
class StringField(Field):

    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')

class IntegerField(Field):

    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')
        
        
class ModelMetaclass(type):

    def __new__(cls, name, bases, attrs):
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model: %s' % name)
        mappings = dict()
        print(attrs)
        for k, v in attrs.items():
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        for k in mappings.keys():
            attrs.pop(k)
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = name # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)
    
    
class Model(dict, metaclass=ModelMetaclass):

    def __init__(self, **kw):
        print(kw)
        super(Model, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v.name)
            params.append('?')
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))
        
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')
```

用元类创建类对象时，__new__方法先于被创建类的__init__方法被调用。

当创建

```
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
```

由于 Model 是 User 类父类，而 Model 类使用了元类，故先调用的方法为元类中 new()。

以 id 为例，在 mappings 字典中此时建立的应该是 {id:IntegerField('id')}。假设没有 attrs.pop() 那么在调用 u.save()时，getattr()首先从类的属性或者父类的属性中找，只有查询不到时，才会到 getattrs 中查找。由于没有把原本的 id 关键字弹出，故 getattr 便能从类的属性 id=IntegerField 得到 IntegerField。一旦有了 attrs.pop() 在类中就查询不到相应的属性，那么就要调用 getattrs 而返回值是 self[k]。

由于 self 不是指类而是类的实例，所以能够成功返回，构建实例时创建 id 的值 12345，这样就正确得到传入的实参值。

### super 和 mro()


### 类属性和实例属性的区别

<https://www.cnblogs.com/wgDream/p/6749643.html>

+ 类属性

在类定义时直接指定的属性 (不是在__init__方法中)

```
class Test: 
    class_attribute1="attr-value" 
```

+ 实例属性

在__init__方法中添加的属性, 在其他位置添加也是可以的, 实际是通过 setattr 内置函数 (调用__setattr__) 完成, 另外也可以直接修改__dict__属性手动添加

+ 属性的访问顺序

```
class Test(object):
    name = 'python'

a = Test()
a.name = 'python good'  # 通过实例进行修改
print Test.name
print a.name
```

发现类的属性没有修改，而实例的属性则修改成功了。

｀instance.attr_name｀ 访问实例属性时, 首先在｀instance.__dict__｀实例属性中查找, 如果找到返回对应值, 否则在
｀instance.__class__.__dict__｀中查找, 也就是在类属性中查找, 如果找到, 返回对应值, 否则产生attributeError 异常。

而当我试图用实例去修改一个在类中不可变的属性的时候，我实际上并没有修改，而是在我的实例中创建了这个属性。而当我再次访问这个属性的时候，我实例中有，就不用去类中寻找了。

