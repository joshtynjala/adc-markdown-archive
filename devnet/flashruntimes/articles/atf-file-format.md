# ATF File Format

## by Vivek Negi

![Vivek Negi](./img/1470652187373.jpg)

## Content

- [Adobe Texture Format](#adobe-texture-format)
- [ETC2 RGB Format](#etc2-rgb-format)
- [ETC2 RGBA Format](#etc2-rgba-format)
- [ATFCOMPRESSEDLOSSY](#atfcompressedlossy)
- [Where to go from here](#where-to-go-from-here)

## Requirements

### Prerequisite knowledge

Some familiarity with texture file formats will help you make the most
of this article.

### User level

Intermediate

### Required products

- [Adobe AIR SDK](https://airsdk.dev/)
- Flash Player or Adobe AIR runtime

Adobe introduced support for Adobe Texture Format (ATF) in Stage3D with
the first release of Adobe Gaming SDK in the year 2012. ATF is a
container format for compressed textures. With AIR 18, the ATF tools
have been moved to the AIR SDK. So the latest ATF tools with support for
ETC2 texture compression (ATF file version 3) are available only in the
AIR SDK and not in the Adobe Gaming SDK. These tools let you create and
inspect ATF texture files. This article provides an overview of the ATF
file format version 3, which provides additional support for ETC2 format
and two new ATF formats.

### Adobe Texture Format

**Table 1**. Adobe Texture Format

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr>
<td><strong>Field</strong></td>
<td><strong>Type</strong></td>
<td><strong>Comment</strong></td>
</tr>
<tr>
<td width="149"><p>Signature</p></td>
<td width="399"><p>U8[3]</p></td>
<td width="366"><p>Always ‘ATF’.</p></td>
</tr>
<tr>
<td width="149"><p>Reserved</p></td>
<td width="399"><p>U32</p></td>
<td width="366"><p>Reserved field.</p>
<p><br />
First byte: 0x00</p>
<p><br />
Second byte: 0x00</p>
<p><br />
Third byte: LSB indicates if "-e" switch is used or not and remaining 7
bits indicates actual number of mipmap packaged if "-n" switch is
used.</p>
<p><br />
Fourth byte: 0xff<br />
</p></td>
</tr>
<tr>
<td width="149"><p>Version</p></td>
<td width="399"><p>U8</p></td>
<td width="366"><p>Version of ATF file format.</p></td>
</tr>
<tr>
<td width="149"><p>Length</p></td>
<td width="399"><p>U32</p></td>
<td width="366"><p>Size of ATF file in bytes, does not include
signature, reserved, version bytes, and this length field.</p></td>
</tr>
<tr>
<td width="149"><p>Cubemap</p></td>
<td width="399"><p>UB[1]</p></td>
<td width="366"><p>0 = normal texture</p>
<p>1 = cube map texture</p></td>
</tr>
<tr>
<td width="149"><p>Format</p></td>
<td width="399"><p>UB[7]</p></td>
<td width="366"><p>0 = RGB888</p>
<p>1 = RGBA88888</p>
<p>2 = Compressed</p>
<p>3 = RAW Compressed</p>
<p>4 = Compressed with Alpha</p>
<p>5 = RAW Compressed with Alpha</p>
<p>0x0c = Compressed Lossy</p>
<p>0x0d = Compressed Lossy with Alpha</p></td>
</tr>
<tr>
<td width="149"><p>Log2Width</p></td>
<td width="399"><p>U8</p></td>
<td width="366"><p>Width of texture expressed as 2^Log- 2Width. Maximum
value allowed is 12.</p></td>
</tr>
<tr>
<td width="149"><p>Log2Height</p></td>
<td width="399"><p>U8</p></td>
<td width="366"><p>Height of texture expressed as 2^Log2Height. Maximum
value allowed is 12.</p></td>
</tr>
<tr>
<td width="149"><p>Count</p></td>
<td width="399"><p>U8</p></td>
<td width="366"><p>Total number of textures encoded per face. Maximum
value allowed is 13.</p></td>
</tr>
<tr>
<td>TextureData</td>
<td>If Cubemap == 0 {<br />
If Format == 0<br />
ATFRGB888[Count]<br />
If Format == 1<br />
ATFRGBA8888[Count]<br />
If Format == 2<br />
ATFCOMPRESSED[Count]<br />
If Format == 3<br />
ATFRAWCOMPRESSED[Count]<br />
If Format == 4<br />
ATFCOMPRESSEDALPHA[Count]<br />
If Format == 5<br />
ATFRAWCOMPRESSEDALPHA[Count]<br />
If Format== 0x0C<br />
ATFCOMPRESSEDLOSSY[Count]<br />
If Format== 0x0D<br />
ATFALPHACOMPRESSEDLOSSY[Count]<br />
}<br />
if Cubemap == 1 {<br />
If Format == 0<br />
ATFRGB888[6][Count]<br />
If Format == 1<br />
ATFRGBA8888[6][Count]<br />
If Format == 2<br />
ATFCOMPRESSED[6][Count]<br />
If Format == 3<br />
ATFRAWCOMPRESSED[6][Count]<br />
If Format == 4<br />
ATFCOMPRESSEDALPHA[6][Count]<br />
If Format == 5<br />
ATFRAWCOMPRESSEDALPHA[6][Count]<br />
If Format== 0x0C<br />
ATFCOMPRESSEDLOSSY[6][Count]<br />
If Format== 0x0D<br />
ATFALPHACOMPRESSEDLOSSY[6][Count]<br />
}</td>
<td><p>The first image encoded is the main texture image encoded at the
size specified above. Each consecutive image encoded contains the next
mip map level of that<br />
main image.</p>
<p>Cube maps contain 6 faces. They are placed in the following order:
[-X, +X, -Y, +Y, -Z, +Z], that is, [Left, Right, Bottom, Top, Back,
Front]</p></td>
</tr>
</tbody>
</table>

 

**Table 2**. ATFRGB888

|           |              |                                  |
| --------- | ------------ | -------------------------------- |
| **Field** | **Type**     | **Comment**                      |
| Length    | U32          | Length of payload data in bytes  |
| Data      | U8\[Length\] | JPEG-XR data (JXRC_FMT_24bppRGB) |

 

**Table 3**. ATFRGBA888

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr>
<td><strong>Field</strong></td>
<td><strong>Type</strong></td>
<td><strong>Comment</strong></td>
</tr>
<tr>
<td width="236"><p>Length</p></td>
<td width="277"><p>U32</p></td>
<td width="387"><p>Length of payload data in bytes</p></td>
</tr>
<tr>
<td width="236"><p>Data</p></td>
<td width="277"><p>U8[Length]</p></td>
<td width="387"><p>JPEG-XR data (JXRC_FMT_32bppBGRA)<br />
</p></td>
</tr>
</tbody>
</table>

**Table 4**. ATFRAWCOMPRESSED

|                        |                              |                                         |
| ---------------------- | ---------------------------- | --------------------------------------- |
| **Field**              | **Type**                     | **Comment**                             |
| DXT1ImageDataLength    | U32                          | Length of DXT1 image data in bytes      |
| DXT1ImageData          | U8\[DXT1ImageDataLength\]    | RAW DXT1 data                           |
| PVRTCImageDataLength   | U32                          | Length of PVRTC4bpp image data in bytes |
| PVRTCImageData         | U8\[PVRTCImageDataLength\]   | RAW PVRTC data                          |
| ETC1ImageDataLength    | U32                          | Length of ETC1 image data in bytes      |
| ETC1ImageData          | U8\[ETC1ImageDataLength\]    | RAW ETC1 data                           |
| ETC2RgbImageDataLength | U32                          | Length of ETC2Rgb image data in bytes   |
| ETC2RgbImageData       | U8\[ETC2RgbImageDataLength\] | RAW ETC2Rgb data                        |

**Table 5**. ATFRAWCOMPRESSEDALPHA

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr>
<th colspan="3">ATFRAWCOMPRESSEDALPHA<br />
</th>
</tr>
&#10;<tr>
<td><strong>Field</strong></td>
<td><strong>Type</strong></td>
<td><strong>Comment</strong></td>
</tr>
<tr>
<td width="246"><p>DXT5ImageDataLength</p></td>
<td width="285"><p>U32</p></td>
<td width="373"><p>Length of DXT5 image data in bytes</p></td>
</tr>
<tr>
<td width="246"><p>DXT5ImageData</p></td>
<td width="285"><p>U8[DXT5ImageDataLength]</p></td>
<td width="373"><p>RAW DXT5 data</p></td>
</tr>
<tr>
<td width="246"><p>PVRTCImageDataLength</p></td>
<td width="285"><p>U32</p></td>
<td width="373"><p>Length of PVRTC4bpp image data in bytes</p></td>
</tr>
<tr>
<td width="246"><p>PVRTCImageData</p></td>
<td width="285"><p>U8[PVRTCImageDataLength]</p></td>
<td width="373"><p>RAW PVRTC data</p></td>
</tr>
<tr>
<td width="246"><p>ETC1ImageDataLength</p></td>
<td width="285"><p>U32</p></td>
<td width="373"><p>Length of ETC1 image data in bytes</p></td>
</tr>
<tr>
<td width="246"><p>ETC1ImageData</p></td>
<td width="285"><p>U8[ETC1ImageDataLength/2]</p></td>
<td width="373"><p>RAW ETC1 data</p></td>
</tr>
<tr>
<td width="246"><p>ETC1AlphaImageData</p></td>
<td width="285"><p>U8[ETC1ImageDataLength/2]</p></td>
<td width="373"><p>RAW ETC1 data</p></td>
</tr>
<tr>
<td width="246"><p>ETC2RgbaImageDataLength</p></td>
<td width="285"><p>U32</p></td>
<td width="373"><p>Length of ETC2Rgba image data in bytes</p></td>
</tr>
<tr>
<td width="246"><p>ETC2RgbaImageData</p></td>
<td width="285"><p>U8[ETC2RgbaImageDataLength]</p></td>
<td width="373"><p>RAW ETC2Rgba data</p></td>
</tr>
</tbody>
</table>

**Table 6**. ATFCOMPRESSED

|                         |                               |                                          |
| ----------------------- | ----------------------------- | ---------------------------------------- |
| **Field**               | **Type**                      | **Comment**                              |
| DXT1DataLength          | U32                           | Length of DXT1 data in bytes             |
| DXT1Data                | U8\[DXT1DataLength\]          | LZMA compressed DXT1 data                |
| DXT1ImageDataLength     | U32                           | Length of DXT1 image data in bytes       |
| DXT1ImageData           | U8\[DXT1ImageDataLength\]     | JPEG-XR data (JXRC_FMT_16bppBGR565)      |
| PVRTCTopDataLength      | U32                           | Length of PVRTC4bpp top data in bytes    |
| PVRTCTopData            | U8\[PVRTCTopDataLength\]      | LZMA compressed PVRTC top data           |
| PVRTCBottomDataLength   | U32                           | Length of PVRTC4bpp bottom data in bytes |
| PVRTCBottomData         | U8\[PVRTCBottomDataLength\]   | LZMA compressed PVRTC bottom data        |
| PVRTCImageDataLength    | U32                           | Length of PVRTC4bpp image data in bytes  |
| PVRTCImageData          | U8\[PVRTCImageDataLength\]    | JPEG-XR data (JXRC_FMT_16bppBGR555)      |
| ETC1TopDataLength       | U32                           | Length of ETC1 top data in bytes         |
| ETC1TopData             | U8\[ETC1TopDataLength\]       | LZMA compressed ETC1 top data            |
| ETC1BottomDataLength    | U32                           | Length of ETC1 bottom data in bytes      |
| ETC1BottomData          | U8\[ETC1BottomDataLength\]    | LZMA compressed ETC1 bottom data         |
| ETC1ImageDataLength     | U32                           | Length of ETC1 image data in bytes       |
| ETC1ImageData           | U8\[ETC1ImageDataLength\]     | JPEG-XR data (JXRC_FMT_16bppBGR555)      |
| ETC2RgbTopDataLength    | U32                           | Length of ETC2Rgb top data in bytes      |
| ETC2RgbTopData          | U8\[ETC2RgbTopDataLength\]    | LZMA compressed ETC2Rgb top data         |
| ETC2RgbBottomDataLength | U32                           | Length of ETC2Rgb bottom data in bytes   |
| ETC2RgbBottomData       | U8\[ETC2RgbBottomDataLength\] | LZMA compressed ETC2Rgb bottom data      |
| ETC2RgbImageDataLength  | U32                           | Length of ETC2Rgb image data in bytes    |
| ETC2RgbImageData        | U8\[ETC2RgbImageDataLength\]  | JPEG-XR data (JXRC_FMT_24bppBGR)         |

**Splitting source texture streams into ATF data streams:**

> Image Missing

<!-- ![Figure 1. DXT1 input stream](./atf-file-format/1434453640235.png) -->

Figure 1. DXT1 input stream

> Image Missing

<!-- ![Figure 2. PVRTC 4bpp input data stream](./atf-file-format/1434454141526.png) -->

Figure 2. PVRTC 4bpp input data stream

> Image Missing

<!-- ![Figure 3. ETC1 input data stream](./atf-file-format/1434454000895.png) -->

Figure 3. ETC1 input data stream

### ETC2 RGB Format

**Table 7**. ETC2 RGB input data stream - Individual mode And
Differential mode

|                                                                                                                      |                                          |                                                       |
| -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **24bits**                                                                                                           | **8bits**                                | **32bits**                                            |
| Color0+1 and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) | Flags --\> ATF ETC2Rgb Top Data (8 bits) | Modulation Data --\> ATF ETC2Rgb Bottom Data(32 bits) |

 

**Table 8.** ETC2 RGB input data stream - T Mode

|                                                                                                                      |                                                                                                                      |                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **28bits**                                                                                                           | **4bits**                                                                                                            | **32bits**                                                                                                           |
| Color0+1 and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) | Color0+1 and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) | Color0+1 and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) |

 

**Table 9.** ETC2 RGB input data stream - H Mode

|                                                                                                                      |                                          |                                                       |
| -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **29bits**                                                                                                           | **3bits**                                | **32bits**                                            |
| Color0+1 and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) | Flags --\> ATF ETC2Rgb Top Data (8 bits) | Modulation Data --\> ATF ETC2Rgb Bottom Data(32 bits) |

 

**Table 10.** ETC2 RGB input data stream - Planar Mode

|                                                                                                                    |                                                          |                                                      |
| ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- | ---------------------------------------------------- |
| **25bits**                                                                                                         | **7bits**                                                | **32bits**                                           |
| ColorO and block mode identifier data --\> ATF ETC2Rgb JPEG-XR color data and block mode identifier data (32 bits) | Flag and Color H Data --\> ATF ETC2Rgb Top Data (8 bits) | Color H,V Data --\> ATF ETC2Rgb Bottom Data(32 bits) |

Please refer to [OpenGL ES Version 3.0](https://registry.khronos.org/OpenGL/specs/es/3.0/es_spec_3.0.pdf) for ETC2 RGB specification.

**Table 11.** ATFALPHACOMPRESSED

|                               |                                  |                                               |
| ----------------------------- | -------------------------------- | --------------------------------------------- |
| **Field**                     | **Type**                         | **Comment**                                   |
| DXT5AlphaDataLength           | U32                              | Length of DXT5 alpha data in bytes            |
| DXT5AlphaData                 | U8\[DXT5AlphaDataLength\]        | LZMA compressed DXT1 data                     |
| DXT5AlphaImgDataLength        | U32                              | Length of DXT5 alpha image data in bytes      |
| DXT5AlphaImageData            | U8\[DXT5AlphaImgDataLength\]     | JPEG-XR data (JXRC_FMT_8bppGray)              |
| DXT5DataLength                | U32                              | Length of DXT5 data in bytes                  |
| DXT5Data                      | U8\[DXT5DataLength\]             | LZMA compressed DXT5 data                     |
| DXT5ImageDataLength           | U32                              | Length of DXT5 image data in bytes            |
| DXT5ImageData                 | U8\[DXT5ImageDataLength\]        | JPEG-XR data (JXRC_FMT_16bppBGR565)           |
| PVRTCTopDataLength            | U32                              | Length of PVRTC4bpp top data in bytes         |
| PVRTCTopData                  | U8\[PVRTCTopDataLength\]         | LZMA compressed PVRTC top data                |
| PVRTCBottomDataLength         | U32                              | Length of PVRTC4bpp bottom data in bytes      |
| PVRTCBottomData               | U8\[PVRTCBottomDataLength\]      | LZMA compressed PVRTC bottom data             |
| PVRTCImageDataLength          | U32                              | Length of PVRTC4bpp image data in bytes       |
| PVRTCImageData                | U8\[PVRTCImageDataLength\]       | JPEG-XR data (JXRC_FMT_16bppBGR555)           |
| ETC1TopDataLength             | U32                              | Length of ETC1 top data in bytes              |
| ETC1TopData                   | U8\[ETC1TopDataLength\]          | LZMA compressed ETC1 top data                 |
| ETC1BottomDataLength          | U32                              | Length of ETC1 bottom data in bytes           |
| ETC1BottomData                | U8\[ETC1BottomDataLength\]       | LZMA compressed ETC1 bottom data              |
| ETC1ImageDataLength           | U32                              | Length of ETC1 image data in bytes            |
| ETC1ImageData                 | U8\[ETC1ImageDataLength\]        | JPEG-XR data (JXRC_FMT_16bppBGR555)           |
| ETC2RgbaAlphaTopDataLength    | U32                              | Length of ETC2RGBA alpha top data in bytes    |
| ETC2RgbaAlphaTopData          | U8\[ETC2RgbaAlphaTopDataLength\] | LZMA compressed ETC2Rgba alpha top data       |
| ETC2RgbaAlphaBottomDataLength | U32                              | Length of ETC2Rgba alpha bottom data in bytes |
| ETC2RgbaAlphaBottomData       | U8\[ETC2RgbaAlphaTopDataLength\] | LZMA compressed ETC2Rgba alpha bottom data    |
| ETC2RgbaAlphaImgDataLength    | U32                              | Length of ETC2Rgba alpha image data in bytes  |
| ETC2RgbaAlphaImageData        | U8\[ETC2RgbaAlphaTopDataLength\] | JPEG-XR data (JXRC_FMT_8bppGray)              |
| ETC2RgbaTopDataLength         | U32                              | Length of ETC2Rgba top data in bytes          |
| ETC2RgbaTopData               | U8\[ETC2RgbaAlphaTopDataLength\] | LZMA compressed ETC2Rgba top data             |
| ETC2RgbaBottomDataLength      | U32                              | Length of ETC2Rgba bottom data in bytes       |
| ETC2RgbaBottomData            | U8\[ETC2RgbaAlphaTopDataLength\] | LZMA compressed ETC2Rgba bottom data          |
| ETC2RgbaImageDataLength       | U32                              | Length of ETC2Rgba image data in bytes        |
| ETC2RgbaImageData             | U8\[ETC2RgbaAlphaTopDataLength\] | JPEG-XR data (JXRC_FMT_24bppBGR)              |

 
**Splitting source texture streams into ATF data streams:**

> Image missing

<!-- ![Figure 4. DXT5 input stream](./atf-file-format/1434454080016.png) -->

Figure 4. DXT5 input stream

### ETC2 RGBA Format

**Table 12.** ETC2 RGBA alpha input data stream (most significant 64
bits contain the alpha data)

|                                                          |                                                 |                                                              |
| -------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| **8bits**                                                | **8bits**                                       | **48bits**                                                   |
| Alpha data --\> ATF ETC2Rgba Alpha JPEG-XR data (8 bits) | Flags --\> ATF ETC2Rgba Alpha Top Data (8 bits) | Modulation Data --\> ATF ETC2Rgba Alpha Bottom Data(64 bits) |

 

**Table 13.** ETC2 RGBA color input data stream (least significant 64
bits contain the color data)

|                                                                                          |
| ---------------------------------------------------------------------------------------- |
| **ETC2 RGBA color input data stream (least significant 64 bits contain the color data)** |
| _Same as ETC2 RGB input data stream. Refer to Tables  7, 8, 9, and 10 above._            |

Please refer to [OpenGL ES Version 3.0](https://registry.khronos.org/OpenGL/specs/es/3.0/es_spec_3.0.pdf) for ETC2 RGBA specification.

### ATFCOMPRESSEDLOSSY

**Table 14.** ATFCOMPRESSEDLOSSY

|                         |                               |                                          |
| ----------------------- | ----------------------------- | ---------------------------------------- |
| **Field**               | **Type**                      | **Comment**                              |
| DXT1DataLength          | U32                           | Length of DXT1 data in bytes             |
| DXT1Data                | U8\[DXT1DataLength\]          | LZMA compressed DXT1 data                |
| DXT1ImageDataLength     | U32                           | Length of DXT1 image data in bytes       |
| DXT1ImageData           | U8\[DXT1ImageDataLength\]     | JPEG-XR data (JXRC_FMT_16bppBGR565)      |
| PVRTCTopDataLength      | U32                           | Length of PVRTC4bpp top data in bytes    |
| PVRTCTopData            | U8\[PVRTCTopDataLength\]      | LZMA compressed PVRTC top data           |
| PVRTCBottomDataLength   | U32                           | Length of PVRTC4bpp bottom data in bytes |
| PVRTCBottomData         | U8\[PVRTCBottomDataLength\]   | LZMA compressed PVRTC bottom data        |
| PVRTCImageDataLength    | U32                           | Length of PVRTC4bpp image data in bytes  |
| PVRTCImageData          | U8\[PVRTCImageDataLength\]    | JPEG-XR data (JXRC_FMT_16bppBGR555)      |
| ETC1TopDataLength       | U32                           | Length of ETC1 top data in bytes         |
| ETC1TopData             | U8\[ETC1TopDataLength\]       | LZMA compressed ETC1 top data            |
| ETC1BottomDataLength    | U32                           | Length of ETC1 bottom data in bytes      |
| ETC1BottomData          | U8\[ETC1BottomDataLength\]    | LZMA compressed ETC1 bottom data         |
| ETC1ImageDataLength     | U32                           | Length of ETC1 image data in bytes       |
| ETC1ImageData           | U8\[ETC1ImageDataLength\]     | JPEG-XR data (JXRC_FMT_16bppBGR555)      |
| ETC2RgbTopDataLength    | U32                           | Length of ETC2Rgb top data in bytes      |
| ETC2RgbTopData          | U8\[ETC2RgbTopDataLength\]    | LZMA compressed ETC2Rgb top data         |
| ETC2RgbModeDataLength   | U32                           | Length of ETC2Rgb mode data in bytes     |
| ETC2RgbModeData         | U8\[ETC2RgbModeDataLength\]   | LZMA compressed ETC2Rgb mode data        |
| ETC2RgbBottomDataLength | U32                           | Length of ETC2Rgb bottom data in bytes   |
| ETC2RgbBottomData       | U8\[ETC2RgbBottomDataLength\] | LZMA compressed ETC2Rgb bottom data      |
| ETC2RgbImageDataLength  | U32                           | Length of ETC2Rgb image data in bytes    |
| ETC2RgbImageData        | U8\[ETC2RgbImageDataLength\]  | JPEG-XR data (JXRC_FMT_24bppBGR)         |

 

**Table 15.** ATFALPHACOMPRESSEDLOSSY

|                               |                                  |                                               |
| ----------------------------- | -------------------------------- | --------------------------------------------- |
| **Field**                     | **Type**                         | **Comment**                                   |
| DXT5AlphaDataLength           | U32                              | Length of DXT5 alpha data in bytes            |
| DXT5AlphaData                 | U8\[DXT5AlphaDataLength\]        | LZMA compressed DXT1 data                     |
| DXT5AlphaImgDataLength        | U32                              | Length of DXT5 alpha image data in bytes      |
| DXT5AlphaImageData            | U8\[DXT5AlphaImgDataLength\]     | JPEG-XR data (JXRC_FMT_8bppGray)              |
| DXT5DataLength                | U32                              | Length of DXT5 data in bytes                  |
| DXT5Data                      | U8\[DXT5DataLength\]             | LZMA compressed DXT5 data                     |
| DXT5ImageDataLength           | U32                              | Length of DXT5 image data in bytes            |
| DXT5ImageData                 | U8\[DXT5ImageDataLength\]        | JPEG-XR data (JXRC_FMT_16bppBGR565)           |
| PVRTCTopDataLength            | U32                              | Length of PVRTC4bpp top data in bytes         |
| PVRTCTopData                  | U8\[PVRTCTopDataLength\]         | LZMA compressed PVRTC top data                |
| PVRTCBottomDataLength         | U32                              | Length of PVRTC4bpp bottom data in bytes      |
| PVRTCBottomData               | U8\[PVRTCBottomDataLength\]      | LZMA compressed PVRTC bottom data             |
| PVRTCImageDataLength          | U32                              | Length of PVRTC4bpp image data in bytes       |
| PVRTCImageData                | U8\[PVRTCImageDataLength\]       | JPEG-XR data (JXRC_FMT_16bppBGR555)           |
| ETC1TopDataLength             | U32                              | Length of ETC1 top data in bytes              |
| ETC1TopData                   | U8\[ETC1TopDataLength\]          | LZMA compressed ETC1 top data                 |
| ETC1BottomDataLength          | U32                              | Length of ETC1 bottom data in bytes           |
| ETC1BottomData                | U8\[ETC1BottomDataLength\]       | LZMA compressed ETC1 bottom data              |
| ETC1ImageDataLength           | U32                              | Length of ETC1 image data in bytes            |
| ETC1ImageData                 | U8\[ETC1ImageDataLength\]        | JPEG-XR data (JXRC_FMT_16bppBGR555)           |
| ETC2RgbaAlphaTopDataLength    | U32                              | Length of ETC2RGBA alpha top data in bytes    |
| ETC2RgbaAlphaTopData          | U8\[ETC2RgbaAlphaTopDataLength\] | LZMA compressed ETC2Rgba alpha top data       |
| ETC2RgbaAlphaBottomDataLength | U32                              | Length of ETC2Rgba alpha bottom data in bytes |
| ETC2RgbaAlphaBottomData       | U8\[ETC2RgbaAlphaBottomData\]    | LZMA compressed ETC2Rgba alpha bottom data    |
| ETC2RgbaAlphaImgDataLength    | U32                              | Length of ETC2Rgba alpha image data in bytes  |
| ETC2RgbaAlphaImageData        | U8\[ETC2RgbaAlphaImgDataLength\] | JPEG-XR data (JXRC_FMT_8bppGray)              |
| ETC2RgbaTopDataLength         | U32                              | Length of ETC2Rgba top data in bytes          |
| ETC2RgbaTopData               | U8\[ETC2RgbaTopDataLength\]      | LZMA compressed ETC2Rgba top data             |
| ETC2RgbaModeDataLength        | U32                              | Length of ETC2Rgba mode data in bytes         |
| ETC2RgbaModeData              | U8\[ETC2RgbaModeDataLength\]     | LZMA compressed ETC2Rgba mode data            |
| ETC2RgbaBottomDataLength      | U32                              | Length of ETC2Rgba bottom data in bytes       |
| ETC2RgbaBottomData            | U8\[ETC2RgbaBottomDataLength\]   | LZMA compressed ETC2Rgba bottom data          |
| ETC2RgbaImageDataLength       | U32                              | Length of ETC2Rgba image data in bytes        |
| ETC2RgbaImageData             | U8\[ETC2RgbaImageDataLength\]    | JPEG-XR data (JXRC_FMT_24bppBGR)              |

### Where to go from here

For details and usage examples for the tools that make up the ATF SDK,
refer to the [ATF tools user's guide](./atf-users-guide.md).
