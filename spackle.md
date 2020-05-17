Often there's a trade-off in programming between code hygiene and code length. Should I have a `Person` class storing a name and a birthday? Or can I get away with just passing around strings and `Date` objects? *Spackle* is what I call any programming language feature which allows you to bridge this gap.

A good example of  sort of thing is the use of tuples in Python. A tuple in Python looks like this:

```
x = (1, "two", 3)
```

We can "unpack" tuples like so:

```
x, y, z = (1, "two", 3)
```

After this operation, `x` and `z` contain the integers 1 and 3, and `y` contains the string "two". We can also return them from functions with a lightweight, parentheseless syntax:

```
def f():
	return 1, "two", 3
```

In this way, rather than having a function `get_person()` return a `Person` class (and therefore have to write a `Person` class), it can return a tuple with a name and a birthday. As long as this use of spackle does not spread too far (we have a hundred functions passing around these undocumented name/birthday tuples), this code is pleasant to write and easy to refactor if we want to add a `Person` class.

Another example of spackle is the use of a [closure](closures-as-objects.md) as a stand-in for a class. Spackle features allow us to delay refactoring. The key to using spackle effectively is to be aware what it is a stand-in for, and how to go about refactoring it later if needed.

