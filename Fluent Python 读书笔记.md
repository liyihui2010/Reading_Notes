# Fluent Python 读书笔记
# Chapter 1: The Python Data Model
## French Card example: 
* we can use `__len__` and `__getitem__` method in our defined classes to implement the `len()` and `[]`(slicing) method. 
```python
import collections

Card = collections.namedtuple(‘Card’, [‘rank’, ‘suit’])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list(‘JQKA’)
    suits = ‘spades diamonds clubs hearts’.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
				for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

>>> deck = FrenchDeck()
>>> len(deck)
52
>>> deck[:3]
[Card(rank=‘2’, suit=‘spades’), Card(rank=‘3’, suit=‘spades’), Card(rank=‘4’, suit=‘spades’)]
```

## Vector example:
* we can overload the `__repr__` (string representation), `__abs__`, `__add__`, `__mul__` methods as well:
* Python will print `__repr__` if `__str__` is not implemented.
```python
from math import hypot

class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return ‘Vector(%r, %r)’ % (self.x, self.y)

    def __abs__(self):
        return hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
	  # However, this will violate 3*Vector(1,2), which will be 		implemented under __rmul__ methods, covered in Chapter 		13.
```

## Boolean value of a custom type
* if the `__bool__` method is not implemented, will call the `__len__` method. If neither is implemented, will be just *truthy*

### Why `len()` is not a method?
* `len` and `abs` are frequently used to built-in types, which are accessed through Cython’s struct. Therefore, these are not methods calls. 


# Chapter 2: An Array of Sequences
## Overview of Built-In Sequences
* **Container sequences** (e.g., `list`, `tuple`) can store distinct objects (only save the reference), while **Flat sequences**(e.g., `str`) stores the objects, thus only allows primitive values.
* `MutableSequence` inherits `Sequence` inherits `Container`, `Iterable` and `Sized`.
* listcomps in 2.7 still leak value, not the case in 3.5
```python
>>> x = ‘ABC’
>>> dummy = [ord(x) for x in x]
>>> x
‘C’ # 2.7
‘ABC’ # 3.5
```
* Using genexp (generator expression, such as `()`) can save memory

## Tuples
* We can use `*` to grab excess items: **Works in any position!**
```python
>>> a,b, *rest = range(5) # 2, 3
>>> rest
[2,3,4] # [2], []

>>> a, *body, c, d = range(5)
>>> body
[1, 2]
>>> *head, b, c, d = range(5)
[0,1]
```

## Using `+` and `*`
Nested list are by reference,
```python
>>> board1 = [[‘_’] * 3 for i in range(3)]
>>> board2 = [[‘_’] * 3] * 3 # WRONG! 
```

##  `+=` and `*=`
* if the `__iadd__ ` method is implemented, will call that. Otherwise, will call `__add__`, which making:
```python
>>> a += b # mutable object is changed
>>> a = a + b  # creating a new object
```
* putting mutable items in tuples is not a good idea
* augmented assignment is not an atomic operation — failure != the item is not modified

## `list.sort` and `sorted`
* `list.sort` will do in place, thus returning `None` to indicate the list has been modified.
* The built-in `sorted` function can apply to any iterables (including immutable sequence), and aways return a `list`.

## When `list` is not an answer
* if pure number, no need to go for `list`, as it will convert to `float` object. We can use `array.array`
* `deque` is thread-safe and efficient for FIFO structure
 

# Chapter 3: Dictionaries and Sets
* the built-in functions live in  `__builtins__.__dict__`

## Generic Mapping Types
```python
>>> my_dict = {}
>>> isinstance(my_dict, abc.Mapping)
True
```
* using `is instance` is better than checking whether a function argument is of `dict` type
* all mapping types in the standard library use the basic `dict` in their implementation, so they share the limitation that the **keys** must be **hashable** — (need a `__hash__()` method and can be compared to other objects — needs a `__eq__()` method)
* If an element is not found in a dict, unless the `__missing__` is defined (which could return something by default), it will raise error.

# Chapter 4: Text versus Bytes
* The identity of a character — its **code point** is a number from 0 to 1,114,111
* from code points to bytes is *encoding*, from bytes to code points is *decoding*

