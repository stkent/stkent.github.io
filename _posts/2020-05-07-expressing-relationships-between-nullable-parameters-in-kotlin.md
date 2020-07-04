---
title: Expressing Relationships Between Nullable Parameters In Kotlin
author: Stuart Kent
tags: android, kotlin

---

Every time we declare a pair of nullable parameters:

```kotlin
fun plotOnMap(
  location: LatLng?,
  accuracyMeters: Double?
)
```

we allow 4 different combinations of parameter values:

<table>
  <tr>
    <td colspan="2" rowspan="2"></td>
    <th colspan="2" style="text-align:center">
      <code>accuracyMeters</code>
    </th>
  </tr>
  <tr>
    <td style="text-align:center">
      <code>null</code>
    </td>
    <td style="text-align:center">
      Non-<code>null</code>
    </td>
  </tr>
  <tr>
    <th rowspan="2" style="text-align:right">
      <code>location</code>
    </th>
    <td style="text-align:right">
      <code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">✅</td>
  </tr>
  <tr>
    <td style="text-align:right">
      Non-<code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">✅</td>
  </tr>
</table>

Sometimes we know extra information about these parameters that makes one or more of these combinations impossible. We can write working code that accepts and ignores these invalid combinations[^1], but that code will be prone to accidental misuse and misinterpretation in the future. This post demonstrates how to use Kotlin’s type system to eliminate these risks.

# Example 1: "Either both parameters are non-null or both parameters are null"

**Original function**

```kotlin
fun plotOnMap(
  location: LatLng?,
  accuracyMeters: Double?
)
```

**Function context**

We’re plotting a user’s location on a map. If we know the user’s location, we must _always_ include a circle representing location accuracy. If we don’t know the user’s location, we plot nothing.

**Valid parameter combinations**

2 out of 4 combinations of parameter values are valid:

<table>
  <tr>
    <td colspan="2" rowspan="2"></td>
    <th colspan="2" style="text-align:center">
      <code>accuracyMeters</code>
    </th>
  </tr>
  <tr>
    <td style="text-align:center">
      <code>null</code>
    </td>
    <td style="text-align:center">
      Non-<code>null</code>
    </td>
  </tr>
  <tr>
    <th rowspan="2" style="text-align:right">
      <code>location</code>
    </th>
    <td style="text-align:right">
      <code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">🚫</td>
  </tr>
  <tr>
    <td style="text-align:right">
      Non-<code>null</code>
    </td>
    <td style="text-align:center">🚫</td>
    <td style="text-align:center">✅</td>
  </tr>
</table>

**Improved function**

We can express this relationship in our code by introducing a wrapper type:

```kotlin
data class LocationData(
  val location: LatLng,
  val accuracyMeters: Double
)
```

and updating our function signature to receive a nullable instance of that type:

```kotlin
fun plotOnMap(locationData: LocationData?)
```

# Example 2: "If parameter one is null, so is parameter two"

