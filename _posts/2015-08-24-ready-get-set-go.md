---
title: "Ready... Get Set... Go!"
layout: default
tags: [allonge, noindex]
---

Once upon a time, there was a language called C,[^bcpl] and this language had something called a `struct`, and you could use it to make heterogenously aggredated data structures that had members. The key thing to know about C is that when you have a struct called `current_user`, and a member like `id`, and you write something like `currentUser.id = 42`, the C complier turned this into extremely fast assembler instructions. Same for `int id = currentUser.id`.

[^bcpl] There was also a language called BCPL, and others before that, but our story has to start somewhere, and it starts with C.

Also of importance was that you could have pointers to functions in structs, so you could write things like `current_user->setId(42)` if you preferred to make setting an `id` a function, and this was also translated into fast assembler.

And finally, C programming has a very strong culture of preferring "extremely fast" to just "fast," and thus if you wanted a C programmer's attention, you had to make sure that you never do something that is just fast when you could do something that is extremely fast. This is a generalization, of course. I'm sure that if we ask around, we'll eventually meet *both* C programmers who prefer elegant abstractions to extremely fast code.

### java and javascript

Then there was a language called Java, and it was designed to run in browsers, and be portable across all sorts of hardware and operating systems, and one of its goals was to get C programmers to write Java code in the browser instead of writing C that lived in a plugin. Or rather, that was one of its strategies, the goal was for Sun Microsystems to stay relevant in a world that Microsoft was commoditizing, but that is another chapter of the history book.

So the nice people behind Java gave it C-like syntax with the braces and the statement/expression dichotomy and the dot notation. They have "objects" instead of structs, and objects have a lot more going on than structs, but Java's designers made a distinction between `currentUser.id = 42` and `current_user.setId(42)`, and made sure that one was extremely fast and the other was just fast. Or rather, that one was fast, and the other was just ok compared to C, but C programmers could feel like they were doing *important thinking* when deciding whether `id` ought to be directly accessed for performance or indirectly accessed for elegance and flexibility.

History has shown that this was the right way to sell a new language. History has also shown that the actual performance distinction was irrelevant to almost everybody. Performance is only for now, code flexibility is forever.

Well, it turned out that Sun was right about getting C programmers to use Java (it worked on me, I ditched CodeWarrior and Lightspeed C), but wrong about using Java in browsers. Instead, people started using another language called JavaScript to write code in browsers, and using Java to write code on servers. The irony is, Javascript was designed to run on servers, so the state of affairs was that everyobody was using a server-side language to write browser code, and a browser language to write server code.[^ignorant]

[^ignorant]: Amazingly, people sometimes complain about Node that "JavaScript was never meant to run on a server." They need to buy Brandon Eich a BEvERage.

Will it surprise you to learn that JavaScript was *also* designed to get C programmers to write code? And that it went with the C-like syntax with the braces and the statement/expression dichotomy and the dot notation? And although JavaScript has a thing that is kinda-sorta like a Java object, and kinda-sorta like a Smalltalk dictionary, will it surprise you to learn that JavaScript *also* has a distinction between `currentUser.id = 42` and `current_user.setId(42)`? And that originally, one was slow, and the other dog-slow, but programmers could do *important thinking* about when to optimize for performance and when to give a hoot about programmer sanity?

No, it will not surprise you to learn that it works kinda-sorta like C in the same way that Java kinda-sort works like C, and for exactly the same reason. And the reason really doesn't matter any more.

### in praise of getters and setters

Very soon after people begun working with Java at scale, they learned that directly accessing instance variables was a terrible idea. JIT compilers narrowed the performance difference between `currentUser.id = 42` and `current_user.setId(42)` to almost nothing of relevance to anybody, and code using `currentUser.id = 42` or `int id = currentUser.id` was remarkably inflexible.

There was no way to decorate such operations with cross-cutting concerns like logging or validation. You could not override the behaviour of setting or getting an `id` in a subclass. (Java programmers love subclasses!) So Java programmers religiously started following a pattern: They made their instance variables private, but exposed access with methods for getting and settingtheir values, e.g. `currentUser.setId(42)` and `currentUser.getId()`.

This is awkward to write, but Java programmers have heavyweight IDEs, and they figured out how to have their code editors do all the tying for them.

Meanwhile, JavaScript programmers carried on writing `currentUser.id = 42`, and eventually they too discovered that this was a terrible idea. One of the catalysts for change was the arrival of frameworks for client-side JavaScript applications. Let's say we have a rediculously simple person class:

{% highlight javascript %}
class Person {
  constructor (first, last) {
    this.first = first;
    this.last = last;
  }

  fullName () {
    return `${this.first} ${this.last}`;
  }
};
{% endhighlight %}

And an equally ridiculous view:

{% highlight javascript %}
class PersonView {
  constructor (person) {
    this.model = person;
  }

  // ...

  redraw () {
    document
      .querySelector(`person-${person.id}`)
      .text(person.fullName())
  }
}
{% endhighlight %}

Every time we update the person class, we have to remember to redraw the view:

{% highlight javascript %}
const currentUser = new Person('Reginald', 'Braithwaite');
const currentUserView = new PersonView(currentUser);

currentUserView.redraw();

currentUser.first = 'Ragnvald';
currentUserView.redraw();
{% endhighlight %}

It didn't take long for JavaScript libraries to figure out how to make this go away by using a `get` and `set` method. Stripped down to