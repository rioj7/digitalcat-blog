Title: Object-Oriented Programming in Python 3 - Metaclasses
Date: 2014-09-01 15:00:00 +0200
Modified: 2019-05-22 23:30:00 +0000
Category: Programming
Tags: Python, Python3, OOP, metaprogramming, metaclasses
Authors: Leonardo Giordani
Slug: python-3-oop-part-5-metaclasses
Image: python-3-oop
Series: Python 3 OOP
Summary: Fundamentals of object-oriented programming in Python - metaclasses

This post is available as an **IPython Notebook** [here](/notebooks/Python_3_OOP_Part_5__Metaclasses.ipynb)

## The Type Brothers

The first step into the most intimate secrets of Python objects comes from two components we already met in the first post: `class` and `object`. These two things are the very fundamental elements of Python OOP system, so it is worth spending some time to understand how they work and relate each other.

First of all recall that in Python _everything is an object_, that is everything inherits from `object`. Thus, `object` seems to be the deepest thing you can find digging into Python variables. Let's check this

``` pycon
>>> a = 5
>>> type(a)
<class 'int'>
>>> a.__class__
<class 'int'>
>>> a.__class__.__bases__
(<class 'object'>,)
>>> object.__bases__
()
```

The variable `a` is an instance of the `int` class, and the latter inherits from `object`, which inherits from nothing. This demonstrates that `object` is at the top of the class hierarchy. However, as you can see, both `int` and `object` are called _classes_ (`<class 'int'>`, `<class 'object'>`). Indeed, while `a` is an instance of the `int` class, `int` itself is an instance of another class, _a class that is instanced to build classes_

``` pycon
>>> type(a)
<class 'int'>
>>> type(int)
<class 'type'>
>>> type(float)
<class 'type'>
>>> type(dict)
<class 'type'>
```

Since in Python everything is an object, everything is the instance of a class, even classes. Well, `type` is the class that is instanced to get classes. So remember this: `object` is the base of every object, `type` is the class of every type. Sounds puzzling? It is not your fault, don't worry. However, just to strike you with the finishing move, this is what Python is built on

``` pycon
>>> type(object)
<class 'type'>
>>> type.__bases__
(<class 'object'>,)
```

If you are not about to faint at this point chances are that you are Guido van Rossum or one of his friends down at the Python core development team (in this case let me thank you for your beautiful creation). You may get a cup of tea, if you need it.

Jokes apart, at the very base of Python type system there are two things, `object` and `type`, which are inseparable. The previous code shows that `object` is an instance of `type`, and `type` inherits from `object`. Take your time to understand this subtle concept, as it is very important for the upcoming discussion about metaclasses.

When you think you grasped the `type`/`object` matter read this and start thinking again

``` pycon
>>> type(type)
<class 'type'>
```

## The Metaclasses Take Python

You are now familiar with Python classes. You know that a class is used to create an instance, and that the structure of the latter is ruled by the source class and all its parent classes (until you reach `object`).

Since classes are objects too, you know that a class itself is an instance of a (super)class, and this class is `type`. That is, as already stated, `type` is the class that is used to build classes.

So for example you know that a class may be instanced, i.e. it can be called and by calling it you obtain another object that is linked with the class. What prepares the class for being called? What gives the class all its methods? In Python the class in charge of performing such tasks is called _metaclass_, and `type` is the default metaclass of all classes.

The point of exposing this structure of Python objects is that you may change the way classes are built. As you know, `type` is an object, so it can be subclassed just like any other class. Once you get a subclass of `type` you need to instruct your class to use it as the metaclass instead of type, and you can do this by passing it as the `metaclass` keyword argument in the class definition.

``` pycon
>>> class MyType(type):
...  pass
...
>>> class MySpecialClass(metaclass=MyType):
...  pass
... 
>>> msp = MySpecialClass()
>>> type(msp)
<class '__main__.MySpecialClass'>
>>> type(MySpecialClass)
<class '__main__.MyType'>
>>> type(MyType)
<class 'type'>
```

#### Metaclasses 2: Singleton Day

Metaclasses are a very advanced topic in Python, but they have many practical uses. For example, by means of a custom metaclass you may log any time a class is instanced, which can be important for applications that shall keep a low memory usage or have to monitor it.

I am going to show here a very simple example of metaclass, the Singleton. Singleton is a well known design pattern, and many description of it may be found on the Internet. It has also been heavily criticized mostly because its bad behaviour when subclassed, but here I do not want to introduce it for its technological value, but for its simplicity (so please do not question the choice, it is just an example).

