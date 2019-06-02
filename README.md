# angular-performance-hacks
Some topics of discussion on how to improve your Angular Application v7.0 onwards

# Angular NG-Show v NG-If
You’ve probably come across **ng-if** and **ng-show** and wondered why they both exist and what’s the difference between them. After all they usually have the same behavior as far as the user is concerned.

The devil is in the details and the differences between these directives can allow your to boost your application’s performance easily.

The Differences
Both **ng-show** and **ng-if** receive a condition and hide from view the directive’s element in case the condition evaluates to **false**. The mechanics they use to hide the view, though, are different.

**ng-show** (and its sibling **ng-hide**) toggle the appearance of the element by adding the CSS display: none style.

**ng-if**, on the other hand, actually **removes** the element from the DOM when the condition is **false** and only adds the element back once the condition turns **true**.

Since **ng-show** leaves the elements alive in the DOM, it means that all of their watch expressions and performance cost are still there even though the user doesn’t see the view at all. In cases where you have a few big views that are toggled with **ng-show** you might be noticing that things are a bit laggy (like clicking on buttons or typing inside input fields).

If you just replace that ng-show with an ng-if you might witness considerable improvements in the responsiveness of your app because those extra watches are no longer happening.

**That’s it**: replace ng-show and ng-hide with ng-if!

Caveats and Pitfalls of ng-if
**Measure, measure, measure:** As with every optimization, you should not apply this without measuring and validating that it does speed up your app. It can, potentially, actually make things slower, as I explain below.
**Your controllers will be rerun:** The controllers and directives in the element that’s being removed and added again will actually be recreated and so their initialization logic will run again. This is in contrast to ng-show where things are always there in memory, and so are only initialized once. You need to make sure your code handles being rerun properly.

- [ ] Do example on stackblitz

**Sometimes initialization is more expensive than keeping things around**: That’s to say that in some cases the cost of removing the element from the DOM and then recreating it to add again can be a heavy operation all by itself. In those cases you might feel that it takes too long for the element to reappear. In those cases this trick might actually degrade your app’s performance, so remember to check and measure!

That’s it. Enjoy making your app a bit snappier. If you care about performance in Angular you really should read about speeding ng-repeat with the track by syntax.

# DOM - Change Detection #

In order to guarantee that the DOM always shows the very latest available data, Angular monkey-patches every entry point to running Javascript with a Zone. So any time Javascript finishes executing—meaning an AJAX request completes, a click event handler runs, or a Promise is fulfilled—Angular 2 checks whether any changes have occurred that would affect the DOM.

When the change detector runs on an Angular 2 component or directive, it evaluates every Javascript expression in that component’s view template (or the “host” entry on the @Component or @Directive decorator settings). It then compares the output of each expression to its previous output. If the result of an expression has changed, Angular 2 updates the DOM accordingly.

As the application grows, things may start to lag a bit. If you have drag & drop in your interface, you may find that you’re no longer getting silky-smooth 60FPS updates as you drag elements around. Eventually you’ll run a profiler on your code to see what’s slow, and you’ll find something that looks like this:

- [ ] TODO add diagram of how the change detection stuff actual works in cycle

[Change Detection Mess](https://1drv.ms/u/s!AstPH6dB0Clt6ETlDq29YwlFBJ4-)

- Have less DOM. This is a critically important piece of the puzzle. If DOM elements aren’t visible, you should remove them from the DOM by using *ngIf instead of simply hiding elements with CSS. As the saying goes, the fastest code is code that is not run—and the fastest DOM is DOM that doesn’t exist.
- Make your expressions faster. Move complex calculations into the ngDoCheck lifecycle hook, and refer to the calculated value in your view. Cache results to complex calculations as long as possible.
-Use the OnPush change detection strategy to tell Angular there have been no changes. This lets you skip the entire change detection step on most of your application most of the time.

** USE ONPUSH CHANGE DETECTION STRATEGY **

By default, the change detection strategy on any component or directive is CheckAlways. There is another strategy, OnPush, which can be much more efficient if you build your application carefully.

OnPush means that the change detector will only run on a component or directive if one of the following occurs:

- An input property has changed to a new value
- An event handler fired in the component or directive
- You manually tell the change detector to look for changes
- The change detector of a child component or directive runs

If you’re able to use OnPush throughout your application, you will ideally only ever run the change detector on components that have actually changed (and their direct ancestors). This reduces the time complexity of the change detector from O(n) to O(log n) in the number of component instances in your application.

**Method 1: Only use immutable inputs**
When you use OnPush, the change detector will run whenever an input property changes. !!!However, if you pass an object or array as an input to a component (which is very common), the change detector will not notice if something in that object or array changes. It only detects when you change to a different object entirely!!!.

The simplest way to make OnPush work perfectly is to use immutable objects throughout a component. If you change an object you are passing to a component, don’t change its properties in place; rather, construct a copy with the change applied.

When the component is constructed, we inject our observable user, a change detector, and the zone. Whenever changes happen on our user, we mark the change detector as needing to be checked within the Angular 2 zone. It’s important to do this if your change callback can come from outside the Angular 2 zone, which does happen regularly in our application.

If the user is passed into the component as a property, we have to watch for changes with the ngOnChanges lifecycle hook, and unbind and rebind changes appropriately.
```
@Component({selector: 'my-cmp', template: `...`})
class MyComponent implements OnChanges {
  // TODO(issue/24571): remove '!'.
  @Input()
  prop !: number;

  ngOnChanges(changes: SimpleChanges) {
    // changes.prop contains the old and the new value...
  }
}
```
