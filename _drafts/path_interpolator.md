---
layout: post
title: An Intro to PathInterpolatorCompat
author: Stuart Kent
tags: android, interpolation
summary: This is a single paragraph describing the post.

---

Version 22.1 of the v4 support library added [several new Interpolator classes](http://android-developers.blogspot.com/2015/04/android-support-library-221.html) to help developers infuse their applications with [Authentic Motion](http://www.google.com/design/spec/animation/authentic-motion.html). Today, we'll begin exploring the highly-flexible [`PathInterpolatorCompat`](http://developer.android.com/reference/android/support/v4/view/animation/PathInterpolatorCompat.html) class.

As the name suggests, `PathInterpolatorCompat` is a utility for creating [`Path`](http://developer.android.com/reference/android/graphics/Path.html)-based interpolators. If it's been a while since you worked with a `Path`, here's the snappy definition from the docs:

> The Path class encapsulates compound (multiple contour) geometric paths consisting of straight line segments, quadratic curves, and cubic curves.

In plainer English:

> A Path is a collection of (not-necessarily-connected) straight lines and curves.

### Interpolators &harr; Functions

Peeking at the (abbreviated) source code of the `TimeInterpolator` interface below, we see that an Android interpolator is nothing more than a function that maps the closed interval $[0,1]$ to the real numbers $\mathbb{R}$:[^1]

{% highlight java %}
public interface TimeInterpolator {

    /**
     * [...]
     *
     * @param input A value between 0 and 1.0 indicating our current point
     *        in the animation where 0 represents the start and 1.0
     *        represents the end
     * @return The interpolation value. This value can be more than 1.0
     *         for interpolators which overshoot their targets, or less
     *         than 0 for interpolators that undershoot their targets.
     */
    float getInterpolation(float input);
    
}
{% endhighlight %}

In general, a function can be represented in one of three ways:

* algebraically: $f(x) = x^2$;
* numerically: 
* graphically:

Each representation 

### Algebraically-defined interpolators

`AccelerateInterpolator`:

{% highlight java %}
public float getInterpolation(float input) {
    if (mFactor == 1.0f) {
        return input * input;
    } else {
        return (float)Math.pow(input, mDoubleFactor);
    }
}
{% endhighlight %}

[^1]:The documentation for the non-compat [`PathInterpolator`](https://developer.android.com/reference/android/view/animation/PathInterpolator.html) confirms this equivalence, albeit in a much more verbose manner.