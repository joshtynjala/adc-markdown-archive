# Delivering video and content for the Flash Platform on TV

## Content

- [Introducing stage video](#introducing-stage-video)
- [The StageVideo API](#the-stagevideo-api)
- [Video encoding guidelines](#video-encoding-guidelines)
- [Video delivery guidelines](#video-delivery-guidelines)
- [Supporting legacy content on Google TV](#supporting-legacy-content-on-google-tv)
- [General optimizations for the Flash Platform](#general-optimizations-for-the-flash-platform)
- [Authoring content for multiple device types and screen sizes](#authoring-content-for-multiple-device-types-and-screen-sizes)
- [Video player and rendering optimizations](#video-player-and-rendering-optimizations)
- [User interface guidelines](#user-interface-guidelines)
- [Content protection guidelines](#content-protection-guidelines)
- [Customize based on target device for Flash Player](#customize-based-on-target-device-for-flash-player)
- [Where to go from here](#where-to-go-from-here)

## Requirements

### Prerequisite knowledge

You should have a solid understanding of delivering video using Flash, including
the ActionScript APIs for video delivery and playback. It would also be helpful
to know the basics of HTTP, video delivery network protocols, and video encoding
standards.

### User level

Intermediate

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)
- Flash Builder or Adobe Animate (formerly Flash Professional)

The Adobe Flash Platform is the leading platform for video and rich interactive
content on the web. Now Adobe is teaming up with companies like Google and
Samsung to bring Adobe Flash Player 10.1 and Adobe AIR 2.5 to television sets so
that consumers can enjoy this same video content in their living rooms. This
article contains recommendations for adapting your content to perform optimally
on televisions.

To help ensure that your video content performs optimally, you should do the
following:

- Use the new StageVideo API
- Verify that your video stream is encoded in a format that is appropriate for
  televisions
- Optimize the performance of your video player and of your application, both
  for new and legacy content
- Ensure that the user interface of your application is appropriate for
  televisions
- Understand the content protection options for your video
- Add conditional logic to your website so that you can customize your video
  stream and video player depending on the target device

### Introducing stage video

Up until now, video in Flash Player has been rendered using the Video object in
ActionScript 3. The Video object is treated the same as any other object on the
stage, which gives developers an unprecedented amount of creative control. For
example, video can be displayed on each face of a spinning cube, or multiple
videos can be blended together with one another.

To support that level of creative control, the Flash runtime must do a
significant amount of processing for each video frame. Depending on the power of
the underlying device, this increased processing may decrease the frame rate of
the video, or it may increase the load that Flash places on the CPU.

To solve for this problem, Adobe has introduced a new way to render video. This
new approach, called "stage video," takes full advantage of the underlying video
hardware. The resulting much lower load on the CPU translates into higher frame
rates on less-powerful devices and also less memory usage.

The performance benefits of stage video are especially pronounced for
televisions and set-top boxes. Those devices do not have CPUs that are as
powerful as desktop computers, but they do have very powerful video decoders
capable of rendering high-quality video content with very little CPU usage.

With this new stage video approach, the video is rendered onto a StageVideo
object instead of a Video object. The StageVideo object is always displayed in a
window-aligned rectangular region of the screen. Although other graphics may be
layered on top of the StageVideo object, you cannot layer objects behind the
video. Furthermore, the following features are not available when the StageVideo
object is used:

- The StageVideo object cannot be rotated.
- The StageVideo object may not have a colorTransform or 3D transformations
  transform applied to it. It may not have a matrix transform that skews the
  video.
- The StageVideo object cannot have an alpha channel, blendMode, filter, mask,
  or scale9Grid applied to it.
- The video data cannot be copied into a BitmapData object.
- The video data must not be embedded in the SWF file. Stage video can only be
  used with videos originating from a NetStream object.
- Depending on the underlying hardware, some color spaces may not be supported.
  In those cases, the Flash runtime will choose a substitute color space. The
  new StageVideo ActionScript API provides a means to query the color space that
  is being used.
- To ensure compatibility between Flash Player on desktop and on TV devices, use
  `wmode=direct`.
- Avoid layering `wmode=transparent` SWF files on top of each other. Platforms
  such as Google TV do not support `wmode=transparent`. This means that all
  Flash instances are supported in `wmode=window` regardless of the embed tag
  parameters.

In practice, none of the above restrictions will affect the most common use
case, which is a video player application. In cases where these restrictions are
acceptable, developers are strongly encouraged to use the StageVideo object.

Stage video is supported on Flash Player 10.1 beta on
[Google TV](https://www.google.com/tv/), on all AIR for TV platforms, and it
will soon be included on all platforms that support Flash Player. The API
examples in this document are specific to those platforms, and the API is
evolving and may change in future version of Flash Platform products.

### The StageVideo API

Flash Player includes a new class called StageVideo which represents a single
video display instance in the hardware video plane. StageVideo objects are
created by the Flash runtime and cannot be instantiated on their own. StageVideo
objects can be accessed from the Stage object as such:

    var v:Vector.<StageVideo> = stage.stageVideos;
    var stageVideo:StageVideo;
    if ( v.length >= 1 ) {
    	stageVideo = v[0];
    }

The length of the `Stage.stageVideos` vector will vary depending on platform and
hardware availability when the stageVideos property is accessed. The length of
the vector will sometimes be zero. Developers are required to check for this
condition, as content is sure to break otherwise. If a StageVideo object is not
available, then the Video object should be used instead.

Once you have obtained a StageVideo object, you can move and resize it.
Coordinates are always in Stage coordinate space as the StageVideo object
essentially exists as a child of the Stage and is not part of the standard
display list. Sample code:

`stageVideo.viewPort = new Rectangle(10,10,320,240);`

Before displaying a video stream, the developer should add a listener for the
`StageVideoEvent.RENDER_STATE` event. Here is the skeleton for an event handler:

    function onStageVideoEvent(event:StageVideoEvent)
    {
    	if ( event.status == StageVideoEvent.RENDER_STATUS_UNAVAILABLE ) {
    		// The video hardware stopped showing video. On the desktop you
    		// would want to fall back to use a normal Video object
    		// to continue rendering video at a cost of performance.
    	}
    	if ( event.status == StageVideoEvent.RENDER_STATUS_SOFTWARE) {
    		// The video is decoded using a software video decoder.
    	}
    	if ( event.status == StageVideoEvent.RENDER_STATUS_ACCELERATED ) {
    		// The video is decoded using a hardware based video decoder.
    	}
    }
    stageVideo.addEventListener(StageVideoEvent.RENDER_STATE, onStageVideoEvent);

To display a video stream in a StageVideo, use the `attachNetStream` method; it
is analogous to the `Video.attachNetStream` method:

`stageVideo.attachNetStream(myNetStream);`

The StageVideo API offers a few methods to track capabilities before any of the
above APIs are used. For example, we can query what video color spaces the
StageVideo object is able to display. The API returns a vector of strings
usually containing "BT.601" and "BT.709."

`var colorSpaces:Vector.<String> = stageVideo.colorSpaces;`

Here is an application fragment that uses the new StageVideo API:

    	public class SamplePlayer extends Sprite
    	{
    		public var netStream:NetStream;
    		public var netConnection:NetConnection;
    		public var video:Video;

    		public function SamplePlayer()
    		{
    			netConnection = new NetConnection();
    			netConnection.connect(null);
    			netStream = new NetStream(netConnection);

    			video = new Video(320,240);
    			video.attachNetStream(netStream); // might be overridden later

    			// Set up normal display list properties
    			video.x = 100;
    			video.y = 100;
    			video.width = 320;
    			video.height = 240;
    			addChild(video);

    			// Try to render as stage video
    			var v:Vector.<StageVideo> = stage.stageVideos;
    			if ( v.length >= 1 ) {
    				var stageVideo:StageVideo = v[0];
    				stageVideo.viewPort =
    					new Rectangle(100,100,320,240);
    				stageVideo.addEventListener(
    					StageVideoEvent.RENDER_STATE,renderStateEventHandler);
    				stageVideo.attachNetStream(netStream);
    			}

    			netStream.play("MyVideo.f4v");
    		}

    		public function renderStateEventHandler(event:StageVideoEvent):void
    		{
    			// If video is not rendered properly let the
    			// video object in the display list try to render it.
    			if ( event.status == StageVideoEvent.RENDER_STATUS_UNAVAILABLE ) {
    				video.attachNetStream(netStream);
    			}
    		}
    	}

### Video encoding guidelines

When streaming video to a TV device, Adobe recommends the following encoding
guidelines:

- **Video codec:** H.264, Main or High profile, progressive encoding
- **Resolution:** 720i, 720p, 1080i, or 1080p
- **Frame rate:** 24 or 30 frames per second
- **Audio codec:** AAC-LC or AC3, 44.1 kHz, stereo
- **Combined bit rate:** up to 2Mbps (or higher depending on available
  bandwidth)
- **Audio bit rate:** up to 192 kbps
- **Pixel aspect ratio:** 1 × 1

Flash Player also supports video that is encoded using the Sorenson Spark or On2
VP6 codecs. However, those codecs are not fully supported by the video hardware,
so they will play at a much lower frame rate. As a result, H.264 should be used
if at all possible.

These video encoding guidelines are not appropriate for some of the other
devices that support Flash Player, such as smartphones. Therefore, your website
should include some conditional logic that delivers different video streams to
different target devices. Additional detail is provided later in this document.

To read more about optimizing video with Flash Player, do not hesitate to check
the document entitled
[Delivering video for Flash Player 10.1 on mobile devices](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/devices/articles/delivering_video_fp10.1.html).
From mobile video encoding guidelines to video player optimizations, the
document covers general optimizations interesting for any Flash developer
working with video.

### Video delivery guidelines

Given the largely unreliable quality of the Internet connections that run into
your users' homes, your video delivery system needs to have some sort of an
adaptive bitrate policy, regardless of device type. For different reasons, this
applies for video to deliver to living room as it does on mobile. In the former
case, as other users start to use the same Internet connection, network
congestion increases; in the latter case, as users move around, the quality of
the signal changes. In both cases, the available bandwidth changes throughout
the time a video is playing back.

Technologies such as Adobe Flash Media Server 4 with adaptive bitrate
capabilities, as well as recovery during network outages, allows for a
high-quality user experience while the network capabilities vary during
playback. Features in Flash Player on Google TV and AIR on Samsung devices
enable this adaptive bitrate playback. In order to take advantage of the various
technologies, as well as take advantage of extensive testing already
accomplished, Adobe strongly recommends using the
[Open Source Media Framework](https://web.archive.org/web/20151003093740/http://www.opensourcemediaframework.com/)
(OSMF) for the basis of any video player going towards an embedded device. Adobe
provides OSMF
[free of charge](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/video/articles/osmf_overview.html),
enabling video players to be quickly enabled to take advantage of all the
various Flash Platform capabilities, including adaptive bitrate and our content
protection solutions.

For the delivery of the content, there are a variety of protocols that are
available in the Flash Platform to enable you to move video over the network:

- HTTP Dynamic Streaming (F4F format)
- RTMP/e Streaming
- HTTP Progressive Download
- RTMFP Peer-to-Peer
- RTMFP Multicast

The choices above give you the flexibility to use your existing delivery
infrastructure to target TV devices.

### Supporting legacy content on Google TV

Going forward, Flash Player will support both the Video object and the
StageVideo object. If a piece of conent displays video in a window-aligned
rectangular region on the screen, then you should use the StageVideo object; if
a piece of content performs complex transformations on the video, then the
slower Video object should be used instead.

Because the StageVideo object is new, all existing content on the web uses the
Video object. When that content is displayed on a high-resolution device using a
less-powerful CPU, the result is sometimes less than ideal.

To solve for this problem, Adobe has decided to make a short-term trade-off. On
the Google TV and AIR for TV platforms, the Video object will be rendered using
the same code path that is used by the StageVideo object. As a result, the Video
object will enjoy the same performance as the StageVideo object but it will be
subject to the same restrictions. If any of the unsupported features (listed in
the "Introducing stage video" section of this article) are applied to the Video
object, then they are simply ignored. For example, if the Video object's `alpha`
property is set to 0.5, the video is rendered as if the `alpha` property were
set to 1.

If you attempt to simultaneously play multiple videos in multiple Video objects,
then only one or two will play at a time. Each call to `NetStream.play()` will
cause a new video to begin playing, and other videos will just stop.

In future releases, Adobe intends to restore the original, intended behavior of
the Video object. When the original behavior of the Video object is restored, it
will render more slowly. StageVideo, on the other hand, will always maximize
performance, maximize quality, and minimize power consumption on all platforms.
Therefore, the StageVideo object should be used for any video content that does
not depend on complex transformations or rendering effects.

### General optimizations for the Flash Platform

This document covers specific optimizations for developers targeting Flash
Player on Google TV or Adobe AIR on TV. Note that another document entitled
[Optimizing performance for the Adobe Flash Platform](http://help.adobe.com/en_US/as3/mobile/index.html)
contains a lot of general optimizations that are true in any embedded systems
context. Do not hesitate to read it; several advanced techniques are covered,
ranging from object pooling to manual bitmap caching. Those techniques will
improve the performance of your content when viewed in Flash Player.

Flash Player and AIR on devices, such as Google TV or Samsung Internet-connected
TVs, support hardware acceleration for H.264 video. As an ActionScript developer
targeting GPU-accelerated devices, you should be aware of things to avoid in
order to get the best performance possible. The document
[Flash Player 10.1 hardware acceleration for video and graphics](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/flashplayer/articles/fplayer10_1_hardware_acceleration.html)
sheds some light on the technical details of hardware acceleration in the Flash
runtime.

Many resources are available to you to see how improve the performance of
ActionScript. Adobe suggests to use ActionScript 3 on embedded platforms, as
Flash can take advantage of the improved performance characteristics that
ActionScript 3 enables:

- [Getting started with ActionScript 3](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/actionscript/getting_started.html)
- [Optimizing performance for the Flash Platform](https://help.adobe.com/en_US/as3/mobile/)

### Authoring content for multiple device types and screen sizes

As Flash Player and AIR proliferate to various screens and on to various kinds
of devices, Flash content needs to be aware of the fact that it will run on any
kind of screen, whatever its size. An AIR application can be designed to run on
a mobile phone, a desktop computer, a TV, or a tablet. Someobody who visits your
website may be visiting from an Android phone, a Google TV, or a desktop
computer.

Being aware of the differences in all these devices, yet leveraging the Flash
Platform, enables you to build a multiscreen application. These applications
respect the soul of the device, but let you, the content developer, share much
of your code in order to reduce the costs of optimizing to that device—or, at
best case, do nothing at all.

Consider a TV screen. TVs offer a small number of screen resolutions for content
to support, namely 960 × 540, 1280 × 720 (720p) or 1920 × 1080 (1080p). Even
with three basic TV screen resolutions, there are many input methods (mouse,
remote control, free motion wand, hybrid keyboards, and more) and CPU/GPU
performance characteristics that mean that your content may need to address
differences in order to be accessible to your users.

To make sure your video content for Flash handles different screen sizes and
resizing common behaviors properly, make sure you read the document
[Authoring mobile Flash content for multiple screen sizes](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/flash/articles/authoring_for_multiple_screen_sizes.html),
which covers the essentials.

### Video player and rendering optimizations

In addition to encoding your video stream properly, it is also important to
optimize your video player and the content around it. If your video player uses
too many CPU cycles, then it may affect the frame rate of the video playback, or
it may cause the user interface to feel sluggish.

Inefficient video players are typically plagued by both excessive script
execution and excessive rendering. The following sections describe a number of
video player optimizations that may help to significantly improve performance on
a television.

#### Simplify rendering complexity

Reduce the number of objects on the Stage as much as possible. For instance,
simplify the shape of essential buttons by using non-complex fill styles like
simple, rectangular, solid-color shapes rather than gradients or multiple small
lines. Be especially judicious using expensive rendering features such as
filters. If shapes such as the playhead are changing shape or position, they
should be updated as infrequently as possible—no more than once or twice per
second.

#### Do not set wmode to "transparent"

This item applies when you are are targeting Google TV or any other platform
that has Flash Player embedded in a TV class device. In that case, the video
player SWF file is embedded in an HTML page using a combination of `<object>`
and `<embed>` tags. If those tags contain a `wmode` parameter, then ensure that
the value of that parameter is set to `"direct"` . Other values of `wmode` may
be supported on some platforms but they cannot be supported consistently. In
particular, setting `wmode="transparent"` will not work on these platforms.
Overlapping SWF files will not layer correctly with each other, nor with other
overlapping HTML elements.

#### Limit use of timer functions and check for duplicates

The goal here is to minimize ActionScript processing intervals: enterFrame
handlers, callback functions registered with NetStream, mouse event handlers,
and other such functionality. When registering a timer function, it will be
called as often as possible up to the rate requested; Flash Player may end up
potentially spending all of its time executing script as opposed to drawing
video frames. This is especially true for timer functions that update UI
components, and hence trigger redraws, which are expensive operations.

#### Do not render items that are not currently visible

If an object is hidden off Stage or behind another object, the Flash renderer
will still waste time attempting to render the hidden object (computing its
bounding box, for instance). If you want to make an object invisible, do not
hide it behind another object, move it off Stage, set its `alpha` property to
zero, or set its visible property to `false` . Instead, call `removeChild()` to
remove the object from the display list entirely. Also, be sure to stop the
timeline of any hidden object.

#### Close the NetStream when the video is finished

If the video associated with a NetStream object has finished playing, call the
`NetStream.close()` method. This allows the Flash runtime to manage video
resources better.

#### Minimize bitmap caching for frequently changing objects

Avoid bitmap caching (via the `cacheAsBitmap` property) on frequently updated
objects. When bitmap caching is enabled, the bitmap needs to be recached—an
expensive operation—each time the object is updated. For example, frequently
updated text fields should not be bitmap cached, as this impacts video playback
time.

#### Avoid transparency

Where possible, avoid transparency or limit transparent objects to smaller
areas. For instance, it may be preferable to convert transparent overlay ads to
pre- or post-roll ads.

#### Avoid drawing APIs

Drawing API calls are particularly expensive because they usually create
objects, such as the scroll bar, dynamically on the Stage before rendering them
via the rasterizer. Furthermore, these objects are destroyed and recreated every
time ActionScript is executed. If possible, create these objects statically in
authoring time on the Stage; these reside in memory through different timelines.

#### Improve ActionScript efficiency

Computer science coding optimizations apply to ActionScript; examples include
avoiding large loops, reducing the depth of function calls, and simplifying the
design of object-oriented classes. Adobe makes available best practices to
improve ActionScript performance. Due to the behavior of many video players,
they may be executing a large amount of script while video is playing back. It
is possible that the amount of time consumed by the executing ActionScript can
cause issues with video playback if the CPU is completely taxed with
ActionScript execution.

Avoid using ActionScript timers with very short intervals (for example, shorter
than a frame update) or spending significant time in any one handler. For Flash
Player, calling from ActionScript to JavaScript can be expensive and should not
be done at regular intervals during video playback. Further, avoid flushing
LocalObjects more often than needed. This is only critical when communicating
between SWF files on a page. If your goal is to communicate between SWF files on
a page, LocalConnection is a higher performing option.

### User interface guidelines

Consider the typical household. The television is in the center of the room,
with various devices connecting to it. There may be one or many people in the
living room, some watching the TV, others engaging in other activites, but also
in the living room. Your application lives in a very unique context, unlike any
other type of platform on which Flash runs. The television is the only consumer
electronics device that is shared between various individuals and members of a
family. Unlike a mobile phone or a desktop or laptop computer, the TV does not
necessarily have an "owner"; the same person who turns it on may not be the same
person who turns it off.

Thus, the user interface for your application and for your video player should
be optimized for television sets and for the living room environment. Consider
what happens if you're building a game and somebody picks up where somebody else
has left off, or if you're building a video player and the play/pause state may
be transient between users.

Your user is likely sitting 10–15 feet from the screen, and the device in the
hand is probably not a mouse and QWERTY keyboard—it's likely a remote control.
It may even be a hybrid remote control, or it may be a mobile phone. The room
may be dark, so imagery needs to be crisp and fonts and colors need to be
legible. Font sizes need to be readable from over 10 feet away, and often at a
glance as somebody is pushing buttons on the remote quickly to get to the
application.

A common practice for most UIs is to be responsive to button clicks in less than
200ms; as such, it's necessary to precache items that may be in the user's
workflow to provide the most optimal experience for the user. It's easy to think
of a television as another PC-class device, but it's also important to consider
the history of television and the decades of behaviors adults and children have
been trained in how a TV should behave and act. People expect TV to be an
entertainment device, either as a lean-back device (in the case of video) or a
sitting-upright device (in the case of gaming). For example, notifications that
require dismissal get in the way of entertainment-oriented expectations, as
consumers expect their TV to just work. Consider using as few modal dialog boxes
as possible and simplifying the user experience to keep it
entertainment-focused. All these factors should influence the design of your
application and any user interface.

### Content protection guidelines

AIR for TV and Google TV devices support Flash Platform content protection for
premium video content.

Many of the leading premium video providers use the Flash Platform to provide a
seamless viewing experience. Streaming video securely from Flash Media Server
(FMS) is possible by using technologies such as RTMPE (Real Time Media Protocol
Encrypted) and SWF Verification. The content protection features in FMS are
supported by the vast majority of content delivery networks (CDNs), enabling an
easy content workflow and broad geographical reach.
[Protecting online video distribution with Adobe Flash media technology](https://web.archive.org/web/20151003093740/http://www.adobe.com/devnet/flashmediaserver/articles/protecting_video_rtmpe.html),
a white paper on the Adobe Developer Connection, provides an overview of typical
content protection workflows using RTMPE, SWF Verification, and related
features.

Adobe also offers Flash Access, an end-to-end content protection and
monetization solution that can provide an even higher level of protection,
increased flexibility, and new opportunities for monetizing content. Flash
Access works for both downloading and streaming use cases, with either FMS or
the new HTTP Dynamic Streaming protocol from Adobe. This technology supports a
broad range of business models including electronic sell-through (EST), video on
demand (VOD), rental, subscription, and pay-per-view (PPV).

Flash Access support is included on desktops starting with Flash Player 10.1 and
Adobe AIR 2. Starting with AIR for TV 2.5, Flash Access is also supported on
Digital Home devices. By providing a common protection solution across different
devices and screens, and integrating content protection into the Flash runtimes,
Flash Access enables content providers to have a single workflow with the
highest level of protection, bringing to consumers a rich, interactive
experience around premium video content.

Developers can leverage the Flash Access server SDK or work with one of our
hosted content protection partners to create solutions that integrate with your
existing back end (such as a subscriber database or a payment processor). The
white paper,
[Adobe Flash Access overview on protected streaming ](https://web.archive.org/web/20151003093740/http://www.adobe.com/products/flashmediaserver/pdfs/flashaccess_wp_protectstreaming.pdf)
(PDF, 319 KB), describes using Flash Access in various workflows, while the
[Flash Access 2.0 Help Resource Center](https://web.archive.org/web/20151003093740/http://www.adobe.com/support/documentation/en/flashaccess/index.html)
provides more detailed information and documentation about the server
components.

On the client, Flash Access introduces a few ActionScript APIs that can be used
to control the acquisition of content licenses, user authentication (where
required), and other functionality. Silicon vendors and OEMs can use the Flash
Access porting kit to leverage the native security hardware for the best user
experience and highest level of protection.

For additional information, refer to the following resources:

- [flash.net.drm package classes](https://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/net/drm/package-detail.html)
  (API documentation)
- [Rich media content: Using digital rights management](https://help.adobe.com/en_US/as3/dev/WS5b3ccc516d4fbf351e63e3d118666ade46-7ce3.html)
  (ActionScript 3 Developer's Guide)

### Customize based on target device for Flash Player

To provide users an optimal experience while browsing on television sets, you
may need to re-encode your video stream or customize the user interface of your
video player. At the same time, you will want to continue providing an optimal
experience on desktop computers and smartphones, which may each require a
different video encoding or a different user interface.

Therefore, you need to add some conditional logic to your website so that the
web page is tailored to the device on which is it displayed. This conditional
logic could be included on your web server, in the JavaScript that runs in your
HTML page, or in the ActionScript that runs in your SWF file.

#### Detection on the server

When a web browser issues an HTTP request, it inserts some information in the
header of the request. One of the inserted fields is called "user-agent." The
user-agent field describes the device on which the browser is running.

By adding logic to your web server, you can serve a television-specific HTML
page when you detect a television browser's user-agent string. A good way to
handle redirecting a browser to a television-specific page on the server is by
using an HTTP Redirect. There is more information on the
["Redirect" directive](https://httpd.apache.org/docs/current/mod/mod_alias.html#redirect)
in the Apache manual and on redirection in the
[Microsoft IIS documentation](https://learn.microsoft.com/en-us/iis/configuration/system.webServer/httpRedirect/).
Also, there are some sample scripts that perform device detection at
[John Boxall's mobile device detection site](https://web.archive.org/web/20151003093740/http://notnotmobile.appspot.com/).

Note that the content of the user agent will vary from one device to the next
and may change if the software on a device is updated. New devices and software
updates appear very frequently, so you should plan to update your server-side
logic accordingly.

#### Detection in JavaScript

A second option is to include the same user-agent logic in the JavaScript code
within your HTML page. Depending on the user agent it detects, it can use the
`document.write` method to embed a SWF file tailored to the device. In this
case, your site contains a single version of the HTML page, but a different
version of the SWF file is delivered to each different target device.

As mentioned above, code that relies on user-agent strings will not work with
future agent strings. In addition, the code may be slow because of all the
string comparisons. The benefit to using the JavaScript approach is the ability
to detect specific devices and operating systems to help you take advantage of
platform-specific capabilities. For more information about user-agent strings
and how to use them, check out the information on
[browser detection and cross-browser support](https://developer.mozilla.org/en/Browser_Detection_and_Cross_Browser_Support)
from Mozilla.org.

To get around the static nature of saving user agents in your JavaScript code,
you can use [WURFL](https://wurfl.sourceforge.net/), a service that maps user
agents to device capabilities. This might be useful if you're trying to use
JavaScript to detect specific attributes of the device that is browsing your web
site.

#### Detection in ActionScript

A third option is to include the detection logic in the ActionScript code within
your SWF file. When this option is used, a single version of the HTML page and a
single version of the SWF file is delivered to all devices. Within the SWF file,
the ActionScript code determines the type of the target device, requests a video
stream that is appropriate for that device, and customizes the video player user
interface for that device.

Flash Player provides many useful ActionScript properties to gather information
about the target device:

- `Capabilities.cpuArchitecture`
- `Capabilities.os`
- `Capabilities.screenDPI`
- `Capabilities.screenResolutionX` , `Capabilities.screenResolutionY`
- `Capabilities.touchscreenType`
- `Mouse.supportsCursor`

On Google TV, the Capabilities.os property is set to `GTV` , and as such you
could infer that an Intel-based Android device with a high-resolution screen and
`Capabilities.os=="GTV"` is a Google TV device.

On other TV platforms, including AIR for TV, the Capabilities class will vary in
what data it returns based on the underlying hardware. You may be able to use
information in the Capabilities class to make various playback and rendering
decisions.

This strategy involves guesswork, and it does not always enable content to
identify the exact model of the target device, but it is a bit more future-proof
than a solution that hard-codes user-agent strings.

### Where to go from here

Flash Player is now available on Google TV, and AIR for TV will soon be
available on Samsung TV and Blu-ray devices, so you can deliver the video
experience that your users have come to love on yet another screen. While you
will be able to leverage many of your assets from existing projects targeting
Flash Player, you may want to customize your video stream and video player for
television sets.
