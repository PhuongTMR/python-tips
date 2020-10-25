### 5. Methods & Attributes
```python
    class MyClass:

        public_attr = "Public Attribute"
        _protected_attr = "Protected Attribute"
        __private_attr = "Private Attribute"

        def method(self, param_1, param_2):
            """
            Epytext http://epydoc.sourceforge.net/manual-epytext.html
            Method explaintation

            @type  param_1: number
            @param param_1: param_1 Description.
            @type  param_2: string
            @param param_2: param_2 Description.
            @rtype:  number
            @return: return description
            """
            return 'instance method called', self

        @classmethod
        def class_method(cls):
            return 'class method called', cls

        @staticmethod
        def static_method():
            return 'static method called'

        @classmethod
        def _protected_method(cls):
            return 'class protected method'

        @classmethod
        def __private_method(cls):
            """
            Python performs name mangling of private variables. Every member with double underscore will be changed to _object._class__variable.
            If so required, it can still be accessed from outside the class, but the practice should be refrained.
            """
            return 'class private method'

    # How to call them
    >>> MyClass().method(1, "sample")
    >>> MyClass.class_method()
    >>> MyClass.static_method()
```

---
### 6. Arguments & Type Hinting
Type hinting is a formal solution to statically indicate the type of a value within your Python code. It was specified in PEP 484 and introduced in Python 3.5.

```python
    from ai_api.models import Lead
    from dataclasses import dataclass

    @dataclass
    class MyClass:

        a_string: str = ""
        a_number: int = 0
        candidate: Lead = None

        def __init__(self, *args, **kwargs):
            self.candidate = kwargs.get("candidate")

        def greet(self, fname: str, lname: str = None, *args, **kwargs) -> str:
            print("CandidateName:", self.candidate.name)
            print("args", args)
            print("kwargs", kwargs)
            return f"Hello, {fname} {lname}"

    >>> In [1]: candidate = Lead(name="James Bond")
    >>> In [2]: MyClass(candidate=candidate).greet(
           ...:     "Phuong", "Nguyen",
           ...:     "arg1", "arg2",
           ...:     kwarg1="kwarg1", kwarg2="kwarg2"
           ...: )
    >>> CandidateName:  James Bond
    >>> args ('arg1', 'arg2')
    >>> kwargs {'kwarg1': 'kwarg1', 'kwarg2': 'kwarg2'}
    >>> Out[3]: 'Hello, Phuong Nguyen'
```

---
### 7. Decorators
- Python has an interesting feature called decorators to add functionality to an existing code.
- This is also called metaprogramming because a part of the program tries to modify another part of the program at compile time.

```python
def star(func):
    def inner(*args, **kwargs):
        print("*" * 30)
        func(*args, **kwargs)
        print("*" * 30)
    return inner


def percent(func):
    def inner(*args, **kwargs):
        print("%" * 30)
        func(*args, **kwargs)
        print("%" * 30)
    return inner

@star
@percent
def printer(msg):
    print(msg)

printer("I'm thanos!")
******************************
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
I'm thanos!
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
******************************
```

---
### 8. ContextManager
https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager

```python
>>> from contextlib import contextmanager
>>> @contextmanager
... def my_context():
...     try:
...         yield
...     finally:
...         pass
...
>>> with my_context():
...     print("I'm Spiderman")

>>> @my_context()
... def some_function():
...     print("I'm Hulk")
>>> some_function()
```

Another way to create a context

https://docs.python.org/3/library/contextlib.html#contextlib.ContextDecorator
```python
from contextlib import ContextDecorator
class my_context(ContextDecorator):

    def __init__(self):
        pass

    def __enter__(self):
        pass

    def __exit__(self, *args):
        pass

>>> with my_context():
...     print("I'm Spiderman")

>>> @my_context()
... def some_function():
...     print("I'm Hulk")
>>> some_function()
```