Singleton has one purpose: to return the same instance every time it is instanced, like a sort of object-oriented global variable. So we need to build a class that does not work like standard classes, which return a new instance every time they are called.

"Build a class"? This is a task for metaclasses. The following implementation comes from [Python 3 Patterns, Recipes and Idioms](http://python-3-patterns-idioms-test.readthedocs.org/en/latest/Metaprogramming.html#intercepting-class-creation).

``` python
class Singleton(type):
    instance = None
    def __call__(cls, *args, **kw):
        if not cls.instance:
             cls.instance = super(Singleton, cls).__call__(*args, **kw)
        return cls.instance
```

We are defining a new type, which inherits from `type` to provide all bells and whistles of Python classes. We override the `__call__` method, that is a special method invoked when we call the class, i.e. when we instance it. The new method wraps the original method of `type` by calling it only when the `instance` attribute is not set, i.e. the first time the class is instanced, otherwise it just returns the recorded instance. As you can see this is a very basic cache class, the only trick is that it is applied to the creation of instances.

To test the new type we need to define a new class that uses it as its metaclass

``` pycon
>>> class ASingleton(metaclass=Singleton):
...  pass
... 
>>> a = ASingleton()
>>> b = ASingleton()
>>> a is b
True
>>> hex(id(a))
'0xb68030ec'
>>> hex(id(b))
'0xb68030ec'
```

By using the `is` operator we test that the two objects are the very same structure in memory, that is their ids are the same, as explicitly shown. What actually happens is that when you issue `a = ASingleton()` the `ASingleton` class runs its `__call__()` method, which is taken from the `Singleton` type behind the class. That method recognizes that no instance has been created (`Singleton.instance` is `None`) and acts just like any standard class does. When you issue `b = ASingleton()` the very same things happen, but since `Singleton.instance` is now different from `None` its value (the previous instance) is directly returned.

Metaclasses are a very powerful programming tool and leveraging them you can achieve very complex behaviours with a small effort. Their use is a must every time you are actually metaprogramming, that is if you are writing code that has to drive the way your code works. Good examples are creational patterns (injecting custom class attributes depending on some configuration), testing, debugging, and performance monitoring.

## Coming to Instance

Before introducing you to a very smart use of metaclasses by talking about Abstract Base Classes (read: to save some topics for the next part of this series), I want to dive into the object creation procedure in Python, that is what happens when you instance a class. In the first post this procedure was described only partially, by looking at the `__init__()` method.

In the first post I recalled the object-oriented concept of _constructor_, which is a special method of the class that is automatically called when the instance is created. The class may also define a destructor, which is called when the object is destroyed. In languages without a garbage collection mechanism such as C++ the destructor shall be carefully designed. In Python the destructor may be defined through the `__del__()` method, but it is hardly used.

The constructor mechanism in Python is on the contrary very important, and it is implemented by two methods, instead of just one: `__new__()` and `__init__()`. The tasks of the two methods are very clear and distinct: `__new__()` shall perform actions needed when _creating_ a new instance while `__init__` deals with object _initialization_.

Since in Python you do not need to declare attributes due to its dynamic nature, `__new__()` is rarely defined by programmers, who may rely on `__init__` to perform the majority of the usual tasks. Typical uses of `__new__()` are very similar to those listed in the previous section, since it allows to trigger some code whenever your class is instanced.

The standard way to override `__new__()` is 

``` python
class MyClass():
    def __new__(cls, *args, **kwds):
        obj = super().__new__(cls, *args, **kwds)
        [put your code here]
        return obj
```

just like you usually do with `__init__()`. When your class inherits from `object` you do not need to call the parent method (`object.__init__()`), because it is empty, but you need to do it when overriding `__new__`.

Remember that `__new__()` is not forced to return an instance of the class in which it is defined, even if you shall have very good reasons to break this behaviour. Anyway, `__init__()` will be called only if you return an instance of the container class. Please also note that `__new__()`, unlike `__init__()`, accepts the class as its first parameter. The name is not important in Python, and you can also call it `self`, but it is worth using `cls` to remember that it is not an instance.

## Movie Trivia

Section titles come from the following movies: _The Blues Brothers (1980)_, _The Muppets Take Manhattan (1984)_, _Terminator 2: Judgement Day (1991)_, _Coming to America (1988)_.

## Sources

You will find a lot of documentation in [this Reddit post](https://www.reddit.com/r/Python/comments/226ahl/some_links_about_python_oop/). Most of the information contained in this series come from those sources.

## Feedback

The [GitHub issues](https://github.com/TheDigitalCatOnline/thedigitalcatonline.github.com/issues) page is the best place to submit corrections.

