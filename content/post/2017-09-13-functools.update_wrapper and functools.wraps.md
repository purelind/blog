---
date: 2017-09-13T19:44:11+08:00
title: functools.update_wrapper和functools.wraps
tags: []
categories: ["技术"]
---

先看python文档中的描述，见[functools.update_wrapper](https://docs.python.org/3/library/functools.html#module-functools)

functools.**update_wrapper**(*wrapper*, *wrapped*, *assigned=WRAPPER_ASSIGNMENTS*, *updated=WRAPPER_UPDATES*)

Update a *wrapper* function to look like the *wrapped* function. 

The optional arguments are tuples to specify which attributes of the original function are assigned directly to the matching attributes on the wrapper function and which attributes of the wrapper function are updated with the corresponding attributes from the original function.

o allow access to the original function for introspection and other purposes (e.g. bypassing a caching decorator such as [`lru_cache()`](https://docs.python.org/3/library/functools.html#functools.lru_cache)), this function automatically adds a `__wrapped__` attribute to the wrapper that refers to the function being wrapped.

The main intended use for this function is in [decorator](https://docs.python.org/3/glossary.html#term-decorator) functions which wrap the decorated function and return the wrapper. If the wrapper function is not updated, the metadata of the returned function will reflect the wrapper definition rather than the original function definition, which is typically less than helpful.

[`update_wrapper()`](https://docs.python.org/3/library/functools.html#functools.update_wrapper) may be used with callables other than functions. Any attributes named in *assigned* or *updated* that are missing from the object being wrapped are ignored (i.e. this function will not attempt to set them on the wrapper function). [`AttributeError`](https://docs.python.org/3/library/exceptions.html#AttributeError) is still raised if the wrapper function itself is missing any attributes named in *updated*.

示例1: 普通的装饰器

```python
>>> def my_decorator(f):
...     def wrapper(*args, **kwds):
...    		"""wrapper Docstring"""
...         print('Calling decorated function')
...         return f(*args, **kwds)
...     return wrapper
...
>>> @my_decorator
... def example():
...     """Docstring"""
...     print('Called example function')
...
>>> example()
Calling decorated function
Called example function
>>> example.__name__  # __name__属性从example变为了wrapper,因为装饰器返回的是wrapper
'wrapper'
>>> example.__doc__  # __doc__属性也从Docstring变成了wrapper()函数的wrapper Docstring
'wrapper Docstring'
```

示例2: 使用update_wrapper的装饰器

```python
>>> from functools import update_wrapper
>>> def my_decorator(f):
...     def wrapper(*args, **kwds):
...    		"""wrapper Docstring"""
...         print('Calling decorated function')
...         return f(*args, **kwds)
...     return update_wrapper(wrapper, f)
...
>>> @my_decorator
... def example():
...     """Docstring"""
...     print('Called example function')
...
>>> example()
Calling decorated function
Called example function
>>> example.__name__  # 这里看到example的__name__属性未改变,难道装饰器返回的不是wrapper函数？
'example'             # 这里只是将原函数example的__module__, __name__, __qualname__, 		...					  # __annotations__ and __doc__属性赋值给了wrapper对应的各个属性,
...                   # 而默认的将wrapper的__dict___属性通过example的对应属性更新
>>> example.__doc__   # 这里我们看到__doc__属性未改变
'Docstring'
```

示例3: 使用@wraps的装饰器

```python
>>> from functools import wraps  # functtools.wraps()则是一个调用update_wrapper()的简便方法
>>> def my_decorator(f):
...     def wrapper(*args, **kwds):
...    		"""wrapper Docstring"""
...         print('Calling decorated function')
...         return f(*args, **kwds)
...     return wrapper
...
>>> @my_decorator
... def example():
...     """Docstring"""
...     print('Called example function')
...
>>> example()
Calling decorated function
Called example function
>>> example.__name__  # 这里看到example的__name__属性未改变，和update_wrapper()一样的效果
'wrapper'             # 但是更加方便
>>> example.__doc__   # 这里我们看到__doc__属性未改变，和update_wrapper()一样的效果
'wrapper Docstring'
```

