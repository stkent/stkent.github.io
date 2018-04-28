---
title: Android Studio Shortcut Secrets
author: Stuart Kent
tags: android, android studio

---

Many “best Android Studio shortcuts” articles inhabit the blogosphere. Here are some perspectives on effective shortcut usage that I don't remember reading anywhere else.

<!--more-->

# Shortcut-Only Navigation

{:.table-left}
| Description                   | Mac Shortcut                                                        | Windows/Linux Shortcut                                           |
|:------------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------|
| Go to class                   | <kbd>Cmd</kbd>&nbsp;<kbd>O</kbd>                                    | <kbd>Ctrl</kbd>&nbsp;<kbd>N</kbd>                                |
| Go to file                    | <kbd>Cmd</kbd>&nbsp;<kbd>Shift</kbd>&nbsp;<kbd>O</kbd>              | <kbd>Ctrl</kbd>&nbsp;<kbd>Shift</kbd>&nbsp;<kbd>N</kbd>          |
| Previous/next cursor position | <kbd>Cmd</kbd>&nbsp;<kbd>[</kbd> / <kbd>Cmd</kbd>&nbsp;<kbd>]</kbd> | <kbd>Ctrl</kbd>&nbsp;<kbd>Alt</kbd>&nbsp;<kbd>◀</kbd> / <kbd>Ctrl</kbd>&nbsp;<kbd>Alt</kbd>&nbsp;<kbd>▶</kbd> |
| Open recently used file       | <kbd>Cmd</kbd>&nbsp;<kbd>E</kbd>                                    | <kbd>Ctrl</kbd>&nbsp;<kbd>E</kbd>                                |

The “Go to class” and “Go to file” actions are ideal for quickly opening up existing files. I usually use “Go to class” for anything I know for sure is a Java or Kotlin class, and “Go to file” for anything else. The latter includes all build.gradle files, the Android manifest, and all resource files (layouts, strings, etc.). “Previous/next cursor position” and “Open recently used files” allow me to quickly step back in my navigation history or cycle between several relevant files.

Once you are comfortable with these shortcuts, you'll find that you hardly ever need to use:

- the project pane, or
- tab-based navigation.

At this point you can leave the project pane collapsed by default and turn off the tab bar in Android Studio (Preferences -> Editor -> General -> Editor Tabs; set Placement to None) to slightly increase the size of your code reading/writing area. So far, so good.

An extra and underrated benefit of these same navigation shortcuts is the ability to use them to search library and Android framework classes and files as well as your own code. To do this, press the shortcuts listed above _twice_ before entering the text you'd like to search. I love introducing people to this functionality because it reduces the mental barrier between “code in my app that I wrote” and “code that supports my app that I didn't write”. Both types of code are equally important to the way your app will function (or not!) on user devices, so knowing shortcuts that makes them almost equally accessible is very beneficial. Take advantage of the availability of Android source code - not every platform is so generous!

Example (stepping into and out of an Android framework class (`View`)):

{% include youtube.html video_id="R-AwwPIWv64" %}

# Navigating Despite Decoupling

{:.table-left}
| Description                   | Mac Shortcut                                                        | Windows/Linux Shortcut                                           |
|:------------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------|
| Go to definition              | <kbd>Cmd</kbd>&nbsp;<kbd>B</kbd>                                    | <kbd>Ctrl</kbd>&nbsp;<kbd>C</kbd>                                |
| Go to implementation          | <kbd>Cmd</kbd>&nbsp;<kbd>Opt</kbd>&nbsp;<kbd>B</kbd>                | <kbd>Ctrl</kbd>&nbsp;<kbd>Alt</kbd>&nbsp;<kbd>C</kbd>            |
| Go to usage                   | <kbd>Cmd</kbd>&nbsp;<kbd>Opt</kbd>&nbsp;<kbd>F7</kbd>               | <kbd>Ctrl</kbd>&nbsp;<kbd>Alt</kbd>&nbsp;<kbd>F7</kbd>           |
| Go to super                   | <kbd>Cmd</kbd>&nbsp;<kbd>U</kbd>                                    | <kbd>Ctrl</kbd>&nbsp;<kbd>U</kbd>                                |

This group of shortcuts operate on almost all Java constructs, like classes, methods and fields. They allow us to jump from any usage of a construct to important related code, like:

