It can be nice to think of program architecture as consisting of a periodic table of "fundamental problems" each of which has a canonical solution. The canonical solution is always a programming language construct, and solving that fundamental problem can be thought of as the canonical *purpose* of that construct. This allows one to think more clearly about programming and justify, if only to yourself, that you're using the right tool for the right job. That is to say, it short-circuits philosophical debates of the type "[is this an exception or just a normal possible behavior of my code](exceptions.md)".

Some examples:

| Problem                                                      | Solution             |
| :----------------------------------------------------------- | :------------------- |
| I need to execute the same code in several contexts          | Procedures/Functions |
| I need to execute code in several contexts which differ only in the values of certain variables | Procedure arguments  |
| [The behavior of certain functions depend on which of those functions were called previously](objects.md) | Objects              |
