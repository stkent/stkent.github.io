---
layout: post
title: Building Smooth Paths using B&eacute;zier Curves
author: Stuart Kent
tags: android, animation, math
summary:
---

[Last post]({% post_url 2015-06-07-an-intro-to-pathinterpolatorcompat %}), we built a super-simple `Path`-based interpolator using straight line segments. To produce smoother interpolators (typically preferred for animating motion), we'll need correspondingly smooth generating Paths.

To avoid today's post getting too long, I've chosen to focus on calculating smooth Paths only. We'll relate our results back to `Path`-based interpolators in the next post. So, our goal is simply the following:

* given a sequence of points in the cartesian plane, calculate a smooth `Path` passing through all points in order.

### Prerequisites

Our Paths will be constructed by patching together multiple cubic B&eacute;zier curves. This amounts to calling `Path.cubicTo(...)` multiple times with appropriate knots (our input sequence) and control points (chosen to make our composite curve smooth at the joins). I'm not going to be covering B&eacute;zier curve basics; if you need a refresher, I recommend the following resources:

* [Wikipedia article on B&eacute;zier curves](https://en.wikipedia.org/wiki/B%C3%A9zier_curve)
* [Wikipedia article on B&eacute;zier curves [PDF]](https://en.wikipedia.org/wiki/B%C3%A9zier_curve)
* [Wikipedia article on B&eacute;zier curves [PDF]](https://en.wikipedia.org/wiki/B%C3%A9zier_curve)

The derivation of the system of linear equations governing control point locations involves some linear algebra and one-dimensional differentiation.

### Notation

Let $\lbrace k_i \in \mathbb{R}^m : i \in 0,\ldots,n \rbrace$ represent a collection of $n+1$ _knots_.

Let $\Gamma_i$ represent any cubic B&eacute;zier curve connecting $k_i$ to $k_{i+1}$ for $i \in 0,\ldots,n-1$. Each $\Gamma_i$ may then be represented by a parametric equation of the form[^1]

$$ \Gamma_i(t) = (1-t)^3 k_i + 3(1-t)^2 t c_{i,0} + 3(1-t) t^2 c_{i,1} + t^3 k_{i+1} $$

where $t$ ranges between $0$ and $1$, and $c_{i,0} \in \mathbb{R}^m $ and $c_{i,1} \in \mathbb{R}^m $ are intermediate _control points_ that determine the curvature of $\Gamma_i$.

### Formal Goal

For any given collection of knots, we aim to compute control points that guarantee the composite curve $\Gamma$ formed by connecting all the individual B&eacute;zier curves satisfies the following conditions:

- $ \Gamma $ is twice-differentiable everywhere;

- $ \Gamma $ satisfies natural boundary conditions (i.e. $\Gamma'' = 0$ at each end).

Each $ \Gamma_i $ is clearly $ C^\infty $ away from the endpoints $ k_i $ and $ k_{i+1} $, so the first condition above is equivalent to requiring that $ \Gamma $ be twice-differentiable at every knot.

### Derivation

Note that

$$ \Gamma_i^{\prime}(t) = 3 \left[ - (1-t)^2 k_i + (3t-1)(t-1) c_{i,0} - t(3t-2) c_{i,1} + t^2 k_{i+1} \right] $$

and

$$ \Gamma_i^{\prime\prime}(t) = 6 \left[ (1-t) k_i + (3t-2) c_{i,0} - (3t-1) c_{i,1} + t k_{i+1} \right]. $$

For $\Gamma$ to be $C^2$ at each interior knot, we require that

$$ \left.\Gamma_{i-1}^{\prime}\right\vert_{k_{i}} = \left.\Gamma_{i}^{\prime}\right\vert_{k_{i}} \hspace{0.2in} \text{ and } \hspace{0.2in} \left.\Gamma_{i-1}^{\prime\prime}\right\vert_{k_{i}} = \left.\Gamma_{i}^{\prime\prime}\right\vert_{k_{i}} $$

for $ i \in \lbrace 1,\ldots,n-1 \rbrace $. Substituting the derivative expressions computed above, we see that these equalities are equivalent to choosing control points that satisfy

$$ c_{i-1,1} + c_{i,0} = 2k_{i} \text{ for } i \in \lbrace 1,\ldots,n-1 \rbrace $$

and

$$ c_{i-1,0} - 2c_{i-1,1} = c_{i,1} - 2c_{i,0} \text{ for } i \in \lbrace 1,\ldots,n-1 \rbrace. $$

So far, we have $2(n-1)$ constraints for $2n$ control points. The final constraints that will uniquely determine the locations of all control points are the boundary conditions

$$ \left.\Gamma_0^{\prime\prime}\right\vert_{k_0} = 0 \hspace{0.2in} \text{ and } \hspace{0.2in} \left.\Gamma_{n-1}^{\prime\prime}\right\vert_{k_n} = 0. $$

Equivalently,

$$ k_0 - 2c_{0,0} + c_{0,1} = 0 $$

and

$$ c_{n-1,0} - 2c_{n-1,1} + k_n = 0. $$

Eliminating $c_{i,1}$ from all these equations gives a system of $n$ equations for $ \lbrace c_{i,0} : i \in 0,\ldots,n-1 \rbrace $:

$$ c_{i-1,0} + 4 c_{i,0} + c_{i+1,0} = 2(2k_{i} + k_{i+1}) \text{ for } i \in \lbrace 1,\ldots,n-2 \rbrace, $$

$$ 2c_{0,0} + c_{1,0} = k_0 + 2k_1, $$

$$ 2c_{n-2,0} + 7c_{n-1,0} = 8 k_{n-1} + k_n $$

Writing these equations in matrix form:

$$
\begin{bmatrix}
    2 & 1 & 0 & 0 & 0 & \dots & 0 \\
    1 & 4 & 1 & 0 & 0 & \dots & 0 \\
    0 & 1 & 4 & 1 & 0 & \dots & 0 \\
    \vdots & \ddots & \ddots & \ddots & \ddots & \ddots & \vdots \\
    0 & \dots & 0 & 1 & 4 & 1 & 0 \\
    0 & \dots & 0 & 0 & 1 & 4 & 1 \\
    0 & \dots & 0 & 0 & 0 & 2 & 7
\end{bmatrix}
\begin{bmatrix}
    c_{0,0} \\
    c_{1,0} \\
    c_{2,0} \\
    \vdots \\
    c_{n-3,0} \\
    c_{n-2,0} \\
    c_{n-1,0}
\end{bmatrix} =
\begin{bmatrix}
    k_0 + 2k_1 \\
    2(2k_{1} + k_{2}) \\
    2(2k_{2} + k_{3}) \\
    \vdots \\
    2(2k_{n-3} + k_{n-2}) \\
    2(2k_{n-2} + k_{n-1}) \\
    8k_{n-1} + k_{n}
\end{bmatrix}
$$

This tridiagonal system can be solved in linear time using [Thomas' Algorithm](http://en.wikipedia.org/wiki/Tridiagonal_matrix_algorithm), which in this case is guaranteed to be stable since the tridiagonal matrix is diagonally dominant. Once all $c_{i,0}$ are calculated, the remaining control points $\lbrace c_{i,1} : i \in 0,\ldots,n-1 \rbrace$ are given by the following formulae:

$$ c_{i,1} = 2k_{i+1} - c_{i+1,0} \text{ for } i \in \lbrace 0,\ldots,n-2 \rbrace, $$

$$ c_{n-1,1} = \frac{1}{2}\left[ k_n + c_{n-1,0} \right]. $$

### Implementation

The following Android/Java code uses Thomas' Algorithm to compute appropriate control points and accomplish our original goal:

> given a sequence of points in the cartesian plane, calculate a smooth `Path` passing through all points in order.

I use an enhanced point class `EPointF` that provides some [componentwise operations](http://en.wikipedia.org/wiki/Pointwise#Componentwise_operations) for convenience. Note that the code was written with readability, rather than performance, in mind.

{% highlight java %}
package com.example;

/**
 * API inspired by the Apache Commons Math Vector2D class.
 */
public class EPointF {

  private final float x;
  private final float y;

  public EPointF(final float x, final float y) {
    this.x = x;
    this.y = y;
  }

  public float getX() {
    return x;
  }

  public float getY() {
    return y;
  }

  public EPointF plus(float factor, EPointF ePointF) {
    return new EPointF(x + factor * ePointF.x, y + factor * ePointF.y);
  }

  public EPointF plus(EPointF ePointF) {
    return plus(1.0f, ePointF);
  }

  public EPointF minus(float factor, EPointF ePointF) {
    return new EPointF(x - factor * ePointF.x, y - factor * ePointF.y);
  }

  public EPointF minus(EPointF ePointF) {
    return minus(1.0f, ePointF);
  }

  public EPointF scaleBy(float factor) {
    return new EPointF(factor * x, factor * y);
  }

}
{% endhighlight %}

{% highlight java %}
package com.example;

import android.graphics.Path;

import com.example.EPointF;

import java.util.Collection;
import java.util.List;

public class PolyBezierPathUtil {

  /**
   * Computes a Poly-Bezier curve passing through a given list of knots.
   * The curve will be twice-differentiable everywhere and satisfy natural
   * boundary conditions at both ends.
   *
   * @param knots a list of knots
   * @return      a Path representing the twice-differentiable curve
   *              passing through all the given knots
   */
  public Path computePathThroughKnots(List<EPointF> knots) {
    throwExceptionIfInputIsInvalid(knots);

    final Path polyBezierPath = new Path();
    final EPointF firstKnot = knots.get(0);
    polyBezierPath.moveTo(firstKnot.getX(), firstKnot.getY());

    /*
     * variable representing the number of Bezier curves we will patch
     * together
     */
    final int n = knots.size() - 1;

    if (n == 1) {
      final EPointF lastKnot = knots.get(1);
      polyBezierPath.lineTo(lastKnot.getX(), lastKnot.getY());
    } else {
      final EPointF[] controlPoints = computeControlPoints(n, knots);

      for (int i = 0; i < n; i++) {
        final EPointF targetKnot = knots.get(i + 1);
        appendCurveToPath(polyBezierPath, controlPoints[i], controlPoints[n + i], targetKnot);
      }
    }

    return polyBezierPath;
  }

  private EPointF[] computeControlPoints(int n, List<EPointF> knots) {
    final EPointF[] result = new EPointF[2 * n];

    final EPointF[] target = constructTargetVector(n, knots);
    final Float[] lowerDiag = constructLowerDiagonalVector(n - 1);
    final Float[] mainDiag = constructMainDiagonalVector(n);
    final Float[] upperDiag = constructUpperDiagonalVector(n - 1);

    final EPointF[] newTarget = new EPointF[n];
    final Float[] newUpperDiag = new Float[n - 1];

    // forward sweep for first control points c_i,0:
    newUpperDiag[0] = upperDiag[0] / mainDiag[0];
    newTarget[0] = target[0].scaleBy(1 / mainDiag[0]);

    for (int i = 1; i < n - 1; i++) {
      newUpperDiag[i] = upperDiag[i] /
          (mainDiag[i] - lowerDiag[i - 1] * newUpperDiag[i - 1]);
    }

    for (int i = 1; i < n; i++) {
      final float targetScale = 1 /
          (mainDiag[i] - lowerDiag[i - 1] * newUpperDiag[i - 1]);

      newTarget[i] =
          (target[i].minus(newTarget[i - 1].scaleBy(lowerDiag[i - 1]))).scaleBy(targetScale);
    }

    // backward sweep for first control points c_i,0:
    result[n - 1] = newTarget[n - 1];

    for (int i = n - 2; i >= 0; i--) {
      result[i] = newTarget[i].minus(newUpperDiag[i], result[i + 1]);
    }

    // calculate second control points c_i,1:
    for (int i = 0; i < n - 1; i++) {
      result[n + i] = knots.get(i + 1).scaleBy(2).minus(result[i + 1]);
    }

    result[2 * n - 1] = knots.get(n).plus(result[n - 1]).scaleBy(0.5f);

    return result;
  }

  private EPointF[] constructTargetVector(int n, List<EPointF> knots) {
    final EPointF[] result = new EPointF[n];

    result[0] = knots.get(0).plus(2, knots.get(1));

    for (int i = 1; i < n - 1; i++) {
      result[i] = (knots.get(i).scaleBy(2).plus(knots.get(i + 1))).scaleBy(2);
    }

    result[result.length - 1] = knots.get(n - 1).scaleBy(8).plus(knots.get(n));

    return result;
  }

  private Float[] constructLowerDiagonalVector(int length) {
    final Float[] result = new Float[length];

    for (int i = 0; i < result.length - 1; i++) {
      result[i] = 1f;
    }

    result[result.length - 1] = 2f;

    return result;
  }

  private Float[] constructMainDiagonalVector(int n) {
    final Float[] result = new Float[n];

    result[0] = 2f;

    for (int i = 1; i < result.length - 1; i++) {
      result[i] = 4f;
    }

    result[result.length - 1] = 7f;

    return result;
  }

  private Float[] constructUpperDiagonalVector(int length) {
    final Float[] result = new Float[length];

    for (int i = 0; i < result.length; i++) {
      result[i] = 1f;
    }

    return result;
  }

  private void appendCurveToPath(Path path, EPointF control1, EPointF control2, EPointF targetKnot) {
    path.cubicTo(
        control1.getX(),
        control1.getY(),
        control2.getX(),
        control2.getY(),
        targetKnot.getX(),
        targetKnot.getY()
    );
  }

  private void throwExceptionIfInputIsInvalid(Collection<EPointF> knots) {
    if (knots.size() < 2) {
      throw new IllegalArgumentException(
          "Collection must contain at least two knots"
      );
    }
  }

}
{% endhighlight %}

### Results

[^1]:The general (parametric) form of a cubic B&eacute;zier curve can be found in [the Wikipedia entry on B&eacute;zier Curves](http://en.wikipedia.org/wiki/B%C3%A9zier_curve#Cubic_B.C3.A9zier_curves).