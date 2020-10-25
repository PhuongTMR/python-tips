### 4. Function/Variable (Naming & scopes)
**1. Naming:**
- You need to know the list of <mark>[Built-in Functions](https://docs.python.org/3/library/functions.html)</mark> to avoid code overriding when naming a function or variable.
- Use the function naming rules: lowercase with words separated by underscores as necessary to improve readability.
- Use one leading underscore only for non-public methods and instance variables.
- To avoid name clashes with subclasses, use two leading underscores to invoke Python's name mangling rules.
- Python mangles these names with the class name: if class Foo has an attribute named __a, it cannot be accessed by Foo.__a. (An insistent user could still gain access by calling Foo._Foo__a.) Generally, double leading underscores should be used only to avoid name conflicts with attributes in classes designed to be subclassed.

**2. Scopes:**
  - At any given moment, there are at least three nested scopes.
    1. Scope of the current function which has local names
    2. Scope of the module which has global names
    3. Outermost scope which has built-in names

![variable scopes](../assets/img/tips/variable-scopes.jpg "variable-scopes")

```python
def outer():
    x = 2
    def inner():
        x = 3
        print(x)
    inner()
    print(x)
x = 1
outer()
print(x)
```

```python
x = 1
def outer():
    x = 2
    def inner():
        global x # <-- Focus on it
        x = 3
        print(x)
    inner()
    print(x)
outer()
print(x)
```