读不下去了 先跳过

# Chapter 5: First-Class Functions
## Treating a function like an object
```python
def factorial(n):
“””returns n!”””
	return 1 if n < 2 else n * factorial(n-1)

>>> factorial.__doc__
‘returns n!’
>>> type(factorial) 
<class ‘function’>
```
## Modern replacement for `map`, `filter`, and `reduce`
* we can use list comprehension and `sum`

## Callable objects:
* call operator `()` can be checked by `callable()` function
* We can implement a `__call__` instance method to make a objects callable:
``` python

>>> bingo = BingoCage(range(3))
>>> bingo.pick()
1
>>> bingo()
0
>>> callable(bingo)
True


import random
class BingoCage:
    def __init__(self, items):
        self._items = list(items)  
        random.shuffle(self._items) 

    def pick(self):  
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError(‘pick from empty BingoCage’) 

    def __call__(self): 
        return self.pick()
```

##  Function introspection
* we can check what methods does a plain function has over a plain class:
```python
>>> class C: pass
>>> obj = C()
>>> def func(): pass
>>> sorted(set(dir(func)) - set(dir(obj)))
[‘__call__’, ‘__class__’, ‘__closure__’, ‘__code__’, ‘__defaults__’, ‘__delattr__’, ‘__dict__’, ‘__format__’, ‘__get__’, ‘__getattribute__’, ‘__globals__’, ‘__hash__’, ‘__init__’, ‘__name__’, ‘__new__’, ‘__reduce__’, ‘__reduce_ex__’, ‘__repr__’, ‘__setattr__’, ‘__sizeof__’, ‘__str__’, ‘__subclasshook__’, ‘func_closure’, ‘func_code’, ‘func_defaults’, ‘func_dict’, ‘func_doc’, ‘func_globals’, ‘func_name’]
```

## Packages for functional Programming
* `itemgetter` and `attrgetter` great functions to extract positional and key-pair values
```python
>>> from operator import itemgetter, attrgetter
>>> getter = itemgetter(1, 0)
>>> getter([1,2,3,4])
(2, 1) # it use [] operator (need to implement __getitem__)
	     # and return a tuple
>>> agetter = attrgetter(“name”, “coord.lat”)
    # agetter needs to acked on somthing like namedtuple
    # it cannot act on dict, as dict does not have `name`
>>> from collections import namedtuple
>>> LatLong = namedtuple(“latLong”, “lat long”)
>>> Metropolis = namedtuple(“Metropolis”, “name cc pop coord”)
>>> metro_areas = [Metropolis(“Tokyo”, “JP”, “40”, LatLong(1, 2))]
>>> agetter(metro_areas[0])
(‘Tokyo’, 1)
```
* `methodcaller` has similar usage:
```python
>>> from operator import methodcaller
>>> s = “FRANK the tank”
>>> upcase = methodcaller(“upper”)
>>> lowcase = methodcaller(“lower”)
>>> upcase(s) # s.upper()
‘FRANK THE TANK’
>>> lowcase(upcase(s)) # s.lower()
‘frank the tank’
```
* `functools.partial` can be a very good `lambda` replacement
```python
>>> def boo(a, b, c):
…     return (a,b,c)
>>> foo = partial(boo, 1,2)
>>> foo(11)
(1, 2, 11)
>>> foo2 = partial(boo, b=1,c=2)
>>> foo2(111)
(111, 1, 2)
```

# Chapter 7: Function Decorators and Closures
## Decorator 101
* function decorators are expected **as soon as ** the module is imported, but the decorated functions only run when they are explicitly invoked . (**import time ** vs **runtime**)

## Variable Scope Rule
* Python does not require you to declare variables, but assumes that a variable assigned in the body of a function is local
```python
>>> b = 6
>>> def f2(a):
…		print(a)
…		print(b)
…		b = 0

>>> f2(3)
3
Traceback (most recent call last):
  File “<stdin>”, line 1, in <module>
  File “<stdin>”, line 3, in f
UnboundLocalError: local variable ‘b’ referenced before assignment

>>> def f3(a):
…		global b
…		print(a)
…		print(b)
…		b = 0

>>> f3(3)
3
6
>>> b
0
```

