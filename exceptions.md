Hey, I'm writing this function called `indexOf` that gives you the index of a certain value in an array. What should I do if the value isn't in the array? Should it return `None`, or should it raise a `NoSuchIndex` exception? Is the value not being in the array really an "exception"? I mean, exceptions are supposed to be for errors, right? Is this an error? Or just a different kind of thing that can happen? Where's the line?

If you find yourself doing metaphysics while programming, this may be a sign you're barking up the wrong tree. In this note I want to think [pragmatically](pragmatism.md) about exceptions. A comparison with return values is a good place to start. Both allow a function to communicate with its caller. But what does an exception allow you to do that a return value doesn't? There are three obvious features:

1. An exception *propagates by default*. With return values, the caller handles the message from the callee by default. In order to escalate the message higher up the call stack, extra code must be written. With exceptions, escalation is the default behavior, and a try/catch must be written in order to prevent this.
2. Exceptions *affect control flow*. Like ifs and loops, exceptions change the order in which program instructions are executed, and in a unique way that can't be recreated using other control flow constructs.
3. Exceptions *must be handled explicitly*. If `indexOf` returns -1 or even `None` when it can't find the value, a line like `index = indexOf(value, array)` implicitly ignores this possibility, allowing bugs to creep in. If instead the function raised an exception, the programmer would be forced to write explicit handling code, even if it's just an empty `catch` block.

In existing programming languages (I'm specifically thinking of Java, which has the purest exception mechanism I'm aware of), these three behaviors are coupled together in exceptions, meaning there are no language features which provide (1) but not (2) and (3). It's instructive to imagine language features which would give us one of the behaviors without the other two.

## Propagation by default

How could a function communicate with its caller in a way that escalates up the call stack by default, needing to be stopped explicitly? By *communicate*, I mean the function can provide a value which the caller can conveniently bind to a variable in its own scope, which can be done with return values by assigning them to a variable, or with exceptions with a catch block.

## Exceptional control flow

There are two ways in which exceptions affect control flow. One is a consequence of the "propagation by default" behavior: they can force the caller to return immediately. We've already covered this. The other way happens within a single function call: an exception can cause the function to jump to a "catch block" defined further down in the code. This is useful for the same reasons footnotes are useful in prose writing: they allow us to separate the "main idea" of the text from the "extra bits" which we can tack on at the end. In a way it models how we tend to explain algorithms (or other kinds of trains of logic) to a fellow human being: you explain the main idea first, explaining the edge cases and odd details at the end.

Consider this exception-based code:

```
try
{
	connection = openDatabase();
	connection.storeData();
}
catch (ConnectionFailure exception)
{
	// ...
}
catch (InsufficientSpace exception)
{
    // ...
}
```

This can be rewritten using return values and `goto`. The unsafe functions can return error codes, and the caller can execute a `goto` if it detects an error code to jump to a handler. To simplify things, let's pretend our hypothetical language has a "Go-style" mechanism of returning multiple return values, so that the `openDatabase()` call can return both a connection and an error code. This looks like this:

```
connection, error = openDatabase();
if (error)
{
	goto handleConnectionFailure;
}
error = connection.storeData();
if (error)
{
	goto handleInsufficientSpace;
}
goto normalFlow;
handleConnectionFailure:
{
	// ...
}
handleInsufficientSpace:
{
	// ...
}
normalFlow:
// ...
```

This is unsatisfying for two reasons:

1. By having to insert the `if (error)` statements, we lose the entire advantage of exception control flow in the first place: our main code is littered with the visual noise of all of our exceptional cases. Even worse, we need the `goto normalFlow` to prevent the exception handlers from executing in the normal case.
2. Many programmers would prefer not to have an unrestrained `goto` in the language because of how it can be misused.

We can answer concern number (2) by introducing a restricted `goto` that can only be used for exception-style control flow. It might look like this:

```
{
	connection, error = openDatabase();
	if (error) { handle connectionFailure(error); }
	error = connection.storeData();
	if (error) { handle storageFailure(error); }
}
handler connectionFailure(ConnectionException error)
{
	// ...
}
handler storageFailure(StorageException error)
{
	// ...
}
```

This still has the problem of requiring the explicit if statements. In existing exception mechanisms, this is done away with by deciding which exception handler to jump to automatically based on the *type* of the exception. This is unsatisfying, because it means we don't know exactly under what circumstances the code might jump to any given exception handler. What if two calls in the `try` block both throw the same type of exception? Worse, what if we have this scenario:

```
try
{
	unsafeOperation1();
	unsafeOperation2();
}
catch (MyException exception)
{
	// ...
}
```

Say initially only `unsafeOperation1()` can raise `MyException`, but later on `unsafeOperation2` is rewritten so as to sometimes throw it as well? Even static exception- and type-checking can't save us here: that catch block is going to get executed in circumstances not initially intended when we wrote the code. Ideally, we want to be able to explicitly state when the code should jump to a given handler, so as to have complete control, but we still want a nice lightweight syntax so as to not litter our code with ifs.

What we can do is have labeled catch blocks, and introduce the `handle` operator. The `|` operator is a binary operator which requires a function call on its left and a handler label on its right. The code would look like this:

```
{
	connection = openDatabase() handle connectionFailure;
	storeData() handle insufficientStorage;
}
handler connectionFailure(ConnectionException exception)
{
	// ...
}
handler insufficientStorage(StorageException exception)
{
	// ...
}
```

