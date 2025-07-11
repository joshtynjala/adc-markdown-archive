# Working with Stage3D profiles

by Vivek Negi

![Vivek Negi](./img/1470652187373.jpg)

## Content

- [Introduction](#introduction)
- [Available Stage3D Hardware-based profiles](#available-stage3d-hardware-based-profiles)
- [Requesting Context with profiles](#requesting-context-with-profiles)
- [Brief overview of all profiles](#brief-overview-of-all-profiles)

## Requirements

### Prerequisite knowledge

A basic understanding of how Stage3D works. To learn more about Stage3D, see:

- [How Stage3D works](./how-stage3d-works.md)
- [What is AGAL](./what-is-agal.md)

### User level

Beginning

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)
- Flash Builder or Adobe Animate (formerly Flash Professional)
- Flash Player or Adobe AIR runtime

### Introduction

While working with Stage3D, we are actually working with Context3D. When we
request a Context3D, we generally request it by specifying a profile. A profile
in Stage3D helps to specify feature-level support. Profiles help developers to
select a particular category of devices/platform and target them for their
applications.

### Available Stage3D Hardware-based profiles

Currently, Flash and AIR provide the following Stage3D hardware-based profiles
in decreasing order of the capabilities they offer:

- Standard
  - Standard Extended
  - Standard
  - Standard Constrained
- Baseline
  - Baseline Extended
  - Baseline
  - Baseline Constrained

### Requesting Context with profiles

When requesting Context3D you can use either requestContext3D or
requestContext3DMatchingProfile methods.

Use requestContext3d as follows:

    public function requestContext3D(context3DRenderMode:String = "auto", profile:String ="baseline")

In argument PROFILE, specify the particular profile for the requested Context.

The default profile provided is Baseline.

Use requestContext3DMatchingProfile as follows:

    public function requestContext3DMatchingProfiles(profiles:Vector.<String>):void

`profiles:Vector.<String>` is a profile array that developers want to use in
their Flash programs. When you pass a profile array to
Stage3D.requestContext3DMatchingProfiles, you get a Context3D based on the high
level profile in that array according to the hardware capability.

### Brief overview of all profiles

The following table gives a brief overview of the capabilities different
profiles offer to AS developers.

|                                            |                          |              |                       |                          |              |                       |
| ------------------------------------------ | ------------------------ | ------------ | --------------------- | ------------------------ | ------------ | --------------------- |
| **Features**                               | **Baseline Constrained** | **Baseline** | **Baseline Extended** | **Standard Constrained** | **Standard** | **Standard Extended** |
| Register:va                                | 8                        | 8            | 8                     | 8                        | 8            | 16                    |
| Register:vc                                | 128                      | 128          | 128                   | 250                      | 250          | 250                   |
| Register:vt                                | 7                        | 8            | 8                     | 26                       | 26           | 26                    |
| Register:v                                 | 8                        | 8            | 8                     | 8                        | 10           | 10                    |
| Register:fc                                | 28                       | 28           | 28                    | 64                       | 64           | 200                   |
| Register:ft                                | 8                        | 8            | 8                     | 26                       | 26           | 26                    |
| Register:fs                                | 8                        | 8            | 8                     | 8                        | 16           | 16                    |
| Register:iid                               | 0                        | 0            | 0                     | 0                        | 0            | 1                     |
| Token                                      | 200                      | 200          | 200                   | 1024                     | 1024         | 2048                  |
| Rectangle Texture                          | NO                       | YES          | YES                   | YES                      | YES          | YES                   |
| 4k+ Texture                                | NO                       | NO           | YES                   | YES                      | YES          | YES                   |
| MRT                                        | NO                       | NO           | NO                    | NO                       | YES          | YES                   |
| Half-precision floating-point texture      | NO                       | NO           | NO                    | YES                      | YES          | YES                   |
| AGAL:Conditional forward jump instructions | NO                       | NO           | NO                    | YES                      | YES          | YES                   |
| AGAL:Fragment depth output                 | NO                       | NO           | NO                    | NO                       | YES          | YES                   |
| AGAL:Partial derivative instructions       | NO                       | NO           | NO                    | YES                      | YES          | YES                   |

**Baseline Constrained:** As baseline profile was released as default profile in
Stage3D, it targeted a lot of devices which were present at that time. But
still, there were a good number of devices which were not supported by Baseline.

These devices did not run applications developed for Baseline and hence a number
of devices were left which couldn't use Stage3d.

So, as we know Stage3D was launched with default profile as baseline, with
Baseline Constrained came actual 'Profile' parameter, by which Context could be
requested from GPU with either Baseline or Baseline constrained profile.

**Baseline:** This profile is default profile and most closely resembles with
Stage3D as a whole feature when it was released.

This profile is primarily targeted at devices that only support PS_2.0 level
shaders like the Intel GMA 9xx series. In addition, this mode tries to improve
memory bandwidth usage by rendering directly into the back buffer.

**Baseline Extended:** This profile was provided to support GPUs which can
render 4096\*4096 textures. Earlier, Baseline and Baseline constrained only
supported rendering of 2048\*2048 textures.

This profile is an extension of baseline, hence it has all the feature of
baseline, as well as support of 4096\*4096 textures.

**Standard Constrained**: On Desktop platforms, most video cards support
"standard" profile features. All non-pepper Flash Players use the graphics
interface from OS directly. So "standard" profile is available on most machines
on non-pepper Flash Player.

However, pepper API wraps graphics interface of OS on Windows (D3D9 or D3D11),
and provide OpenGL ES2.0 interface to Flash. Now pepper only supports Multiple
Render Target (one feature in "standard" profile") on D3D11 backend but not on
D3D9 backend.

Using D3D9 or D3D11 is determined by the Chrome Browser (visit chrome://gpu/ and
search GL_RENDERER to see what the backend is on your machine).

However, Chrome forces the use of D3D9 backend on many machines for some reasons
(driver bugs, old hardware, and other such reasons.), which prevents Flash from
using 'standard profile' widely.

Now all Intel graphics cards are forced to use D3D9 by Chrome.

On most devices that have OpenGLES2.0, Multiple Render Target (MRT), and
fragmentation depth features are not available.

To provide a tradeoff solution between functionality and end-user coverage, like
the baseline constrained profile, we provided the Standard Constrained profile.

**Standard Profile:** Standard profile introduced a number of new features for
Stage3D such as AGAL2, Multiple Render Target (MRT), and Floating point texture.
For more information, see
[Adobe Blogs: Stage3D "standard" profile](https://web.archive.org/web/20150202073513/http://blogs.adobe.com:80/flashplayer/2014/09/stage3d-standard-profile.html)

**Standard Extended:** This feature is implemented to enable you to use AGAL v3
in your apps. AGAL v3 increases the limit of various registers used in Stage3D.
