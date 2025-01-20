# TGLDPNG ver 3.4

1998,2000 Copyright tarquin all rights reserved.
2004/5/9

Table of Contents

1. Introduction
2. Setting Up the Environment
3. Usage
  a. Basic Usage
  b. Linking Function
  c. Using in the IDE Environment
  d. Saving in Grayscale
  e. Alpha channel
  f. Image Encryption
  g. Background Color and Transparency Color
  h. Compiler Definitions
  i. Migration from Older Versions
4. License Agreement
5. Support
6. Changelog
7. References
  1. Class Reference
  2. FAQ

## 1. Introduction

This class (hereinafter referred to as TGLDPNG) allows you to read and write PNG image files in the same way as handling TJPEGImage. It also provides the ability to configure settings specific to the PNG format.

**Main Features:**

- Naturally, reading and writing PNG format images
- Saving with filters supported by default
- Optimization of selected filters
- Configuration of compression level similar to ZLIB
- Support for the OnProgress event
- Support for image encryption
- Reading and writing of alpha channels
- Support for saving grayscale images
- Reading and writing of 15-bit (32,768 colors) and 16-bit (65,536 colors) bitmaps

**Note:** To fully utilize this class, a certain level of understanding of TBitmap is recommended.

Before using this software, please carefully read the license agreement and use it only if you agree with the terms.

Supported programming languages: Delphi 3 or higher. Although compilation is possible in C++Builder 3 or higher, functionality is not guaranteed.

**For Users of Previous Versions:** If you are upgrading from an older version to version 3.4, please refer to the migration guide from the previous version, make the necessary adjustments, and then use the updated version.

## 2. Setting Up the Environment

Copy all the included files to a folder that is part of the Delphi search path [^1]. If you are using an older version (such as GLDPNG 3.3 or TKPNG), make sure to uninstall the previous version from the IDE environment before copying the new files. (Note: This applies if you have installed the component). After uninstalling, delete all files from the old version before copying the new ones.

[1]: To add a folder to the Delphi search path, go to [Tools (T)] → [Environment Options (O)] → [Library] → [Library Path]. By adding the folder there, the compiler will automatically search for unit files from that path during compilation.

## 3. Usage

### a. Basic Usage

The basic usage of TGLDPNG is similar to handling TJPEGImage. When using TGLDPNG, make sure to add the GLDPNG unit to the Uses section. Below is an example of simple reading and writing.

```delphi
// Load an image saved in PNG format into the specified TBitmap class
procedure LoadPNGFile(const filename: string; bitmap: TBitmap);
var
  png: TGLDPNG;
begin
  png := TGLDPNG.Create;
  try
    // Load PNG file
    png.LoadFromFile(filename);
    // Copy the loaded image to bitmap
    bitmap.Assign(png);
  finally
    png.Free;
  end;
end;

// Save the specified TBitmap class as a PNG file
procedure SavePNGFile(const filename: string; bitmap: TBitmap);
var
  png: TGLDPNG;
begin
  png := TGLDPNG.Create;
  try
    // Assign bitmap to PNG class
    png.Assign(bitmap);
    // Save as PNG file
    png.SaveToFile(filename);
  finally
    png.Free;
  end;
end;
```

To perform simple file reading and writing, simply call the `LoadFromFile` or `SaveToFile` methods.

The `Assign` method is used for TBitmap because TGLDPNG is a class designed specifically for reading and writing PNG files, and does not have the functionalities of a TBitmap, such as shape drawing or pixel-level manipulation. Therefore, if you're only displaying the image, you can directly assign it to a `TImage` without needing to use `Assign` with a `TBitmap`.

TGLDPNG has many properties and events, most of which are used to configure settings during reading and writing. For more details, refer to the class reference. Properties and events should be set before calling methods like `LoadFromFile` or `SaveToFile`. Below is an example of how to set properties.

**Example:**

```delphi
// Load an image saved in PNG format into the specified TBitmap class
procedure LoadPNGFile(const filename: string; bitmap: TBitmap);
var
  png: TGLDPNG;
begin
  png := TGLDPNG.Create;
  try
    // Check if MacBinary is enabled
    // This is just a sample, so it is set here, but
    // it defaults to TRUE even if not explicitly set.
    png.MacBinary := TRUE;
    // Load PNG file
    png.LoadFromFile(filename);
    // Copy the loaded image to bitmap
    bitmap.Assign(png);
  finally
    png.Free;
  end;
end;

// Save the specified TBitmap class as a PNG file
procedure SavePNGFile(const filename: string; bitmap: TBitmap);
var
  png: TGLDPNG;
begin
  png := TGLDPNG.Create;
  try
    // Assign bitmap to PNG class
    png.Assign(bitmap);
    // Set compression type to "Best Compression"
    png.CompressLevel := gplBest;
    // Set filter type to "None"
    png.FilterType := gpfNone;
    // Save as "Interlaced"
    png.InterlaceType := gptAdam7;
    // Save as PNG file
    png.SaveToFile(filename);
  finally
    png.Free;
  end;
end;
```

