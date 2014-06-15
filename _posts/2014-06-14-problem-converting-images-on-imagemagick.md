---
layout: post
title: "Problem converting images on imagemagick"
modified: 2014-06-14 19:39:09 -0700
tags: [technical, imagemagick, dragonfly]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
---

During my recent time machine restore to my laptop, I came across these errors while rendering my images on my rails app.

{% highlight css %}
DRAGONFLY: shell command: 'convert' '/Users/kc/Documents/sm_portfolio/public/system/dragonfly/development/2014/06/14/75v5688cse_2014_01_27_032738.jpg' '-resize' '600x400^^' '-gravity' 'Center' '-crop' '600x400+0+0' '+repage' '/var/folders/_x/_8_gx72j2_b429n7m1nvz7q00000gn/T/dragonfly20140614-52534-8molud.jpg'
DRAGONFLY: caught error - Command failed ('convert' '/Users/kc/Documents/sm_portfolio/public/system/dragonfly/development/2014/06/14/75v5688cse_2014_01_27_032738.jpg' '-resize' '600x400^^' '-gravity' 'Center' '-crop' '600x400+0+0' '+repage' '/var/folders/_x/_8_gx72j2_b429n7m1nvz7q00000gn/T/dragonfly20140614-52534-8molud.jpg') with exit status  and stderr dyld: Library not loaded: /usr/local/lib/libpng16.16.dylib
  Referenced from: /usr/local/lib/libfreetype.6.dylib
  Reason: image not found
{% endhighlight %}

So all my images wouldn't load no matter what. I started looking up solutions online (including stackoverflow and what not), but everything I found didn't fix my errors.

I tried all kinds of ways including reinstalling imagemagick, libpng, unlinked and symlinked the libpng path manually (and etc) using brew. Nothing helped.

Then I called "convert" specifically:

{% highlight css %}
$ convert -version
dyld: Library not loaded: /usr/local/lib/libpng16.16.dylib
  Referenced from: /usr/local/lib/libfreetype.6.dylib
  Reason: image not found
Trace/BPT trap: 5
ks-macbook-pro:sm_portfolio kc$ find /usr |grep "libpng16"
/usr/local/Cellar/libpng/1.6.10/bin/libpng16-config
/usr/local/Cellar/libpng/1.6.10/include/libpng16
/usr/local/Cellar/libpng/1.6.10/include/libpng16/png.h
/usr/local/Cellar/libpng/1.6.10/include/libpng16/pngconf.h
/usr/local/Cellar/libpng/1.6.10/include/libpng16/pnglibconf.h
/usr/local/Cellar/libpng/1.6.10/lib/libpng16.16.dylib
/usr/local/Cellar/libpng/1.6.10/lib/libpng16.a
/usr/local/Cellar/libpng/1.6.10/lib/libpng16.dylib
/usr/local/Cellar/libpng/1.6.10/lib/pkgconfig/libpng16.pc
/usr/local/lib/pkgconfig/libpng16.pc
{% endhighlight %}

Hmmm....~!

So my next approach, I did this:

{% highlight css %}
$ brew unlink libpng && brew link libpng
Unlinking /usr/local/Cellar/libpng/1.6.10... 2 symlinks removed
Linking /usr/local/Cellar/libpng/1.6.10... 18 symlinks created
ks-macbook-pro:sm_portfolio kc$ ls /usr/local/lib/*png*
/usr/local/lib/libpng.a     /usr/local/lib/libpng16.a
/usr/local/lib/libpng.dylib   /usr/local/lib/libpng16.dylib
/usr/local/lib/libpng16.16.dylib
{% endhighlight %}

After that, I tried reloading the images on my browser, now I get this new error in terminal:

{% highlight css %}
DRAGONFLY: shell command: 'convert' '/Users/kc/Documents/sm_portfolio/public/system/dragonfly/development/2014/06/14/75v5688cse_2014_01_27_032738.jpg' '-resize' '600x400^^' '-gravity' 'Center' '-crop' '600x400+0+0' '+repage' '/var/folders/_x/_8_gx72j2_b429n7m1nvz7q00000gn/T/dragonfly20140614-52820-pmardu.jpg'
DRAGONFLY: caught error - Command failed ('convert' '/Users/kc/Documents/sm_portfolio/public/system/dragonfly/development/2014/06/14/75v5688cse_2014_01_27_032738.jpg' '-resize' '600x400^^' '-gravity' 'Center' '-crop' '600x400+0+0' '+repage' '/var/folders/_x/_8_gx72j2_b429n7m1nvz7q00000gn/T/dragonfly20140614-52820-pmardu.jpg') with exit status 1 and stderr convert: unable to load module `/usr/local/Cellar/imagemagick/6.8.9-1/lib/ImageMagick//modules-Q16/coders/jpeg.la': file not found @ error/module.c/OpenModule/1282.
convert: no decode delegate for this image format `JPEG' @ error/constitute.c/ReadImage/501.
convert: no images defined `/var/folders/_x/_8_gx72j2_b429n7m1nvz7q00000gn/T/dragonfly20140614-52820-pmardu.jpg' @ error/convert.c/ConvertImageCommand/3187.
{% endhighlight %}

Oh man, new errors. But hey, that fixed my old error! Good, at least we're moving along... meaning, more research...

Found [this](http://stackoverflow.com/questions/9586048/imagemagick-no-decode-delegate) on stackoverflow.

So let's check my current package list:

{% highlight css %}
$identify -list format
   Format  Module    Mode  Description
-------------------------------------------------------------------------------
      3FR  DNG       r--   Hasselblad CFV/H3D39II
        A* RAW       rw+   Raw alpha samples
      AAI* AAI       rw+   AAI Dune image
      ART* ART       rw-   PFS: 1st Publisher Clip Art
      ARW  DNG       r--   Sony Alpha Raw Image Format
      AVI  MPEG      r--   Microsoft Audio/Visual Interleaved
      AVS* AVS       rw+   AVS X image
        B* RAW       rw+   Raw blue samples
      BGR* BGR       rw+   Raw blue, green, and red samples
     BGRA* BGR       rw+   Raw blue, green, red, and alpha samples
      BMP* BMP       rw-   Microsoft Windows bitmap image
...
...
...
{% endhighlight %}

Ah, jpeg wasn't in there, that's why! So, off we go...

{% highlight css %}
$brew uninstall imagemagick jpeg
$brew install imagemagick
{% endhighlight %}

Now, reload the page, voila~~! Issue fixed!
