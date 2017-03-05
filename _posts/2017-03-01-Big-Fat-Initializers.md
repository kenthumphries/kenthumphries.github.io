A dicsussion about whether you really want to build that big, fat initializer.

*Ugh, gross. Big, fat, bloated inits. They're about as cool as these dorky tab bars. Just give me a fat hamburger menu and a skinny init. With a side of customisable properties please. Let me configure my object in the state I want, and ignore the properties that I don't care about. It's 2011, I think I know what I'm doing...*

\<cough....ahem>

I Like Big Inits, And I Can Not Lie
----------------------------------
 
Turns out that I find big, fat initializers way sexier in 2017 than I did in 2011.

Unthinkingly, I used the term 'big, fat inits' in a recent email thread. I was explaining that a detailed init could provide us with a lot of benefits, where the only side effect is having to create a 'big, fat initializer'. I wish I hadn't made this sound like a bad thing. I wish I'd used the term 'deterministic, reliable initializer' instead.

The more I write in Swift and think about [value types](https://developer.apple.com/swift/blog/?id=10) and immutability, the more I appreciate injecting state at initialization. Make objects as immutable as possible and create objects that behave in fewer, more deterministic ways. The smaller surface area that your state has, the less likely that two instances of the same class can perform in unexpected ways.

How Big Is Too Big?
-------------------

If your initializer really is getting too big, there are a couple of solutions.

One symption of big initializers is too many responsibilities. If your object is trying to do many things, it will likely require many dependencies to be injected. Think of ways to cut back your objects responsibilities. Could the responsibilities be split amongst multiple objects? Is there a design pattern that could improve your architecture?

If your object's initializer is large because of many related state variables, consider creating a container class. In Swift this would be a value type - an immutable collection of properties that are simpler to pass around as a group. Do not use a dictionary - I guarantee someone will mistype one of your carefully named keys.

Init Got Back!
--------------

In summary, some advantages to using a deterministic, reliable initializer:

1. **Immutability**  
Move towards immutable objects (the power of value types in swift) so that an object will behave in a well defined way for it's entire lifetime.

1. **Clear interface**  
The init shows which parameters (variables or dependencies) are required to make your object function correctly. Ideally, tend towards `nonnull` parameters.

1. **Resist Multiple Responsibilities**  
Declaring all dependencies/state in one place makes it easier to identify a class that is taking on too many responsibilities.

And finally, the disadvantages:

1. **Can't Think Of Any**  
Stop supporting countless states, with countless properties, with countless bugs. 

**Want to talk about big, fat initializers? [Tweet me](https://twitter.com/kentios).**
  
PS. We have enough real world problems in 2017, please stop using hamburger menus.
