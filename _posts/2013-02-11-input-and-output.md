---
layout: post
title: Input and Output
---

Programs take input and produce output. The output is the result of doing something with the input. Input, transform, output, done.

This pattern is easy to see when the program is a UNIX tool. Take a string, count the words, print out the result. But it's a lot harder to see when we're writing an iOS app with a UI, lots of different features, periodic tasks, etc.

What's the input and output?

The input is all the sources of action for your app. It's taps. It's keyboard events. It's timer triggers, GPS events, and web service responses. These things are all inputs. They all feed into the app, and the app combines them all in some way to produce a result: the output.[^1]

The output is often a change in the app's UI. A switch is toggled or a list gets a new item. Or it could be more than that. It could be a new file on the device's disk, or it could be an API request. These things are the outputs of the app.

But unlike the classic input/output design, this input and output happens more than once. It's not just a single input → work → output—the cycle continues while the app is open. The app is always consuming inputs and producing outputs based on them.

To put it another way, the output at any one time is the result of combining all inputs. The output is a _function_ of all inputs up to that time.[^2]

So **why should we care?** Because fresh perspectives are powerful and good and necessary and cool. And in this case, it gives us a fantastic new tool.

### State

There's no intrinsic idea of state from this perspective. There's just a change in an input resulting in a new output. State might be an implementation detail with how the app handles its inputs, but it's not necessary. It's not intrinsic to the idea.

Most problems worth solving have some intrinsic state. State _can_ be essential. But that's not how we treat it. We solve everything with state. Because we treat all the inputs to our app as different things—a touch event here, a web response there—we can't combine them in any meaningful way. We can't transform them uniformly. And so our only tool for dealing with all these different things is state. When our only tool is state, every problem looks like a stateful nail.

State is bad. State introduces complexity. And worse, it introduces complexity that grows more than linearly with the size of our app.[^3] We're in the habit of constantly introducing more state into our app. New feature? New state. New complexity. New bugs.[^4]

But happily this perspective of our app's output as a function of its inputs over time gives us a new tool: functional reactive programming. Functional reactive programming (FRP) is a paradigm built around the idea of time-varying values produced by time-varying functions.

### Time

"Time-varying values" might sound like a bit of sleight-of-hand. Isn't that just another way of saying "state?" They're both trying to capture the same idea—that things change as the program runs. But by formalizing and reifying time variance, we can reason about change safely.

Time-varying values can be derived from other time-varying values, which are themselves derived from the time-varying inputs to the app. So while traditional state places the burden of ensuring our app is always in a known consistent state on us, the programmers, FRP lets us define our app in terms of the time-varying values and ensures changes propagate as needed.

Before, state was discrete pieces of data all moving independently. But time-varying values are cogs all fitting together in a gear. When one turns, it turns all its connected cogs, which turns their cogs, which turns… and ends up running the entire mechanism all by themselves.

It's beautiful.[^5]

There are a lot of other things that behave like time-varying values. The result of asynchronous work is really just a time-varying value that only has a value once the work is done.[^6] Or a UI element's value could be seen as a time-varying value that changes as the user interacts with it. If my app is running on a mobile device, the device's GPS coordinates is a time-varying value.

It's all time-varying values, all the way down.[^7] State, inputs, and outputs.

### Functional Reactive Programming

As the _functional_ in "functional reactive programming" suggests, FRP comes from functional programming languages.

