# GLDPNG FAQ

Copyright 2000-2001 tarquin all rights reserved.

Here, we will provide answers to questions and doubts that have been raised regarding GLDPNG, along with detailed explanations.

---
1. I can't install it with C++Builder.
2. I want to handle PNG data from the clipboard.
3. I want to load a PNG saved in 16-bit format and keep it in 16-bit.
4. Doesn't it support 48-bit?
5. The compilation fails.
6. I can't save it properly in 16-bit.
7. When saving or loading in 256 colors, the transparent color or background color palette index is different. What should I do?
8. Please tell me how to use the alpha channel.
9. I can't load a large image (5000x5000).
10. I used a PNG image created with GLDPNG in a web browser, but when viewed on environments other than Windows, like a Mac, the colors look slightly different.
11. Does it support Kayrix? Or is there a plan to port it?
12. I used a PNG image created with GLDPNG on IE5, but it doesn't show transparency.
13. This is unrelated to GLDPNG, but I want to get the palette color with an index using TBitmap...
---

## Q1

I can't install it with C++Builder.

## A1

Since I only have C++Builder 1, I'm not sure about versions 3 and above. However, there is an explanation in Mr. Nakamura's "nkdib," so you should be able to install it by following similar steps.

First, open the package source (*.bpk) using "File" | "Open," add GLDPNG.pas to the project, and compile it.

Next, copy the generated GLDPNG-related hpp files from ($BCB)\lib to ($BCB)\include.

When using it, include GLDPNG.hpp and link GLDPNG.obj with:

#pragma link "GLDPNG.obj"

Please note that C++Builder is not officially supported, so use it at your own risk.

(2001/1/5 Note)

If you encounter the error:

"[C++ Error] GLDStream.hpp(26): E2015 'TBitmap' and 'Windows::TBitmap' are ambiguous,"

please rewrite the typedef in the error line as follows:

typedef Graphics::TBitmap TGLDBmp;

## Q2

I want to handle PNG data from the clipboard.

## A2

GLDPNG only supports reading and writing the clipboard in Bitmap format. This is done to ensure compatibility with various applications.

## Q3

I want to load a PNG saved in 16-bit format and keep it in 16-bit.

## A3

Since version 3.3, a property called `Read16Bit` has been added. If you set this property to TRUE, it will read the image in 16-bit instead of converting it to 24-bit.

## Q4

Doesn't it support 48-bit?

## A4

It may be supported in the future, but currently, it is not supported. It is also not supported on the Windows side...

## Q5

The compilation fails.

## A5

The translation of the provided text is:

If old versions of files with the same name remain, it will not compile correctly, so please delete all old version files before installing.

## Q6

I can't save it properly in 16-bit.

## A6

Is the `PixelFormat` property of `TBitmap` set to `pfDevice`, or the `HandleType` property set to `bmDDB`? If so, please change it to `bmDIB` and then convert it to match the screen's bit count using the `PixelFormat` property.

Regarding the distinction between 16-bit and 15-bit, in Windows, if no mask is specified, it defaults to 5:5:5 (30,000 colors), and in `TBitmap`, since no mask is specified, 5:5:5 (30,000 colors) is used as the default. However, when a bitmap is loaded, if a mask is specified, it will be treated as 16-bit.

## Q7

When saving or loading in 256 colors, the transparent color or background color palette index is different. What should I do?

## A7

The reason is that there are multiple instances of the same color (transparent or background color) in the palette. In GLDPNG versions 3.3 and earlier, transparent and background colors were handled in RGB format rather than in Index format, which led to this recognition error.

Starting from version 3.4, Index format is also supported, so the issue mentioned above has been resolved.

## Q8

Please tell me how to use the alpha channel.

## A8

When compositing images using the alpha channel, the color can be calculated using the following formula. Let `c0` be the destination color, `c1` the source color, and `a` the alpha channel (where 0 = opaque, 255 = transparent). The formula is:

```
c0 = c1 + (( c0 - c1 ) * a ) / 255
```

