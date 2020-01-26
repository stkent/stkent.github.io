---
title: A Map Marker Tapping Bug In Google Maps Android SDK v2
author: Stuart Kent
tags: android

---

Google Maps Android SDK v2 includes [default logic to help users tap through groups of overlapping markers](https://developers.google.com/maps/documentation/android-sdk/marker#marker_click_events):

> Clicking on a cluster of markers causes subsequent clicks to cycle through the cluster, selecting each in turn.

Unfortunately, this default logic contains a bug in version 17.0.0. This bug occurs whenever your first tap on a cluster of markers causes the map camera to move by a non-trivial distance. The effect is that your `OnMarkerClickListener` is called **twice** for the rearmost marker before being correctly called once each for all other markers in the group. This is confusing for users, who are led to believe their second tap was inaccurate or ignored.

<!--more-->

Below is a video demonstrating the bug using visible taps and snackbars. Note that the red marker receives two taps before the green marker receives its first tap:

<div class="image-container">
  <img src="/assets/images/a-map-marker-tapping-bug-in-google-maps-android-sdk-v2-v2-bug.gif" width="50%" />
</div>

The bug occurs regardless of whether the map camera animation is triggered by the default `OnMarkerClickListener` behavior:

{% highlight kotlin %}
googleMap.setOnMarkerClickListener { marker ->
  Snackbar.make(rootView, "${marker.tag} marker tapped", LENGTH_SHORT).show()

  false // Allow default animation to occur.
}
{% endhighlight %}

or by custom `OnMarkerClickListener` code:

{% highlight kotlin %}
googleMap.setOnMarkerClickListener { marker ->
  Snackbar.make(rootView, "${marker.tag} marker tapped", LENGTH_SHORT).show()

  // Manually perform animation:
  googleMap.animateCamera(CameraUpdateFactory.newLatLng(marker.position))

  true // Prevent default animation from occurring.
}
{% endhighlight %}

The bug does not occur if you do not move the map camera at all from your `OnMarkerClickListener`:

{% highlight kotlin %}
googleMap.setOnMarkerClickListener { marker ->
  Snackbar.make(rootView, "${marker.tag} marker tapped", LENGTH_SHORT).show()

  true // Prevent default animation from occurring.
}
{% endhighlight %}

but that is a significant restriction and unpleasant deviation from the behavior users expect:

<div class="image-container">
  <img src="/assets/images/a-map-marker-tapping-bug-in-google-maps-android-sdk-v2-v2-workaround.gif" width="50%" />
</div>

Helpfully, this bug (and many others) is already fixed in the latest beta of Google Maps Android SDK **v3**:

<div class="image-container">
  <img src="/assets/images/a-map-marker-tapping-bug-in-google-maps-android-sdk-v2-v3-fixed.gif" width="50%" />
</div>

This beta represents [the future direction of the Google Maps Android SDK](https://cloud.google.com/blog/products/maps-platform/whats-next-for-google-maps-platform) but is still a little [rough around the edges](https://issuetracker.google.com/issues/148084488) and adds a bunch of weight (approximately 5-7MB) to your APK/AAB.

Given their stated goals and [past comments](https://issuetracker.google.com/issues/69629563#comment7), it seems unlikely to me that the Google Maps team will backport fixes from v3 to v2:

> The Maps SDK for Android has recently received a major overhaul, now available in Beta, which made many bugs and feature requests inapplicable or obsolete.
>
> We believe that the issue or feature request reported [in the v2 issue tracker] has been fixed or is inapplicable with this new release, and therefore closed this report.

If the bug described in this post is unacceptable for your app I therefore recommend that you [update to Google Maps Android SDK v3](https://developers.google.com/maps/documentation/android-sdk/v3-client-migration#install_the_client_library) when possible.

If you do update, be sure to report any remaining bugs using the [issue tracker component dedicated to the beta SDK](https://issuetracker.google.com/issues?q=componentid:541018).

The code for this post is available [here](https://github.com/stkent/google-maps-marker-tap-cycling-bug).
