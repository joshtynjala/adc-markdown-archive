# Using Anisotropic texture filtering to render a Stage3D scene

by Kratika Agarwal

![Kratika Agarwal](img/1446808297797.jpg)

## Content

- [Introduction](#introduction)
- [How to implement?](#how-to-implement)
- [Where to go from here?](#where-to-go-from-here)

## Requirements

### Prerequisite knowledge

Familiarity working with the Stage3D API and basic understanding of how it is
structured is required. Before completing this tutorial, be sure to follow along
with the previous tutorials in this series on Stage3D:

1.  [How Stage3D works](../../flashplayer/articles/how-stage3d-works.md)
2.  [What is AGAL](../../flashplayer/articles/what-is-agal.md)

### User level

Beginning

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)
- Flash Builder or Adobe Animate (formerly Flash Professional)
- Flash Player or Adobe AIR runtime

### Sample files

- Sample Project Missing
<!-- - [Anisotropic_Filtering.zip](/web/20160624123431/http://www.adobe.com/content/dotcom/en/devnet/actionscript/articles/using-stage3d-anisotropic-filtering/_jcr_content/articlePrerequistes/multiplefiles/node_1462188594690/file.res/Anisotropic_Filtering.zip)
  (36 KB)-->

### Introduction

In this article we will learn to use the Stage3D anisotropic filtering support
in ActionScript application, which is based on the Stage3D API. This feature is
available on Flash Player 14.0 and later for both FP and AIR.

Anisotropic filtering provides a new texture filter method called anisotropic
filter in Stage3D, which enhances the image quality of textures on the surfaces
of computer graphics that are at oblique viewing angles.

Until now, there were two texture filter methods available ─ nearest and linear.
See the effect shown in the figure below when you use these two methods to
render a Stage3D scene.

Stage 3D scene rendered using Nearest and Linear texture filtering methods

> Image Missing

<!-- ![file](./using-stage3d-anisotropic-filtering/1462183483881.png)-->

The surface is at oblique viewing angles. So, the far end of the texture is
blurred.

To reduce the blur as shown in the image above, the texture should use
anisotropic filter. See the effect shown below when you use anisotropic filter
with radio 16 to render the scene.

Stage 3D scene rendered using Anisotropic texture filtering method

> Image Missing

<!--
![Stage 3D scene rendered using Anisotropic texture filtering method](./using-stage3d-anisotropic-filtering/1462184305563.png)--->

The following types of filter are available:

- nearest
- linear
- anisotropic2x
- anisotropic4x
- anisotropic8x
- anisotropic16x

See the Action Script documentation of
[Context3DTextureFilter](https://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/display3D/Context3DTextureFilter.html)
for more details.

Anisotropic filtering can be applied on all types of textures and all the
formats of texture (including compressed texture). If a system does support
_anisotropic filter_, the filter will fall back to _linear_.

**\*Note:** The anistropic filter takes effect only when mipmap are used. If the
mipmap is set as none, the texture filter falls back to linear. The feature is
not available in Software rendering mode.\*

### How to implement?

We can set anisotropic texture filtering method both in AGAL and in the
`Context3D::setSamplerStateAt` function when rendering.

**To set anisotropic texture filtering method in AGAL**:

Add anisotropic2x, anisotropic4x, anisotropic8x, or anisotropic16x option to
sampler's filter in AGAL. The sampling format is:

`tex destination, UV, sampler <type, filtering, mip-maping, wrapping mode>`

Here, filtering can be set to any of following:

- anisotropic2x
- anisotropic4x
- anisotropic8x
- anisotropic16x

**To set anisotropic texture filtering method in the
`Context3D::setSamplerStateAt` function:**

Add anisotropic2x, anisotropic4x, anisotropic8x, or anisotropic16x option to
Context3DTextureFilter, which can be used in Context3D::setSamplerStateAt
function when rendering.

`Context3D::setSamplerStateAt(sampler:int, wrap:String, filter:String, mipfilter:String)`

Here, filtering can be set to any of following:

- anisotropic2x
- anisotropic4x
- anisotropic8x
- anisotropic16x

To use anisotropic filtering in your Action Script code, download the Flash
Builder project <u>Anisotropic_Filtering.zip</u> (attached as sample file on the
top of this page) and import it in Flash builder.

### Where to go from here?

From here on as you use Anisotropic filtering with Stage3D apps, things will
only get deeper and more interesting.

To learn more about Stage3D, read
[Using advanced texture features in Stage3D](../../flashruntimes/articles/using-advanced-texture-features-in-stage3d.md).
