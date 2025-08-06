# Resource management strategies in Flash Player

by Grant Skinner

![Grant Skinner](./img/1334095867017.jpg)

## Content

- [Why is resource management an issue?](#why-is-resource-management-an-issue)
- [Using System.totalMemory](#using-systemtotalmemory)
- [Weak references](#weak-references)
- [Where to go from here](#where-to-go-from-here)

## Requirements

### User level

Advanced

ActionScript 3.0 empowers Flash developers with faster code execution and many
new API enhancements. From a developer's standpoint, these changes require a
much higher level of responsibility than ever before. This article focuses on
the implications that some of the new ActionScript 3.0 features have on resource
management, and looks briefly at some of the new tools in ActionScript 3.0 that
can help you to track and manage memory more effectively.

The biggest change in ActionScript 3.0 that affects resource management is the
new display list model. In Flash Player 8 and earlier, when a display object was
removed from the screen (using `removeMovie` or `unloadMovie`), the display
object and all of its descendants were immediately removed from memory and code
execution ceased. Flash Player 9 introduces a much more flexible display list
model, where display objects (sprites, movie clips, etc.) are treated the same
way as normal objects.

This means that developers can now do really cool things like reparenting
(moving a display object from one display list to another), and instantiating
display objects from loaded SWFs. Unfortunately, it also means that display
objects are now treated the same as every other object by the garbage collector,
which raises a whole slew of interesting (and possibly nonobvious) issues.

### Why is resource management an issue?

Flash developers are likely to look at some of the new resource management
considerations in ActionScript 3.0 and think the concepts are complicated. Java
developers, on the other hand, would probably think nothing of them. This
disparity is understandable: Flash developers are not accustomed to implementing
manual resource management beyond basic best practices—such as killing
references when no longer needed—whereas Java developers have been through all
this before. These issues are par for the course for most modern memory managed
languages. Unfortunately, there is no way to completely avoid them.

Although resource management is a fact of life, Flash raises many challenges
that are rare in other languages (including Flex). Flash content tends to
include a lot of idle or reactive code execution—unlike Java and Flex which are
mostly interactive, meaning that the CPU-intensive code usually only executes
upon user interaction. Additionally, Flash projects tend to load external
content from third-party sources (possibly using poor coding standards) far more
often than other platforms. Flash developers also have fewer tools, profilers,
and frameworks to utilize.

Finally, Flash developers generally come from a much less formal programming
background. Most Flash developers I know have backgrounds in music, art,
business, philosophy, or just about anything other than programming. This
diversity results in _awesome_ creativity and content but it does not really
prepare the community for dealing with resource management issues.

#### Issue 1: Dynamic content

One of the more obvious issues encountered with resource management is related
to sprites (or other display objects) that you instantiate dynamically, and then
wish to remove at a later time. Display objects continue to exist in memory when
you remove the object from the Stage because they no longer live and die on the
display list. If you have not cleaned up all other references to the clip,
including the object's listeners, it may never be removed. If you have done a
good job of cleaning up all references, the clip will be removed from memory the
next time the garbage collector runs a sweep, which will occur at some
indeterminate point in the future based loosely on memory usage.

It is very important to note that not only will the display object continue to
use memory, it will also continue to execute any "idle" code, such as timers,
enterFrames, and listeners that are outside its scope.

A couple of examples will help illustrate this issue:

- A game sprite subscribes to its own `enterFrame` event. In every frame it
  moves, the application performs some calculations to determine its proximity
  to other game elements. In ActionScript 3.0, even after you remove the sprite
  from the display list and null all references to it, the application continues
  to run that code in every frame until it is removed by garbage collection. You
  must remember to explicitly remove the `enterFrame` listener when the sprite
  is removed.
- Consider a movie clip that follows the mouse by subscribing to the Stage's
  `mouseMove` event—which is the only way to achieve this effect in the new
  event model. Unless you remember to remove the listener, the clip will
  continue to execute code every time the mouse is moved, even after the clip is
  "deleted." By default, the clip executes forever because a reference to it
  exists from the Stage for event dispatch. I will discuss how to avoid this
  issue in a future article.

Now imagine the implications of instantiating and removing several sprites
before the garbage collector does its sweep—or what will occur if you fail to
remove all references to the sprites. You could inadvertently max out the CPU
resources fairly easily, causing your application or game to slow to a crawl, or
even stalling the users' computers entirely. There is currently no way to force
Flash Player to kill a display object and stop it from executing. It is up to
the Flash developer to do this manually when the object is removed from the
display.

#### Issue 2: Loaded content

Remember that the contents of loaded SWFs are also now treated the same way as
every other object and you can begin to imagine some of the problems that you
could encounter with loaded content. Similar to other display objects, there is
no way to explicitly remove a loaded SWF and its contents from memory, or to
stop it from executing. Calling Loader.unload simply nulls the loader's
reference to the SWF; it will continue to exist and keep executing until it has
been picked up by the next garbage collection sweep (assuming all other
references to the loaded content have been properly cleared).

Consider the following two scenarios:

- You build a shell that loads your experimental Flash projects. This
  experimental work is cutting-edge and pushes the CPU's resources to the limit.
  A user clicks a button to load one experiment, views it, then clicks a button
  to load a second experiment. Even if all references are cleared to the first
  experiment, it will continue to run in the background, which will likely max
  the processor out when the second experiment starts running at the same time.
- A client commissions you to build an application that loads ActionScript 3.0
  SWFs created by other developers. These developers add listeners to the Stage
  or otherwise create external references to their own content. Because there is
  no way to unload their content, it will live in memory and continue to consume
  CPU resources until the user quits your application. Even if the loaded
  content does not have any external references, it will continue to execute
  indefinitely until the next garbage collection sweep.

When you design an application that loads untrusted content, it is very
important to be aware that the code could continue to execute after you have
unloaded it. While this content will act according to the appropriate
[Flash Player security model](./cross-domain-policy.md), it is always a good
idea to think through potential vulnerabilities in your application during the
development process.

### Using System.totalMemory

Although System.totalMemory is a simple tool, it is important because it marks
the first runtime profiling tool that developers can use in Flash. It allows you
to monitor how much memory is in use by Flash Player at runtime. This gives you
some ability to profile your own work during development without using a system
monitor. More importantly, it makes it possible for you to preemptively deal
with major memory leaks in your content before it could cause a serious issue
for your user. It's always better to throw an error and abort your application,
rather than bog down the user's system or even stall it completely.

Here's a simple example of how you could accomplish this:

    import flash.system.System;
    import flash.net.navigateToURL;
    import flash.net.URLRequest;
    ...
    // check our memory every 1 second:
    var checkMemoryIntervalID:uint = setInterval(checkMemoryUsage,1000);
    ...
    var showWarning:Boolean = true;
    var warningMemory:uint = 1000*1000*500;
    var abortMemory:uint = 1000*1000*625;
    ...
    function checkMemoryUsage() {
        if (System.totalMemory > warningMemory && showWarning) {
            // show an error to the user warning them that we're running out of memory and might quit
            // try to free up memory if possible
            showWarning = false; // so we don't show an error every second
        } else if (System.totalMemory > abortMemory) {
            // save current user data to an LSO for recovery later?
            abort();
        }
    }
    function abort() {
        // send the user to a page explaining what happpened:
        navigateToURL(new URLRequest("memoryError.html"));
    }

This could obviously be enhanced in a number of ways, but hopefully this code
demonstrates the basic concept behind this process.

It is important to note that `totalMemory` is a shared value within a single
process. A single process may be just one browser window, or all open browser
windows, depending on the browser, the operating system, and how the windows
were opened. For example, in Mac OS X, all Safari browser windows share a single
process and `totalMemory` value, whereas the processes and values are much more
convoluted in Microsoft Windows.

### Weak references

One of the features I'm really happy to see implemented in ActionScript 3.0 is
called _weak references_. These can be described as references to objects that
are not counted by the garbage collector in determining an object's availability
for collection. If the only references remaining to an object are weak, that
object will be removed by the garbage collector on its next pass.

Unfortunately, weak references are only supported in two contexts. The first is
event listeners—which is great because event listeners are one of the most
common references that cause problems with garbage collection. I strongly
recommend _always_ using weak references with listeners. This is accomplished by
passing `true` as the fifth parameter of your `addEventListener` calls, as shown
here:

    someObj.addEventListener("eventName",listenerFunction,useCapture,priority,weakReference);
    stage.addEventListener(Event.CLICK,handleClick,false,0,true);
    // the reference back to handleClick (and this object) will be weak.

To learn more about this topic, read the article on my blog about
[weakly referenced listeners](https://www.gskinner.com/blog/archives/2006/07/as3_weakly_refe.html).

ActionScript 3.0 also supports weak references in the Dictionary object. Simply
pass true as the first parameter when you instantiate a new Dictionary to have
it use weak references as its keys, as shown here:

    var dict:Dictionary = new Dictionary(true);
    dict[myObj] = myOtherObj;
    // the reference to myObj is weak, the reference to myOtherObj is strong

### Where to go from here

Resource management is an important part of ActionScript 3.0 development.
Ignoring the issues described in this article could result in sluggish content.
There's also a risk of potentially stalling users' systems completely. There is
no longer any way to explicitly remove a display object from memory and stop its
code from executing—which means all Flash developers have a responsibility to
clean up properly after objects are no longer used in an application.

While ActionScript 3.0 has substantially increased the amount of work developers
must do to manage resources in their applications, there are new tools provided
in Flash Player 9 to help manage memory usage. Pairing these new tools with
effective strategies and approaches (the subject of my companion article,
[Understanding garbage collection in Flash Player 9](./garbage-collection.md))
should allow you to successfully manage resources in your upcoming Flash and
Flex projects.
