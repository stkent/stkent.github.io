---
layout: post
title: Really Rounded Polygons
author: Stuart Kent
tags: android, animation, math
comments: true

---

Nick Butcher recently demonstrated a `Path`-and-`CornerPathEffect`-based method for drawing regular polygons with rounded corners. My library [PolygonDrawingUtil](https://github.com/stkent/PolygonDrawingUtil) solves the same problem but produces noticeably different results:

<a name="animation"></a> 

<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-comparison.gif" style="max-width: 400px" />
</div>

This inspired me to dig into how `CornerPathEffect` is implemented and compare it to the internals of PolygonDrawingUtil. Read on for commentary, code, and conclusions.

<!--more-->

# The Base Case

The light gray triangle of radius 500px in the [animation above](#animation) is the shape we will be rounding throughout this post. Here it is drawn a little darker for clarity:

<!-- Images for this post created using Pixel XL screenshots + convert -crop 1000x1050+40+275 -->
<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-base-shape.png" style="max-width: 400px" />
</div>

All subsequent discussion applies to any n-sided regular polygon.

# PolygonDrawingUtil

PolygonDrawingUtil dynamically generates and draws rounded polygon `Path`s based on user-specified attributes:

- polygon side count,
- polygon center coordinates,
- polygon radius (center to corner), and
- desired corner radius.

These `Path`s use exactly two components: straight lines for sides, and [circular arcs](https://en.wikipedia.org/wiki/Arc_(geometry)) for corners.

Below are some examples created using this method. The full circles used to create the rounded corner arcs are shown in light gray for illustration:

<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-polygondrawingutil-construction.png" />
</div>

This approach creates corners with exact and uniform radii. If the corner radius becomes too large relative to the polygon radius, the drawn shape gracefully degrades to a circle:

<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-polygondrawingutil-degenerate.png" style="max-width: 400px" />
</div>

The code that constructs these `Path`s is shown below. I'm not going to dissect it line by line, but the essential steps are:

1. Calculate the length of arc to use for each corner;
2. Calculate where each arc should be centered to ensure a smooth join with the sides;
3. Draw each arc in turn, using a [neat feature of `Path.arcTo`](https://developer.android.com/reference/android/graphics/Path.html#arcTo(android.graphics.RectF,%20float,%20float)) to automatically insert the sides:
    > If the start of the path is different from the path's current last point, then an automatic lineTo() is added to connect the current contour to the start of the arc.

{% highlight java %}
private void constructRoundedPolygonPath(
    int   sides,
    float centerX,
    float centerY,
    float polyRadius,
    float cornerRadius) {

  float arcSweep = 180.0 / sides;
  double arcCenterRadius = polyRadius - cornerRadius / sin(radians(90 - arcSweep));

  for (int nCorner = 0; nCorner < sides; nCorner++) {
    double cornerAngle = 360.0 * nCorner / sides;
    float arcCenterX = (float) (centerX + arcCenterRadius * cos(radians(cornerAngle)));
    float arcCenterY = (float) (centerY + arcCenterRadius * sin(radians(cornerAngle)));

    arcBounds.set(
        arcCenterX - cornerRadius,
        arcCenterY - cornerRadius,
        arcCenterX + cornerRadius,
        arcCenterY + cornerRadius);

    backingPath.arcTo(
        arcBounds,
        (float) (cornerAngle - 0.5 * arcSweep),
        arcSweep);
  }

  backingPath.close();
}
{% endhighlight %}

# CornerPathEffect

`PathEffect`s are used to modify how an existing `Path` is drawn. Applied via `Paint.setPathEffect`, they grant `Paint`s some "artistic license". For example, `CornerPathEffect` allows the `Paint` to draw rounded corners in place of any sharp corners. Users can specify a corner "radius" that influences the amount of rounding applied. Note that the `Path` itself is never altered by the application of a `PathEffect`.

`CornerPathEffect` draws rounded polygon `Path`s using two components: straight lines for sides, and quadratic B&eacute;zier curves for corners.

Below are some examples created using this method:

<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-cornerpatheffect-construction.png" />
</div>

This approach creates corners with nonuniform radii. Note too that the actual corner radius does not seem to match the specified corner radius if we interpret it as a value in pixels (this discrepancy is also illustrated by the earlier [animation](#animation) which shows that PolygonDrawingUtil corner roundness and `CornerPathEffect` corner roundness differ for the same input radius.)

If the corner radius becomes too large relative to the polygon radius, the drawn shape settles in this form:

<div class="image-container">
  <img src="/assets/images/really-rounded-polygons-cornerpatheffect-degenerate.png" style="max-width: 400px" />
</div>

As far as I can tell, there's no easy way to predict what this degenerate shape will look like ahead of time.

The relevant native code from [SkCornerPathEffect.cpp](https://android.googlesource.com/platform/external/skia/+/android-8.0.0_r4/src/effects/SkCornerPathEffect.cpp) is below[^2]. For each line in the original path, the essential steps are:

1. Calculate the control and end point locations for the corner curve;
2. Draw a quadratic B&eacute;zier curve using these points;
3. If necessary (i.e. the corner radius is small compared to the polygon radius), draw the remainder of the original straight side.

{% highlight c++ %}
bool SkCornerPathEffect::filterPath(SkPath* dst, const SkPath& src, SkStrokeRec*, const SkRect*) const {
  SkPath::Iter    iter(src, false);
  SkPath::Verb    verb;
  SkPoint         pts[4];
  SkVector        step;

  for (;;) {
    switch (verb = iter.next(pts, false)) {
      case SkPath::kLine_Verb: {
        bool drawSegment = ComputeStep(pts[0], pts[1], fRadius, &step);
                
        dst->quadTo(pts[0].fX, pts[0].fY, pts[0].fX + step.fX, pts[0].fY + step.fY);

        if (drawSegment) {
          dst->lineTo(pts[1].fX - step.fX, pts[1].fY - step.fY);
        }

        break;
      }

      // Other verb cases omitted.
    }
  }

  // Other iteration implementation omitted.
}

static bool ComputeStep(const SkPoint& a, const SkPoint& b, SkScalar radius, SkPoint* step) {
  SkScalar dist = SkPoint::Distance(a, b);

  *step = b - a;

  if (dist <= radius * 2) {
    *step *= SK_ScalarHalf;
    return false;
  } else {
    *step *= radius / dist;
    return true;
  }
}
{% endhighlight %}

# Usage

The implementation differences highlighted above are interesting but perhaps still a little abstract. Let's get real. Which rounding method should you use?

My recommendation would be to consider PolygonDrawingUtil if:

- you are drawing regular polygons, and
- you prefer or need exactly controllable corner radius[^1], or
- you prefer or need uniform corner radius, or
- you prefer or need a simple, predictable degenerate shape.

On the other hand, consider `CornerPathEffect` if:

- you are rounding an existing `Path` that's not a regular polygon, or
- you don't care about exact corner shape or radius, or
- you prefer to or need to avoid third-party dependencies.

Either way, I hope you learned a little about `Path`s, `PathEffect`s, and geometry during this exploration :)

[^1]: Trivia: PolygonDrawingUtil was originally inspired by games based on hexagonal grids, which is why it's polygon-specific and allows such precise corner control!
[^2]: More useful native code if you want to go deeper: [SkPath.h](https://android.googlesource.com/platform/external/skia/+/android-8.0.0_r4/include/core/SkPath.h), [SkPath.cpp](https://android.googlesource.com/platform/external/skia/+/android-8.0.0_r4/src/core/SkPath.cpp), [SkPathEffect.h](https://android.googlesource.com/platform/external/skia/+/android-8.0.0_r4/include/core/SkPathEffect.h), [SkPathEffect.cpp](https://android.googlesource.com/platform/external/skia/+/android-8.0.0_r4/src/core/SkPathEffect.cpp).
