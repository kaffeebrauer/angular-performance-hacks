# angular-performance-hacks
Some topics of discussion on how to improve your Angular Application v7.0 onwards

# Angular NG-Show v NG-If
You’ve probably come across ng-if and ng-show and wondered why they both exist and what’s the difference between them. After all they usually have the same behavior as far as the user is concerned.

The devil is in the details and the differences between these directives can allow your to boost your application’s performance easily.

The Differences
Both ng-show and ng-if receive a condition and hide from view the directive’s element in case the condition evaluates to false. The mechanics they use to hide the view, though, are different.

ng-show (and its sibling ng-hide) toggle the appearance of the element by adding the CSS display: none style.

ng-if, on the other hand, actually removes the element from the DOM when the condition is false and only adds the element back once the condition turns true.

Since ng-show leaves the elements alive in the DOM, it means that all of their watch expressions and performance cost are still there even though the user doesn’t see the view at all. In cases where you have a few big views that are toggled with ng-show you might be noticing that things are a bit laggy (like clicking on buttons or typing inside input fields).

If you just replace that ng-show with an ng-if you might witness considerable improvements in the responsiveness of your app because those extra watches are no longer happening.

That’s it: replace ng-show and ng-hide with ng-if!

Caveats and Pitfalls of ng-if
Measure, measure, measure: As with every optimization, you should not apply this without measuring and validating that it does speed up your app. It can, potentially, actually make things slower, as I explain below.
Your controllers will be rerun: The controllers and directives in the element that’s being removed and added again will actually be recreated and so their initialization logic will run again. This is in contrast to ng-show where things are always there in memory, and so are only initialized once. You need to make sure your code handles being rerun properly.
Sometimes initialization is more expensive than keeping things around: That’s to say that in some cases the cost of removing the element from the DOM and then recreating it to add again can be a heavy operation all by itself. In those cases you might feel that it takes too long for the element to reappear. In those cases this trick might actually degrade your app’s performance, so remember to check and measure!
That’s it. Enjoy making your app a bit snappier. If you care about performance in Angular you really should read about speeding ng-repeat with the track by syntax.