## Closure
* `nonlocal` vs `global` (`nonlocal` is a Python3 feature)
```python
a = 1
def A():
	a = 2
	def B():
		nonlocal a # if this is global, the `outer` a will be 	
					# modified
		print(a)
		a = 3
	return B


>>> fn = A()
>>> fn # 1 if using `global`
2
>>> fn # this will be the same
3
>>> a
1
```
* we can use `functools.lru_cache` to save time in recursion by:
```python
import functools
from clockdeco import clock

@functools.lru_cache() # we need `()`
@clock
def fibonacci(n):
		if n < 2:
			return n
		return fibonacci(n-2) + fibonacci(n-1)

if __name__ == ‘__main’:
		print(fibonacci(6))
```

* if want to include parameter in decorator (`lru_cache()` above), we need to make that decorator to return another decorator: therefore:
```python
def f():
	…

@functools.lru_cache()
f
# equal to
f = functools.lru_cache()(f)
``` 


# Chapter 8: Object References, Mutability, and Recycling
* The `is` operator is faster than `==`, because it cannot be overloaded.
* `a == b` is syntactic sugar for `a__eq__(b)`, if `a` and `b` are objects, the `__eq__` method will fetch the id. so will get the same result as `is`. However, most built-in types override `__eq__` with more meaningful implementations. (e.g., comparing large collections or deeply nested structures.)

## Copies are shallow by default
```python
l1 = [1, [2, 3]]
l2 = list(l1) # or l1[:], both shallow copy
l1[1].append(4)
l2 # will give [1, [2, 3, 4]]
```
* we can also use `copy.copy` as shallow copy and `copy.deepcopy` as deep copy. 
* Some cases, such as singleton, we want to avoid making such copy. Therefore, we need to implement the `__copy__()` and `__deepcopy__()` method.

## Function parameters as references
* The only mode of parameter passing in Python is **call by sharing**. It means that each formal parameter of the function gets a copy of each reference in the arguments. In other words, the parameters inside the function become aliases of the actual argument.
* This means that: **a function may change any mutable object passed as a parameter, but it cannot change the identity of those objects**.
```python
>>> def f(a, b):
…     a += b
…     return a
…
>>> x = 1
>>> y = 2
>>> f(x, y)
3
>>> x, y
(1, 2)

>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b
([1, 2, 3, 4], [3, 4])

>>> t = (10, 20)
>>> u = (30, 40)
>>> f(t, u)
(10, 20, 30, 40)
>>> t, u
((10, 20), (30, 40))

>>> def g(a, b):
…     print(id(a))
…     a += b
…     print(id(a)) # id will change if a is immutable
…     pass
…
>>> g(x, y)
4300251512
4300251464
>>> g(a, b)
4301581848
4301581848
>>> g(t, u)
4301587808
4301454864
```
* **Mutable types as parameter defaults is a BAD IDEA!**. if a default value is a mutable object, and you change it, the change will affect every future call of the function.

## `del` and garbage collection
* The `del` statement deletes names, not objects. An object my be garbage collected as result of a `del` command, but only if the variable deleted holds the last reference to the object, or if the object becomes unreachable.
* we can use `weakref` as a weak references to an object, which do not increase its reference count.

## interning
* never use `is` on `int` or `str`…
```python
>>> a = 1
>>> b = 1
>>> a is b
True
>>> c = 10000
>>> d = 10000
>>> c is d
False
```

## `classmethod`versus `staticmethod`
* `classmethod` changes the way the method is called, so it receives the class itself as the first argument, instead of an instance. Its most common use is for alternative constructors.  (first argument is `cls`)
* in contrast, the `staticmethod` decorator changes a method so that it receives no special first argument. in essence, a static method is just like a plain function that happens to live in a class body, instead of being defined at the module level. 

# Chapter 9: A Pythonic Object
## Hashable
* we can use `@property` decorator to call without using `()`
* hide `self.x` to `self.__x` and load it using the following:
```python
class boo():
	def __init__(self, x):
		self.__x = x			
	
	@property
	def x(self):
		return self.__x

# this will avoid assignmentlike:
# this is Python3
>>> a = boo(1)
>>> a.x = 2
Traceback (most recent call last):
  File “<stdin>”, line 1, in <module>
AttributeError: can’t set attribute
```

