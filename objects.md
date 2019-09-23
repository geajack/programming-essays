An object is a collection of functions whose behavior depends on which of those functions have been called previously.

In "classical" OOP, an object consists of a collection of private variables ("properties") and public methods. There can be public properties as well, but those are just [another kind of public method](public-properties.md). This is what I mean by an object: a collection of functions, and a collection of variables that only those functions are allowed to modify.

When should we introduce objects in our code? [Any time the problem domain prominently involves a noun](overmodelling.md)? If we look at things strictly [pragmatically](pragmatism.md) we might realize the use cases for objects are a lot narrower than we thought. Novice programmers raised on Java sometimes write classes consisting of only a constructor and a getter method, which as has been pointed out, may as well be just a function (as long as your language supports free functions). Picture a class for fetching information about a date:

```python
class DateParser:
    
    def __init__(self, date_string):
        self.date_string = date_string
        
    def get_day(self):
        # Do parsing and somehow return Monday, Tuesday etc
        pass
```

In practice there may be architectural reasons to make it a class anyway, but strictly pragmatically it only needs to be a function - if what we want to do is go from some values (the parameters to the constructor) and turn them into a result (the return value of the getter), then functions are the tool invented for specifically this job. That is, it could just be `def get_day(date_string)`.

The only reason it would 

Why wouldn't you know in which order the functions will be called?

- Because it depends on I/O.
- Because you're pretending you don't know in order to make it easier to reason about your program.

An object is a function with certain parameters filled in by someone other than the caller. This is essentially [a very lightweight form of polymorphism](constructor-polymorphism.md) - the function can do many different things, and [someone other than the caller of the function is able to decide which of those things it will do](polymorphism.md). It's really a special case of the above - the constructor is a method of the object (it's just automatically called as soon as the object is created), and the object's lone other method has behavior which depends on how the constructor was called.