### b. Linking Function

When performing standard PNG file reading and writing with TGLDPNG, the internal bitmap is duplicated for processing. This can result in slower performance, especially when trying to save multiple images with the same settings at once.

To address this, TGLDPNG offers a feature called the "linking function." The linking function allows direct reading and writing to a bitmap by linking to the bitmap without using an intermediary internal bitmap. Below is an example of how to use the linking function.

**Example:** (Note: `png` is an instance of TGLDPNG, and `Bitmap` is an instance of TBitmap.)

```delphi
// Load a PNG file into the Bitmap
png.Image := Bitmap;
png.LoadFromFile('filename');

// Save the Bitmap to a PNG file
png.Image := Bitmap;
png.SaveToFile('filename');
```

### c. Using in the IDE Environment

By installing `GLDPNG.pas` as a component in the IDE environment, you can load PNG files into components like TImage during design-time.

When loading a PNG file during design-time, make sure to add `GLDPNG` to the `Uses` section of a unit in your project. Otherwise, the PNG file will not be displayed at runtime.

### d. Saving in Grayscale

To save an image in grayscale, set the `GrayScale` property to `TRUE`.

### e. Alpha Channel

TGLDPNG supports handling the alpha channel. In TGLDPNG, the alpha channel is treated as a separate bitmap from the image itself. The bitmap used for the alpha channel is an 8-bit bitmap.

#### (1) During Loading:

- **When the `AlphaBitmap` property is not specified (its value is `nil`):**
  The alpha channel is loaded into the internal alpha channel bitmap. You need to extract it using the `AlphaBitmapAssignTo` method.
- **When another bitmap is specified for the `AlphaBitmap` property:**
  The specified bitmap is used directly without going through the internal alpha channel bitmap. The same bitmap cannot be used for both the Image and AlphaBitmap properties.

The presence of an alpha channel can be checked using the `AlphaChannel` property. Below is an example of its usage:

**Example:** (Note: `png` is an instance of TGLDPNG, and `AlphaBmp` is an instance of TBitmap.)

```delphi
// Internal loading
png.AlphaBitmap := nil;
png.LoadFromFile('PNG filename');
if png.AlphaChannel then  // Check for alpha channel presence
  png.AlphaBitmapAssignTo(AlphaBmp); // Extract alpha channel here

// Load to a linked destination
if png.AlphaChannel then  // Check for alpha channel presence
  png.AlphaBitmap := AlphaBmp; // Specify linked destination
png.LoadFromFile('PNG filename');
```

#### (2) During Saving:

To use the alpha channel, set the `AlphaChannel` property to `TRUE`.

- **Using the internal alpha channel bitmap (set `AlphaBitmap` property to `nil`):**
  The internal alpha channel bitmap will be used. To set the internal alpha channel bitmap, use the AlphaBitmapAssign method.
- **Specifying another bitmap for the `AlphaBitmap` property:**
  Instead of using the internal alpha channel bitmap, the specified bitmap is used directly as the alpha channel. The same bitmap cannot be used for both the Image and `AlphaBitmap` properties.

Additionally, when specifying an alpha channel, the following conditions must be met:

- The alpha channel bitmap must be the same size as the image.
- The alpha channel bitmap must be a 256-color image (i.e., the `PixelFormat` property must be `pf8bit`). **Note:** The palette is not relevant.

**Example:** (Note: `png` is an instance of TGLDPNG, and `AlphaBmp` is an instance of TBitmap.)

```delphi
// Use internal alpha channel bitmap
png.AlphaChannel := TRUE;
png.AlphaBitmap := nil;
png.SaveToFile('PNG filename');

// Use linked destination alpha channel bitmap
png.AlphaChannel := TRUE;
png.AlphaBitmap := AlphaBmp; // Specify linked destination
png.SaveToFile('PNG filename');
```

TGLDPNG inverts the alpha channel values when loading. Specifically, `0` is treated as fully opaque, and `255` is treated as fully transparent. However, if you define the compiler directive `GLD_NOREVERSE_ALPHA`, this inversion will not occur.

When loading 2/4/16/256-color images, if the alpha channel contains only one transparent color and all others are opaque, the transparent color is automatically handled as the transparent pixel.

TGLDPNG does not support palette-based alpha channels in PNG files. During loading, any such formats are converted to either a transparent color specification or a bitmap-based alpha channel.

### f. Image Encryption