## `__slots__`
* we can implement the `__slots__` method to save memory. It will save all variables to tuple, instead of dict. However: no other variables can be declared in the class. And one needs to overwrite `__slots__` in the inheritance class as it will be ignored.

 # Chapter 10: Sequence Hacking, Hashing, and Slicing
## How slice work:
* `__getitem__` will return a slice object or a single number
```python
def __getitem__(self, index):
	cls = type(self)
	if isinstance(index, slice):
		return cls(self._components[index]) # this will invoke the class to build another `Vector` instance form a slice of the `_components array
	elif isinstance(index, numbers.Integral):
		return self._components[index] # in this case we no need to build another `Vector`
	else:
		# deal with error msg here
```

## Dynamic Attribute Access
* we can implement `__getattr__` in case when some attribute are not explicitly defined, while we still want to use these easy access methods:
* It is always a good practice to also implement the `__setattr__` method, incase we assign new value to these attributes. (e.g., to protect readonly attributes)
* we could also implement `__setitem__` to enable `v[0] = 1.1` and/or `__setattr__` to make `v.x = 1.1` work. But `Vector` will remain immutable because we want to make it washable in the coming section.

# Chapter 11: Interfaces: From Protocols to ABCs
## Interfaces and protocols in Python Culture
* **protocols** are defined as the informal interfaces that make polymorphism work in languages with dynamic typing like Python
* **Interface** is  the subset of an object’s public methods that enable it to play a specific role in the system
* **Duck typing**: operating with objects regardless of their types, as long as they implement certain protocols

We don’t need to make the `FrenchDeck` class a sub-class of sequence, as long as we implement all the functions, like `__getitem__`, `__setitem__` (for `random.shuffle` to work).  — this is called **duck typing**.

## Subclassing an ABC
* Python does not check for the implementation of the abstract methods at import time, but only **at runtime** when we actually try to instantiate them.
* An abstract method **can** actually have an implementation. Even if it does, subclasses will still be forced to override it. But they will be able to invoke the abstract method with `super()`

## Virtual subclass
* an essential characteristic of **goose typing** is the ability to register a class as a **virtual subclass** of an ABC, even if it does not inherit from it. When doing do, we promise that the class faithfully implements the interface defined in the ABC.

```python
@Tombola.register # python 3.5
class TomboList(list):
	…
```

# Chapter 12: Inheritance: For Good or For Worse
## Subclassing built-in types
* if  we want to do inheritance from built-in types, such as `dict`, it will mess up because some methods are implemented in C, while others are not. Therefore, we need to use `collections.UserDict` instead.

## Multiple inheritance and method resolution order
```python
class A:
	def ping(self):
		print(“ping:”, self)

class B(A):
	def pong(self):
		print(“pong:”, self)

class C(A):
	def pong(self):
		print(“PONG:”, self)

class D(B, C):
	def ping(self):
		super().ping() # or A.ping(self)
		print(“post-ping:”, self)
	
	def pingpong(self):
		self.ping()
		super().ping()
		self.pong()
		super().pong()
		C.pong(self)

