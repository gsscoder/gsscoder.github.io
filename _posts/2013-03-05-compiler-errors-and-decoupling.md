---
layout: post
title: "Compiler errors and decoupling"
tags: [Productivity]
---

With this post I want to share and reason about a __brutal__ tecnique, that helped me when refactoring some [code smells](http://en.wikipedia.org/wiki/Code_smell).

I could have called this post also _Compiler Errors and Refactoring_, but I'll ask you license to use the term **decoupling** in broad sense.

##### Scenario

It's not hard to be called for a consulence and discover customer code into an highly disorganized state.

As a side note people always blame programmers and designers that preceded, and I'm tempted to ask _why don't you attempted to bring some order?_.
But this question often lacks a concrete reply...

Often customers pretend a fast evaluation of the problem that always resolved in three options:

* __Reverse engineer__ it, rewrite a specification with [TDD](http://en.wikipedia.org/wiki/Test-driven_development) and rebuild it (if budget allows)

* __Re-iterate refactorings__ with the help of your testing framework of choice, a sort of TDR; again with licence, _Test-Driven-Refactoring_ (if budget allows)

* And if budget doesn't allow, choose the more _diplomatic way to quit_ (if you can).

##### Quick Metrics

One of the most generalized code smells I encouter is __Contrived complexity__ (from Wikipedia):
> forced usage of overly complicated design patterns where simpler design would suffice

In my expierience this is got with __tightly coupling__ (who inspired post title) and __Inappropriate intimacy__ smell (from Wikipedia):
> a class that has dependencies on implementation details of another class

In such cases the first thing I do, is enabling ``Show All Files`` from __Visual Studio__ _Solution Explorer_ and start excluding ``.cs`` source file from various projects.

There's no need to explain what happens next. C# compiler start spitting out errors like:
```
Error	6	The name 'XyzFactory' does not exist in the current context	X:\CustProj\Proj1\CoreFeature.cs	326	30	Proj1
```

Normally errors are not best friend of developers, but in this case the compiler is giving you useful information.

Also if some error can mask others, you've a good outlook of the number of times ``XyzFactory`` type is used in that codebase.

As always by selecting the ``File`` column you can reorder the error list, making even clear in which source file ``XyzFactory`` is used.

If the convention of putting one class for file is complied, this is also a list of types that depends on ``XyzFactory``.

##### Conclusion

Never to mention that such way of proceeding, doesn't free you from properly anlyze code and use your expierience to measure how the code base differs [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) principles.