Haskell has a [number of implementations](http://www.haskell.org/haskellwiki/Functional_Reactive_Programming), with varying levels of completeness and usability.

Racket, a Lisp-y language, has [FrTime](http://docs.racket-lang.org/frtime/).

There've also been a number of interpretations for imperative languages. The 900lb gorilla is the [Reactive Extensions](http://msdn.microsoft.com/en-us/data/gg577609.aspx) (Rx) from Microsoft. It's the first compelling example of how to bring the principles of FRP to an imperative language.

Netflix [re-implemented Rx](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html) on the JVM. They have [adapters](https://github.com/Netflix/RxJava/tree/master/language-adaptors) for Clojure, Scala, Groovy, and JRuby.

There are a few different FRP-inspired libraries for Javascript. Microsoft has [RxJS](https://github.com/Reactive-Extensions/RxJS) and there's also [Flapjax](http://www.flapjax-lang.org/), and [BaconJS](https://github.com/raimohanska/bacon.js).

[Elm](http://elm-lang.org/) is a Haskell-like language that compiles to JavaScript. It's the most beautiful, practical FRP implementation I've seen.

Last but not least, myself and some of the other fine folks at GitHub brought Rx to Objective-C: [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa).

### Example

Let's take a quick look at how this works out practically. FRP is most beautiful in a functional language, but sadly most of us don't do our everyday work in a functional language.

Let's suppose we're writing an iOS app. We're going to use ReactiveCocoa to implement a Create Account view. The user will enter their first name, last name, and email (and we'll make them re-enter their email just to be annoying) and there will be a Create button for them to tap.

Think in terms of inputs and output. What are the inputs to the view? Obviously the user will input their information. And then they'll tap the Create button. That's it. Those are all the inputs to our view. Our outputs should be defined in terms of them.

There will be a number of outputs in the form of UI changes. The Create button should only be enabled once the user has entered values for first name, last name, and the email and re-enter email fields match.

If we were writing this traditionally, we'd update the Create button manually whenever we detected that our form changed:

{% highlight objc %}

self.createButton.enabled = [self isFormValid];

{% endhighlight %}

But what we really want to do is declare the relationship between the form's validity and the Create button's enabledness across the entire lifetime of the view.

We want to derive our output from our input. ReactiveCocoa lets us do that:

{% highlight objc %}

RACSignal *formValid = [RACSignal
  combineLatest:@[
    self.firstNameField.rac_textSignal,
    self.lastNameField.rac_textSignal,
    self.emailField.rac_textSignal,
    self.reEmailField.rac_textSignal
  ]
  reduce:^(NSString *firstName, NSString *lastName, NSString *email, NSString *reEmail) {
    return @(firstName.length > 0 && lastName.length > 0 && email.length > 0 && reEmail.length > 0 && [email isEqual:reEmail]);
  }];
 
RAC(self.createButton.enabled) = formValid;

{% endhighlight %}

Now whenever the first name, last name, email, or re-enter email fields change, we'll reduce them down to a boolean indicating whether the form is valid and update the Create button's enabledness.

Note that last line of the example. Instead of an assignment that sets the value at one time, it's actually establishing a _relationship_. Instead of assigning it once, we're assigning it over time. The Create button's enableness is derived from the form's validity. The output is a function of the input.

When the Create is executing, we want to disable the text fields and change their text color to a light gray:

{% highlight objc %}

RACSignal *fieldTextColor = [executing map:^(NSNumber *x) {
  return x.boolValue ? UIColor.lightGrayColor : UIColor.blackColor;
}];
 
RAC(self.firstNameField.textColor) = fieldTextColor;
RAC(self.lastNameField.textColor) = fieldTextColor;
RAC(self.emailField.textColor) = fieldTextColor;
RAC(self.reEmailField.textColor) = fieldTextColor;
 
RACSignal *notProcessing = [executing map:^(NSNumber *x) {
  return @(!x.boolValue);
}];
RAC(self.firstNameField.enabled) = notProcessing;
RAC(self.lastNameField.enabled) = notProcessing;
RAC(self.emailField.enabled) = notProcessing;
RAC(self.reEmailField.enabled) = notProcessing;

{% endhighlight %}

We can wire the rest of our outputs up similarly. They're all just combinations or transformations of our inputs.

### Fin

That's a small, practical example of the principles of FRP in an imperative language. The whole example view is on GitHub: [RACSignupDemo](https://github.com/joshaber/RACSignupDemo).

Functional reactive programming offers a way to once again view our programs as simply input and output. We get to minimize state while also embracing a unified view of what our app is doing. It's all just inputs and outputs.

[^1]: This is kinda interesting itself because it means that the "meaningful" part of the app is the bit that combines all the inputs into the output.
[^2]: It might be easier to break it down on a per-view basis than thinking of the entire app at each point in time.
[^3]: See ["Out of the Tar Pit"](http://shaffner.us/cs/papers/tarpit.pdf)
[^4]: Most state is introduced simply used because it's an easy solution to a problem or because it's how we're trained to solve problems—not because it's intrinsic to the problem at hand.
[^5]: Don't hate.
[^6]: Which sounds a lot like futures. Time-varying values are a higher-level concept that encompasses futures.
[^7]: [Until you reach the turtles](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html).