>>> d = D()
>>> d.pong() # this will return B’s pong
(‘pong:’, <__main__.D instance at 0x10197f6d8>)
>>> C.pong(d) # or call a method on a superclass directly
(‘PONG:’, <__main__.D instance at 0x10197f6d8>)
>>> d.ping() # `self` is always D
ping: <__main__.D object at 0x10197f6d8>
post-ping: <__main__.D object at 0x10197f6d8>
>>> d.pingpong()
ping: <__main__.D object at 0x10197f6d8>
post-ping: <__main__.D object at 0x10197f6d8>
ping: <__main__.D object at 0x10197f6d8>
pong: <__main__.D object at 0x10197f6d8>
pong: <__main__.D object at 0x10197f6d8>
PONG: <__main__.D object at 0x10197f6d8>
```

* Python follows a specific order when traversing the inheritance graph. The order is called **MRO**: Method Resolution Order.


# Chapter 13: Operator Overloading: Doing It Right
## Operator Overloading 101
* we cannot overload operators for the built-in types
* We cannot create new operators, only overload existing ones
* A few operators cannot be overloaded: `is`, `and,` `or`, `not`(but the bitwise `&`, `|`, `~`, can )
* Interesting case: when `x` and `+x` are not equal (in built-in)
```python
>>> ct = Counter(“abracadabra”)
>>> ct
Counter({‘a’: 5, ‘r’: 2, ‘b’: 2, ‘c’: 1, ‘d’: 1})
>>> ct[‘r’] = -3
>>> ct[‘d’] = 0
>>> +ct
Counter({‘a’: 5, ‘b’: 2, ‘c’: 1})
```

## Augmented Assignment Operators
* if we do not implement `__iadd__` method, we might get a new object if the previous one is immutable, because (`a+=b` -> `a = a+b`)

# Chapter 14: Iterables, Iterators, and Generators
##  Why sequences are operable: the `iter` function
* when the interpreter needs to iterate over an object `x`, it automatically calls `iter(x)`
* If `__iter__` function is not implemented, will call the `__getitem__` to iterate, starting form index 0

## Iterables vs iterators
* `iterator` inherits from `iterable`, where the former has `__next__` and `__iter__` (return `self`) while later only has `__iter__` 
* **Iterator**: any object that implements the `__next__` no-argument method that returns the next item in a series or raise `StopIteration` when there are no more items. Python iterators also implement the `__iter__` method so they are `itrable` as well

## Generator function
```python
def gen_123(): # this will return a generator
	yield 1
	yield 2
	yield 3

>>> gen_123
<function gen_123 at 0x102800b18>
>>> gen_123()
<generator object gen_123 at 0x1007ee0f0>
```

* a generator function builds a generator object that wraps the body of the function. 
* we can also use `genexp` to make generator:
```python
>>> a = (i for i in range(100))
>>> a
<generator object <genexpr> at 0x102177c60>
```

## `yield from`
We can use `yield from` to replace nested for-loop
```python
def chain(*iterables):
	for it in iterables:
		for i in it:
			yield i

# equivalent
# this is not only a syntax sugar
def chain(*iterables):
	for it in iterables:
		yield from i
``` 

* `yield from` creates a channel connecting the inner generator directly to the client of the outer generator. This channel becomes really important when generators are used as **coroutines** and not only produce but also consume values form the client code.

## closer look at `iter` function
* we can add a second params to `iter` function for termination:
```python
def d6():
	return randint(1,6)

>>> d6_iter = iter(d6, 1) # will stop when roll 1
>>> for roll in d6_iter:
…     print(roll)
…
4
5
6
6
5
2
```

# Chapter 15: Context Managers and else Blocks
## `else` blocks beyond `if`
```python
for item in my_list:
	if item.flavor == "banana":
		break
else: # will call when for runs to completion
	raise ValueError("No banana flavor found!")


try:
	dangerous_call()
	after_call()
except OSError:
	log("OSError")

# do this!
try:
	dangerous_call()
except OSError:
	log("OSError")
else: # will run if no exception is raised in the try block
	after_call()

# while: the else block will run only if and when the `while` loop exits because the condition become falsy (o.e., not aborted with a `break`)
```

## Context managers and `with` blocks
* The `with` statement was designed to simplify the `try/finally` pattern, which guarantees that some operation is performed after a block of code, even if the block is aborted because of an exception, a `return` or `sys.exit()`call.
* need to implement `__enter__` and `__exit__`(play by `finally`) protocols.
* the `with` syntax call `__enter__`, and `__exit__` method is invoked on the context manager object, not on whatever is return by `__enter__` (with … as ..:)


# Chapter 16: Coroutines
* A **coroutine** is syntactically like a generator: just a function with the `yield` keyword in its body. However, in a coroutine, `yield` usually appears on the right side of an expression (e.g., `datum = yield`), and it may or may not produce a value.
* It is even possible that no data goes in or out through the `yield` keyword. Regardless of the flow of data, `yield` is a control flow device that can be used to implement cooperative multitasking: each coroutine yields control to a central scheduler so that other coroutines can be activated.

```python
def simple_coroutine():
	print(“-> coroutine started”)
	x = yield
	print(“-> coroutine received:”, x)

