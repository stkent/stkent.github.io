---
title: Capturing Nougat Screenshots Using adb shell
author: Stuart Kent
tags: android

---

Prior to Android Nougat, I used the following bash function to speedily save screenshots to my local machine:

{% highlight bash %}
function screenshot() {
  adb shell screencap -p | perl -pe 's/\x0D\x0A/\x0A/g' > "${HOME}/Desktop/screenshot.png"
}
{% endhighlight %}

However, executing this function when using a device running Android Nougat results in a corrupted output file. What gives?

<!--more-->

# Background

The output of the adb shell's screencap utility is known to be somewhat funky on older (read: pre-Nougat) versions of Android. In particular, "adb shell" performs an automatic [line feed (LF) to {carriage return (CR) + line feed (LF)}](http://stackoverflow.com/a/13593914/2911458) conversion. This can be observed by capturing a “naive” screenshot (no [perl sanitization](http://blog.shvetsov.com/2013/02/grab-android-screenshot-to-computer-via.html)):

{% highlight bash %}
adb shell screencap -p > "screenshot.png"
{% endhighlight %}

and then inspecting the corresponding [hex dump](https://en.wikipedia.org/wiki/Hex_dump):

{% highlight text %}
$ hexdump -C screenshot.png | head
00000000  89 50 4e 47 0d 0d 0a 1a  0d 0a 00 00 00 0d 49 48  |.PNG..........IH|
00000010  44 52 00 00 02 d0 00 00  05 00 08 06 00 00 00 6e  |DR.............n|
00000020  ce 65 3d 00 00 00 04 73  42 49 54 08 08 08 08 7c  |.e=....sBIT....||
00000030  08 64 88 00 00 20 00 49  44 41 54 78 9c ec bd 79  |.d... .IDATx...y|
00000040  9c 1d 55 9d f7 ff 3e 55  75 f7 de b7 74 77 d2 d9  |..U...>Uu...tw..|
00000050  bb b3 27 10 48 42 16 c0  20 01 86 5d 14 04 11 dc  |..'.HB.. ..]....|
00000060  78 44 9d c7 d1 d1 11 78  70 7e 23 33 8e 1b 38 33  |xD.....xp~#3..83|
00000070  ea 2c 8c 8e 0d 0a 08 a8  23 2a 0e 10 82 ac c1 40  |.,......#*.....@|
00000080  12 02 81 24 64 ef ec 5b  ef fb 5d 6b 3b bf 3f ea  |...$d..[..]k;.?.|
00000090  de db dd 49 27 e9 ee 74  77 3a e3 79 bf 5e 37 e7  |...I'..tw:.y.^7.|
{% endhighlight %}

All valid PNG files start with the following [8-byte header](https://en.wikipedia.org/wiki/Portable_Network_Graphics#File_header):

{% highlight text %}
89 50 4e 47 0d 0a 1a 0a
{% endhighlight %}

But look at the first 10 bytes of our “naively”-captured pre-Nougat screenshot:

{% highlight text %}
89 50 4e 47 0d 0d 0a 1a 0d 0a
{% endhighlight %}

They don't match! Each occurrence of the byte “0a” in the correct byte sequence has been replaced with a pair of bytes, “0d” “0a” in the incorrect byte sequencing. This is precisely the line feed (LF) to {carriage return (CR) + line feed (LF)} conversion mentioned earlier. The global perl search-and-replace in the original script reverts this heavy-handed manipulation and results in a valid PNG file.

# Nougat

Here's the hex dump of a “naively”-captured Nougat screenshot:

{% highlight text %}
$ adb shell screencap -p > "screenshot.png"
$ hexdump -C screenshot.png | head
00000000  89 50 4e 47 0d 0a 1a 0a  00 00 00 0d 49 48 44 52  |.PNG........IHDR|
00000010  00 00 05 a0 00 00 0a 00  08 06 00 00 00 47 f4 85  |.............G..|
00000020  cf 00 00 00 04 73 42 49  54 08 08 08 08 7c 08 64  |.....sBIT....|.d|
00000030  88 00 00 20 00 49 44 41  54 78 9c ec dd 79 78 54  |... .IDATx...yxT|
00000040  f5 f9 fe f1 7b 96 4c b6  c9 1e f6 9d 20 06 65 09  |....{.L..... .e.|
00000050  a0 b2 c8 aa 20 b8 07 6d  d5 aa 55 b4 ae b5 56 d4  |.... ..m..U...V.|
00000060  5f ad 7c ad 56 d4 6a 37  04 6c d5 2a 2a d8 82 5b  |_.|.V.j7.l.**..[|
00000070  55 c0 1d 50 d9 41 ac ec  c8 be 86 2d 40 42 12 b2  |U..P.A.....-@B..|
00000080  4f 66 f9 fd 11 83 42 66  26 db 4c e6 4c f2 7e 5d  |Of....Bf&.L.L.~]|
00000090  17 57 cb 99 33 e7 3c 33  cc 04 bc cf e7 3c 8f 49  |.W..3.<3.....<.I|
{% endhighlight %}

The first 8 bytes **do** match the expected PNG file header, so we can deduce that “adb shell” has **not** performed the line feed (LF) to {carriage return (CR) + line feed (LF)} conversion in this case! This also explains why my original bash function **always** produces invalid Nougat screenshots. By attempting to undo a conversion that was never applied, the perl search-and-replace mutilates the PNG file header, replacing the required “0d” “0a” sequence with a single “0a”.

# Solution

For now, I've defined separate pre-Nougat and post-Nougat screenshot bash functions to deal with this change in behavior:

{% highlight bash %}
function screenshot() {
  adb shell screencap -p | perl -pe 's/\x0D\x0A/\x0A/g' > "${HOME}/Desktop/screenshot.png"
}

function screenshot-n() {
  adb shell screencap -p > "${HOME}/Desktop/screenshot.png"
}
{% endhighlight %}

There's probably a slick way to unify these functions by inspecting the incoming bytes and conditionally applying the global perl search-and-replace. Sadly, my bash-fu is not at that level - a challenge for the future!