For those who may not understand such a formula, I have prepared a simple sample for reference. :)

Delphi version:

```delphi
procedure AlphaBlt(dest, source, alpha: TBitmap);
var
  pSource, pDest, pAlpha: pbyte;
  x, y, r, g, b, a: integer;
begin 
  for y := 0 to source.Height - 1 do
  begin
    pDest := dest.ScanLine[y];
    pSource := source.ScanLine[y];
    pAlpha := alpha.ScanLine[y];
    for x := 0 to source.Width - 1 do
    begin
      a := pAlpha^;
      with PRGBQUAD(pSource)^ do
      begin
        r := rgbRed; g := rgbGreen; b := rgbBlue;
      end;
      with PRGBQUAD(pDest)^ do
      begin
        rgbRed := r + (((rgbRed - r) * a) div 255);
        rgbGreen := g + (((rgbGreen - g) * a) div 255);
        rgbBlue := b + (((rgbBlue - b) * a) div 255);
      end;
      Inc(pSource, 3);
      Inc(pDest, 3);
      Inc(pAlpha);
    end;
  end;
end;
```

C++Builder version:

```cpp
void AlphaBlt(TBitmap *dest, TBitmap *source, TBitmap *alpha) {
  BYTE *pSource, *pDest, *pAlpha;
  int x, y, r, g, b, a;

  for (y = 0; y < source->Height; y++) {
    pSource = (BYTE*)(source->ScanLine[y]);
    pDest = (BYTE*)(dest->ScanLine[y]);
    pAlpha = (BYTE*)(alpha->ScanLine[y]);
    for (x = 0; x < source->Width; pDest += 3, pSource += 3, pAlpha++, x++) {
      a = *pAlpha;
      r = ((RGBQUAD*)pSource)->rgbRed; // Might be incorrect here.. ^^;
      g = ((RGBQUAD*)pSource)->rgbGreen;
      b = ((RGBQUAD*)pSource)->rgbBlue;
      ((RGBQUAD*)pDest)->rgbRed = r + (((((RGBQUAD*)pDest)->rgbRed - r) * a) / 255);
      ((RGBQUAD*)pDest)->rgbGreen = g + (((((RGBQUAD*)pDest)->rgbGreen - g) * a) / 255);
      ((RGBQUAD*)pDest)->rgbBlue = b + (((((RGBQUAD*)pDest)->rgbBlue - b) * a) / 255);
    }
  }
}
```

This is just a sample, so it's very slow. It's for reference purposes only. :)

## Q9

I can't load a large image (5000x5000).

## A9

This is a specification of Windows 95/98. (^^;;

GLDPNG uses `TBitmap` for reading and writing image data. `TBitmap` holds pixel data in memory-mapped form and creates a bitmap handle using `CreateDIBSection`. However, with this method, it fails with large images.

This issue does not occur in Windows 2000 (and I believe it should not occur in NT4 either...).

## Q10

I used a PNG image created with GLDPNG in a web browser, but when viewed on environments other than Windows, like a Mac, the colors look slightly different.

## A10

This happens because the gamma value is not set. Starting from version 3.4, GLDPNG sets the gamma value to 0.45455 by default. However, this is only a reference value for Windows environments, so if you're in a special environment, you will need to adjust the gamma value to suit your environment.

## Q11

Does it support Kayrix? Or is there a plan to port it?

## A11

There are no plans to release a Kayrix version, and since PNG is supported by default, there is no need for a separate version.

## Q12

I used a PNG image created with GLDPNG on IE5, but it doesn't show transparency.

## A12

This is because applications like IE5 do not fully support displaying PNG images.

## Q13

This is unrelated to GLDPNG, but I want to get the palette color with an index using TBitmap...

## A13

You can use the `GetPaletteColor` function in the included `SFunc.pas` file for this purpose.

```delphi
// Get the color of palette index 10
cor := GetPaletteColor(bitmap.Palette, 10);
```