>>> my_coro = simple_coroutine()
>>> my_coro
<generator object simple_coroutine at 0x1007ee0f0>
>>> next(my_coro)
-> coroutine started
>>> my_coro.send(42)
(‘-> coroutine received:’, 42)
Traceback (most recent call last):
  File “<stdin>”, line 1, in <module>
StopIteration
```
* we can use the `inspect.getgeneratorstate(…)` to check the state.
* `my_coro.send(42)` works only when the state is currently suspended:
```python
>>> my_coro = simple_coroutine()
>>> my_coro.send(42)
Traceback (most recent call last):
  File “<stdin>”, line 1, in <module>
TypeError: can’t send non-None value to a just-started generator
```
* The `a  = yield b` syntax will assign `a` to `.send(…)` and prints `b`. Moreover, `yield` will be evaluated first before the assignment `=`. In this case, `b` will be assigned only when coroutine is activated later by the client code

## Using `yield from`
* the main feature of `yield from` is to open a bidirectional channel from the outermost caller to the innermost subgenerator, so that values can be sent and yielded back and forth directly from them, and exceptions can be thrown all the way in without adding a lot of exception handling boilerplate code in the intermediate coroutines.

## The meaning of `yield from`
* any values that the subgenerator yields are passed directly to the caller of the delegating generator (i.e., the client code) 
* any values sent to the delegating generator using `send()` are passed directly to the subgenerator. If the sent value is `None`, the subgenerator’s `__next__()` method is called. If the sent value is not `None`, the subgenerator’s `send()` method is called. If the call raises `StopIteration`, the delegating generator is resumed. Any otters exception is propagated to the delegating generator.
* `return expr` in a generator (or sub) causes `StopIteration(expr)` to be raised upon exit from the generator
* the value of the `yield from` expression is the first argument to the `StopIteration` exception raised by the subgenerator  when it terminates.


# Chapter 19: Dynamic Attributes and Properties
* it’s hard to access data like `feed[“xxx”][“yyy”][3][“zzz”]`, while faster to do like: `feed.xxx.yyy[3].zzz`. Therefore we can dynamically allocate attributes to the data.

## Flexible object creation with `__new__`
* We ofter refer to `__init__` as the constructor method, but that’s because we adopted jargon from other languages. The special method that actually constructs an instance is `__new__`: it’s a class method, and it must return an instance. That instance will in turn be passed at the first argument `self` of `__init__`. Because `__init__` get an instance when called, and it’s actually forbidden from returning anything, `__init__` is really an “initializer”.  The real constructor is `__new__` — which we rarely need to code because the implementation inherited from `object` suffices.

```python
# pseudo-code for object construction
def object_maker(the_class, some_arg):
	new_object = the_class.__new__(some_arg)
	if isinstance(new_object, the_class):
		the_class.__init__(new_object, some_arg)
	return new_object

# the following statements are roughly equivalent
x = Foo(‘bar’)
x = object_maker(Foo, ‘bar’)
```

## `property`
* this is the full signature of the `property` constructor:
	* `property(fget=None, fset=None, fdel=None, doc=None)`
* all arguments are optional, and if a function is not provided for one of them, the corresponding operation is not allowed by the resulting property object

```python
class LineItem:

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight  # self.weight’s setter
        self.price = price

    def subtotal(self):
        return self.weight * self.price

    @property 
    def weight(self): 
        return self.__weight  # where the actual value is stored

    @weight.setter  # the decorated getter has a `.setter` attribute, which is also a decorator; this ties the getter and setter together
    def weight(self, value):
        if value > 0:
            self.__weight = value  # <6>
        else:
            raise ValueError(‘value must be > 0’)  # <7>
