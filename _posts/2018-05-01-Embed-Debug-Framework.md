---
layout: post
title: Embedding a Debug Only Framework
---

When using [fastlane/Snapshot](https://docs.fastlane.tools/getting-started/ios/screenshots/) & [SimulatorStatusMagic](https://github.com/shinydevelopment/SimulatorStatusMagic) (which are both ace!!) to produce screenshots, how should SimulatorStatusMagic be added to a project?

## Just Use a Dependency Manager
Alright - that's the easy way out. Continue reading if your project is not using CocoaPods or Carthage.

## Just Throw It In
The SimulatorStatusMagic [installation instructions](https://github.com/shinydevelopment/SimulatorStatusMagic/blob/master/INSTALLATION.md) outline how to build the framework and add it to your project without a dependency manager.

Embedding the build works fine for both Debug and Release builds. However, **SimulatorStatusMagic cannot be submitted to Apple in an app binary**.

1. The framework does not contain bitcode. I unwisely [tried to add bitcode](https://github.com/shinydevelopment/SimulatorStatusMagic/pull/59) - let's not talk about that anymore.

1. The framework contains non-public APIs which are detected by Apple's automatic code scan.

&nbsp; 
![Apple's email about non-public APIs]({{ site.url }}/assets/non-public-apis.png)
&nbsp; 

Alas, there is no built-in Xcode mechanism for linking a framework against Debug builds, while excluding Release builds.

## Just Embed It For Debug

**Update: I added to the [official installation instructions on GitHub](https://github.com/shinydevelopment/SimulatorStatusMagic/blob/master/INSTALLATION.md) to describe this process.**

Custom Run Script build phases allow for the embedding of a framework under specified conditions, such as Debug configuration only.

The Framework must be added to the Framework Search Paths so it can be found at linking time. For example, if the framework is inside a folder `ThirdPartyFrameworks`, then set:

``
Framework Search Paths->Debug: $(SRCROOT)/ThirdPartyFrameworks/
``

I created a script that embeds a framework only if the build configuration is set to Debug. [Download it here](https://gist.github.com/kenthumphries/cf04683184217c7331f9c213c556c65a) with full usage instructions. Hopefully, *It Just Works*™.

![Configured run script build phase]({{ site.url }}/assets/custom-run-script-build-phase.png)

The script solution was adapted from this [SO post](https://stackoverflow.com/a/40484337/9051514) which shows how to embed framework for certain architectures.

## Just Embed It In the UITest Target

The above solution allows `SDStatusBarManager` to be called from within the main app target. When using fastlane/Snapshot, the screenshots are captured from a UITest target. 

**Is it possible to use SimulatorStatusMagic from the UITest target and keep the app target clean from 'Debug frameworks'?**

Yep.

But it's a little tricky. There are three players involved when running UITests; the UITest bundle, the app itself, and an Apple generated test runner app.

#### The UITest Bundle

The test bundle doesn't actually run - it merely contains the tests for the test runner to perform on the app itself.

However, it does need to be compiled against the framework, so update the UITest target build settings: `Framework Search Paths: $(SRCROOT)/ThirdPartyFrameworks/`

#### The App Itself

The app itself does not require the framework to compile. It does require the framework to run, because it's performing a UITest which calls SimulatorStatusMagic.

In the UITest target, add a custom Run Script build phase to embed the framework into the app (do insert the red underlined name):

![Configured run script build phase for main app]({{ site.url }}/assets/embed-in-main-app.png)

#### The Generated Test Runner

The test runner also requires the framework to run.

In the UITest target, add a custom Run Script built phase to embed the framework in the test runner (do insert the red underlined name):

![Configured run script build phase for test runner]({{ site.url }}/assets/embed-in-test-runner.png)

-----

**Comments? [Tweet me](https://twitter.com/kentios).**
