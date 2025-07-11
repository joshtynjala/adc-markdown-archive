# Apache Flex mobile skins – Part 2: Handling different pixel densities

by Jason San Jose

## Content

- [Why does screen density matter?](#why-does-screen-density-matter)
- [Automatic application scaling](#automatic-application-scaling)
- [Flash Builder 4.5 device configurations](#flash-builder-45-device-configurations)
- [Density-specific Button skin tutorial](#density-specific-button-skin-tutorial)
- [Density-specific Splash Screen Tutorial](#density-specific-splash-screen-tutorial)

## Requirements

### Prerequisite knowledge

Familiarity with ActionScript, CSS, and Flex skinning is required. Read Part 1
of this series before completing this tutorial.

### User level

Intermediate

### Required products

- Apache Flex SDK
- Adobe Flash Builder
- Adobe Illustrator CS5

### Sample files

- [mobile-skinning-part2](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/mobile-skinning-part2)

In [Part 1](mobile-skinning-part1.html) of this series on creating Flex mobile
skins, I discussed the rationale behind the performance optimizations the Flex
team made in the Mobile theme and I walked through a simple Button skin example.
With performance being a primary concern, the next major topic for mobile
skinning is adapting the look and feel of an application across the wide variety
of mobile screens available today.

Flex 4.5 adds significant new features to support mobile devices with different
pixel densities, also known as pixels per inch (PPI) or dots per inch (DPI).
Those features include application scaling, multi-DPI bitmaps, CSS @media
query-based style rules, and a DPI-specific Mobile theme.

### Why does screen density matter?

If you're coming from a desktop Flex development background, you may be
wondering why you should care about screen density.

Today's mobile devices come in different sizes and screen DPIs (see Table 1). In
the desktop world, there are different screen sizes, but DPI values are largely
similar. So, desktop applications (native or browser-based) are typically
designed based on pixel size.

**Table 1.** Sampling of mobile device resolutions, sizes and DPI values.

| **Manufacturer** | **Device**       | **Resolution (px)** | **Diagonal Screen Size (in)** | **DPI** |
| ---------------- | ---------------- | ------------------- | ----------------------------- | ------- |
| Apple            | iPhone 4, iPod 4 | 960 x 640           | 3.5                           | 326     |
| Apple            | iPad 1, iPad 2   | 1024 x 768          | 9.7                           | 132     |
| BlackBerry       | PlayBook         | 1024 x 600          | 7                             | 170     |
| HTC              | Evo              | 800 x 480           | 4.3                           | 217     |
| Motorola         | Atrix            | 960 x 540           | 4                             | 275     |
| Motorola         | Xoom             | 1280 x 800          | 10.1                          | 150     |
| Samsung          | Galaxy Tab       | 1024 x 600          | 7                             | 170     |

The key point is that some devices pack many more pixels into the same physical
space. If you design a button to be 80 x 80 pixels in size, that button will be
approximately 0.5 inches square on a 160 DPI device, such as a BlackBerry
PlayBook or Samsung Galaxy Tab. If you run the same application on a higher DPI
device (for example, an iPhone 4), that button is now 0.25 inches square, making
it more difficult to interact with on a touch screen.

The same principle applies to font size selection, pixel-based layout, bitmap
graphics, and so on. The position and size of any on-screen element need to be
set based on the DPI of the screen. Failure to address screen density will
result in applications that look either too big or too small on many mobile
devices.

Flex 4.5 introduces several new features to address screen density. Developers
can choose between automatic application scaling for convenience or manually use
density-specific graphics and logic for design fidelity and control.

### Automatic application scaling

Flex 4.5 has support for DPI classifications. Devices come in a wide range of
pixel densities, but for the purposes of layout it makes sense to group these
into categories; for example, 238 DPI or 249 DPI displays are essentially
equivalent to a 240 DPI device. Flex supports three DPI classifications: 160,
240, and 320.

A new `applicationDPI` property has been added to the Application class and its
subclasses. When this property is set to one of the DPI classifications
specified above, your layout is assumed to be tuned for that DPI. If your
application runs on a device of a different DPI classification, your entire
application will automatically be scaled so that your UI appears at the same
approximate physical size on that device.

The sample files for this article include TwitterTrendsFinal.fxp. In this
example project, you can see that `applicationDPI` is set to 160, and various
layout values (such as the `iconWidth` and `iconHeight` in the TweetsView list
as well as the padding/gaps in the UserInfoView) have been sized appropriately
for 160 DPI screens. On a 160 DPI device, such as the Motorola Droid Pro, the
application is sized exactly as specified. On a 240 DPI device, such as the
Droid X or Droid 2, the application scales uniformly by 150%. Figure 1 compares
the size of a 160 DPI screen to a 240 DPI screen in absolute pixels. Figure 2
shows the same screenshots, but the 240 DPI image has been scaled down to show
approximate physical size. Internally to your MXML and ActionScript code,
however, all values are their original 160 DPI values; the scaling is applied at
the stage level.

![Actual pixels at 320 x 480 160 DPI (left) and 480 x 800 240 DPI (right).](mobile-skinning-part2/img/fig01.png "Actual pixels at 320 x 480 160 DPI (left) and 480 x 800 240 DPI (right).")

Figure 1. Actual pixels at 320 x 480 160 DPI (left) and 480 x 800 240 DPI
(right).

![Physical size (approx.) at 320 x 480 160 DPI (left) and 480 x 800 240 DPI (right).](mobile-skinning-part2/img/fig02.png "Physical size (approx.) at 320 x 480 160 DPI (left) and 480 x 800 240 DPI (right).")

Figure 2. Physical size (approx.) at 320 x 480 160 DPI (left) and 480 x 800 240
DPI (right).

Note that in general, it is better to scale upwards rather than downwards, due
to artifacts with vector rendering. So it's best to target a lower density (such
as 160 DPI) and let the application scale up on higher-density devices.

#### Density-specific bitmaps

When using `applicationDPI` to scale the application automatically for different
densities, text is not included in vector scaling; instead, the font size is
scaled, so it's rendered with high fidelity at different sizes. Bitmaps,
however, are scaled. This is not ideal, as upscaling bitmaps typically produces
unacceptable artifacts. To alleviate this, Flex 4.5 includes a new
MultiDPIBitmapSource class. This class has three properties (`source160dpi`,
`source240dpi`, and `source320dpi`). It uses the most appropriate bitmap for
each device based on its actual DPI. For example, suppose you have a button icon
that should be 24 x 24 at 160 DPI. Instead of directly specifying a single asset
as the icon property of your Button component, you can set the `icon` property
to a MultiDPIBitmapSource instance that additionally refers to a 36 x 36 icon to
use at 240 DPI and a 48 x 48 icon to use at 320 DPI. The code looks like this:

    <s:Button>
    	<s:icon>
    		<s:MultiDPIBitmapSource
    			source160dpi="@Embed('/assets/refresh160.png')"
    			source240dpi="@Embed('/assets/refresh240.png')"
    			source320dpi="@Embed('/assets/refresh320.png')"/>
    	</s:icon>
    </s:Button>

Continuing the scenario above, suppose that you had set `applicationDPI` to 160,
and the application is running on a 320 DPI device. Even though vectors will be
scaled by 200%, if you specify a multi-DPI bitmap, Flex will not scale the 160
DPI bitmap up by 200%; instead, it will use the 320 DPI bitmap, and set up the
proper transforms on the bitmap so that it appears at the proper size with all
its pixels unscaled.

You can specify a MultiDPIBitmapSource object almost anywhere you can use an
ordinary embedded or dynamically loaded image reference—for example, as the
`source` property of an Image, or as the `icon` property of a Button. The one
exception is that you cannot use a MultiDPIBitmapSource object as the splash
screen of an application; in that case, you will need to specify a
high-resolution bitmap and let Flex downscale it for lower-density devices.

#### Application scaling summary and considerations

If you want to use application scaling, you'll need to:

- Create a single set of skins and view/component layouts targeted at the
  `applicationDPI` value you specify. Adobe recommends using a value of 160 to
  scale up 1.5x and 2x to 240 DPI and 320 DPI respectively.
- Create multiple versions of any bitmap asset used in your skins or elsewhere
  in your application, and specify them using MultiDPIBitmapSource. Vector
  assets and text in your skins do not need to be density-specific if you are
  using application scaling.
- Ignore `@media` rules since your application only needs to consider a single
  target DPI.
- Test your application on devices of different densities in order to ensure
  that the appearance of the scaled application is acceptable on each device. In
  particular, check devices that will require scaling factors that are not
  integers. For example, if `applicationDPI` is 160, check your application on
  240 DPI devices.

One potential drawback when using scaling is that there can be visual artifacts.
When the scale factor is not an integer (for example, when scaling up 1.5 times
from 160 to 240), this can cause visual artifacts (in some cases) with vector
artwork. Particularly, strokes may exhibit blurred lines or be slightly offset
depending on stroke width, stroke alignment, and the pixel snapping performed by
Flash Player.

Text and Bitmaps are generally not a concern. Text should not exhibit any
artifacts because text is scaled by scaling the font size value rather than
scaling the actual vectors. Bitmaps should not exhibit scaling artifacts when
using MultiDPIBitmapSource to specify appropriate bitmaps for each DPI.

In practice, the Flex team has found only minor visual differences between using
160 DPI skins scaled up to 240 DPI and using 240 DPI skins unscaled. For most
applications, this approach should work well, and you can stop reading right
here. However, if you find unacceptable visual artifacts form scaling, or have
other reasons why you need more detailed control over the behavior of your
application at different densities, you can manage density manually, as
described in the next section.

### Density-specific skins and styles

If you do not explicitly set `applicationDPI`, the value will be set to the DPI
classification of the device. The Mobile theme shipped with Flex 4.5 chooses
appropriate graphic assets, as well as the positioning and sizing of component
skins based on the `applicationDPI` value. For non-skinned elements such as
layout elements, it is assumed that your application will automatically manage
layout values for different densities. In particular, any pixel values in your
code should be dynamically calculated at runtime based on `applicationDPI`
(through imperative code or data binding), and you should take advantage of
MultiDPIBitmapSource for bitmaps.

#### CSS `@media` queries for density-specific styles

Flex 4.5 introduces a limited set of CSS media query functionality used to
declare CSS rules that apply only at certain densities. For example, the Mobile
theme uses DPI-specific rules to specify default font sizes for the ActionBar
title:

    ActionBar #titleDisplay { fontSize: 24pt; }
    @media (application-dpi: 160)
    {
    	ActionBar #titleDisplay { fontSize: 18pt; }
    }
    @media (application-dpi: 320)
    {
    	ActionBar #titleDisplay { fontSize: 36pt; }
    }

The first rule is intended for 240 DPI; the other two rules override the first
for 160 DPI and 320 DPI.

The DPI classification used to match these @media rules is the current effective
DPI of the application. If `applicationDPI` is set on the `Application` tag,
then that is used; otherwise, the DPI classification corresponding to the
runtimeDPI of the device is used. For example, if `applicationDPI` is 160, and
the application is running on a 320 DPI device, the `application-dpi`:160 rule
will be selected, causing Flex to set the text to be 18pt. This text will then
be automatically scaled by a factor of two, so the final text will actually
render at 36pt. In this case, the 320 DPI `@media` rule is actually never used.
However, if you do not specify `applicationDPI`, and you run the application on
a 320 DPI device, then the 320 DPI `@media` rule will be used.

#### Density-specific skinning and styling considerations

If you choose not to use autoscaling by leaving the Application `applicationDPI`
unset, your application will need to read the `applicationDPI` value at runtime
in your skins and layouts to determine the actual DPI classification of the
device. You'll then need to implement density-specific skinning and styling
using the following techniques:

- Make multiple sets of skins and view/component layouts targeted at each
  runtime DPI classification, or make a single set of skins and layouts that
  dynamically adapts to different densities. The Mobile theme skins take the
  latter approach—each skin class checks `applicationDPI` and configures itself
  appropriately.
- Use @media to filter CSS rules based on the device's DPI classification.
  Typically, you will customize font sizes and padding values for each DPI.
- Create multiple versions of any bitmap asset used in your skins or elsewhere
  in your application, and specify them using MultiDPIBitmapSource. Vector
  assets and text in your skins do not need to be density-specific if you are
  using application scaling.

Test your application on devices of different densities to ensure your skins and
layouts are properly adapting.

### Flash Builder 4.5 device configurations

Flash Builder 4.5 supports the Flex 4.5 density-specific features in both Design
View and also when launching the Adobe Debug Launcher (ADL) utility. While
previewing in Design View or simulating with ADL are both quick and easy,
launching applications on-device will always provide the most accurate
representation of your application.

#### Creating a new mobile project in Flash Builder

When creating a Flex Mobile project in Flash Builder, you can enable autoscaling
in the project wizard. You can always edit your application file to add (or
remove) `applicationDPI` after the fact.

#### Authoring applications in Flash Builder

Flash Builder 4.5 also includes a new Device Configurations preference page,
which you can use to specify a pixel density and resolution for any arbitrary
device. The built-in configurations already have the proper pixel density and
resolution specified for their corresponding devices. This means that when you
choose a device to preview in Design View, or launch an application on the
desktop using ADL, it will properly simulate what would happen at runtime if you
were to run the application on that device.

For example, if your `applicationDPI` is 160 and you choose Google Nexus S as
your preview device in Design View, then Design View will scale your application
150% and display it at the dimensions of the Nexus S. This means that one pixel
in your Flex code now corresponds to 1.5 pixels in Design View (see Figure 3).

![Using device configurations in Design View](mobile-skinning-part2/img/fig03.png "Using device configurations in Design View")

Figure 3. Using device configurations in Design View

Edits you make in the Properties View are not affected by autoscaling, since
these edits actually modify the underlying code. For example, in the above
scenario, if you set the width of a component to 100 in the Properties View, it
will actually appear 150 pixels wide in Design View, just as it would on a 240
DPI device.

Similarly, running the application on the desktop in ADL simulating a Nexus S
will also scale the application up 150%, displaying it at the correct scale
within the Nexus S screen. Both Design View and ADL will properly show the
effects of @media queries and MultiDPIBitmapSource as well.

Using the Properties View, you can set multiresolution bitmaps anywhere they're
allowed. To do so, select a component (such as a Button). Next to the icon
setting in the Properties View, the browse folder is now a dropdown that lets
you choose either a single bitmap or a multiresolution bitmap. Choosing the
latter will bring up a dialog box that you can use to specify bitmaps for each
DPI classification (see Figure 4).

![The Multiresolution Bitmap dialog box.](mobile-skinning-part2/img/fig04.jpg "The Multiresolution Bitmap dialog box.")

Figure 4. The Multiresolution Bitmap dialog box.

#### Testing

If you have devices with different pixel densities, you can try out these
density independence features for yourself. For example, if you have a Motorola
Droid Pro or a Samsung Galaxy Tab (DPIClassification 160) and another modern
Android phone (DPIClassification 240), you can write an app with
`applicationDPI`="160" and run it on both phones, and you should see that UI
components on each device appear similarly-sized when compared side-by-side. (Of
course, the Galaxy Tab also has a physically larger screen, so in that case your
overall UI will be resized larger than it would be on a phone.)

If you don't have phones of different densities, you can still experiment with
these features when running on the desktop from Flash Builder. For example,
Flash Builder 4.5 now has a built-in device configuration for Droid Pro. You can
simply choose that device configuration in the Run Configurations (see Figure 5)
or Debug Configurations dialog box. ADL will pass the screen density specified
in the device configuration to the Flex application, so it will exhibit the same
scaling behavior you would see on a device.

![The Run Configurations dialog box.](mobile-skinning-part2/img/fig05.png "The Run Configurations dialog box.")

Figure 5. The Run Configurations dialog box.

### Density-specific Button skin tutorial

Since automatic scaling requires very little work, the tutorial for this article
focuses on the steps required to write a density-specific skin. The goal is to
have high-fidelity graphics at each DPI. Therefore, you'll be using three
versions of a button graphic. Instead of creating a fully rounded, pill-shaped
button as in Part 1 of this series, this tutorial will use a back button
graphic. I'll leave out the Adobe Illustrator details this time for brevity.

#### Step 1: Create FXG Graphics

For this tutorial, you can start with the premade graphic (see Figure 6) in the
DensitySpecificButtonTutorial.fxp sample project. You'll find the starter FXG
files in the skins.assets160 package.

![The transparent, rounded button.](mobile-skinning-part2/img/fig06.png "The transparent, rounded button.")

Figure 6. The transparent, rounded button.

I covered the basics of creating FXG files with Illustrator CS5 in Part 1. The
additional work needed to create graphics for a density-specific skin includes:

- Scaling the original graphic for 160, 240, and 320 DPI values. You'll start
  with a 160 DPI graphic and transform it in Illustrator by using a scale
  factor.
- Update scale grid information for each DPI.

You might wonder why you want to scale in Illustrator if you can scale the
graphic programmatically at runtime. There are two reasons:

- Design fidelity - A designer might find that simply scaling a 160 DPI graphic
  isn't acceptable. Perhaps a hairline stroke, corner radius parameters, pixel
  snapping, or shape detail can be improved at a higher DPI.
- Complexity – Applying scaling (for DPI) and resize transforms (to shrink/grow
  using scale grid data) together add more complexity. Having DPI-specific
  graphics that resize with scale grids can be done with a simpler, more
  predictable approach.

Scaling a graphic in Illustrator is deceivingly simple (see Figure 7):

![Transform Panel in Adobe Illustrator CS5](mobile-skinning-part2/img/fig07.png "Transform Panel in Adobe Illustrator CS5")

Figure 7. Transform Panel in Adobe Illustrator CS5

1.  Find the Transform Panel in the toolbar.
2.  Scale the width and height values for each scale factor. If you're starting
    with a 160 DPI graphic, scale by 1.5x and 2x for 240 and 320 DPI
    respectively.
3.  Review the graphic and alter it as necessary.

In the Mobile theme library, the graphic assets are organized in package names
based on DPI. You'll find assets in the spark.skins.assets160,
spark.skins.assets240, and spark.skins.assets320 packages in mobiletheme.swc. In
practice, some minor design tweaks were needed after scaling, particularly with
hairline stroke widths, to pass the design review. You may not need any
modifications after scaling, but it's a good idea to check. I said that scaling
was deceivingly simple earlier for a reason. When scaling the 160 DPI graphics
in the example project, you'll see that Illustrator will move the x and y
positions of the outer stroke from half-pixel alignment (0.5,0.5) to (0.75,0.75)
and (1,1) for 240 and 320 DPI values respectively. If you want to maintain a
hairline stroke with pixel snapping across all three DPI values, you have to
adjust the x and y positions back to (0.5,0.5) and make adjustments to the width
and height to compensate.

Before moving on to the ActionScript skin code, you'll still need to repeat two
steps explained in Part 1, specifically:

- (Optional) Clean up FXG for readability
- Remove Groups and update scale grid values

#### Step 2: Add `applicationDPI` logic

Once you have your DPI-specific FXG graphics and your package structure, you can
tailor the graphics in your skin based on the value of `applicationDPI`. The
MobileSkin base class exposes `applicationDPI` as a property. You initialize the
graphics classes in the constructor as follows:

    public class TransparentRoundedButtonSkin extends ButtonSkin
    {
    	public function TransparentRoundedButtonSkin()
    	{
    		super();

    		switch (applicationDPI)
    		{
    			case DPIClassification.DPI_320:
    			{
    				upBorderSkin = skins.assets320.TransparentRoundedButton_up;
    				downBorderSkin = skins.assets320.TransparentRoundedButton_down;

    				break;
    			}
    			case DPIClassification.DPI_240:
    			{
    				upBorderSkin = skins.assets240.TransparentRoundedButton_up;
    				downBorderSkin = skins.assets240.TransparentRoundedButton_down;

    				break;
    			}
    			default:
    			{
    				// default DPI_160
    				upBorderSkin = skins.assets160.TransparentRoundedButton_up;
    				downBorderSkin = skins.assets160.TransparentRoundedButton_down;

    				break;
    			}
    		}
    	}
    }

#### Step 3: Add CSS @media rules for each DPI classification

So far, not much has changed in the approach to skinning from Part 1. That's
good news! The last step for the Button skin is to pick appropriate font sizes
based on the DPI classification. The mobiletheme.swc defaults.css file already
specifies a global style rule that sets default global font sizes as follows:

- 160 DPI – 16
- 240 DPI – 24
- 320 DPI – 32

To override the default font sizes, create new style rules in your CSS file or
fx:Style block; for example:

    @media (application-dpi: 160)
    {
    	Button
    	{
    		fontSize: 20;
    	}
    }

    @media (application-dpi: 240)
    {
    	Button
    	{
    		fontSize: 30;
    	}
    }

    @media (application-dpi: 320)
    {
    	Button
    	{
    		fontSize: 40;
    	}
    }

### Density-specific Splash Screen Tutorial

The Flex 4.5 framework introduced the ability for a mobile application to
display a splash screen when an application is launched. This feature allows
developers to customize their application's startup experience by showing
branding information, such as a logo, while the Flex application initializes.

Showing a splash screen is simple and works with all Flex Application classes
(Application, ViewNavigatorApplication and TabbedViewNavigatorApplication). To
enable this feature, an application only needs to assign the `splashScreenImage`
property in the main Application file to an embedded image or graphic.

    <?xml version="1.0" encoding="utf-8"?>
    <s:Application xmlns:fx="http://ns.adobe.com/mxml/2009"
    			   xmlns:s="library://ns.adobe.com/flex/spark"
    			   splashScreenImage="@Embed('assets/SplashScreen.png')"
    			   splashScreenMinimumDisplayTime="2000"
    			   splashScreenScaleMode="zoom">

    </s:Application>

The example above will cause the application to display the SplashScreen.png
image when the application starts up. Once the application is fully loaded, the
image will be removed and the application will be ready for use by the user. An
application can use the `splashScreenMinimumDisplayTime` property to force the
splashScreen to be visible for at least the specified time. This allows you to
ensure that even on faster devices, the image is visible for a minimum amount of
time.

As stated in this document, one of the main challenges of mobile development is
producing an application that can adapt to different devices. Since the
characteristics of each device can be different, an application may need to
tweak its user interface to produce an ideal experience. One of the main
differentiating factors is the screen resolution of a device. Since each device
supports a different screen resolution, it's hard to create a single image that
displays properly on all devices. To help solve this, SplashScreen supports the
ability to resize itself based on the resolution of the device it's run on.
Using the `splashScreenScaleMode` property, the developer can select the scaling
behavior exhibited by the splash screen. The possible values are:

- **none** (default): The splash screen will not scale but will be centered
- **letterbox**: The splash screen will scale to the available screen size of
  the device while maintaining its aspect ratio
- **stretch**: The splash screen scales to fill the dimensions of the screen.
  The aspect ratio is not maintained.
- **zoom**: The splash screen scales until it completely fills the available
  screen size while maintaining the aspect ratio. As a result, part of the image
  can go outside the bounds of the screen.

#### Customizing the SplashScreen to Support DPI

Scaling the splash screen can help produce an experience that works well on some
devices, but when pixel density changes between screens, simply resizing an
image isn't necessarily going to produce the best result. Scaling can cause an
image to appear too big, too small or distorted on some devices. In addition,
scaling can sometimes lead to visual artifacts and pixilation. If more control
is necessary, an application may want to display a different splash image based
on the density of the device. This allows a more refined experience, and can
produce much better results. The framework can easily be extended to support
this by writing a custom SplashScreen preloader. By doing so, the preloader can
decide what image to show when the application is launching. The process for
doing this is outlined below:

#### Step 1: Create a custom object that extends the spark.preloaders.SplashScreen component and embed your graphical assets

    package
    {
    import spark.preloaders.SplashScreen;
    public class MultiDPISplashScreen extends SplashScreen
    {
    	[Embed(source="assets/splash160.png")]
    	private var SplashImage160:Class;

    	[Embed(source="assets/splash240.png")]
    	private var SplashImage240:Class;

    	[Embed(source="assets/splash320.png")]
    	private var SplashImage320:Class;

    	public function MultiDPISplashScreen()
    	{
    		super();
    	}
    }
    }

#### Step 2: Override the getImageClass method and return the appropriate embedded asset based on the dpi of the device

    package
    {
    import spark.preloaders.SplashScreen;
    import mx.core.DPIClassification;
    import mx.core.mx_internal;
    use namespace mx_internal;
    public class MultiDPISplashScreen extends SplashScreen
    {
    	[Embed(source="assets/splash160.png")]
    	private var SplashImage160:Class;

    	[Embed(source="assets/splash240.png")]
    	private var SplashImage240:Class;

    	[Embed(source="assets/splash320.png")]
    	private var SplashImage320:Class;

    	public function MultiDPISplashScreen()
    	{
    		super();
    	}

    	override mx_internal function getImageClass(dpi:Number, aspectRatio:String):Class
    {
    		if (dpi == DPIClassification.DPI_160)
    			return SplashImage160;
    		else if (dpi == DPIClassification.DPI_240)
    			return SplashImage240;
    		else if (dpi == DPIClassification.DPI_320)
    			return SplashImage320;
    		return null;
    	}
    }
    }

#### Step 3: Assign your custom splash screen preloader as the application's preloader

    <?xml version="1.0" encoding="utf-8"?>
    <s:Application xmlns:fx="http://ns.adobe.com/mxml/2009"
    			   xmlns:s="library://ns.adobe.com/flex/spark"
    			   preloader="MultiDPISplashScreen">
    </s:Application>

When the custom preloader is run, it will display a different embedded asset
depending on the dpi of the device. Notice that there are no limitations to the
heuristic you use in the getImageClass method. For example, you could extend the
example above to use the aspectRatio parameter and the stage dimensions to show
different images based on the orientation of the device and whether the
application is running on a tablet.

### Where to go from here

When designing multiscreen applications with Flex 4.5, you can choose between
convenience and pixel-perfect control. Before building your application,
consider how screen density factors into your design and make the most of the
tools that Flex provides.

[Part 3](mobile-skinning-part3.html) of the series covers theme authoring and
platform-specific skinning.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
