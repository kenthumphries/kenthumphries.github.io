General advice for when and how to hide features behind a toggle.

In an ideal world every feature we write is bite sized and shippable. When development is complete, we get our little featureling a ticket on the next release train and we move on. We are content that our hard work will soon be in the hands of happy users.

Alas, the real world is not ideal. There are epics and rewrites and features with creepin scopes. There are releases that simply **must go out** and cannot wait for all features to be completed. At this point there are two common options:

- a long lived feature (or epic) branch 
- a toggle that enables the feature

In my experience **long lived feature branches** tend to get muddy. Someone adds a terrific utility that everyone wants on `develop` and after a cherry pick or two our branches become entangled.

A **toggle** makes life simpler. We can merge everything onto `develop` and share the benefits. The risk is that features are not properly toggled and incomplete work slips into the hands of our users.

Toggle Time
-----------
How can we make sure that toggles are safe and effective? Here are some tips.

1. **Aim for one toggle**  
A toggle that enables/disables all unfinished features reduces possible test states. *Wait did you test with feature A on, but feature B off?*

1. **Make the toggle OFF by default**  
Toggling unfinished features on should always be a conscious act.

1. **Disable the toggle in release builds**  
Use `#ifdef DEBUG` or your own compiler flag so that the toggle is *not even compiled* into a release.

1. **Control the toggle from a single point**  
A well architected toggle should enable/disable a feature as early as possible so that code is not riddled with `if (featureEnabled)` statements.

1. **Don't let the toggle become part of the furniture**  
Make the toggle control stand out!! Make it hurt your eyeballs!!  
If the toggle is hidden under four layers of submenus*, or nestled in amongst regular controls, it may sneak onto that release train unnoticed.

1. **Backup the toggle with a remote kill-switch**  
The above points all relate to development toggles.  
Add an extra layer of security by adding a remote toggle that can disable or enable a feature without changing the code.

_*If your app has four layers of submenus, you have bigger problems than toggles..._

Remote Toggling
---------------

A remote toggle can be a value returned in a server response, eg `supportsFeatureX, or something enabled using an A/B testing tool. These remote toggles allow us to perform a staged roll out, or roll back a feature that is not working as expected out in the wild.

A development toggle and a remote toggle can work well together. Consider the point in the code where the remote toggle is introduced. For a server response, this is likely the parsing code. For an A/B tool this is likely some kind of if statement. Use this point for the development toggle also.


Think of the development toggle as a hack that allows us to force or circumvent the remote toggle: 

- When the development toggle is disabled (or removed), the remote toggle is solely responsible for enabling the new feature
- When the development toggle is enabled, the remote toggle is ignored, and the new feature is enabled

-----

**Feeling passionate about toggles? [Tweet me](https://twitter.com/kentios).**