This is an asymmetric version of [example 1](#example-1-either-both-parameters-are-non-null-or-both-parameters-are-null).

**Original function**

```kotlin
fun plotOnMap(
  location: LatLng?,
  accuracyMeters: Double?
)
```

**Function context**

We’re plotting a user’s location on a map. If we know the user’s location, we include a circle representing location accuracy _if that accuracy is available_. If we don’t know the user’s location, we plot nothing.

**Valid parameter combinations**

3 out of 4 combinations of parameter values are valid:

<table>
  <tr>
    <td colspan="2" rowspan="2"></td>
    <th colspan="2" style="text-align:center">
      <code>accuracyMeters</code>
    </th>
  </tr>
  <tr>
    <td style="text-align:center">
      <code>null</code>
    </td>
    <td style="text-align:center">
      Non-<code>null</code>
    </td>
  </tr>
  <tr>
    <th rowspan="2" style="text-align:right">
      <code>location</code>
    </th>
    <td style="text-align:right">
      <code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">🚫</td>
  </tr>
  <tr>
    <td style="text-align:right">
      Non-<code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">✅</td>
  </tr>
</table>

**Improved function**

We can express this relationship in our code by introducing a wrapper type (note that `accuracyMeters` is nullable this time):

```kotlin
data class LocationData(
  val location: LatLng,
  val accuracyMeters: Double?
)
```

and updating our function signature to receive a nullable instance of that type:

```kotlin
fun plotOnMap(locationData: LocationData?)
```

# Example 3: "Exactly one parameter must be non-null"

**Original function**

```kotlin
data class NewCreditCard(/*...*/)
data class SavedCreditCard(/*...*/)

fun chargeOrder(
  newCreditCard: NewCreditCard?,
  savedCreditCard: SavedCreditCard?
)
```

**Function context**

We’re placing an online order. An order should be charged to exactly one credit card.

**Valid parameter combinations**

2 out of 4 combinations of parameter values are valid:

<table>
  <tr>
    <td colspan="2" rowspan="2"></td>
    <th colspan="2" style="text-align:center">
      <code>savedCreditCard</code>
    </th>
  </tr>
  <tr>
    <td style="text-align:center">
      <code>null</code>
    </td>
    <td style="text-align:center">
      Non-<code>null</code>
    </td>
  </tr>
  <tr>
    <th rowspan="2" style="text-align:right">
      <code>newCreditCard</code>
    </th>
    <td style="text-align:right">
      <code>null</code>
    </td>
    <td style="text-align:center">🚫</td>
    <td style="text-align:center">✅</td>
  </tr>
  <tr>
    <td style="text-align:right">
      Non-<code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">🚫</td>
  </tr>
</table>

**Improved function**

We can express this relationship in our code by introducing a sealed type:

```kotlin
sealed class PaymentMethod {
  data class NewCreditCard(/*...*/) : PaymentMethod()
  data class SavedCreditCard(/*...*/) : PaymentMethod()
}
```

and updating our function signature to receive a non-`null` instance of that sealed type:

```kotlin
fun chargeOrder(paymentMethod: PaymentMethod)
```

In this example we’ve managed to remove nullability entirely!

# Example 4: "At most one parameter is non-null"

**Original function**

```kotlin
data class NewBankAccount(/*...*/)
data class NewCreditCard(/*...*/)

fun chargeOrder(
  newBankAccount: NewBankAccount?,
  newCreditCard: NewCreditCard?
)
```

**Function context**

We’re placing an online order. An order can be charged to a new bank account or a new credit card. If the user provides neither of these new payment methods, the order is charged to the user’s default saved payment method.

**Valid parameter combinations**

3 out of 4 combinations of parameter values are valid:

<table>
  <tr>
    <td colspan="2" rowspan="2"></td>
    <th colspan="2" style="text-align:center">
      <code>savedCreditCard</code>
    </th>
  </tr>
  <tr>
    <td style="text-align:center">
      <code>null</code>
    </td>
    <td style="text-align:center">
      Non-<code>null</code>
    </td>
  </tr>
  <tr>
    <th rowspan="2" style="text-align:right">
      <code>newCreditCard</code>
    </th>
    <td style="text-align:right">
      <code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">✅</td>
  </tr>
  <tr>
    <td style="text-align:right">
      Non-<code>null</code>
    </td>
    <td style="text-align:center">✅</td>
    <td style="text-align:center">🚫</td>
  </tr>
</table>

**Improved function**

We can express this relationship in our code by introducing a sealed type as in [example 3](#example-3-exactly-one-parameter-must-be-non-null):

```kotlin
sealed class NewPaymentMethod {
  data class NewBankAccount(/*...*/) : NewPaymentMethod()
  data class NewCreditCard(/*...*/) : NewPaymentMethod()
}
```

and updating our function signature to receive a `nullable` instance of that sealed type:

```kotlin
fun chargeOrder(newPaymentMethod: NewPaymentMethod?)
```

# Summary

Use Kotlin’s type system to make illegal parameter combinations impossible. You'll thank yourself later.

[^1]: Typically such code will deliberately include no-ops or theoretically throw a `NullPointerException`.