Image encryption is not defined in the PNG format itself and is a custom feature of TGLDPNG. However, encrypted PNG files can be read by other viewers without any issues.

TGLDPNG does not handle the encryption process directly. It merely triggers dedicated events during encryption and decryption.

Thus, the following processes must be handled:

- Encrypting data
- Decrypting encrypted data

For encryption, the process is carried out during the `OnEncode` event, and for decryption, the process is carried out during the `OnDecode` event.

In the `OnEncode` event, the data pointed to by the `pbuf` argument is encrypted for the number of data bytes specified by `buflen`. The `lineno` parameter corresponds to the Y-coordinate of the `pbuf` data, and `password` is the specified password data. Below is an example of the encryption process:

```delphi
// Encryption event (this is just an example with a simple method)
procedure TForm1.EncodeEvt(sender: TObject; pbuf: pbyte; buflen, lineno: integer; password: string);
var
  i, st, mx, af: integer;
  pc: pchar;
begin
  mx := length(password);
  pc := pchar(password);
  st := ((lineno + 3) and 2);
  if st >= mx then st := st - mx;
  Inc(pc, st);
  af := 0;
  for i := 0 to Pred(buflen) do
  begin
    af := ((pbuf^ - pbyte(pc)^) and $FF) xor af;
    pbuf^ := af;
    Inc(st); Inc(pc);
    if st >= mx then
    begin
      st := 0;
      pc := pchar(password);
    end;
    Inc(pbuf);
  end;
end;
```

In the `OnDecode` event, the data pointed to by the `pbuf` argument is decrypted for the number of data bytes specified by `buflen`. The `lineno` parameter corresponds to the Y-coordinate of the `pbuf` data, and `password` is the specified password data. Below is an example of the decryption process:

```delphi
// Decryption event (this is just an example with a simple method)
procedure TForm1.DecodeEvt(sender: TObject; pbuf: pbyte; buflen, lineno: integer; password: string);
var
  i, st, mx, af, n: integer;
  pc: pchar;
begin
  mx := length(password);
  pc := pchar(password);
  st := ((lineno + 3) and 2);
  if st >= mx then st := st - mx;
  Inc(pc, st);
  af := 0;
  for i := 0 to Pred(buflen) do
  begin
    n := pbuf^;
    pbuf^ := ((pbuf^ xor af) + pbyte(pc)^) and $FF;
    af := n;
    Inc(st); Inc(pc);
    if st >= mx then
    begin
      st := 0;
      pc := pchar(password);
    end;
    Inc(pbuf);
  end;
end;
```

### g. Background Color and Transparency Color

TGLDPNG supports both background and transparency colors.

During loading, if transparency or background colors exist, their values are assigned to the `TransColor` and `BGColor` properties. If they do not exist, the value `GLD_NONECOLOR` is assigned.

During saving, if the `TransColor` or `BGColor` properties are set to values other than `GLD_NONECOLOR`, those colors will be saved as the transparency or background colors. If the alpha channel is enabled, the transparency color will be ignored.

For 1, 16, or 256 color images, it is possible to specify a palette index for the `TransColor` and `BGColor` properties. When using an index, the index value should be OR'd with `$1000000`.

**Example:**

```delphi
// Extracting
if ((png.TransColor and $1000000) <> 0) and (png.TransColor <> GLD_NONECOLOR) then
  idx := png.TransColor and $FF;

// Setting
png.TransColor := (idx or $1000000);
```

When loading 1, 16, or 256 color images, the `TransColor` will always be an index value, so if you're using an older version of the software, modifications may be required.

For images other than 1, 16, or 256 colors, index specification will be ignored and treated like any other regular color. For example, specifying `$1000001` (Index=1) will be treated as `$000001` (R=1, G=0, B=0).

### h. Compiler Definitions

TGLDPNG provides several compile-time definitions, allowing for tailored compilation based on your needs.

The following are the available compiler definitions:

- **GLD_READONLY**
  Compiles TGLDPNG as a read-only class. If write functionality is not needed, this option can be specified to save space in the executable file.
- **GLD_SUPPORT_BIT15**
  Specifies the PNG format to be expanded as 15-bit (R:G:B = 5:5:5). If this is not specified, it defaults to 16-bit (R:G:B = 5:6:5) expansion.
- **GLD_NOREVERSE_ALPHA**
  Prevents the inversion of alpha channel values. This ensures the alpha channel matches the original PNG alpha values.

Note that the definition names have changed starting from version 3.4, so users of older versions will need to make adjustments.

### i. Migration from Older Versions

When migrating from TKPNG or TKPNGLE, the following changes need to be taken into account. Additionally, when migrating from an older version, be sure to **uninstall the old version (delete all files)** before installing GLDPNG.

