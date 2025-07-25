# Apache Flex mobile skins – Part 1: Optimized skinning basics

by Jason San Jose

## Content

- [Comparing the Spark and Mobile themes](#comparing-the-spark-and-mobile-themes)
- [Button skin tutorial overview](#button-skin-tutorial-overview)
- [Tutorial step 1: Create the FXG graphic](#tutorial-step-1-create-the-fxg-graphic)
- [Tutorial step 2: Create the skin class](#tutorial-step-2-create-the-skin-class)
- [Tutorial Step 3: Test](#tutorial-step-3-test)
- [FXG tips and tricks](#fxg-tips-and-tricks)
- [Where to go from here](#where-to-go-from-here)

## Requirements

### Prerequisite knowledge

Familiarity with ActionScript and Flex 4.0 Spark skinning is recommended, as is
some experience with Adobe Illustrator.

### User level

Intermediate

### Required products

- Apache Flex SDK
- Adobe Flash Builder
- Adobe Illustrator CS5

### Sample files

- [mobile-skinning-part1](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/mobile-skinning-part1)

The mobile-optimized skins provided in Flex 4.5 are designed with touch
interaction, performance, and memory usage in mind. Despite the relatively
powerful devices available on the market today, typical Spark skins (including
the default skins introduced in Flex 4) do not perform well enough for real
world use on mobile devices. The Adobe mobile-optimized skins are built to
balance two opposing goals: overall performance and ease of skin creation. While
MXML skins are usable to a certain extent, Adobe recommends following a few
simple guidelines to ensure that Flex 4.5 mobile applications meet your
performance expectations as well as your users' performance expectations.

This is the first in a series of articles covering mobile skinning for Flex 4.5.

This article covers the basics of creating mobile-optimized skins including:

- Differences between the Spark theme and the Mobile theme
- Performance optimizations built into the MobileSkin base class
- Using FXG instead of MXML graphics
- Creating a custom Button skin based on the Mobile theme ButtonSkin class
- Follow-up articles in this series will cover more advanced topics including:
- Creating new skins based using the MobileSkin base class
- Creating density-aware skins that adapt to all screen sizes
- Creating custom themes per platform using CSS media queries

### Comparing the Spark and Mobile themes

The Mobile theme is essentially a subset of the functionality introduced with
the Spark theme in Flex 4. The Mobile theme makes critical optimizations for
performance and memory use.

#### MobileSkin base class

Spark skins typically subclass Skin (which extends Group) or SparkSkin (which
extends Skin). Skins in the Mobile theme use a new MobileSkin base class, which
extends UIComponent.

#### States

States support in MobileSkin is one major feature that is optimized for mobile
development. Normal usage of states (in MXML or procedurally in ActionScript)
adds memory and performance overhead costs. MobileSkin optimizes this by
manually handling state changes in procedural code instead of using the State
class and its associated overrides, such as `SetProperty` and `AddChild` .

#### Layout

Since MobileSkin is not a Group, it cannot employ Spark layouts such as
HorizontalLayout, VerticalLayout, or even BasicLayout for use with
constraint-based layouts.

Instead, contents of a MobileSkin are laid out manually through programmatic
code. MobileSkin adds a new lifecycle method, `layoutContents(),` which is
called from `updateDisplayList()` . This method is used to position child
elements of the skin.

#### FXG vs. MXML Graphics

Graphics for skins in the Mobile theme rely on a combination of compiled FXG and
programmatic ActionScript drawing. Drawing code is only used for trivial
graphics or when required to support styling. All other graphics are done with
compiled FXG for optimized performance. MXML graphics are not an option with the
MobileSkin base class unless they are nested in a Group.

#### Text

The Mobile theme does not use the Flash Text Engine (FTE) or the Text Layout
Framework (TLF) in the default skins. This is primarily for performance reasons
and to support native predictive text input and editing.

Instead, Flex 4.5 introduces a new StyleableTextField primitive found in
mobilecomponents.swc. This class extends TextField and adds support for styles.
It is designed to be used in mobile ActionScript skins and item renderers only
and is not intended for use in MXML.

When using StyleableTextField and Label (based on FTE) simultaneously,
developers must embed fonts twice. Label uses `embedAsCFF=true` while TextField
and StyleableTextField use `embedAsCFF=false` .

Adobe also recommends against using the RichEditableText component in mobile
projects. Use the TextArea component instead.

#### MXML vs. ActionScript

Because MobileSkin removes many of the main declarative features of MXML skins
(states, layout, and MXML Graphics), Adobe recommends that skins are written in
ActionScript. Without these three features, there is only a small benefit to
using MXML for declarative markup.

**Note:** In Flash Builder, Design View can render MXML skins from source or
compiled ActionScript skins from SWCs in the library path or theme. Design View
cannot render ActionScript skins from source.

#### Unsupported global styles

The Mobile theme omits some styles based on one or more of the above
constraints. Those styles include:

- `rollOverColor` – Not supported as touch-based interfaces are the main use
  case.
- `borderAlpha` , `borderColor` , `cornerRadius` – Not supported as these
  parameters are properties of compiled FXG and are not changed at runtime.
- `dropShadowVisible` – Not supported to minimize use of filters for
  performance.
- Flash Builder correctly shows and hides style properties in MXML and CSS
  editors based on the current theme selection.

#### Components to Avoid for Mobile

Some Spark components do not have a skin in the Mobile theme for various
reasons. For example, the component may have limited utility in a mobile UI or
it simply was not targeted for the Flex 4.5 release and will be optimized for
mobile in a subsequent release of Flex. Such components include:

- ComboBox and DropDownList
- NumericStepper
- ToggleButton
- VideoDisplay and VideoPlayer
- VSlider
- Panel
- TabBar (the component used in TabbedViewNavigator is actually a ButtonBar)
- TitleWindow

### Button skin tutorial overview

One of the best places to start skinning is to create a custom Button skin based
on the default Button skin. To keep things simple, this example does not take
screen DPI into account. I'll discuss these considerations later in this article
series.

First, using Adobe Illustrator CS5, you'll create an FXG file for the Button
graphics. This one file will represent both the `up` and `down` skin states of
the Button component. Since you are targeting touch input, you won't create an
`over` state.

Second, you'll add `chromeColor` style support by overriding the
`drawBackground()` method of MobileSkin. Alternatively, you can hard code static
background colors into separate up and down graphics and override
`drawBackground()` to do nothing. The `disabled` state will simply change the
`alpha` property of the `up` state. This functionality is already built-in to
the mobile ButtonSkin.

### Tutorial step 1: Create the FXG graphic

Depending on your preference, you can use Adobe Flash Professional, Adobe
Illustrator, or Adobe Fireworks to create the FXG. You can also edit the FXG by
hand in Flash Builder, and then visually verify the results by referencing the
FXG in an MXML file and using Design View to render it.

#### Using Illustrator to create the FXG

This example uses Illustrator to create a pill-shaped Button graphic.

1.  In Illustrator, choose File \> New.
2.  Type a name for the graphic; for example, **RoundedButtonExport**.
3.  Set the New Document Profile to Flash Catalyst.
4.  Set the size to typical phone dimensions, 480 px x 800 px (just for
    context).
5.  Click OK
6.  Use the Rectangle Tool to create a rectangle with a gray stroke and gradient
    fill.
7.  In the Stroke panel, find Align Stroke and select the middle option for
    Align Stroke To Inside. By default, strokes align to center. I'll discuss
    why it's a good idea to avoid this default behavior later in the article.
8.  Change the X and Y positions to 0
9.  With the rectangle selected, choose Effect \> Stylize \> Round Corners.
10. Specify a corner radius of 22 px.
11. Save as FXG without private data.

#### Cleaning up the FXG

After exporting the FXG file, you may prefer to clean up any unnecessary markup
like extra Group tags or private namespace data. This step is not necessary, but
it helps to make FXG more human-readable.

**RoundedButtonExport.fxg**

    <?xml version="1.0" encoding="utf-8" ?>
    <Graphic version="2.0" viewHeight="800" viewWidth="480" ai:appVersion="15.0.2.399" ATE:version="1.0.0" flm:version="1.0.0" d:using="" xmlns="http://ns.adobe.com/fxg/2008" xmlns:ATE="http://ns.adobe.com/ate/2009" xmlns:ai="http://ns.adobe.com/ai/2009" xmlns:d="http://ns.adobe.com/fxg/2008/dt" xmlns:flm="http://ns.adobe.com/flame/2008">
      <Library/>
      <Group ai:seqID="1" d:layerType="page" d:pageHeight="800" d:pageWidth="480" d:type="layer" d:userLabel="Artboard 1">
    	<Group ai:seqID="2" d:type="layer" d:userLabel="Layer 1">
    	  <Group ai:seqID="3" flm:knockout="false" d:type="layer" d:userLabel="RoundedButton">
    		<Group ai:seqID="4" flm:knockout="false">
    		  <Path x="0.5" y="1" winding="nonZero" ai:seqID="5" data="M21.5 43C9.64502 43 0 33.355 0 21.5 0 9.64502 9.64502 0 21.5 0L197.5 0C209.355 0 219 9.64502 219 21.5 219 33.355 209.355 43 197.5 43L21.5 43Z">
    			<fill>
    			  <LinearGradient x="109.5" y="0" scaleX="43" rotation="90">
    				<GradientEntry ratio="0" color="#F0F0F0"/>
    				<GradientEntry ratio="0.478788" color="#C8C8C8"/>
    				<GradientEntry ratio="0.50303" color="#BBBBBB"/>
    				<GradientEntry ratio="1" color="#F0F0F0"/>
    			  </LinearGradient>
    			</fill>
    		  </Path>
    		  <Path winding="nonZero" ai:seqID="6" data="M198 1C209.58 1 219 10.4204 219 22 219 33.5796 209.58 43 198 43L22 43C10.4204 43 1 33.5796 1 22 1 10.4204 10.4204 1 22 1L198 1M198 0 22 0C9.8999 0 0 9.8999 0 22 0 34.1001 9.8999 44 22 44L198 44C210.1 44 220 34.1001 220 22 220 9.8999 210.1 0 198
     0L198 0Z">
    			<fill>
    			  <SolidColor color="#DDDDDD"/>
    			</fill>
    		  </Path>
    		</Group>
    	  </Group>
    	</Group>
      </Group>
      <Private/>
    </Graphic>

**RoundedButtonCleanup.fxg**

    <?xml version="1.0" encoding="utf-8" ?>
    <Graphic version="2.0" xmlns="http://ns.adobe.com/fxg/2008">
    		<Path x="0.5" y="1" winding="nonZero" data="M21.5 43C9.64502 43 0 33.355 0 21.5 0 9.64502 9.64502 0 21.5 0L197.5 0C209.355 0 219 9.64502 219 21.5 219 33.355 209.355 43 197.5 43L21.5 43Z">
    		  <fill>
    			<LinearGradient x="109.5" y="0" scaleX="43" rotation="90">
    			  <GradientEntry ratio="0" color="#F0F0F0"/>
    			  <GradientEntry ratio="0.478788" color="#C8C8C8"/>
    			  <GradientEntry ratio="0.50303" color="#BBBBBB"/>
    			  <GradientEntry ratio="1" color="#F0F0F0"/>
    			</LinearGradient>
    		  </fill>
    		</Path>
    		<Path winding="nonZero" data="M198 1C209.58 1 219 10.4204 219 22 219 33.5796 209.58 43 198 43L22 43C10.4204 43 1 33.5796 1 22 1 10.4204 10.4204 1 22 1L198 1M198 0 22 0C9.8999 0 0 9.8999 0 22 0 34.1001 9.8999 44 22 44L198 44C210.1 44 220 34.1001 220 22 220 9.8999 210.1 0 198
     0L198 0Z">
    		  <fill>
    			<SolidColor color="#DDDDDD"/>
    		  </fill>
    		</Path>
    </Graphic>

#### Testing the FXG

After exporting and cleaning up the FXG, you can quickly validate how it will be
rendered by dropping the component into an MXML file and previewing it in Design
View.

To try this, follow these steps:

1.  Create a new project (named whatever you like) in Flash Builder.
2.  Add the FXG file to your project source folder.
3.  Switch to Design View.

The components panel will show FXG files as custom components.

4.  Drag and drop the component onto the design area. You'll see the FXG
    rendered at its natural size.
5.  While in Design view, try resizing the FXG width.

You'll notice that the rounded corners are stretching horizontally (see Figure
1). What you really want is the shape of the corners to stay the same and
instead let the middle section of the graphic stretch. To accomplish this, you
need to add scale grid information to the FXG.

![Missing scale grid data.](mobile-skinning-part1/img/fig01.jpg)

Figure 1. Missing scale grid data.

#### Adding scale grids manually

You'll need to add left, right, top and bottom scale grid positions in order for
scaling to work as expected. Since you used the Round Corners stylize option,
you know that the radius in this case is 22px. Given this information, you can
make the following changes to the FXG file after opening it in Flash Builder:

1.  Add `scaleGridLeft="22"`, and   `scaleGridRight="198px"` (the full graphic
    width is 220px) to the root `<Graphic>` tag.

You also want to make sure the border stroke doesn't scale.

2.  For this graphic,  add `scaleGridTop="1px"` and `scaleGridBottom="43px"`
    (the full graphic height is 44px) to the same tag.

**Note:** For some graphics, it can be difficult to find the exact cut-off point
of an arbitrary path. For those cases, use the selection tool in Illustrator and
hover over an anchor point to find its coordinates (see Figure 2).

![Finding anchor coordinates.](mobile-skinning-part1/img/fig02.jpg)

Figure 2. Finding anchor coordinates.

After adding scale grid data on the root Graphic tag, if you try the new FXG in
Design View, you'll notice nothing has changed. The scale grid didn't take
effect because the path data doesn't fill the full dimensions of the graphic.

3.  To fix this, add a transparent Rect to occupy the full Graphic space as
    shown below.
4.  Save your changes as RoundedButton.fxg.

**RoundedButton.fxg**

    <?xml version="1.0" encoding="utf-8" ?>
    <Graphic version="2.0" xmlns="http://ns.adobe.com/fxg/2008" scaleGridLeft="22" scaleGridRight="198" scaleGridTop="1" scaleGridBottom="43">
    		<Path x="0.5" y="1" winding="nonZero" data="M21.5 43C9.64502 43 0 33.355 0 21.5 0 9.64502 9.64502 0 21.5 0L197.5 0C209.355 0 219 9.64502 219 21.5 219 33.355 209.355 43 197.5 43L21.5 43Z">
    		  <fill>
    			<LinearGradient x="109.5" y="0" scaleX="43" rotation="90">
    			  <GradientEntry ratio="0" color="#F0F0F0"/>
    			  <GradientEntry ratio="0.478788" color="#C8C8C8"/>
    			  <GradientEntry ratio="0.50303" color="#BBBBBB"/>
    			  <GradientEntry ratio="1" color="#F0F0F0"/>
    			</LinearGradient>
    		  </fill>
    		</Path>
    		<Path winding="nonZero" data="M198 1C209.58 1 219 10.4204 219 22 219 33.5796 209.58 43 198 43L22 43C10.4204 43 1 33.5796 1 22 1 10.4204 10.4204 1 22 1L198 1M198 0 22 0C9.8999 0 0 9.8999 0 22 0

    34.1001 9.8999 44 22 44L198 44C210.1 44 220 34.1001 220 22 220 9.8999 210.1 0 198
     0L198 0Z">
    		  <fill>
    			<SolidColor color="#DDDDDD"/>
    		  </fill>
    		</Path>
    		<!-- scale grid fix -->
    		<Rect x="0" y="0" width="220" height="44">
    		  <fill>
    			<SolidColor color="#000000" alpha="0"/>
    		  </fill>
    		</Rect>
    </Graphic>

In Design View the button will now scale properly (see Figure 3).

![The component after adding the scale grid.](mobile-skinning-part1/img/fig03.jpg)

Figure 3. The component after adding the scale grid.

### Tutorial step 2: Create the skin class

The process of creating a new Button skin based on the provided Mobile theme
ButtonSkin comprises three main steps.

1.  Create a subclass of spark.skins.mobile.ButtonSkin.
2.  In the constructor, assign `upBorderSkin` and `downBorderSkin` to the FXG
    Class. Note that these properties are typed as Class, not an instance of
    your FXG. Also assign `measuredDefaultHeight` and `measuredDefaultWidth` to
    the dimensions of the FXG (see the code snippet below for an example).
3.  Use CSS or MXML to assign `skinClass` to Button, to use your new Button
    skin.

**Note:** The sample files for this project include
RoundedButtonSkinProject.fxp, which you can import into Flash Builder to see how
a complete application, including the skin, is implemented.

To support the `chromeColor` style, you have three basic options:

- Don't support `chromeColor` – Override `drawBackground()` to do nothing
- Draw `chromeColor` to match the FXG shape – Override `drawBackground()` and
  programmatically draw `chromeColor` . Add alpha to the FXG so that the
  `chromeColor` is visible.
- Tint the FXG – Override `drawBackground()` and use the helper function
  `applyColorTransform()` to tint the FXG from its _base_ color to the specified
  `chromeColor` .

This example uses the third option, tinting.

The code below is the final version of the skin.

**RoundedButtonSkin.as**

    package skins
    {
    import skins.assets.RoundedButton;

    import spark.skins.mobile.ButtonSkin;

    public class RoundedButtonSkin extends ButtonSkin
    {
    	private var colorized:Boolean = false;

    	public function RoundedButtonSkin()
    	{
    		super();

    		// replace FXG asset classes
    		upBorderSkin = skins.assets.RoundedButton;
    		downBorderSkin = skins.assets.RoundedButton;

    		measuredDefaultHeight = 44;
    		measuredDefaultWidth = 220;
    	}

    	override protected function drawBackground(unscaledWidth:Number, unscaledHeight:Number):void
    	{
    		// omit call to super.drawBackground() to apply tint instead and don't draw fill
    		var chromeColor:uint = getStyle("chromeColor");

    		if (colorized || (chromeColor != 0xDDDDDD))
    		{
    			// apply tint instead of fill
    			applyColorTransform(border, 0xDDDDDD, chromeColor);

    			// if we restore to original color, unset colorized
    			colorized = (chromeColor != 0xDDDDDD);
    		}
    	}
    }
    }

### Tutorial Step 3: Test

The code below assigns the custom RoundedButtonSkin to be the default skin for
buttons. To demonstrate `chromeColor` support and advanced CSS, I've added some
down state-specific styles.

    <fx:Style>
    @namespace s "library://ns.adobe.com/flex/spark";
    s|Button
    {
    	skinClass: ClassReference("skins.RoundedButtonSkin");
    }

    s|Button:down
    {
    	chromeColor: #0000FF;
    	color: #FFFFFF;
    	textShadowColor: #000000;
    }
    </fx:Style>

After adding some Buttons to your application, you should see the new skin and
its up and down skin states (see Figure 4). The down skin state shows that the
advanced CSS style rule is in effect.

![Buttons with the new custom skin.](mobile-skinning-part1/img/fig04.jpg)

Figure 4. Buttons with the new custom skin.

### FXG tips and tricks

When working with FXG, keep the following tips in mind.

#### Path vs. shape primitives

Path data may not be easy to read, but in some cases, paths will render with
fewer anti-aliasing artifacts. Your mileage may vary. If you get hard-to-read
path data from Illustrator, resist the urge to simplify to shape primitives
right away.

#### blendMode

By default, on graphic elements, `blendMode` is set to `auto` . When you use
`auto` , the Flash Player or AIR runtime will correctly figure out whether or
not the element needs to use the layer `blendMode` based on the `alpha` value.

#### Avoid strokes when possible

By default, strokes will straddle the shape edges (for example, when using Align
Stroke To Center in Illustrator).

For example, take a vertical line from (0,0) to (0,100) with a 1px wide stroke.
That stroke will stretch in the x-axis from -.5 to .5. However, since nothing
can be drawn at a fractional pixel, Flash will draw an anti-aliased line two
pixels wide to approximate the fractional position. To counteract this
anti-aliasing, place your shapes with odd numbered stroke weights on a half
pixel boundary. For example, a Rect with a `SolidColorStroke` of weight `1` at
(10,10) should be placed at (10.5,10.5) instead.

If possible, use a filled Rect in place of a stroked Rect or Line. If you have a
Rect with a `fill` and a `stroke` , then try to split it into two filled Rects
(assuming that the fill is completely opaque). If you have a Line, try turning
it into a filled Rect. For example, if you have a horizontal Line, create a
filled Rect with a height equal to the Line's stroke weight.

#### Scale grid

Scale grids have some limitations that must be considered when designing
graphics using FXG.

Scale grid values must be inside the boundaries of the graphic and must not
overlap (that is, left boundary \< `scaleGridLeft` \< `scaleGridRight` \< right
boundary).

Scale grids will not work if the graphic contains any Group elements.

Scale grids will not work if elements have `alpha` applied. Instead, apply
`alpha` to the `stroke` and `fill` elements.

#### FXG specification

For complete details on FXG, see FXG 2.0 - Functional and Design Specification.

### Where to go from here

As a mobile skin developer, you should take some time to familiarize yourself
with the MobileSkin base class and the default mobile skins found in the Flex
4.5 SDK source. You can find the source files in your Flash Builder install
location; for example,
`Adobe Flash Builder 4.5/sdks/4.5.0/frameworks/projects/mobiletheme`.

[The next article](./mobile-skinning-part2.md) in this series will cover the
process for creating DPI-aware skins to scale across multiple screen sizes. The
final article will address using CSS media queries to create custom themes based
on the target platform.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
