# Python Cheatsheet

## Dictionary versus object's attributes

- Use `in` when checking for keys in a dictionary or items in a collection.
- Use `hasattr/getattr/setattr` when checking for or accessing attributes on objects.

## Decorators

Modify a function by wrapping it in another function.

**REMEMBER:**

- Everything inside Python is an object, even functions. Functions is a first-class citizen.
- Any object implements the special `__call__()` method is deemed "callable"
  => A decorator is a callable returns a callable.

```py
def outer(x):
    def inner(y):
        return x + y

    return inner  # function is an obj, they can be returned


add_five = outer(5)
result = add_five(6)

print(result)


def add(x, y):
    return x + y


def cal(func, x, y):
    return func(x, y)


result = cal(add, 5, 5)
print(result)


def make_pretty(func):
    def inner():
        print("I got decorated")
        func()

    return inner


def ordinary():
    print("I am ordinary")


decorated_func = make_pretty(ordinary)
decorated_func()


# equivalent to calling
# ordinary2=make_pretty(ordinary2)
@make_pretty
def ordinary2():
    print("I am ordinary2")


ordinary2()


def smart_divide(func):
    def inner(a, b):
        print(f"Divide: {a}/{b}")
        if b == 0:
            print(f"Invalid dividend: {b}")
            return

        return func(a, b)

    return inner


# divide=smart_divide(divide)
@smart_divide
def divide(a, b):
    print(a / b)


divide(2, 5)
divide(2, 0)


def star(func):
    def inner(*args, **kwargs):
        print("*" * 15)
        func(*args, **kwargs)
        print("*" * 15)

    return inner


def percent(func):
    def inner(*args, **kwargs):
        print("%" * 15)
        func(*args, **kwargs)
        print("%" * 15)

    return inner


@star
@percent
def print_banner(msg):
    print(msg)


# print_banner=star(percent(print_banner))

print_banner("Hello")
```

## References

- [Python Decorators](https://www.programiz.com/python-programming/decorator)
