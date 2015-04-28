# Playstation TIM format

This document serves to explain the TIM format used by Konami in a number of
their rhythm games. The format is quite common for other Playstation games.

## About this format

The TIM format appears to be very convenient to use for those who developed
software for Sony's Playstation. It's no surprise this standard format was one
chosen to be used for in-game visual assets in a number of Konami's rhythm
games.

This format does not include all the information needed to parse a TIM image
fully; rather, it contains information that pertains only to the features used
by Konami in their rhythm games.

## In a nutshell

A TIM file consists of a header, possibly a palette (depending on the image
pixel format), image transformation information, and raw pixel data.

## Header

The first eight bytes determine if this is a TIM file, and what kind of data
we can expect to follow.

```
Offset(h) Type      Length    Description
0         int32     4         File Identifier (0x00000010)
4         int32     4         Image type
```

### Image types

The image type in the header determines what data follows. Here is a table
that should be used to determine if a palette is present, and what format the
pixel data is in:

```
Value(h)  BitDepth  Palette?  PixelFormat
00        4bpp      No        4 bits per pixel, from an external palette
01        8bpp      No        8 bits per pixel, from an external palette
02        16bpp     No        16 bits per pixel, not indexed
03        24bpp     No        24 bits per pixel, not indexed
08        4bpp      Yes       4 bits per pixel, from a palette of 16 colors
09        8bpp      Yes       8 bits per pixel, from a palette of 256 colors
```

## Palette (CLUTs)

If the bit depth is 4 or 8 bits per pixel, and does not use an external
palette, a palette follows in the file. This palette, often called a CLUT (or
Color LookUp Table) also has its own header. There can be multiple CLUTs, but
this header will only appear once.

```
Offset(h) Type      Length    Description
0         int32     4         Length of all the CLUT data after the header
4         int16     2         Palette Origin X
6         int16     2         Palette Origin Y
8         int16     2         Colors per CLUT
A         int16     2         Number of CLUTs
C         ?         ?         CLUT data
```

If `ColorsPerCLUT` is zero, assume it's 16 for 4bpp images, and 256 for 8pp
images.

All colors are stored as 16 bits (two bytes) for each entry. To find out how
many bytes you should have, use this formula:

```
CLUTDataLength = ColorsPerCLUT * NumberOfCLUTs * 2;
```

The CLUTs are stored one after another. Each 16-bit color can be decoded using
the following method. Note that each color value will be in the range of 0 to
31: no intensity to full intensity for that particular color.

```
Red = ColorData & 0x1F;
Green = (ColorData >> 5) & 0x1F;
Blue = (ColorData >> 10) & 0x1F;
Mask = ((ColorData >> 15) & 0x1) != 0;
```

The purpose of the mask color in Bemani games is unknown. It could possibly
be used for transparency.

## Image metadata

After the header and palette sections (if applicable), the image metadata
section is present.

```
Offset(h) Type      Length    Description
0         int32     4         Length of pixel data after the header in bytes
4         int16     2         Image origin X
6         int16     2         Image origin Y
8         int16     2         Image stride*
A         int16     2         Image height
C         ?         ?         Raw image data
```

Image stride is the number of bytes required in internal memory per line of the
image data. You can get the width of the image in pixels by using the following
method.

```
Width = (Stride * 16) / BitsPerPixel;
```

## Pixel data

The pixel data itself is interpreted in a number of different ways based on
the bits per pixel.

### 4bpp

Each byte represents two pixels. The lower four bits make up the color index
for the leftmost pixel, and the upper four bits make up the color index for
the rightmost pixel.

```
PixelL = (PixelByte & 0xF);
PixelR = (PixelByte >> 4);

PixelColorL = CLUT[PixelL];
PixelColorR = CLUT[PixelR];
```

### 8bpp

Each byte represents one pixel. The byte value is used directly in the CLUT to
determine the output color.

```
PixelColor = CLUT[PixelByte];
```

### 16bpp

Two bytes represent one pixel. The color value is derived directly from the
16-bit value. You can convert the color using the formula mentioned above in
the CLUT section.

### 24bpp

Three bytes represent one pixel. The color value is derived directly from the
24-bit value. Red intensity comes from the first byte, green comes from the
second, and blue comes from the third. Each intensity value is in a range of
0 to 255, as opposed to the range of intensities used in the rest of the
format.

## Konami specific uses

Often, Konami will combine a number of TIM files back-to-back in the arcade
version of Dance Dance Revolution. When the end of pixel data is encountered,
check to see if the end of the file has also been reached. If not, and the
next four bytes are the TIM header bytes (10 00 00 00) then try reading another
TIM image.