1. TGLDPNG ver 3.2 Changes
  - The GetAlphaBitmap method has been renamed to AlphaBitmapAssignTo.
2. TGLDPNG ver 3.3 Changes
  - The values for transparency and background colors for 1/16/256 color images have changed, so adjustments are required.
  - The type of the `CompressLevel` property has changed, so modifications may be necessary.
  - The compiler definition names have changed, so if you are using them, updates will be needed.

## 4. License Agreement

The copyrights for TGLDPNG, as well as for functions and classes other than ZLIB, are held by the author (Tarquin). When using TGLDPNG, please read the following license agreement carefully before use.

**License Agreement (Revised 2004/05/09)**

The license is the "Modified BSD License."

**(Summary of the BSD License)**

There are no restrictions on the use (execution) of the software; it can be used freely. However, when redistributing the software, either in source code or binary form, the following must be included:

- Copyright notice
- List of conditions
- Disclaimer

Additionally, no support will be provided for this software, so use it at your own risk.

## 5. Support

No support is provided, so please use the software at your own risk.

## 6. Changelog

- **2004/5/9**
  - Changed the license to the BSD License
- **2002/3/17** ver 3.4.1 → ver 3.4.3
  - Fixed a bug with obtaining transparent colors for images with 256 colors or less.
  - Fixed a bug in the TGLDPNG Assign process.
- **2001/10/23**
  - Changed email address.
- **2001/07/08** ver 3.4 → ver 3.4.1
  - Fixed an issue where the GLD_REVERSEALPHA condition was omitted during output.
- **2001/07/04** ver 3.3.3 → ver 3.4
  - Allowed index specification for transparent and background colors.
  - Made the standard gamma value (0.45455) for Windows environments the default for output.
  - Ignored transparent color of TBitmap during output (due to special specifications).
  - Allowed preventing alpha channel reversal.
  - Fixed conversion bugs related to 16-bit images.
  - Updated ZLIB to 1.1.3.
  - Revised the license agreement.
  - Updated manual.
  - Added FAQ section.
- **2000/12/27** ver 3.3.1 → ver 3.3.3
  - Fixed the issue preventing "read-only" compilation.
- **2000/11/06** ver 3.3 → ver 3.3.1
  - Fixed issues with PixelFormat property when set to pfCustom (probably).
  - Fixed an issue where a palette was attached when saving grayscale images.
  - Fixed issues with transparent and background color settings when saving grayscale images with 2-step or 16-step.
  - Ensured that comments are read more strictly.
- **2000/10/30** ver 3.2.6 → ver 3.3
  - Added support for loading and saving 15-bit and 16-bit bitmaps.
  - Added almost complete grayscale saving functionality.
  - Added save time functionality.
  - Added color information.
  - Fixed minor bugs.
  - Made some specification changes.
  - Made adjustments for MNG internal streams.
  - Fully updated manual.
- **2000/07/21** ver 3.2.5 → ver 3.2.6
  - Fixed issues occurring with C++Builder 4 and later.
  - Revised the terms of use.
- **2000/03/26** ver 3.2.2 → ver 3.2.5?
  - Fixed a bug where the progress event would not trigger correctly when loading images with an alpha channel.
  - Fixed a memory leak issue.
  - Help file was updated to version 3.2.3 (thanks again, Kougetsu-san!).
- **2000/02/18** ver 3.2.2 → ver 3.2.3
  - Fixed bugs related to reading 4-color images.
  - Added a standard help file (thanks to Kougetsu Rin-ka!).
  - Revised part of the reference documentation.
  - Revised the terms of use.
- **2000/02/10** ver 3.2.1 → ver 3.2.2
  - Fixed bugs with ZLIB-related functions during writing.
  - Added compiler directives to turn off range checks and overflow checks.
  - Changed TBitmap to Graphics.TBitmap (to distinguish from Windows.TBitmap).
- **2000/02/03** ver 3.2 → ver 3.2.1
  - Fixed bugs related to reading 4-step and 256-step grayscale images with alpha channels, and 4-color images.
- **2000/02/02** ver 3.1 → ver 3.2
  - Fixed the issue of missing override for some methods with abstract.
  - Added support for saving images as 30,000 or 60,000-color images using the sBIT chunk.
  - Added resolution input/output support, allowing aspect ratios and DPI values to be saved.
  - Fixed a bug where transparent colors were output even when an alpha channel was present.
  - Added "read-only" compilation mode.
  - Improved TGLDPNG to behave more like a graphics class.
  - Modified to return more information about the loaded data.
  - Fixed bugs related to converting palette-based alpha channels.
  - Major revisions and additions to the reference manual.
- **2000/01/28**
  - Initial release.
