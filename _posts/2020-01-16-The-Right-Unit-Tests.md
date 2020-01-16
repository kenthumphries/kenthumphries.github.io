---
layout: post
title: The *Right* Unit Tests
---

This isn't a post to teach the basics of unit testing on iOS. It's designed for people who already write unit tests, but aren't sure if they're doing it _right_.

&nbsp;

![Unit testing can get messy!]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/alice-dietrich-messy-unsplash.jpg)

<p align="center"><b>Unit testing can get messy</b><br><i>(photo by <a href="https://unsplash.com/@alicegrace?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alice Dietrich</a>)</i></p>

&nbsp;

Just like code, tests should be clear, concise and simple for everyone to understand. It should require no effort to understand a test, even for someone who is not the author, and does not have the full context.

This post is all about getting back to fundamentals and understanding what's most valuable for unit tests.

*[Jump to the Summary](#summary)*

# Tools

![All The Tools!]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/cesar-carlevarino-aragon-tools-unsplash.jpg)

<p align="center"><b>All The Tools!</b><br><i>(photo by <a href="https://unsplash.com/@carlevarino?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Cesar Carlevarino Aragon</a>)</i></p>

&nbsp;

There are some great tools for unit testing iOS in Swift (for example [Quick](https://github.com/Quick/Quick)/[Nimble](https://github.com/Quick/Nimble), [SwiftyMocky](https://github.com/MakeAWishFoundation/SwiftyMocky), [Snapshot-testing](https://github.com/pointfreeco/swift-snapshot-testing)). I encourage you to use them! However, even with great tools, developers can still write bad tests. As an example, Quick (or any [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) franework) improves test naming & readability, but it's important to first understand _why_ better naming is important. To explain these fundamental ideas, let's just talk about plain vanilla XCTests.

# FIRST Things First

If you're not familiar with it, please take a minute to read about the [FIRST principle](https://hackernoon.com/test-f-i-r-s-t-65e42f3adc17).

Here's a short summary for context:

 - **Fast:** Speedy tests are run more often
 - **Isolated:** One test must never be able to break another test
 - **Repeatable:** A test must always return exactly the same  result
 - **Self-Validating:** A clear name and single assertion identifies point of failure
 - **Thorough:** About confidence, not percentages

FIRST is a terrific set of values for tests. However, it's a general testing principle, not a specific set of tips.

&nbsp;

Here are practical tips to improve the quality of your tests, taking FIRST into account. The most important tips are presented first.

# 1. K.I.S.S. And Short, Stupid

Keep your tests short.

Please.

A long test is complex. Code correctness and testability are intertwined. Dirty code smells often cause bugs in their own right, but `this code is difficult to test` is reason enough to refactor. If you, the author, find it difficult to test your own code, think about someone wanting to refactor this code in 6 months time.

When tests are complex or long, it is almost certain that code can be further improved.

**Try to keep your tests to 10* logical statements or fewer!**  

**choose any number that increases your test quality over time*

# 2. Test Quality != Code Quality

It's important to remember that good tests have different characteristics to code. A former colleague spent countless PR reviews teaching me that **shared** test utilities (mocks, convenience factories, comparators) are dangerous. As developers, we learn the [DRY](https://wikipedia.org/wiki/Don't_Repeat_Yourself) principle at birth. But, as FIRST shows, isolation is actually a more important quality for tests.

&nbsp;

![We don't always need things to be DRY]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/erik-witsoe-dry-washing-unsplash.jpg)

<p align="center"><b>We don't always need things to be DRY</b><br><i>(photo by <a href="https://unsplash.com/@ewitsoe?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Erik Witsoe</a>)</i></p>

&nbsp;

Shared test utilities and test data inevitably become as complex as the code itself when you handle every possible input combination. Recreating a utility for a single purpose in each test means the code is simpler and completely independent from any other test.

Note, there is a difference between a shared utility and a function shared by similar tests within the same test class. Here the scope is localized and the usage specific, not generic.

**It's OK to repeat test utilities & data. Above all, we want to avoid breaking Test A when someone modifies a shared utility so that it works with Test B.**

# 3. Test Your Tests

A common pitfall is to write tests that return false positives. It is necessary to satisfy two things:

 1. Your tests **pass when code is correct**
 1. Your tests **fail when code is incorrect**

This sounds so obvious, but so many times I've broken code only to find an existing unit test still passes. 😢

[TDD](https://clean-swift.com/step-by-step-walkthrough-of-ios-test-driven-development-in-swift/) is an obvious antidote to this problem. Consider that tests are the specifications of what your code must do. It seems obvious that you would write the specifications before you implement them. In this way you will see the failing test when code is incomplete, and the passing test only when code is complete. If your test passes before you've finished implementing the functionality, it will likely return false positives.

**You should always run a test twice, once to see it pass, and once to see it fail**.

# 4. Test Naming

![Names are important]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/chuttersnap-names-unsplash.jpg)

<p align="center"><b>Names are important</b><br><i>(photo by <a href="https://unsplash.com/@chuttersnap?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">chuttersnap</a>)</i></p>

&nbsp;

Ideally, when a test fails, the name of the test should be all a developer needs to pinpoint the problem in code. As elaborated below (_Test Coverage_), we want our tests to act like lots of small, independent, precise status lights.

Compare two different test names for the same test:
- `testMyMethodResponses()`
- `test_myMethod_withFriday_returnsSmile()`

If `testMyMethodResponses()` fails, we know that some  _myMethod_ responses are not correct. We will need to analyse the test and the failing assert before we can start debugging _myMethod_.

If `test_myMethod_withFriday_returnsSmile()` fails, the test name alone tells us:

- The method being tested: _myMethod_
- The input being tested: _Friday_
- The expected output: _Smile_

**Given a test name, we should be able to determine which method is being tested, in which state, and what is the expected output/behaviour.**

# 5. Test Coverage

Think about the term `test coverage` literally. Not with percentages, but imagine that your tests are little green ants swarming all over your class. 

&nbsp;

![That's one test...]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/vlad-tchompalov-green-ant-unsplash.jpg)

<p align="center"><b>That's one test...</b><br><i>(photo by <a href="https://unsplash.com/@tchompalov?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Vlad Tchompalov</a>)</i></p>

&nbsp;

As soon as one of the ants finds something wrong with the class, it will turn red. We want to write lots of small, independent tests that act like status lights, quickly identifying precisely which part of our functionality is broken.

If all the ants were to turn red at the same time, they would not help pinpoint which functionality is broken.

**In an ideal world, a single bug will cause one (and only one) test to fail, giving us precise information about where and how the bug occurs.**

# 6. Code Safety

If we continue the ant analogy, the sweetest parts of each class should have the most ants. That is, you should focus your testing on methods that are more *safety critical*.

What does *safety critical* code mean? Focus on testing the parts of your class that *could*:

- Ireversibly lose user data/progress
- Enter an unrecoverable state
- Compromise user's private data/security
- Affect a user's wallet (banking/buying online)

**Prioritise writing tests for safety critical code.**

# 7. Fail Clearly

When causing a newly written test to fail, it's important to think how that failure will look to a new pair of eyes. Consider a decodable struct with three variables that are all optional strings. It's tempting, and quite simple to write one test:

```
func test_decodes_fullValidJSON_allFieldsDecoded() {
    let json = """
                {
                  "string1": "This is string1",
                  "string2": "This is string2",
                  "string3": "This is string3"
                }
               """
    let expected = MyStruct(string1: "This is string1",
                            string2: "This is string2",
                            string3: "This is string3")

    guard let jsonData = json.data(using: .utf8) else {
        XCTFail("Invalid JSON test data")
        return
    }
    do {
        let decoded = try JSONDecoder().decode(MyStruct.self, from: jsonData)
        XCTAssertEqual(decoded, expected)
    } catch {
        XCTFail("Could not decode JSON into MyStruct")
    }
}
```

The test name is clear and the test is well isolated. However, what happens when the test fails?

```
XCTAssertEqual failed: ("MyStruct(string1: Optional("This is string1."), string2: Optional("This is
string2."), string3: Optional("This is string3."))") is not equal to ("MyStruct(string1: Optional("
This is string1,"), string2: Optional("This is string2."), string3: Optional("This is string3."))")
```
This is a trivial example but I've seen tests where the failure takes up half the screen, and requires opening a diff tool to figure out which property failed the equality test.

The solution is to break this test up into 3 smaller tests, one per field. This would look like:

```
func test_decodes_string1() {
    let json = """
                {
                  "string1": "This is string1.",
                }
               """
    let expected = MyStruct(string1: "This is string1,")

    guard let jsonData = json.data(using: .utf8) else {
        XCTFail("Invalid JSON test data")
        return
    }
    do {
        let decoded = try JSONDecoder().decode(MyStruct.self, from: jsonData)
        XCTAssertEqual(decoded.string1, expected.string1)
    } catch {
        XCTFail("Could not decode JSON into MyStruct")
    }
}

...

func test_decodes_string3() {
  ...
}

```
And the corresponding assertion failure:
```
XCTAssertEqual failed: ("Optional("This is string1.")") is not
equal to ("Optional("This is string1,")") -
```

**Ensure that test _failures_ are trivial to understand. It's better to write lots of simple tests than one complex test.**

# Summary

Writing tests is not about quantity, it's about quality. Essentially we are writing documentation, or specifications, for our future selves;  

*"Here are some examples of how my class should work. If you refactor, or add features to the class, these examples should still pass. If the class functionality changes, some of these examples should break."*

&nbsp;

![Quality tests save you time]({{ site.url }}/assets/2020-01-16-The-Right-Unit-Test/pierre-bamin-quality-pocket-watch-unsplash.jpg)

<p align="center"><b>Quality tests save you time</b><br><i>(photo by <a href="https://unsplash.com/@bamin?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Pierre Bamin</a>)</i></p>

&nbsp;

When a developer treats their tests as a second class citizen, their tests will be second class. It's that simple. You should be as proud of your tests as your production code. Put the time in to make your unit tests great. Just as great code is easy to extend, great tests are easy to augment.

1. **Try to keep your tests to 10 logical statements or fewer!**

1. **It's OK to repeat test utilities & data. Above all, we want to avoid breaking Test A when someone modifies a shared utility for Test B**

1. **You should always run a test twice, once to see it pass, and once to see it fail**

1. **Given a test name, we should be able to determine which method is being tested, in which state, and what is the expected output/behaviour**

1. **In an ideal world, a single bug will cause one (and only one) test to fail, giving us precise information about where and how the bug occurs**

1. **Prioritise writing tests for safety critical code**

1. **Ensure that test _failures_ are trivial to understand. It's better to write lots of simple tests than one complex test**

1. **After 6 months, is it trivial to understand how your test works?**  
**If the answer is no, you've written a low quality test. Try again! 😄**

&nbsp;

----
**Comments? Contact me on [Twitter](https://twitter.com/kentios)**

*A special thanks to my former colleague, friend and testing guru Rich Moult for teaching me so much about unit testing.* 🙏
