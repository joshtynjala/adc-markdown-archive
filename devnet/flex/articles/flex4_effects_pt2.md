# Effects in Adobe Flex 4 – Part 2: Advanced graphical effects

by Chet Haase

## Content

- [3D: Taking effects into the next dimension](#3d-taking-effects-into-the-next-dimension)
- [Using Pixel Bender shader effects](#using-pixel-bender-shader-effects)

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

- [flex4-effects-pt2](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/flex4-effects-pt2)

In [Part 1](flex4_effects_pt1.html) of this article series on using Flex 4 to
create basic effects, we looked at the superclass of the new effects, Animate,
and some of the fundamental effect subclasses. In this article, we'll take a
look at some other effect subclasses that enable more advanced graphics
capabilities in Flex 4.

The code snippets included in this article are included in the sample files that
you can download below. If you feel like playing with the applications, grab the
source code, download the open source Flex SDK and start coding.

### 3D: Taking effects into the next dimension

While the Flex team was busy working on Flex 4 and thinking about component
architectures, skinning and namespaces, the Flash team was releasing Flash
Player 10 with all kinds of cool, new features. One of those features was 3D;
the ability to position and orient Flash display objects in Z as well as x and
Y.

Flash Player does not support a complete 3D graphics engine. In particular,
there is no Z buffer, which means you may encounter some overlapping artifacts
if you are trying to render complex 3D models. However, Flash Player does
support correct 3D positioning and perspective projection, which allows you to
create some stunning visual effects. You may not be able to easily write a 3D
modeler without some extra processing on the objects to avoid depth issues, but
you can certainly write cover flow-like or other 3D UI effects in a much easier
and more visually-correct way than was previously possible.

With all of new features included in Flash Player 10, it behooved the Flex team
to take advantage of them. Flex 4 includes new effects that allow you to easily
manipulate objects in 3D—just as you can in 2D. In particular, the new
subclasses of the transform effects (discussed in Part 1 of this article series)
make it possible to manipulate the 3D attributes of an object's transform
matrix:

- **Move3D** exposes from, to, and by properties for x, y, and z to allow you to
  animate objects between 3D positions (the default position of Flex objects are
  at z=0).
- **Rotate3D** exposes from and to properties for the x, y, and z axes for
  rotation in 3D (the usual 2D rotation is around the z axis only and the
  rotation values around the other axes are typically 0).
- **Scale3D** exposes the from, to, and by properties for scaling in x, y, and z
  to allow scaling in 3D. Note that scaling in 2D is just in x and y, with the z
  scale factor being 1.

These three classes expose both 2D and 3D properties and make it easy to
manipulate objects in all three dimensions simultaneously.

#### **Example: 3D mouseOver**

Let's look at the code used to animate the 3D properties of some UI elements.

If you've downloaded the sample files provided with this article and want to
follow along, this sample is found in the application named ThreeDButtons.

The ThreeDButtons application is a simple application that animates buttons
whenever the mouse hovers over them. In this example, both rotation and movement
in 3D are animated. You can switch between the effects using the radio buttons
at the top of the application. As the user's mouse moves over the buttons,
effects are run on each button that either rotate the button 360 degrees around
the y axis or move the button toward the viewer and then back into place. The
effects are declared like this:

    <s:Rotate3D id="rotator" angleYFrom="0" angleYTo="360"
    	autoCenterTransform="true"
    	effectEnd="effectEndHandler(event)"/>
    <s:Move3D id="mover" duration="200" zBy="-30" repeatCount="2"
    	repeatBehavior="{RepeatBehavior.REVERSE}"
    	autoCenterTransform="true"
    	effectEnd="effectEndHandler(event)"/>

The "rotator" effect spins a target object around the y axis, with the transform
center anchored at the center of the object. The "mover" effect will move an
object toward the viewer (using a decreasing Z value) and then move it back
again.

The effects are played when the buttons detect that the mouse is over them, via
the mouseOver event. An event handler calls a script function to start the
appropriate effect, like this:

    <s:Button id="button0" width="100" height="100"
    	mouseOver="animateHover(button0)"/>

The animateHover() function is defined in a script block, as shown below:

    private function animateHover(target:Object):void
    {
    	if (animatingTargets[target.id] === undefined)
    	{
    		var effect:Effect;
    		if (rotationButton.selected)
    			effect = rotator;
    		else
    			effect = mover;
    		effect.target = target;
    		animatingTargets[target.id] = effect;
    		effect.play();
    	}
    }

In the code example above, the appropriate effect is set up (based on which
radio button the user selected), the target is set and the effect is played. We
also add an entry into an Object map for the target, to note that we are
currently playing an effect on this target. This prevents the effect from
restarting if another mouseOver event is broadcast while the effect is still
playing. We also register an event listener for the effectEnd event in our
effects so that we can remove that entry from the map when the effect is done:

    private function effectEndHandler(event:EffectEvent):void
    {
    	delete animatingTargets[event.effectInstance.target.id];
    }

Setting up and playing 3D effects using Flex 4 is fairly straightforward. Next,
let's see how we can use Pixel Bender to create shader effects with Flex 4.

### Using Pixel Bender shader effects

Another amazing graphics feature included in Flash Player 10 is support for
Pixel Bender shaders. Pixel Bender Toolkit is a technology available from Adobe
Labs that allows you to create small programs to manipulate images on a
per-pixel basis, based on arbitrary numbers of image inputs and other constants
and calculations defined in your programs. Think of this process as iterating
through each pixel of an image, running your program at each step and
calculating the final pixel value for the image based on the inputs and
computations specified by your program.

A thorough discussion of the Pixel Bender Toolkit and how to use it is beyond
the scope of this article. This section is about how this technology can be used
in Flex.

The great thing about Pixel Bender support in Flash is that it is now possible
to create custom filters on Flash display objects. Flash filters are powerful
mechanisms for changing the way that objects are displayed on the screen. With
Flash filters, it is a simple matter to perform tasks that would otherwise
require a lot of tedious code to get right, such as adding a drop shadow to a
component or a glow around a button. Prior to Flash Player 10, it was not
possible to create custom Flash filters; you had to rely on the pre-built
filters included with Flash. But with the Pixel Bender Toolkit, you now have a
framework to plug in very custom filters to make objects display in arbitrary
ways on the screen. And it is possible to animate these filters with the effects
in Flex 4 to get all kinds of different visual results.

There are two effects in Flex 4 which enable the use of Pixel Bender shaders:
AnimateFilter and AnimateTransitionShader.

#### **Working with AnimateFilter**

The AnimateFilter effect is not specific to Pixel Bender shaders; it can be used
to animate the properties of any filter object attached to a UI element. For
example, AnimateFilter can animate a glow around a component.

If you're following along and you've downloaded the sample files provided with
this article, this sample is found in the application named GlowingButton.

    <s:GlowFilter id="glow" blurX="20" blurY="20" color="0 x 80ffff"/>
    <s:AnimateFilter id="glower" target="{button}" bitmapFilter="{glow}"
    	duration="600" repeatCount="0"
    	repeatBehavior="{RepeatBehavior.REVERSE}">
    	<s:SimpleMotionPath property="alpha" valueFrom="0" valueTo="1"/>
    </s:AnimateFilter>

This code defines an appropriate GlowFilter to use, then applies an
AnimateFilter effect that varies the alpha property of that filter over time. To
make the animation run while the mouse is hovering over a button, define the
glowing button like this:

    <s:Button id="button" x="100" y="100"
    	mouseOver="if (!animating) { glower.play(); animating = true}"
    	mouseOut="if (animating) { glower.end(); animating = false}"/>

The button uses the boolean flag `animating` to determine whether the animation
is already running, in order to prevent it from continually restarting as the
mouse moves around and over the button.

This example shows the basic process of using the AnimateFilter effect: You pass
in an effect, define the properties you want to animate on that effect and then
play it. This works like most of the other effects in the system except that
rather than animating the properties on the target object, it animates those of
the filter you provide instead, and then sets that filter on the target object
while the effect is playing.

But what does this have to do with Pixel Bender? There is a new filter in Flash
Player 10 called ShaderFilter, and you can use this filter as an input to
AnimateFilter to make it animate the properties of a Pixel Bender shader —just
like we animated the properties of GlowFilter in the example above.

If you'd like to run the next example, open the AnimatedCrossfade application
provided in the sample files.

This example performs a simple cross-fade between two images, which are embedded
as follows (awesome San Francisco pictures provided by Romain Guy):

    [Embed(source="images/GoldenGate.jpg")]
    [Bindable]
    private var GoldenGate:Class;

    [Embed(source="images/Harbor.jpg")]
    [Bindable]
    private var Harbor:Class;

An Image component is used to hold the current image, like this:

    <mx:Image id="img" source="{GoldenGate}" click="clickHandler()"/>

Meanwhile, the code defines a ShaderFilter to hold the Pixel Bender shader. The
shader we are using is one of the samples that is included with the Pixel Bender
Toolkit. This shader has not been modified from its original source, but I've
simply compiled it into a "pbj" file for use with Flash Player 10.

**Note:** You can use the menu options in the toolkit application to compile
shaders into pbj files.

The shader performs a cross fade between two image inputs, calculating the final
output pixel value by combining the two input image pixel values in a ratio
determined by the intensity parameter, which is a value from 0 to 1. As the
value of intensity gets closer to 1, the result becomes closer to the second
image input. Here's the code:

    <s:ShaderFilter id="crossfadeFilter"
    	shader="@Embed(source='shaders/crossfade.pbj')"/>

Finally, we declare our AnimateFilter effect using the code snippet below. Note
that AnimateFilter takes our ShaderFilter as an input for its shader property so
that it can animate the `intensity` variable in the filter:

    <s:AnimateFilter id="crossfader" target="{img}"
    	bitmapFilter="{crossfadeFilter}"
    	effectEnd="effectEndHandler()">
    	<s:SimpleMotionPath property="intensity"
    		valueFrom="0" valueTo="1"/>
    </s:AnimateFilter>

Now let's see how to set up and play the effect. First, we have to get a handle
to the underlying BitmapData objects of our images, which are provided to the
shader as inputs when we play the effect. We grab the handle to the BitmapData
objects in our creationComplete() handler when the application first starts up,
like this:

    private var goldenGateBD:BitmapData;
    private var harborBD:BitmapData;
    private function creationComplete():void
    {
      goldenGateBD = (new GoldenGate()).bitmapData;
      harborBD = (new Harbor()).bitmapData;
    }

When the user clicks on the image, the clickHandler() function is called. This
sets up the image inputs for the shader filter and also sets up the
`newImageSource` property, which will be used when the effect ends to switch the
image control's source in our Image control:

    private function clickHandler():void
    {
    	var bd0:BitmapData;
    	var bd1:BitmapData;

    	if (img.source == GoldenGate)
    	{
    		newImageSource = Harbor;
    		bd0 = goldenGateBD;
    		bd1 = harborBD;
    	}
    	else
    	{
    		newImageSource = GoldenGate;
    		bd0 = harborBD;
    		bd1 = goldenGateBD;
    	}
    	crossfadeFilter.shader.data.frontImage.input = bd0;
    	crossfadeFilter.shader.data.backImage.input = bd1;
    	crossfader.play();
    }

Finally, once the effect ends, we set up our Image control to display the
correct image that we cross-faded to, so that it is now displayed on screen:

    private function effectEndHandler():void
    {
    	img.source = newImageSource;
    }

#### **Using AnimateTransitionShader**

The other way in which Flex effects use Pixel Bender shaders is in a new set of
effects that are subclasses of the AnimateTransitionShader class. In Flex 4,
there are currently just the Wipe and Crossfade effects. More effects may be
added in the future, but it is also relatively easy for you to create your own
effects using these classes and shaders that you define to create completely
new, custom transitions.

The idea behind the AnimateTransitionShader effect, and its subclass effects, is
that the effect captures a bitmap snapshot of the target object in its before
and after states. The effect sets up a ShaderFilter with those two images as
inputs and the shader defined by the effect (or passed in by you for a more
custom effect), and runs an animation on the ShaderFilter to vary the look of
the target object over time between the before and after images.

Because these effects perform very specific tasks, combining two input images
over time, the structure of the Pixel Bender shader is required to conform to
some simple constraints. In particular, shaders used with
AnimateTransitionShader must have three input images, with the second and third
named `from` and `to` and must have a floating point value named `progress`,
which is the property that the effect will use to animate between 0 and 1 as the
animation runs. The reason for the first image input is that Flash expects the
first image input to be the object being filtered. In this case, however, we
derive the output of these shaders just from the bitmap images, so we only need
that first image input to satisfy the Flash Player requirement.

By default, the target object for this effect must be a UIComponent or
GraphicElement subclass, because the effect calls a known function to take a
bitmap snapshot on those classes. However, you can supply your own BitmapData
objects if you want to animate between other types of objects that do not have
that snapshot function—or if you just want to provide the bitmaps for other
reasons.

As its name implies, AnimateTransitionShader is specifically intended to be used
in transitions. While you can play it manually, it doesn't make much sense to do
that because it's really intended for use in situations where it automatically
determines the before/after images and runs the shader on those images.

The Wipe and Crossfade effects are particularly easy to use because they embed
all of the logic of providing the appropriate shader internally. You only need
to specify the target object to affect and, in the case of Wipe, the direction
in which you want the effect to run.

To see the next example, run the ShaderTransitions application provided in the
sample files folder.

The code below smoothly transitions between two different images. The images are
displayed in a single Image control:

    <mx:Image id="img" source="{GoldenGate}" source.state2="{Harbor}"
    	click="currentState=(currentState=='state1')?'state2':'state1'"/>

When the user clicks the control, it changes the `currentState` variable, which
causes the following transition to run:

    <s:Transition id="transition" effect="{crossfader}"/>

By default, this transition will run a CrossFade effect, which is set up in the
Declarations section along with a Wipe effect:

    <fx:Declarations>
    	<s:CrossFade id="crossfader" target="{img}"/>
    	<s:Wipe id="wiper" target="{img}"/>
    </fx:Declarations>

The radio buttons at the top of the window allow users to toggle which effect
will run. The code below sets up the Wipe Left button:

    <s:RadioButton id="wipeLBtn" label="Wipe Left"
    	groupName="EffectType" change="changeHandler(event)"/>

When the user selects one of these controls, the changeHandler() function is
called. This function sets up the transition to point to the appropriate effect,
based on which radio button is selected. It also sets up the Wipe effect
direction property appropriately:

    switch (event.target)
    {
    	case wipeLBtn:
    		transition.effect = wiper;
    		wiper.direction = WipeDirection.LEFT;
    		break;
    	// other cases are similar
    }

As you can see in this simple example, the fact that these effects use Pixel
Bender shaders is not immediately apparent to developers. You don't need to know
how to use the Pixel Bender Toolkit in order to create these effects with
Flex 4. However, this information is handy to know if you'd like to write
shaders of your own to create unique custom transitions.

Using Pixel Bender filters in Flex applications is a powerful way to create
interesting visual effects. And animating these filters over time using the
effects in Flex 4, you can create very interesting animations and transitions.

### Where to go from here

There are many other elements of effects in Flex 4 that you could investigate:

- Using easing to create more realistic or effective animations
- Using interpolators to handle animating custom objects and non-numeric types
- Using the new KeyFrame object to define animations between multiple points
  (instead of just between two endpoints)
- Understanding how the underlying Animation class runs effects and animations
  and how it can be used directly for custom animations
- Investigating how the new Action effects work for performing simple one-shot
  actions like calling functions or pausing until particular events occur

Hopefully the material covered in this article series has given you some idea of
how effects work in Flex 4 and what new facilities are available to create
compelling applications.

If you haven't done so already, be sure to check out
[Part 1](flex4_effects_pt1.html) of this article series.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
