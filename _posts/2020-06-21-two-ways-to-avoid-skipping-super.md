---
title: Two Ways To Avoid Skipping Super
author: Stuart Kent
tags: android

---

{% include kramdown_definitions.md %}

Most Android apps that I've worked on have included a `BaseFragment` that contains universal logic. This logic is often written within one or more lifecycle methods, creating an implicit contract: subclasses **must** call through to `super` if they override any of these lifecycle methods[^1].

We can make this contract harder to accidentally break in a couple of different ways.

# The @CallSuper Annotation

The [`@CallSuper`](https://developer.android.com/reference/kotlin/androidx/annotation/CallSuper){:new_tab} annotation is one of many [Android Support Annotations](https://developer.android.com/studio/write/annotations){:new_tab}. When added to a method declaration, it signals that any overriding implementation should include a call to `super`:

```kotlin
// in BaseFragment:

@CallSuper
open override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    runUniversalLogic()
}
```

If an overriding implementation fails to call `super`:
 
```kotlin
// in HomeFragment, extends BaseFragment:

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    runCustomLogic()
}
```

then Android Studio will show an inline warning:

<div class="image-container">
  <img
    src="/assets/images/two-ways-to-avoid-skipping-super-android-studio-warning-opt.png"
    alt="A screenshot of Android Studio showing the warning 'Overriding method should call super.onViewCreated'."
  />
</div>

This warning **will not** prevent you from compiling and running your application. Whether or not this is a good thing depends on your build process and personal preferences.

This warning **will** be flagged as a lint error with priority 9/10:

<div class="image-container">
  <img
    src="/assets/images/two-ways-to-avoid-skipping-super-lint-error-opt.png"
    alt="A screenshot of an Android lint report including an error of type MissingSuperCall."
  />
</div>

Android Support Annotations are used extensively within Android's own AndroidX libraries, so they are probably already available for you to use in your own project (no new dependency needed).

# The Template Method Pattern

The [template method pattern](https://en.wikipedia.org/wiki/Template_method_pattern){:new_tab} allows us to enforce our implicit contract even more strongly. In this pattern, the method containing universal logic is marked `final` so that it can no longer be overridden by subclasses. Calls to one or more empty "template methods" are then inserted into the `final` method for subclasses to override instead. This allows subclasses to add to (but never skip or erase) superclass logic:

```kotlin
// in BaseFragment:

@CallSuper
final override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    runUniversalLogic()
    afterViewCreated(view, savedInstanceState)
}

open fun afterViewCreated(view: View, savedInstanceState: Bundle?) {
    // Template method, intentionally left blank.
}
```

```kotlin
// in HomeFragment, extends BaseFragment:

override fun afterViewCreated(view: View, savedInstanceState: Bundle?) {
    runCustomLogic()
}
```

Attempting to override `BaseFragment::onViewCreated` directly will cause your application to fail to compile. The price paid for this safety is the reduced discoverability of the template method. Folks are very familiar with the names of built-in lifecycle methods and are used to overriding them; `afterViewCreated` is not a standard lifecycle method name and it may take a few minutes for others to locate it and understand its purpose. This friction can be reduced by choosing template method naming conventions and applying them consistently across your codebase.

# Summary

It is a good idea to make implicit contracts explicit. This post describe two approaches for making "must call `super`" contracts more explicit in Android codebases. Picking either is safer than picking neither. Talk with your team to determine which best suits your needs.

[^1]: This type of contract can also exist between non-lifecycle methods and between non-framework classes. The approaches in this post apply to these situations too.