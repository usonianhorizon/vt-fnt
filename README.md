# vt-fnt

# Introduction

A collection of bitmap .FNT fonts created from various programming fonts, and to be used by FreeBSD's vt(-ng) console terminal.

At present, only **IBMPlexMono-Light** and **IBMPlexMono-Medium** each in pixel height ranges of 12&ndash;16 have been converted.

_These were generated using FontForge under FreeBSD with 160dpi setting on output to best match FHD (1920x1080) on a 14" diagonal screen size._

## Notes on vtfontcvt-ng

The relevant lines in vtfontcvt-ng are 300-348 at [vtfontcvt-ng.c](https://github.com/t6/vtfontcvt-ng/blob/master/vtfontcvt-ng.c) which relate to parsing the section commented as:


_Step 1: Parse FONT logical font descriptor and FONTBOUNDINGBOX bounding box_

## Recipe to generate .FNT from .OTF

Here, in any case, is my recipe:

### Part I&mdash;Convert .OTF to .BDF

1. Use FontForge to load up .OTF file (in the weight of choice).
1. Force encoding through `Encoding > Force Encoding > ISO 8859-1`
1. Generate bitmaps through `Element > Bitmap Strikes Available`
1. Generate font through `File > Generate Fonts...` after selecting BDF output

 _(Don't worry too much about the other options, as a hand-edit in Part II will take care of things.)_

### Part II&mdash;Fix generated BDF files [BDF font format](https://en.wikipedia.org/wiki/Glyph_Bitmap_Distribution_Format)

Now the BDF will have a couple of things that will trip up vtfontcvt-ng:

1. The dreaded `font spacing "C" (character cell) required` error.
1. `bitmap with unsupported DWIDTH`.
1. Even after you fix this you might/will get `PIO_FONT: Invalid argument`.

A couple of edits in the BDF will solve this:

1. `FONTBOUNDINGBOX <fbbw>` has to match all the `DWIDTH <dwidth>` lines below (or x2 according to source, but we'll put that aside). This will require some hand-tuning because if you change fbbw to match dwidth the characters will be scrunched; and if you change dwidth to match fbbw, then your characters will be spaced too far apart. Anyway the important thing is that they match.
1. The other edit was to delete all characters after 255.  The `STARTCHAR` might be descriptive or not (IBMPlexMono has it as _ydieresis_), but just delete all of the remaining chars while leaving the final `ENDFONT` line in place.  In the case of IBMPlexMono, I simply searched for `ENCODING 256`; selected from the previous `STARTCHAR` line to the penultimate line; and then deleted that selected block, leaving the `ENDFONT` line intact.
1. In `FONT` line ensure that `SPACING` as per X- logical font description is 'C' for _Character Cell_ \(See: [X logical font description](https://en.wikipedia.org/wiki/X_logical_font_description)\).  And also ensure that the line that explicitly states `SPACING` matches with a corresponding 'C'.

## Part III&mdash;Install FNT files
With these changes go ahead and run the usual commands to install:

```
$ vtfontcvt-ng IBMPlexMono/IBMPlexMono-Light-14.bdf fnt/IBMPlexMono-Light-14.fnt
$ sudo cp fnt/*.fnt /usr/share/vt/fonts
$ sudo vidcontrol -f /usr/share/vt/fonts/IBMPlexMono-Light-14.fnt # While in vt, and not in X-
```

That should give you a path to a font of choice.  Of course, this doesn't ensure that the generated font will be at all aesthetically pleasing, and certainly won't be as nice as a handcrafted bitmap font.  But it did allow me to get a decent approximation of IBMPlexMono under vt.

Regards.
