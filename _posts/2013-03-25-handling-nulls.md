---
layout: post
title: "Handling nulls"
tags: [Software Design]
---

Avoid returning null is all you need to avoid handling them. Quoting [NOOO](http://notonlyoo.org/):

> Options over nulls

Before digging into the argument, let’s take something simpler.

One of the worst thing of imperative code is all that _ugly null checks_.

Anyway please don't feel disappointed, before realizing and getting in touch with [FP](http://en.wikipedia.org/wiki/Functional_programming); I as first filled my code with these.

For example take a __C# code snippet__ like this that returns a collection of `Order` types.

```csharp
var orders = DB.GetOrdersByDate(DateTime.Now);
if (orders != null)
{
  foreach (var order in orders)
  {

    // do something with order
  }
}
```

There’s no reason to design `DB.GetOrdersByDate()` to returns a `null` when no `Order` satisfies the request. An empty collection, easly obtainable with

```csharp
return Enumerable.Empty<Order>();
```

would be great, the following code could became

```csharp
var orders = DB.GetOrdersByDate(DateTime.Now);
foreach (var order in orders)
{
  // do something with order
}
```

When `orders` value is empty (or better an empty collection) no iteration will be performed, hence no kind of check is required.

Anyway cases could present in which you must check for order presence (in the old semantic the `null` check).
Now we can elegantly write:

```csharp
var orders = DB.GetOrdersByDate(DateTime.Now);
if (!orders.Any())
{
  // no orders
}
```

Because this code is semantically _correct_,  hence is intrinsically _more robust_.

##### MayBe monad

As Mark Seemann says in this article: [the BCL already has maybe monad](http://blog.ploeh.dk/2011/02/04/TheBCLalreadyhasaMaybemonad/).

He proposes the following construct through extension method:

```csharp
public static class LightweightMaybe
{
    public static IEnumerable<T> Maybe<T>(this T value)
    {
        return new[] { value };
    }
}
```

So when you face an API (which you can change or not wrote by you) you can treat it like the sample above.

```csharp
var client = LegacyDB.GetSingleClient(id: “101”);

if (client.Maybe().Any())
{
  // result is not null
}
```

As you can see this extension method put the `T value` inside a generic `IEnumerable<T>`, in this way you can use the beatiful __Linq syntax__ to write readable code that clearly states its intent.

But this mean you should design code that returns `null`? No, you shouldn’t. I take the extreme position that returning a `null` is comparable to having _a bug in the code_.

If you can refactor `LegacyDB.GetSingleClient()` change its signature from:

```csharp
public Client GetSingleClient(string id)
{
  if (found)
    return new Client(...);

  return null;
}
```

to

```csharp
public IEnumerable&lt;Client&gt; GetSingleClient(string id)
{
  if (found)
    return new Client[] { new Client(...) };

  return Enumerable.Empty&lt;Client&gt;();
}
```

There’s nothing bad neither _logically_ neither _semantically_ to have a method handling one item returns _a collection containing one item_ or otherwise _an empty collection_.

To me it’s wonderfully clean, clear and concise!

##### Alternative

However I can understand criticism to this design, that could distract you from the main argument of this post: not returning `null` to avoid handling it.

You can design the method above using a singleton like `string.Empty` following the _Write once immutability pattern_, as explained in [this](http://blogs.msdn.com/b/ericlippert/archive/2007/11/13/immutability-in-c-part-one-kinds-of-immutability.aspx) Eric Lippert article.

```csharp
public Client GetSingleClient(string id)
{
  if (found)
    return new Client(...);

  return Client.Empty;
}
```

The fundamental point here is that having a value, is always better than handling a `null`.

This is the reason why functional language like F# use [option](http://msdn.microsoft.com/en-us/library/dd233245.aspx) monad to describe if a result has or has not a value.

##### When keep it

New compiler constructs and projects like [Code Contracts](http://research.microsoft.com/en-us/projects/contracts/) could remove also this necessity.

But for now there's nothing bad to check `null` to validate method parameters.

```csharp
public Client GetSingleClient(string id)
{
  if (id == null)
    throw new ArgumentNullException("id");

  if (found)
    return new Client(...);

  return Client.Empty;
}
```

However also here my opinion is a bit extreme: __guard clauses__ are just against `null`; everything else falls into other application __concerns__ such as _validation_. 

But deepen it goes beyond the purpose of this post.

##### Conclusion

This blog post starts with a reference to [NOOO](http://notonlyoo.org/). This is because this work is extracted from an unfinished and partly unpublished document.

The project aims to defined guidelines for developers who want to design API adhering to functional concepts from multi-paradigm languages like C#.
 
If you're interested in this collaborative work, write to me (gsscoder AT gmail DOT com) or ping me via Twitter (@gsscoder).

Or visit this [web site](http://yes-to-nooo.github.com/) in status of _work in progress_.