# Effects in Adobe Flex 4 – Part 1: Basic effects

by Chet Haase

## Content

- [Working with the Animate class: "One effect to rule them all"](#working-with-the-animate-class-one-effect-to-rule-them-all)
- [Using basic effects](#using-basic-effects)
- [Resize effect](#resize-effect)
- [Transform effects: Move, Rotate and Scale](#transform-effects-move-rotate-and-scale)
- [Fade effect](#fade-effect)
- [AnimateColor effect](#animatecolor-effect)

## Requirements

### Prerequisite knowledge

Basic understanding of the Flex Builder environment and prior experience
developing Flex applications is recommended.

### User level

Beginning

### Required products

- Apache Flex SDK
- Adobe Flash Builder

### Sample files

- [flex4-effects-pt1](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/flex4-effects-pt1)

Flex effects are, to this graphics geek, one of the coolest and most interesting
parts of the Flex platform. Flex effects make it easy to create rich
applications and compelling user experiences. In fact, Flex effects are so
useful that we decided to make them even more powerful in Adobe Flex 4. We've
enabled them to target arbitrary objects and property types, to take advantage
of new technologies in Flash Player, and to be more powerful and functional in
general.

This article describes the new effects in Flex 4. In order to get you up to
speed quickly, I'll dive right into the main classes and provide code examples
to implement basic effects. The details of how Flex effects work in general are
outside the scope of this article—I'll focus on using the new capabilities of
Flex 4 effects to create animations and describe how to incorporate them into
your applications. This article doesn't cover all of the new Flex 4 features
that you may see in some of the code snippets, such as the new spark components,
multiple namespaces, the Declarations block, and the new states syntax, because
there are other articles available that address those enhancements. That leaves
us free to focus on animation.

Because of the numerous animation features available in Flex 4, it was
impossible to fit descriptions of all of the new effects functionality into one
bite-sized article. Instead, this topic is broken up into two parts: Part 1
describes the basic infrastructure of the new effects and covers the basic,
fundamental effects that most Flex developers may use on a regular basis. Part 2
describes some of the more advanced effects that build on new functionality
available in Flash Player 10.

### Working with the Animate class: "One effect to rule them all"

All of the new effects in Flex 4 are subclasses from the class Animate, which is
itself a subclass of the Effect class. Rather than break backward compatibility
with the old effects—which use the TweenEffect superclass—we created a new
effect hierarchy in Flex 4. Now these systems live in parallel: the older Flex 3
effects still work as usual and can be used on components, without any changes.
Flex 4 developers can take advantage of the new Flex 4 effects, which work on
old and new components, as well as the new graphic elements and even arbitrary
objects.

The Animate superclass provides common functionality used by the new effects,
including the ability to target arbitrary objects and types. Animate allows you
to create, manipulate and play animated effects using the underlying Animation
subclass. The Animation class contains the functions that actually run the
animations, calculating and setting the animated values of the animated
properties. We will not have time to go into the details on the lower-level
Animation class itself, but I would encourage you to download the SDK and ASDocs
for Flex 4 and see how it works.

Creating and playing an effect using the Animate class is easy: you provide a
target object to be modified, the names of the properties of the target that
will be animated, and the values that the properties will use to animate the
object while the effect plays. You also have the ability to set some optional
parameters on the effect, such as the duration. After specifying the target,
properties and values, use the play() method to start playing the effect.

In the code example below, an animation is applied to a button which causes it
to move 100 pixels to the right when the button is clicked:

    <s:Animate id="mover" target="{button}">
    	<s:SimpleMotionPath property="x" valueFrom="0" valueTo="100"/>
    </s:Animate>

    <s:Button id="button" click="mover.play()"/>

The button will move from an x location of 0 to an x location of 100 as the
effect plays. There are other approaches to specifying the same animation, such
as using the valueBy property to specify the delta value instead of the absolute
values, or by supplying only the valueTo property—implicitly assigning the other
value from the current value of that property on the target.

In Flex 4, it is also possible to animate several properties simultaneously by
declaring several SimpleMotionPath objects.

**Note:** Flex 3 developers will notice the difference here between using the
Animate class in comparison with the older AnimateProperty effect, which could
only handle a single property.

To affect multiple properties at once, use the following code snippet to animate
several different properties on our button simultaneously:

    <s:Animate id="mover" target="{button}" duration="1000">
      <s:SimpleMotionPath property="x" valueFrom="0" valueTo="100"/>
      <s:SimpleMotionPath property="y" valueTo="100"/>
      <s:SimpleMotionPath property="width" valueBy="20"/>
    </s:Animate>

**Note:** If you've downloaded the sample files provided with this article,
you'll find this example in the AnimateButton application.

In the code example above, notice how the properties use three different ways to
specify the property values.

There are various other properties you can use to change the behavior of an
animation that uses the Animate class, such as repetition (including the new
[RepeatBehavior](https://flex.apache.org/asdoc/spark/effects/animation/RepeatBehavior.html)
flag, which allows you to automatically reverse effects with each repetition,
unlike the previous looping behavior in Flex 3).

You can also supply different easing behaviors to create more realistic motion,
and use different type interpolation for non-numeric types. A discussion of
these is outside the scope of this article, but look for future blogs and
articles that cover these topics.

At this point we've just scratched the surface, because there are so many Flex 4
effects to describe. In the next section, I'll cover the process for using basic
effects in Flex 4.

### Using basic effects

The code examples provided in the previous section show that it is fairly
straightforward to specify arbitrary property names to the Animate effect. When
you play the effect, an animation is run which changes the properties of the
target object over time.

However, you will typically want to set up a specific type of animation using
some standard properties. Or perhaps you want to use an effect which has
built-in logic to take into account other considerations—such as the visibility
of the target objects or the fact that the target object is positioned with
layout constraints in its container. For this reason, Flex 4 includes several
subclasses of the Animate class to handle the majority of effects that will be
used most frequently. These effect classes make it easy to specify the desired
behavior, and then the effects themselves perform the necessary functionality of
wrapping up the properties for the Animate class and performing any other
related functionality to get the job done. Of course, you can always use the
Animate class directly, and you can also create custom effects that subclass
Animate and build on the capabilities of that class. But in most cases, you'll
probably find it easier to just use one of these subclasses to add a basic
effect. You can think of these subclass effects as wrappers around the Animate
class, exposing specific properties and then packaging up the values in an
Animate-friendly way.

For example, the Move effect, which we will see more of later, allows you to
specify location values that you are animating from, to, and by, and then it
creates the underlying SimpleMotionPath objects that the Animate class expects
in order to apply the animations on the target object's x and y properties.

In this example, a Move effect is used to perform the same button animation
shown in the previous section of this article:

    <s:Move id="mover" target="{button}" xTo="100" yTo="200"/>

As you can see, the code that you have to write to move the button can be much
more compact when an effect can make assumptions about what you want to animate.
This means you don't have to provide all of the details as you do when you are
creating a raw animation using the Animate class.

In this case, the Move subclass assumes that you want to animate x and y, so you
don't need to set up SimpleMotionPath objects to supply those details. These
effects can handle other situations automatically, so that you don't have to.
For example, the Move effect and the Resize effect allow you to optionally
animate layout constraints on the target object, making it easier to animate
objects which are being positioned by constraints.

There are several of these fundamental effects in Flex 4 which will probably
constitute most of the effects that you'll need and use on a regular basis. They
include the Move effect (shown above), as well as the Resize, Scale, Rotate,
Fade, and AnimateColor effects. Let's take a look at how these effects work.

### Resize effect

The Resize effect is a simple wrapper around the Animate class that has
from/to/by properties for both width and height. You use the effect in the same
way shown for the Move effect example above. Basically, you just need to specify
the values for the properties in order to set up the animation.

This code example will change the width of the button to 100 and the height to
50:

    <s:Resize id="resizer" widthTo="100" heightTo="50"/>

Like most of the Flex effects, the Resize effect includes some extra, built-in
logic to handle other related situations. For example, if your target object is
positioned with layout constraints (for example, if it is pinned to the right
side of its container with right="0"), then there are facilities for either
disabling those constraints while the effect is playing or animating those
constraints automatically as the Resize effect runs.

### Transform effects: Move, Rotate and Scale

The Move, Rotate, and Scale effects are closely related because they all affect
properties in the underlying transform matrix of the target object. These
effects work together to ensure that they do not supply conflicting information
to that matrix.

For example, if you Rotate an object, you are changing that object's rotation
property. But the (x,y) location of that object may also change over time as a
side effect. (Imagine the object rotating around its center, with its upper-left
point moving while the center point stays constant). If you also want to apply a
Move effect to that same object while it is rotating, the two effects work
together to make sure that the object moves and rotates in a reasonable way.

Flex 3 users should note that this is a change from the way the Move and Rotate
effects used to work; Move and Rotate used to be completely separate from each
other. For the most part, they worked well, but it was possible to get artifacts
when combining them because the two effects could override each others'
instructions for the target object's x and y properties.

##### Shared properties for transform effects

In most circumstances, it is not necessary for you to know what is going on
under the hood of the transform effects in order to use them. You can just add
the code for the effects you need and the effects will work together to ensure
that you get the results you expect.

In many cases, you'll probably only apply one of these effects at a time to a
target object, so there won't be anything complicated going on behind the scenes
– the subclass for the specified effect will simply change the appropriate
properties over time. But in some situations, it is helpful to know that these
three effects are interrelated so that you can give correct parameters to create
the overall coordinated animation. In particular, the transform effects share
the values of the transformation center (if specified) and repetition
parameters, taking their values from the first one of these effects in a
composite effect. When using any of these shared properties, it is a best
practice to specify them exactly the same in all transform effects to make the
code clearly reflect the changes that the effects are actually performing.

These are the shared properties of Move, Rotate, and Scale:

- **autoCenterTransform**: When this flag is set to true, the effects will
  operate around the center of the target object (width/2, height/2). For
  example, the Rotate effect will cause the object to rotate around its center
  and Scale will cause the object to be scaled in relation to its center. The
  Move effect will visually display the same as if the flag were not set, but it
  is important to remember that you are specifying the change in movement of the
  transform center point, not the upper-left point (x=0, y=0) of the object.
  This is a change from the way the older Flex 3 Move effect works – the Flex 3
  Move effect operates directly on the x and y properties of the target object.
- **transformX, transformY**: When autoCenterTransform is not set, the effects
  will use the transform center of the target object (transformX and
  transformY). But if these same properties are specified on the effect itself,
  they override the values on the object. For example, by default the transform
  center of a component or graphical object is the upper-left point, or x=0,
  y=0. If you set the transformX property on the effect to 50, then the center
  around which the transform effects will operate will be (50, 0), using the
  value from the transformX property on the effect and—since transformY is unset
  on the effect, the transformY property on the target object is used, which
  is 0.
- **Repetition settings**: An important note about the repetition-related
  properties (repeatCount, repeatDelay, and repeatBehavior): The values that are
  set on the first transform effect in a composite effect will be applied to all
  of the transform effects in that overall effect. Because of this, it is
  generally not advisable to use repetition settings on the transform effects.
  In fact, the repetition properties have been excluded from the
  [Apache Flex ASDoc Reference](https://flex.apache.org/asdoc/) and code hints
  for these effects because doing so can lead to rather unpredictable results.
  However, if you are applying only one of the transform effects at a time, you
  can set repetition properties and the animation will work as expected.

##### Per-effect properties

In addition to the shared properties described above, each of the transform
effects exposes unique properties to specify their behavior:

- **Move**: As in the Flex 3 version of the Move effect, the Flex 4 Move effect
  exposes xFrom, xTo, xBy, and yFrom, yTo, yBy. These properties are used
  internally to calculate the change in position of the transform center used by
  the effect.
- **Rotate**: Like the Move effect, this Flex 4 effect is also similar to the
  Flex 3 version of the Rotate effect, exposing the angleFrom and angleTo
  properties. However, the Flex 4 version adds a new property: angleBy. These
  property values result in the target object rotating by the specified angle
  (in degrees) around the transform center for the effect.
- **Scale**: This is a brand new effect in Flex 4 that did not exist in Flex 3,
  although it provides similar functionality to the Flex 3 Zoom effect. The
  scaleXFrom, scaleXTo, scaleXBy, scaleYFrom, scaleYTo, and scaleYBy properties
  specify the change in scale factor around the transform center in the x and y
  directions. Note that this effect is quite different from the Resize effect,
  which works on the width and height properties of the target. The width and
  height properties specify how large the object should be with a scale factor
  of 1, whereas the scale properties specify the magnification of the object
  given its current width and height. The difference in the results can be seen
  in resizing a button, for example, where doubling the width and height
  properties using the Resize effect would cause the button itself to become
  larger, but the look of the button, such as the text label, would remain the
  original size. Contrast this with applying a Scale effect on button to double
  its size, which would cause the text label to be scaled along with the button.

The example below illustrates how the transform effects can move, rotate, and
scale a button around its center.

    <s:Parallel id="transformer" target="{button}">
    		<s:Move xFrom="50" xTo="150" autoCenterTransform="true"/>
    		<s:Rotate angleFrom="0" angleTo="90" autoCenterTransform="true"/>
    		<s:Scale scaleXFrom="1" scaleXTo="2" autoCenterTransform="true"/>
    	</s:Parallel>
    	<s:Button id="button" x="50" y="100" label="Transform Me" click="transformer.play()"/>

**Note:** If you've downloaded the sample files provided with this article,
you'll find this example in the TransformEffects application.

In the example above, all three effects set the shared autoCenterTransform
property equal to true, to ensure that they all animate around the center of the
button object.

If you are following along with the sample files and running the demo
applications, run the TransformEffects application and notice how the text in
the button scales and rotates during the effect, which is a big change from
Flex 3. Previously, it was necessary to embed the font for the button text in
order to keep it visible during these kinds of effects. Now, with the new text
capabilities in Flash Player 10, Spark components automatically ensure that the
text remains visible as it animates.

### Fade effect

The Fade effect in Flex 4 is extremely useful in transitions, allowing you to
fade objects in and out as they come into being or go away between states of the
application. This effect exists in Flex 3, but Flex 4 includes more logic to
automatically handle different situations of fading in your application.

On first inspection, working with the Fade effect in Flex 4 is very simple: you
specify the alphaFrom, alphaTo, and alphaBy properties which create an animation
on the alpha property of the target object.

For example, the following code snippet will fade the button out completely from
whatever its current alpha value is (usually 1):

    <s:Fade target="{button}" alphaTo="0"/>

The extra logic in the Fade effect kicks in when other, related properties of
visibility are involved. When any effect is played within a state transition, it
automatically picks up the "to" and "from" values of the target object in the
states that the object is transitioning. In the case of the Fade subclass, the
effect automatically determines whether the target object changes visibility
between the states.

Visibility comes in many forms. The alpha property, which Fade acts on directly,
can be used to make an object visible or invisible (setting alpha="0" results in
the object being invisible). But you could also set the visible property of the
object to true or false. Finally, the object can simply exist (that is, it has a
parent) in the GUI in one, both, or neither states. In all of these cases, the
Fade effect in Flex 4 will figure out whether the object is becoming visible or
invisible and will automatically animate the object's alpha property
appropriately.

In the following example, states and transitions have been set up.

    <s:states>
      <s:State name="state1"/>
      <s:State name="state2"/>
    </s:states>
    <s:transitions>
      <s:Transition>
    	<s:Fade targets="{[button0, button1, button2]}"/>
      </s:Transition>
    </s:transitions>

**Note:** If you've downloaded the sample files provided with this article,
you'll find this example in the AutoFade application.

The states block in the code above is very simple; it just declares all of the
states that will be used later in state-specific properties of the objects. The
transitions block says that no matter what states the objects are transitioning
from and to, apply the Fade effect to buttons 0, 1, and 2.

Now let's examine how the buttons are declared. In this step, each button has a
label indicating its different method of toggling its visibility between the
states:

      <s:Button id="button0" label="Visible" x="100" y="0"
    	 visible="true" visible.state2="false"/>
      <s:Button id="button1" label="Alpha" x="100" y="50"
    	 alpha="0" alpha.state2="1"/>
      <s:Button id="button2" label="Existence" x="100" y="100"
    	 includeIn="state2"/>

Finally, to finish off the code, we declare a single button to toggle the
current state:

    <s:Button label="Toggle State"
    	 click="currentState=(currentState=='state1')?'state2':'state1'"/>

If you are following along with the sample files, run the application named
AutoFade. Click the Toggle State button to toggle the current state, and you'll
see all three buttons fade in and out between the states. The first button fades
out when going to state2 and the other two buttons fade in. When you click the
Toggle State button again (going to state1), the first button fades out and the
other two buttons fade in.

This example demonstrates that while Fade effect animates only the value of
alpha over time, it derives the values for that animation from several
properties which determine the visibility of the target object. This is a small
demonstration of how effects in Flex 4 do more than simply animate named
properties—they also make it easier for you to do what you need to do get
achieve a polished look and feel for your application.

### AnimateColor effect

The new AnimateColor effect in Flex 4 makes it very easy to animate the color of
a target object. By default, the AnimateColor effect will animate the property
named "color" on the target object by linearly interpolating each of the red,
green, and blue channels separately in RGB color space. However, both of these
default behaviors can be changed: you can specify a different property name to
be animated, (in situations where the color property you want to animate is not
called "color"), and you can also specify a different way to interpolate the
color other than using the default linear RGB method.

Using the AnimateColor effect is very simple: you specify the colorFrom and
colorTo values that you want to animate. As with other effects in Flex 4, the
AnimateColor effect will automatically pick up one or both of these values from
either the current value of the target or the state values when running in a
transition. When the effect is played, it automatically uses the interpolator
named RGBInterpolator to handle interpolating the animated RGB values; you can
tell the effect to use your own custom interpolator or the built-in
HSBInterpolator, which interpolates colors in the HSB (Hue, Saturation, and
Brightness) space instead.

Suppose you wish to change the look of an object to indicate that the user has
pressed the mouse down on the object. This example uses an ellipse for the
object, filled with a simple radial gradient. The ellipse is placed inside a
Group object in order to detect mouse down/up user events and change the state
accordingly.

    <s:Group mouseDown="currentState='state2'" mouseUp="currentState='state1'">
      <s:Ellipse x="50" y="50" width="100" height="100">
    	<s:fill>
    	  <s:RadialGradient>
    		<s:GradientEntry id="center" color="0xf0f0f0" color.state2="0x808080" ratio="0"/>
    		<s:GradientEntry id="edge" color="0x404040" ratio="1"/>
    	  </s:RadialGradient>
    	</s:fill>
      </s:Ellipse>
    </s:Group>

**Note:** If you've downloaded the sample files provided with this article,
you'll find this example in the AnimatedGradient application.

If you run the AnimatedGradient application, you'll see that in the default
state (state1) the center of the ellipse is brighter, giving it a 3D-ish look.
When you press the mouse down on the ellipse (state2), the center becomes darker
to give it a slightly depressed look and provides users with interactive
feedback based on their mouse actions.

The next part involves creating a transition effect to animate the center
gradient color between its values in state1 and state2, like this:

    <s:Transition>
      <s:AnimateColor target="{center}" duration="150"/>
    </s:Transition>

This effect will animate the "color" property of that center gradient entry
between the two colors defined in the state information over a period of 150
milliseconds. (You may notice that the AnimateColor effect uses a faster
animation time than the default 500 milliseconds, because otherwise this effect
would appear to be too slow and unresponsive).

Besides being an interesting and creative effect to use, this example of working
with the AnimateColor effect also provides a good demonstration of the new
capability of effects in Flex 4 in general. Previous versions of Flex effects
only operated on numbers, but effects in Flex 4 allow you to animate arbitrary
property types. In this particular example, the type of the colors is still a
Number (because colors are represented as 32-bit unsigned integers). But the
values are not interpolated as a number, but rather as three separate numbers
from the object's component red, green, and blue channels. After you become
familiar with AnimateColor, try adding a custom interpolator for your own custom
types.

### Where to go from here

The effects covered in this article: Fade, Resize, AnimateColor, and the
transform effects Move, Rotate, and Scale are the basic working effects of any
UI application. To add more interactive animations to your application, you
should consider using these effects as your application switches between states.
Flex effects make it easy to add these kinds of visual flourishes to your
application.

If you want to take your GUI to the next level of richness, there are many more
powerful things that you can do with effects in Flex 4.
[Part 2](./flex4_effects_pt2.md) of this article will dive into other effects
that build on the capabilities of Flash Player 10.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
