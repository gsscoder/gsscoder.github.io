---
layout: post
title: "Discriminated unions in C#"
tags: [Software Design]
---

Discriminated unions are an expressive construct used to describe values that can be modeled under one or more cases.
This is the syntax as in [MSDN documentation](http://msdn.microsoft.com/it-it/library/dd233226.aspx):
```fsharp
type type-name =
   | case-identifier1 [of type1 [ * type2 ...]
   | case-identifier2 [of type3 [ * type4 ...]
   ...
```

Discriminated unions describe also F# lists and when recursive they can model tree data structure.

Let's start with an example that models the booking in a vacation facility:
```fsharp
type Booking =
    | Free
    | Single of DateTime
    | Range of DateTime * DateTime
```

This type can be used to define following values:
```fsharp
let free = Free
let single = Single(DateTime(2013,04,23))
let range = Range(DateTime(2013,04,23), DateTime(2013,04,26))
```

Main advantage of working with discriminated unions is that we can decompose them using [pattern matching](http://msdn.microsoft.com/en-us/library/dd233242.aspx):
```fsharp
let describeBooking b =
    match b with
        | Single day -> "vacation on " + day.ToShortDateString()
        | Range (startDay, endDay)  -> "from " + startDay.ToShortDateString() + " to " + endDay.ToShortDateString()
        | _ -> "no vacation"
```

The function `describeBooking` matches against a `Booking` type and returns a descriptive string.

Obviously we only scratched the surface on the subject, so I invite you to read F# documentation or this [wiki book](http://en.wikibooks.org/wiki/F_Sharp_Programming/Discriminated_Unions).

##### Discriminated Unions in CSharp

Finally we get to the core of the post, where we I'll follow the tecnique described in [Real World Functional Programming](http://goo.gl/AqWj8) to define discriminated unions in C#.

As you can imagine the OO construct we'll use is inheritance.

As first we need define an enumeration with all cases:
```csharp
enum BookingType { Free, Single, Range }
```

Then an abstract class that accepts the discriminator in the constructor and expose it publicly:
```csharp
abstract class Booking
{
  private readonly BookingType tag;<br/>
  protected Booking(BookingType tag)
  {
    this.tag = tag;
  }<br/>
  public BookingType Tag
  {
    get { return this.tag; }
  }
}
```

Now we define one sealed derived class for each case:
```csharp
sealed class Free : Booking { public Free() : base(BookingType.Free) }

sealed class Single : Booking
{
  private readonly DateTime date;<br/>
  public Single(DateTime date)
    : base(BookingType.Single)
  {
    this.date = date;
  }
  public DateTime Date
  {
     get { return this.date; }
  }
}

sealed class Range : Booking
{
  private readonly DateTime fromDate;
  private readonly DateTime toDate;<br/>
  public Single(DateTime fromDate, DateTime toDate)
    : base(BookingType.Range)
  {
    this.fromDate = fromDate;
    this.toDate = toDate;
  }
  public DateTime FromDate
  {
     get { return this.fromDate; }
  }
  public DateTime ToDate
  {
     get { return this.toDate; }
  }
}
```

Since C# lacks ``match`` constuct we can use ``switch`` against ``Tag`` discriminator and cast specific derivates:
```csharp
string describeBooking(Booking booking)
{
  switch (booking.Tag)
  {
    case Booking.Single:
      return "vacation on " + ((Single)booking).Date.ToShortDateString();
    case Booking.Range:
      var range = (Range)booking;
      return "from " + range.FromDate.ToShortDateString() + " to "
        + range.ToDate.ToShortDateString()
    default:
      return "no vacation";
  }
}
```

##### Conclusion

It's clear that this is a particular use of inheritance intended to mimic discriminated unions. Here the __lack of extensibility__ is by design.

I suggest you such construct when the number of cases is predefined or will change little over the time.

I'll report the same BCL usage of this pattern as in the book quoted above, [System.Linq.Expression](http://msdn.microsoft.com/it-it/library/system.linq.expressions.expression.aspx).
In this type the desciminator is named ``NodeType`` and is defined as ``ExpressionType`` enumeration.

This post is part of [YES! to NOOO](http://yes-to-nooo.github.com/) initiative.