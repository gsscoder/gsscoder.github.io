---
layout: post
title: "Scraping Google with AngleSharp"
tags: [API Usage, Code]
---

Web scraping is the act of extracting data from web sites. From a programming standpoint it's performing an **HTTP** request and parsing the **HTML** response. This may involve taking care of various _low level_ details, like handling a stateful session and run into _bad formed_ **HTML**.

Since **.NET Community** is a very vibrant one, we can take advantage of existing software components without _reiventing the wheel_ (excuse me for the _clichè_). In my opinion [AngleSharp](https://anglesharp.github.io/) is the best library to accomplish the aforesaid task. It's so complete (including **HTML DOM**, **CSS Selector** and **JavaScript** support) that I would define it a _web scraping framework_.

In this post I'll show you how it's easy submit a query to Google and parse results.

##### Create a sample

Jump into your terminal and create a stub project:
```sh
$ cd your/projects/path
$ mkdir GoogleScraper
$ cd GoogleScraper
$ dotnet new console
```

Then add reference to main **AnlgeSharp** package:
```sh
$ dotnet add package AngleSharp --version 0.14.0-alpha-788
```

I said _main_ because **AngleSharp** is a very large project and has other components, like [AngleSharp.Io](https://github.com/AngleSharp/AngleSharp.Io) that provides additional requesters and IO helpers.

##### Basic usage

AngleSharp operates through a _browsing context_ defined by the `IBrowsingContext` interface type. So the first thing to do in your `Program.cs` (after adding the required `using` statements) is to create such instance. Let's see how:
```csharp
var context = BrowsingContext.New(Configuration.Default.WithDefaultLoader());
```

Now yuo've everything you need to access Google web site and this is done using `OpenAsync` method:
```csharp
var document = await Context.ActiveContext.OpenAsync("https://www.google.com/");
```

The `document` variable is an instance of a type that implements `IDocument`. Via this interface you can extract data using both [HTML DOM](https://en.wikipedia.org/wiki/Document_Object_Model) and [CSS Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).

As first we need a reference to search form:
```csharp
var form = document.QuerySelector<IHtmlFormElement>("form[action='/search']");
```

The query passed to `QuerySelector` uses **CSS** syntax to select a `form` tag with a specified `action` attribute. Now we can submit the form creating an anonymous type with a field that matches the search form `input` element:

```csharp
// we want ask Google about Bill Gates
var result = await form.SubmitAsync(new { q = "bill gates" });
```

What remains is just extracting all `a` tags (hyperlinks) and consuming data we need:
```csharp
var links = result.QuerySelectorAll<IHtmlAnchorElement>("a"); // CSS
foreach (var link in links) {
    var url = link.Attributes["href"].Value; // HTML DOM
    Console.WriteLine(url);
}
```

The `Attributes` property is accessed like a dictionary and gives access to element attributes. 

The sample is available in [this gist](https://gist.github.com/gsscoder/350998503650d7fc389a0b4d268a5cdf). It's very basic and will grab also URLs of low interest, like ones that are part of web page UI.

For a more complete Google scraper you can see the source of [this PickAll searcher](https://github.com/gsscoder/pickall/blob/master/src/PickAll/Searchers/Google.cs). Based on **AngleSharp**, [PickAll](https://github.com/gsscoder/pickall) is a project of mine that makes even easier scraping results from multiple search engines.

Using **PickAll** allo the job is essentially a matter of one line of code:
```csharp
var results = await SearchContext.Default.SearchAsync("bill gates");
foreach (var result in results) {
  Console.WriteLine(result.Url);
}
```

##### Conclusion

As you can see with some knowledge of **DOM** and **CSS** is pretty easy, and I would add enjoyable, scraping data using **AngleSharp**. Data gathered from the web can be archived, processed or stored in a database for further mining.

Now it's up to you! Experiment and have fun.