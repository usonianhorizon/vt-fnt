# vt-fnt
A collection of bitmap .FNT fonts created from various programming fonts, and to be used by FreeBSD's vt console terminal.

These were generated using FontForge with 160dpi setting on output to best match FHD (1920x1080) on a 14" diagonal screen size.


The relevant lines in vtfontcvt-ng are 300-348 at https://github.com/t6/vtfontcvt-ng/blob/master/vtfontcvt-ng.c which relate to parsing the section commented as:


Step 1: Parse FONT logical font descriptor and FONTBOUNDINGBOX bounding box


In any case, here is my recipe:


    Use FontForge to load up .OTF file (in the weight of choice).
    Force encoding through Encoding > Force Encoding > ISO 8859-1
    Generate bitmaps through Element > Bitmap Strikes Available
    Generate font through File > Generate Fonts... after selecting BDF output
    (Don't worry too much about the other options, as a hand-edit in Part II will take care of things.)

Now the BDF will have a couple of things that will trip up vtfontcvt-ng:


    The dreaded [CMD]font spacing "C" (character cell) required[/CMD] error.
    [CMD]bitmap with unsupported DWIDTH[/CMD]
    Even after you fix this you might/will get [CMD]PIO_FONT: Invalid argument[/CMD]

A couple of edits in the BDF will solve this:


    [CMD]FONTBOUNDINGBOX <fbbw>[/CMD] has to match all the [CMD]DWIDTH <dwidth>[/CMD] lines below (or x2 according to source, but we'll put that aside). This will require some hand-tuning because if you change fbbw to match dwidth the characters will be scrunched; and if you change dwidth to match fbbw, then your characters will be spaced too far apart. Anyway the important thing is that they match.
    The other edit was to delete all characters after 255.  The [CMD]STARTCHAR[/CMD] might be descriptive or not (IBMPlexMono has it as ydieresis), but just delete all of the remaining chars while leaving the final [CMD]ENDFONT[/CMD] line in place.
    In [CMD]FONT[/CMD] line ensure that [CMD]SPACING[/CMD] as per X logical font description is 'C'.  And also ensure that the line that explicitly states [CMD]SPACING[/CMD] matches with a corresponding 'C'.

With these changes go ahead and run the usual commands to install:


[CMD]$ vtfontcvt-ng IBMPlexMono/IBMPlexMono-14.bdf fnt/IBMPlexMono-14.fnt

$ sudo cp fnt/*.fnt /usr/share/vt/fonts

$ sudo vidcontrol -f /usr/share/vt/fonts/IBMPlexMono-14.fnt # While in vt, and not X-[/CMD]


That should give you a path to a font of choice.  Of course, this doesn't ensure that the generated font will be at all aesthetically pleasing, and certainly won't be as nice as a handcrafted bitmap font.  But it did allow me to get a decent approximation of IBMPlexMono under vt.