- the original definition of a class;
- places where an interface method is implemented;
- places where a method is invoked;
- the super implementation that our current method overrides.

These same shortcuts are especially worth learning if you're using interfaces to decouple classes from their dependencies, since they make skipping between interfaces and implementations of those interfaces very fast. This is especially common when a codebase uses an architecture like MVP or MVVM in which every view (and sometimes presenter/view model) is modeled by an interface.

Just like the first group of navigation shortcuts, this second group also work equally well for navigating into and around library and Android framework code.

Example (navigating through several Android framework classes):

{% include youtube.html video_id="0C2cfKDDhfg" %}

# Renaming Resources

{:.table-left}
| Description                   | Mac Shortcut                                                        | Windows/Linux Shortcut                                           |
|:------------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------|
| Rename                        | <kbd>Shift</kbd>&nbsp;<kbd>F6</kbd>                                 | <kbd>Shift</kbd>&nbsp;<kbd>F6</kbd>                              |
| Select in project view        | <kbd>Opt</kbd>&nbsp;<kbd>F1</kbd>, <kbd>Enter</kbd>                 | <kbd>Alt</kbd>&nbsp;<kbd>F1</kbd>, <kbd>Enter</kbd>              |
| Show/hide project view panel  | <kbd>Cmd</kbd>&nbsp;<kbd>F1</kbd>                                   | <kbd>Alt</kbd>&nbsp;<kbd>1</kbd>                                 |

The “Rename” shortcut is incredibly powerful, able to refactor almost anything with a name. However, renaming a project file can still be time-consuming as it usually involves manually locating the relevant file in the project view panel[^1]. The “Select in project view” shortcut will open the project panel if it's currently closed, then jump straight to the file we're currently editing in the file tree. Keep “Show/hide project view panel” handy for closing the project panel when your rename is completed.

Example (renaming an `Activity` layout ready for use with a `Fragment`):

{% include youtube.html video_id="N9Mvzu992cc" %}

# Rapid Error Eradication

{:.table-left}
| Description                   | Mac Shortcut                                                        | Windows/Linux Shortcut                                           |
|:------------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------|
| Jump to next error in file    | <kbd>F2</kbd>                                                       | <kbd>F2</kbd>                                                    |
| Show quick-fixes & actions    | <kbd>Opt</kbd>&nbsp;<kbd>Enter</kbd>                                | <kbd>Alt</kbd>&nbsp;<kbd>Enter</kbd>                             |

If you haven't already encountered “Show quick-fixes & actions”, go practice with it now. It allows us access to all of Android Studio's context-aware automatic fixes, as well as partial access to certain common refactors (e.g. “Replace if with switch”). This shortcut pairs especially well with the lesser-known “Jump to next error in file”, which moves your cursor to the next most important error in the current file. Alternating between “Jump to next error in file” and “Show quick-fixes & actions” is a great way to squash errors without needing to use the gutter or red squiggly underlines to locate problems in our file.

Example (fixing multiple errors and warnings in a class):

{% include youtube.html video_id="YOW0bJ8oKS4" %}

# Customizing While Collaborating

{:.table-left}
| Description                   | Mac Shortcut                                                        | Windows/Linux Shortcut                                           |
|:------------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------|
| Quick switch keymap           | <kbd>Ctrl</kbd>&nbsp;<kbd>&#96;</kbd>, <kbd>3</kbd>                 | <kbd>Ctrl</kbd>&nbsp;<kbd>&#96;</kbd>, <kbd>3</kbd>              |

Some developers prefer to create their own custom keymaps. Good reasons for this include: leveraging existing knowledge from another IDE, accommodating alternative input devices, or better aligning ease of execution with frequency of use.

A potential downside to using a custom keymap is that it can make pairing difficult, since your partner will not be able to use the stock shortcuts they are used to. Rather than allowing this to be a barrier to customizing your IDE, I recommend learning the “Quick switch keymap” shortcut. This allows you to quickly toggle between different keymaps. 

Example (toggling between stock and custom macOS keymaps):

{% include youtube.html video_id="ZxNpcl-6AAU" %}

# Over To You

I'm always looking to improve my workflow. What tips do you have for underrated shortcuts or combos?

[^1]: Note that renaming class files is typically easier than naming resource files, because refactoring the class name in Java also automatically refactors the file name.