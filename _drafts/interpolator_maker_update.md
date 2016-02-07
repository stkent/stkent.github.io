---
layout: post
title: Upgrading Interpolator Maker with `PathInterpolatorCompat`
author: Stuart Kent
tags: android, animation
summary:

---

Back in February, I released the first version of Interpolator Maker. It looked like this:
![]({{ site.baseurl }}/assets/images/apps/interpolator_maker.gif)


# v1 Drawbacks:

* Constructing a function f(x) - in your head - such that ... represented a valid interpolator is quite difficult.
* Feedback was limited (mostly because of the 1-week development time)
* Input slow, fiddly, editing painful, intermediate feedback poor
* Exp4j missing some important features (no pi)

# Can we do better?[^1]

(attribute to Tim Roughgarden)

* Do developers really care about the expression powering an interpolator? How many developers have looked closely at the source code for the existing interpolators? (Few, I'll wager).
* Can we restrict input to allow valid functions only, negating the need for a feedback loop? Partially achieved in v1 (input must represent a function) but far from perfect. `test`

# Bezier Paths

[^1]: Sure can