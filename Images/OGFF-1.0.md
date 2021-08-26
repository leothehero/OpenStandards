mp)àmiu!ç:è;iy kj,bn# Open Graphics Format (OGF)
Note: any text between `<` and `>` is not literal text (of course) and is just are just placeholders for the actual values.

OGF can contain multiple images, if the animated flag is set, it should be interpreted as an animation, otherwise if there are
multiple images that aren't animated, they should be just treated as a list of image, if this doesn't make sense in the current usage, the first image only should be used.

**THIS IS A DRAFT AND ISN'T FINISHED!**

## Types
Any `Text` type is a UTF-8 string starting with a 2 bytes unsigned integer (0-65535) for its length, any optional `Text` is counted as not present if the length is equals to 0. 

A `Boolean` type is a byte equals to 0 for false or 1 for true. If the boolean value is any other value, the resulting boolean is true.

## Flags
If least-significant bit is equals to 1 (flags & 0b00000001 == 0b00000001), the file is an animated image. The images are in
increasing order dependending on entry ID of Image Data entries.

## Header
```
<Signature> - ASCII "OGFIMAG" followed by ASCII End Of Text (ETX, byte equals to 3)
<Version> - Unsigned byte (currently equals to 2)
<Flags> - Byte, see above
<Entries Length> - Unsigned byte (0-255)
<Entries>
```

## Entries
Any entry start with a byte for its type and a 4 bytes integer for their length.
So every entry start with `<Type> - Unsigned byte (0-255)` and `<Length> - Unsigned 4 bytes integer`.
Entries might come in any order, but if an entry (ex: image data) have a dependency over another one (ex: rgb palette), the dependency must come before.
Notice that unless marked so, an entry type can be found multiple times in a single file, including image data, which allows for multiple images in a single file.

### Metadata
This entry **MUST** be on every file!
The entry type is 0.
```
<Frame Time> - Integer, for animated images only, it is the time between each frames in milliseconds, equals to 0 if image isn't animated.
<Author> - Text, the name of the author of the picture, optional
<Extended Name> - Text, the extended name of the picture, optional
<Location> - Text, the location where the picture was made, can be in X-Y-Z format or just a name, optional
<Custom> - Text, optional, for custom/unofficial values, this must be in a key=pair format where each entry is ended by ; (ex: mycustom=thing;thisisnot=pork;abc=123)
```

### RGB Palette
The entry type is 1 and the entry is optional. The bit depth is determined from the image linking to this palette.
So if a 8-bit color depth image is linking to this palette, this should be interpreted as 8-bit palette.
12-bit and 16-bit bit depth are 8-bit color depth (as they all use 8-bit for color, but just have additional bits for alpha), which mean a 8-bit palette is suited for those bit depths (8-bit, 12-bit and 16-bit)

This also means images should never be a different bit depth than the palette was made for.
```
<Values>
```
Yep it just contain a Values field. This field corresponds to a concatenation of the 24-bit RGB values. Meaning the entry is 768 bits long for a 8-bit/12-bit/16-bit depth. What this means is that for example, 1-bit have 2 RGB colors, 2-bit have 4, 4-bit have 16, 8-bit 256, and 24-bit doesn't use any palette.

### Image Data
There **MUST** be atleast one of the following image data in a file.

#### OC Image Data
The entry type is 2, this image data type is mandatory for image that contain OC text, if the image doesn't contain any form of OC text, then Bitmap Image Data should be used instead.

```
<Width> - Unsigned 2 bytes integer (0-65535)
<Height> - Unsigned 2 bytes integer (0-65535)
<Bit Depth> - Unsigned byte.
<Compression> - Unsigned byte.
<RGB Palette> - offset to the first byte of a palette, relative to the file start
<Text Length> - The BYTE length of the text, remember a character, stored in a 4 bytes unsigned integer.
<Text> - A 160x50 character array, appended in XY order from top-left. (this is not a Text type) N<DC3>AAF<ETX>ote: a character is not always one byte!!! Use unicode aware functions to manipulate this.
<Pixels> - A list of pixel structs (format depending on bit depth) appended in XY order from top-left.
```

Pixel struct:
```
<Background> - format depending on bit depth
<Foreground> - format depending on bit depth
```

Example:
```
2 (Width)
1 (Height)
4 (24-bit depth)
0 (not compressed)
2
Ab
0x000000 (bg for char 1)
0xFFFFFF (fg for char 1)
0x000000 (bg for char 2)
0xFFFFFF (fg for char 2)
```
This creates a 2x1 image, with the text

The character for each pixel is to be found in the `<Text>`

#### Bitmap Image Data
This image data type is very very useful for bitmaps (it is more efficient to treat braille-"extended" images as bitmap rather than OC image data)
The entry type is 3.

```
<Width> - Unsigned 2 bytes integer (0-65535)
<Height> - Unsigned 2 bytes integer (0-65535)
<Bit Depth> - Unsigned byte.
<Compression> - Unsigned byte.
<RGB Palette> - offset to the first byte of a palette, relative to the file start
<Bitmap> - Described below
<Foregrounds> - A list of foreground color (format depending on bit depth) appended in XY order from top-left.
<Backgrounds> - A list of background color with same format as foregrounds
```

Bitmap:
Each entry is exactly one byte long and contains the bits of the [braille pattern as defined by Unicode](https://en.wikipedia.org/wiki/Braille_Patterns#Identifying,_naming_and_ordering). If the source image uses braille, this is as simple as substrating 0x2800 from the Unicode codepoint.

Example: U+2813 is encoded to 0x13.

Background - the color when braille pixel is reset/unset, format depending on bit depth
Foreground - the color when braile pixel is set, format depending on bit depth

#### Bit Depths

Byte Value | Bit Depth
---------- | ---------
3 | 8-bit
4 | 24-bit
5 | 32-bit (24-bit + alpha 8-bit)
6 | 16-bit (8-bit + alpha 8-bit)

8-bit colors are changed to 24-bit RGB values via the RGB palette. If the reference is null (equals to 255) then the default [OC palette is used](https://ocdoc.cil.li/_media/api:oc-256-color.png). 24-bit RGB is interpreted as it and the alpha values too.

If the bit depth is 12-bit and the image ends not being padded on 8-bit, 4-bit padding must be added to fit. The padding can be anything as it is ignored.

#### Compression Types
Byte Value | Compression Type
---------- | ----------------
0 | No Compression
1 | RLE

Compression is always only applied to the pixels of an image.

#### RLE Encoding
A parser will parse byte by byte for an opcode.
Here is a list of the opcode it can have:
Bits / Hex | Action
---------- | ------
000xxxxx | Repeat x times 255 (x = bits & 0x1F)
001xxxxx | Repeat x times 0 (x = bits & 0x1F)
01xxxxxx | Treat the following x bytes as a non-RLE encoded byte array (x = bits & 0x3F)
1xxxxxxx |  Repeat x times the following byte (x = bits & 0x7F)
