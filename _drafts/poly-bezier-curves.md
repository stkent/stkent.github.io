---
layout: post
title: Building Twice-Differentiable Poly-B&eacute;zier Curves
author: Stuart Kent
tags: android, interpolation, math
summary: When patching together a finite number of cubic B&eacute;zier curves to form a single spline (subject to natural boundary conditions), the correct choice of control points will produce a composite curve that is everywhere twice-differentiable. In this post, we derive the system of equations that can be used to compute these control points.

---

Version 22.1 of the Support v4 library[^1]. The most flexible of these new classes is `PathInterpolatorCompat`. Using this class, we can convert (almost) any `Path` that connects the points (0,0) and (1,1) into an interpolator.

![Too many choices!](/path/to/img.jpg)

Since animators (and therefore interpolators) are often used to describe motion

### Notation

Let $\lbrace K_i \in \mathbb{R}^m : i \in 0,\ldots,n \rbrace$ represent a collection of $n+1$ _knots_.

Let $\Gamma_i$ represent any cubic B&eacute;zier curve connecting $K_i$ to $K_{i+1}$ for $i \in 0,\ldots,n-1$. Each $\Gamma_i$ may then be represented by a parametric equation of the form[^2]

$$ \Gamma_i(t) = (1-t)^3 K_i + 3(1-t)^2 t P_{i,0} + 3(1-t) t^2 P_{i,1} + t^3 K_{i+1} $$

where $t$ ranges between $0$ and $1$, and $P_{i,0} \in \mathbb{R}^m $ and $P_{i,1} \in \mathbb{R}^m $ are intermediate _control points_ that determine the curvature (and presumably the torsion, if $m \geq 3$) of $\Gamma_i$.

### Goal

For any given collection of knots, we aim to compute control points that guarantee the composite curve $ \Gamma $ formed by connecting all the individual B&eacute;zier curves satisfies the following conditions:

- $ \Gamma $ is twice-differentiable everywhere;

- $ \Gamma $ satisfies natural boundary conditions (i.e. $\Gamma'' = 0$ at each end).

Each $ \Gamma_i $ is clearly $ C^\infty $ away from the endpoints $ K_i $ and $ K_{i+1} $, so the first condition above is equivalent to requiring that $ \Gamma $ be twice-differentiable at every knot.

// insert a note here about the fact that we've made a choice; we could relax c2 and instead improve localization of response; link to one of the pdfs here

### Derivation

Note that

$$ \Gamma_i^{\prime}(t) = 3 \left[ - (1-t)^2 K_i + (3t-1)(t-1) P_{i,0} - t(3t-2) P_{i,1} + t^2 K_{i+1} \right] $$

and

$$ \Gamma_i^{\prime\prime}(t) = 6 \left[ (1-t) K_i + (3t-2) P_{i,0} - (3t-1) P_{i,1} + t K_{i+1} \right]. $$

For $\Gamma$ to be $C^2$ at each interior knot, we require that

$$ \left.\Gamma_{i-1}^{\prime}\right\vert_{K_{i}} = \left.\Gamma_{i}^{\prime}\right\vert_{K_{i}} \hspace{0.2in} \text{ and } \hspace{0.2in} \left.\Gamma_{i-1}^{\prime\prime}\right\vert_{K_{i}} = \left.\Gamma_{i}^{\prime\prime}\right\vert_{K_{i}} $$

for $ i \in \lbrace 1,\ldots,n-1 \rbrace $. Substituting the derivative expressions computed above, we see that these equalities are equivalent to choosing control points that satisfy

$$ P_{i-1,1} + P_{i,0} = 2K_{i} \text{ for } i \in \lbrace 1,\ldots,n-1 \rbrace $$

and

$$ P_{i-1,0} - 2P_{i-1,1} = P_{i,1} - 2P_{i,0} \text{ for } i \in \lbrace 1,\ldots,n-1 \rbrace. $$

So far, we have $2(n-1)$ constraints for $2n$ control points. The final constraints that will uniquely determine the locations of all control points are the boundary conditions

$$ \left.\Gamma_0^{\prime\prime}\right\vert_{K_0} = 0 \hspace{0.2in} \text{ and } \hspace{0.2in} \left.\Gamma_{n-1}^{\prime\prime}\right\vert_{K_n} = 0. $$

Equivalently,

$$ K_0 - 2P_{0,0} + P_{0,1} = 0 $$

and

$$ P_{n-1,0} - 2P_{n-1,1} + K_n = 0. $$

Eliminating $P_{i,1}$ from all these equations gives a system of $n$ equations for $ \lbrace P_{i,0} : i \in 0,\ldots,n-1 \rbrace $:

$$ P_{i-1,0} + 4 P_{i,0} + P_{i+1,0} = 2(2K_{i} + K_{i+1}) \text{ for } i \in \lbrace 1,\ldots,n-2 \rbrace, $$

$$ 2P_{0,0} + P_{1,0} = K_0 + 2K_1, $$

$$ 2P_{n-2,0} + 7P_{n-1,0} = 8 K_{n-1} + K_n $$

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
    P_{0,0} \\
    P_{1,0} \\
    P_{2,0} \\
    \vdots \\
    P_{n-3,0} \\
    P_{n-2,0} \\
    P_{n-1,0}
\end{bmatrix} =
\begin{bmatrix}
    K_0 + 2K_1 \\
    2(2K_{1} + K_{2}) \\
    2(2K_{2} + K_{3}) \\
    \vdots \\
    2(2K_{n-3} + K_{n-2}) \\
    2(2K_{n-2} + K_{n-1}) \\
    8K_{n-1} + K_{n}
\end{bmatrix}
$$

This tridiagonal system can be solved in linear time using Thomas' Algorithm[^3], which in this case is guaranteed to be stable since the tridiagonal matrix is diagonally dominant. Once all $P_{i,0}$ are calculated, the remaining control points $\lbrace P_{i,1} : i \in 0,\ldots,n-1 \rbrace$ are given by the following formulae:

$$ P_{i,1} = 2K_{i+1} - P_{i+1,0} \text{ for } i \in \lbrace 0,\ldots,n-2 \rbrace, $$

$$ P_{n-1,1} = \frac{1}{2}\left[ K_n + P_{n-1,0} \right]. $$

### Implementation

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

    // forward sweep for first control points P_i,0:
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

    // backward sweep for first control points P_i,0:
    result[n - 1] = newTarget[n - 1];

    for (int i = n - 2; i >= 0; i--) {
      result[i] = newTarget[i].minus(newUpperDiag[i], result[i + 1]);
    }

    // calculate second control points P_i,1:
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

### Further reading

[^1]:http://android-developers.blogspot.com/2015/04/android-support-library-221.html

[^2]:The general (parametric) form of a cubic B&eacute;zier curve can be found in [the Wikipedia entry on B&eacute;zier Curves](http://en.wikipedia.org/wiki/B%C3%A9zier_curve#Cubic_B.C3.A9zier_curves).

[^3]:Thomas' algorithm is described in [the Wikipedia entry on the Tridiagonal Matrix Algorithm](http://en.wikipedia.org/wiki/Tridiagonal_matrix_algorithm).