```

## Properties override instance attributes
* Properties are always class attributes, but they usually manage attribute access in the instances of the class
* However, with `@property`, overwrite instance level does not affect. It will still call the property getter. The only way around is the change Class level prop
* The main point of the above is that an expression like `obj.attr` does not search for `attire` staring with `obj`. The search actually starts at `obj.__class__`, and only if there is no property named `attire` in the class, Python looks in the `obj` instance itself. **This rule applies to the whole category of descriptors**

## Coding a property factory
* **Properties are class attribute!**
```python
def quantity(storage_name): 
    def qty_getter(instance):  
        return instance.__dict__[storage_name] 

    def qty_setter(instance, value): 
        if value > 0:
            instance.__dict__[storage_name] = value  
        else:
            raise ValueError(‘value must be > 0’)

    return property(qty_getter, qty_setter) 


class LineItem:
    weight = quantity(‘weight’)  # use the factory to create a class attribute  
    price = quantity(‘price’) 

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight  # <3>
        self.price = price

    def subtotal(self):
        return self.weight * self.price  # <4>
```


# Chapter 20: Attribute Descriptors
* A class implementing a `__get__`, a `__set__`, or a `__delete__` method is a **descriptor**.

## A simpler descriptor
```python
class Quantity:  # <1>

    def __init__(self, storage_name):
        self.storage_name = storage_name  # <2>

    def __set__(self, instance, value):  # <3>
        if value > 0:
            instance.__dict__[self.storage_name] = value  # <4>
        else:
            raise ValueError(‘value must be > 0’)
```

* #3  `__set__` is called when there is an attempt to assign to the managed attribute. Here, `self` is the descriptor instance (i.e., `LineItem.weight`) `instance` is the managed instance (a `LineItem` instance), and the `value` is the value being assigned
* #4 must handle the managed instance `__dict__` directly. Trying to use the `stator` built-in would trigger the `__set__` method again, leading to infinite recursion

# Chapter 21: Class Metaprogramming
## A class factory
* we can use `type` to build a class:
```python
MyClass = type('MyClass', (MySuperClass, MyMixin),
			{"x": 42, "x2" : lambda self: self.x*2})

# is equivalent to 
class MyClass(MySuperClass, MyMixin):
	x = 42
	
	def x2(self):
		return self.x*2
```

* therefore, we can create class on the fly:
```python
def record_factory(cls_name, field_names):
    try:
        field_names = field_names.replace(',', ' ').split()  # <1>
    except AttributeError:  # no .replace or .split
        pass  # assume it's already a sequence of identifiers
    field_names = tuple(field_names)  # <2>

    def __init__(self, *args, **kwargs):  # <3>
        attrs = dict(zip(self.__slots__, args))
        attrs.update(kwargs)
        for name, value in attrs.items():
            setattr(self, name, value)

    def __iter__(self):  # <4>
        for name in self.__slots__:
            yield getattr(self, name)

    def __repr__(self):  # <5>
        values = ‘, ‘.join(‘{}={!r}’.format(*i) for i
                           in zip(self.__slots__, self))
        return ‘{}({})’.format(self.__class__.__name__, values)

    cls_attrs = dict(__slots__ = field_names,  # <6>
                     __init__  = __init__,
                     __iter__  = __iter__,
                     __repr__  = __repr__)

    return type(cls_name, (object,), cls_attrs)  # <7>
```

## Import time vs. runtime
* The interpreter executes a `def` statement on the top level of a model when the module is imported, but what does that achieve? The interpreter compiles the function body, and binds the function object to its global name, but it does not execute the body of the function. The functions are invoked at runtime
* For classes, the story is different: at import time, the interpreter executes the body of every class, even the body of classes nested in other classes. Execution of a class body means that the attributes and methods of the class are defined, and then the class object itself is built. In this sense, the body of classes is “top-level code”: it runs at import time.

## Metaclasses
* A meta class is a class factory, written in class
* Consider the Python object model: **classes are objects**. Therefore each class must be an instance of some other class. By default, Python classes are instances of `type`. In other word, `type` is the metaclass of most built-in and user-defined classes:
```python
>>> “spam”.__class__
<type ‘str’>
>>> str.__class__
<type ‘type’>
>>> type.__class__
<type ‘type’> # type is an instance of itself
```
* **they are NOT inherit from type, but are instances of type**. They are all subclasses of `object`
* `object` is an instance of `type`, and `type` is a subclass of `object`.
* a metaclass inherits from `type` the power to construct classes.

