# What isn't polymorphism?

When do we subclass? As [already discussed](overmodelling.md), the answer is not just "whenever one thing feels like a particular case of another thing". A novice programmer will sometimes create many subclasses of one root class because there are many different things which all need to be treated differently, i.e. Cats and Dogs should be treated differently so we make two classes to distinguish between them, and since we all know they're both Animals, we better make them subclasses of a Animal class.

This is precisely the situation in which we should *not* be subclassing. We don't create subclasses to be able to distinguish between different things, we subclass specifically so as to *not* have to distinguish between different things. In a statically typed language like Java, the following is legal:

```java
Animal myRide = new Cat();
```

This is not a type mismatch, since Cat is a subtype of Animal. However, the moment that `Cat` gets put inside a variable of type `Animal`, we have to treat it as if it's a `Animal`. It's still a cat, and it will still meow if we call `.speak()`, but we have to pretend we don't know what kind of an animal it is from now on. If there are Cat-specific methods like `.scratchFurniture()`, we can't use them anymore, because that's not a method of `Animal`.

If you are never doing this sort of thing - declaring a variable with a more general type than the type of the actual value assigned to it - you almost certainly do not need subclassing. If all your cats are declared `Cat` and all your dogs are declared `Dog`, then they probably don't need to have a common superclass. There may be a case for subclassing for the sake of [code reuse](subclassing-as-code-reuse.md), but you are definitely not doing polymorphism.

# What is polymorphism?

Polymorphism is when I call `myAnimal.speak()`, and since I don't know if the animal is a `Cat` or a `Dog`, I don't know if this is going to call `Cat.speak()` or `Dog.speak()` or whether the call will result in a bark or a meow. It is the capacity in a programming language to write a piece of code which can call a function without knowing exactly which function is going to be called, or in more general terms, to execute an action without knowing what exactly the action will entail.

## Isn't that just what functions do?

Whenever I call *any* function, in some sense, the caller doesn't know what exactly will happen. The whole point of functions to hide the details of how the function works from the caller. However, there still is only one implementation of the function.

## Other ways of implementing polymorphism

Executing a function without knowing exactly which function it is - isn't that just a callback? Yes - callbacks are one very simple way of implementing polymorphism in a language. Another way is through a class that can be "configured" through its constructor. What I'm referring to here are "[configurable procedure](objects.md)" classes - classes which:

1. Have a constructor which does nothing but take in arguments and store them internally as private fields.
2. Expose one or more public methods which make read-only use of those private fields.

This is a very common pattern. If you think about this for a while, you'll realize that those private fields may as well just be parameters passed to the public methods. However, by putting them in the constructor we accomplish something very important: the caller of the methods does not need to know those values. We have separated the subproblems of knowing what those values are and of knowing which of the methods to call.

Notice that there's no actual subtyping involved here. This sort of thing could be done in a language with objects, but no notion of inheritance. Since [closures](closures-as-objects.md) are just a lightweight syntax for creating objects, it can be done with closures as well.

# Why would we want polymorphism?

Polymorphism is a technique for separating the parts of the code that know *how* to do something from the parts of your code that know *when* to do it.

In a language with subclassing, one part of the code could create a `Cat` or a `Dog` and pass it to another part of the code as an `Animal`, for example by passing it as an argument to a function whose argument is declared only as `Animal`. The second part of the code can now decide which methods of the `Animal` should be called, while the first part of the code decided what those methods should do if they do get called.

## Example 1: Format conversion

Since nobody ever learned anything about programming from Cat and Dog examples, let's look at some real world examples of how this can be useful. Suppose we have a database consisting of information records. Each record is identified by a unique ID, and we want to create an HTTP server or some other type of interface to retrieve records by ID. The trouble is, records come in two formats: version 1 and version 2. It's possible to convert between the two formats, and we want to create two different interfaces to retrieve data records, a v1 API and a v2 API (maybe HTTP APIs running on `example.com/v1` and `example.com/v2`). Importantly, when a record is retrieved on the vN API, it should be returned in version N format, *no matter what format it's stored as in the database*.