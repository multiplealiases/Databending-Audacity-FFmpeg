# Databending with Audacity: FFmpeg as an Intermediary for Images

## Required Software
(These are not strictly needed to achieve the same result; this is just what I know works. Feel free to port this piece to other OSes or automate/optimize it)
 - Windows 7 or newer
 - [Irfanview](https://www.irfanview.com/)
 - [Audacity](https://www.audacityteam.org/)
 - [FFmpeg](https://ffmpeg.org/)
 - [YUView](https://ient.github.io/YUView/) (optional, but used for demonstration purposes)
## Required Reading
If you wish to follow along, please read and understand these two articles first:

- [Databending using Audacity](https://www.hellocatfood.com/databending-using-audacity/) by Antonio Roberts

 - [Databending with Audacity: What I do Differently/Required Reading](https://github.com/multiplealiases/Databending-In-Audacity-Required-Reading/blob/main/README.md) by me

I'll assume you understand the basic idea of converting images into raw data, importing them into Audacity, applying effects to it, exporting the results, then opening the raw data in a suitable image viewer. That's covered by the first one.

The second one covers how I do databending with Audacity, and how my method differs from the one detailed in the first article.

## Required Skills
I will assume you have some ability to navigate a command-line interface. At the very least, you need to be able to change directories.

## A note on replicating my results
The images that I'm using to demonstrate this can be found at unsplash.com. I have downloaded the Medium (something × 1920) versions of the images.

## FFmpeg?
To quote the FFmpeg website, it is "a complete, cross-platform solution to record, convert and stream audio and video.". If you've ever used HandBrake or other video transcoders, chances are, you've used FFmpeg before without realizing it. Video editing software (especially open-source ones)? FFmpeg is how those manage to render video and audio, and it would not be unreasonable to say that the entire digital video editing landscape owes its entire existence to FFmpeg in some way.

### That's great, but how does that relate to this article?
While they say FFmpeg is for video, it's absolutely possible to use it for raw images as well. After all, an image is just a single-frame video.

So the plan here is to use FFmpeg to convert our pictures into different pixel formats (ways to encode image data), databend the intermediates, then use FFmpeg again to reconvert the databent images into a format that our programs understand.

## How?
First thing, of course, is to get ourselves a copy of FFmpeg, but we encounter a small roadblock the moment we go to the Downloads page. Quoth the page: "FFmpeg only provides source code. Below are some links that provide it already compiled and ready to go.". Okay. 

Let's get the gyan.dev copy.  Just get the full-git release; may as well get the full version. The version shouldn't matter. I'm writing this on the 7th of February, 2021, so maybe this may be outdated if you're reading this in the future, but then again, FFmpeg is one of the backbones of the modern video world, so it shouldn't change too much.

![enter image description here](https://i.imgur.com/9xSPpKx.png)

Alright, that's our beautiful new copy of FFmpeg ready to go. Let's extract it:

![enter image description here](https://i.imgur.com/IhNNTcP.png)

Alright. Let's copy an image into the \bin directory. This will be today's test subject:

![white, pink, purple, and green flowers wall decor](https://i.imgur.com/5IXouiI.jpg)
[Photo](https://unsplash.com/photos/koYklSUjAXc) by [Amy Shamblen](https://unsplash.com/@amyshamblen) on [Unsplash](https://unsplash.com/)

As usual, convert this image into planar (keep this in mind for later) RAW, and we can finally dip our toes into the command line. For reasons that will be obvious later, I'd suggest a simple, 1-word name for the RAW image. I used the name "flowers.raw".

## Sanity-checking FFmpeg
First thing you want to do right after opening Command Prompt is to change directories to (FFmpeg directory)\bin\. Oh, and change disks if it's on a different disk.

	Microsoft Windows [Version 10.0.19042.746]
	(c) 2020 Microsoft Corporation. All rights reserved.
	
	C:\Windows\System32>D:
	
	D:\>cd D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin
	
	D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin>

By the way, you don't have to actually type in that long mess of alphanumeric soup. Just go to the directory in Explorer, and click on the folder icon on the left side of the address bar.

![enter image description here](https://i.imgur.com/y7OJzG7.png)

Press Ctrl+C to copy, then go back to Command Prompt, type in "cd", then press Ctrl+V to paste in the whole directory. Press Enter, and you've just changed to that directory in a few keystrokes (and a lot of mouse movement). Easy.

Alright, let's check that it's actually working properly. Type in

	ffmpeg.exe -s 1920x1920 -f rawvideo  -pix_fmt rgb24 -i flowers.raw -f rawvideo -pix_fmt rgb24 flowers-sanity-check.raw

At this point, you may be asking "What the hell is this?". Let's break down what this actually means. Keep in mind the switches before "flowers.raw" apply to the input file, "flowers.raw", while the switches after "flowers.raw" but before "flowers-sanity-check.raw" apply to the output file, "flowers-sanity-check.raw".

### Input Parameters

	ffmpeg.exe
This runs ffmpeg.exe. It should be simple enough to understand.

	-s 1920x1920
This tells FFmpeg that the input has a resolution of 1920×1920. In general, this has to be set to the resolution of your image.

	-f rawvideo
This indicates to FFmpeg that the input is in the rawvideo codec. This is raw video, though only one frame of that.

	-pix_fmt rgb24
This means that FFmpeg uses the RGB24 pixel format. This corresponds to the interleaved 24-bit RGB format. This guide will assume that you're using 24-bit RGB images.

	-i flowers.raw
This points FFmpeg to flowers.raw as the input file.

That's the input section done. Let's look at the output parameters.

### Output Parameters

	-f rawvideo
Same thing; just forces FFmpeg to use the rawvideo codec on the output.

	-pix_fmt rgb24
This means that FFmpeg uses the RGB24 pixel format on the output. This corresponds to the interleaved 24-bit RGB format.

	flowers-sanity-check.raw
This tells FFmpeg to name the output file "flowers-sanity-check.raw".

#### What's a "sanity check"?
It's a common term in programming. It refers to a basic test to quickly evaluate whether a claim or the result of a calculation can possibly be true. In this case, we're testing that our copy of FFmpeg is actually working correctly by just converting an image back to itself. If it can't do that right, something has gone horribly wrong.

### Back to FFmpeg
So in essence,

	ffmpeg.exe -s 1920x1920 -f rawvideo  -pix_fmt rgb24 -i flowers.raw -f rawvideo -pix_fmt rgb24 flowers-sanity-check.raw
	
 takes a 1920x1920 raw RGB 24-bit image called "flowers.raw" and converts it back to a raw RGB 24-bit image "flowers-sanity-check.raw". This should make an identical copy of the image under a different name.

You should see the following, or something like it:

	D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin>ffmpeg.exe -s 1920x1920 -f rawvideo  -pix_fmt rgb24 -i flowers.raw -f rawvideo -pix_fmt rgb24 flowers-sanity-check.raw
	ffmpeg version 2021-02-02-git-2367affc2c-full_build-www.gyan.dev Copyright (c) 2000-2021 the FFmpeg developers
	  built with gcc 10.2.0 (Rev6, Built by MSYS2 project)
	  configuration: --enable-gpl --enable-version3 --enable-static --disable-w32threads --disable-autodetect --enable-fontconfig --enable-iconv --enable-gnutls --enable-libxml2 --enable-gmp --enable-lzma --enable-libsnappy --enable-zlib --enable-libsrt --enable-libssh --enable-libzmq --enable-avisynth --enable-libbluray --enable-libcaca --enable-sdl2 --enable-libdav1d --enable-libzvbi --enable-librav1e --enable-libsvtav1 --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxvid --enable-libaom --enable-libopenjpeg --enable-libvpx --enable-libass --enable-frei0r --enable-libfreetype --enable-libfribidi --enable-libvidstab --enable-libvmaf --enable-libzimg --enable-amf --enable-cuda-llvm --enable-cuvid --enable-ffnvcodec --enable-nvdec --enable-nvenc --enable-d3d11va --enable-dxva2 --enable-libmfx --enable-libglslang --enable-vulkan --enable-opencl --enable-libcdio --enable-libgme --enable-libmodplug --enable-libopenmpt --enable-libopencore-amrwb --enable-libmp3lame --enable-libshine --enable-libtheora --enable-libtwolame --enable-libvo-amrwbenc --enable-libilbc --enable-libgsm --enable-libopencore-amrnb --enable-libopus --enable-libspeex --enable-libvorbis --enable-ladspa --enable-libbs2b --enable-libflite --enable-libmysofa --enable-librubberband --enable-libsoxr --enable-chromaprint
	  libavutil      56. 64.100 / 56. 64.100
	  libavcodec     58.120.100 / 58.120.100
	  libavformat    58. 65.101 / 58. 65.101
	  libavdevice    58. 11.103 / 58. 11.103
	  libavfilter     7.101.100 /  7.101.100
	  libswscale      5.  8.100 /  5.  8.100
	  libswresample   3.  8.100 /  3.  8.100
	  libpostproc    55.  8.100 / 55.  8.100
	[rawvideo @ 000002a7108ae840] Estimating duration from bitrate, this may be inaccurate
	Input #0, rawvideo, from 'flowers.raw':
	  Duration: 00:00:00.04, start: 0.000000, bitrate: 2211840 kb/s
    Stream #0:0: Video: rawvideo (RGB[24] / 0x18424752), rgb24, 1920x1920, 2211840 kb/s, 25 tbr, 25 tbn, 25 tbc
	Stream mapping:
	  Stream #0:0 -> #0:0 (rawvideo (native) -> rawvideo (native))
	Press [q] to stop, [?] for help
	Output #0, rawvideo, to 'flowers-sanity-check.raw':
	  Metadata:
    encoder         : Lavf58.65.101
    Stream #0:0: Video: rawvideo (RGB[24] / 0x18424752), rgb24(progressive), 1920x1920, q=2-31, 2211840 kb/s, 25 fps, 25 tbn
    Metadata:
      encoder         : Lavc58.120.100 rawvideo
	frame=    1 fps=0.0 q=-0.0 Lsize=   10800kB time=00:00:00.04 bitrate=2211840.0kbits/s speed=2.51x
	video:10800kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.000000%

	D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin>

Alright, let's use Irfanview to open up this image. The basic things to remember at the RAW open dialog are:

 - Resolution is 1920×1920
 - Select 24BPP
 - Select Planar

 I've gone through and exported it to JPEG to share this, obviously, but you should be seeing something like this:
 
![enter image description here](https://i.imgur.com/lnkprkI.jpg)

If that's correct, then we should be good. If not, please share it; the name of my game is databending, so the more distorted something gets, the better.

## Using FFmpeg to Convert Between Pixel Formats
Alright, let's actually start doing the thing you came for.
Run:

	ffmpeg.exe -s 1920x1920 -f rawvideo  -pix_fmt rgb24 -i flowers.raw -f rawvideo -pix_fmt yuv444p flowers-yuv444p.raw

This converts an interleaved RGB 24-bit to planar YUV 4:4:4. Let's use YUView to open it.

![enter image description here](https://i.imgur.com/PgdkJ9l.png)

Go to File > Open File..., and you'll see the Open File dialog. Not much here, just navigate to the FFmpeg folder and make sure the file type is:

![Any files (asterisk)](https://i.imgur.com/9xXDNLi.png)

YUView will ask you what type of file it is, so just click OK, since it's correct:

![enter image description here](https://i.imgur.com/PAsz4Vb.png)

You'll then see almost the same thing, but something's loaded in.

![enter image description here](https://i.imgur.com/7tB5mXN.png)

Write the relevant parameters; set resolution to 1920x1920, and YUV format to YUV 4:4:4 8-bit. Zoom out a bit, and you'll see...
![](https://i.imgur.com/bWlxOSt.png)
### A diversion into planar and interleaved RGB, and how misinterpretation affects the output

"What?" is probably what you're asking right now. You see, this is a byproduct of pushing *planar* RGB images into a thing that's expecting *interleaved* RGB images.

Let's demonstrate with good old Irfanview. Open the original "flowers.raw" image, but instead of selecting Planar, select Interleaved.

![enter image description here](https://i.imgur.com/JKsl0az.png)

The same thing happens. It's caused by planar images being opened as interleaved. 

For those wondering, here's what happens in the reverse scenario; interleaved opened as planar:

![enter image description here](https://i.imgur.com/eWSPZAK.jpg)

They're definitely interesting databends themselves, but I don't think I could write an entire article on just this.



### Back on track

Right. So we now have a RAW YUV 4:4:4 image. Let's databend it like any other image. Open it in Audacity. As usual, we'll import with the following settings:

![enter image description here](https://i.imgur.com/hRg62Bm.png)

And...

![enter image description here](https://i.imgur.com/DwwEsff.png)

Since I'm a one-trick pony, I'll do my favorite trick: Paulstretch at 50 seconds, and Amplify to avoid clipping.

![enter image description here](https://i.imgur.com/HQPaZoz.png)

Now, to prevent FFmpeg from rejecting the file outright as completely corrupt, we need to add some space to the end of the file. Select just the end of the file, and go under Generate > Silence...

![enter image description here](https://i.imgur.com/8oE6or5.png)

This isn't an exact science or anything, just add a minute or two of silence to the end of the audio. For larger files, I'd suggest a quarter of the original file's length to be safe. We just need to trick FFmpeg into accepting our databent file, and all it's really checking is if the input file's size is equal or bigger than what it's expecting.

Export this audio as raw unsigned 8-bit, as usual. For those following along at home, name this one "flowers-ps50-yuv444p.raw"

![enter image description here](https://i.imgur.com/bAJozHO.png)

...and now we have this mess. Let's look at it in YUView for a bit.

![insert description here](https://i.imgur.com/2CjhIAr.jpg)

Just looks like one of those Paulstretched interleaved RGB images. Don't worry, this isn't the final result. 

A note on YUV is found at the bottom of this article.

### Using FFmpeg to convert from databent YUV 4:4:4 to  24-bit RGB

I assume you haven't closed Command Prompt in the meantime (like I have), else you'll have to navigate back to the (FFmpeg directory)\bin directory.

Once that's done, run

	ffmpeg -s 1920x1920 -f rawvideo -pix_fmt yuv444p -i flowers-ps50-yuv444p.raw -f rawvideo -pix_fmt rgb24 flowers-rgb24-ps50-yuv444p.raw

This is the reverse process of what we did earlier; convert an image from YUV 4:4:4 planar into 24-bit interleaved RGB .

You should see something like this:

	D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin>ffmpeg -s 1920x1920 -f rawvideo -pix_fmt yuv444p -i flowers-ps50-yuv444p.raw -f rawvideo -pix_fmt rgb24 flowers-rgb24-ps50-yuv444p.raw
	ffmpeg version 2021-02-02-git-2367affc2c-full_build-www.gyan.dev Copyright (c) 2000-2021 the FFmpeg developers
	  built with gcc 10.2.0 (Rev6, Built by MSYS2 project)
	  configuration: --enable-gpl --enable-version3 --enable-static --disable-w32threads --disable-autodetect --enable-fontconfig --enable-iconv --enable-gnutls --enable-libxml2 --enable-gmp --enable-lzma --enable-libsnappy --enable-zlib --enable-libsrt --enable-libssh --enable-libzmq --enable-avisynth --enable-libbluray --enable-libcaca --enable-sdl2 --enable-libdav1d --enable-libzvbi --enable-librav1e --enable-libsvtav1 --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxvid --enable-libaom --enable-libopenjpeg --enable-libvpx --enable-libass --enable-frei0r --enable-libfreetype --enable-libfribidi --enable-libvidstab --enable-libvmaf --enable-libzimg --enable-amf --enable-cuda-llvm --enable-cuvid --enable-ffnvcodec --enable-nvdec --enable-nvenc --enable-d3d11va --enable-dxva2 --enable-libmfx --enable-libglslang --enable-vulkan --enable-opencl --enable-libcdio --enable-libgme --enable-libmodplug --enable-libopenmpt --enable-libopencore-amrwb --enable-libmp3lame --enable-libshine --enable-libtheora --enable-libtwolame --enable-libvo-amrwbenc --enable-libilbc --enable-libgsm --enable-libopencore-amrnb --enable-libopus --enable-libspeex --enable-libvorbis --enable-ladspa --enable-libbs2b --enable-libflite --enable-libmysofa --enable-librubberband --enable-libsoxr --enable-chromaprint
	  libavutil      56. 64.100 / 56. 64.100
	  libavcodec     58.120.100 / 58.120.100
	  libavformat    58. 65.101 / 58. 65.101
	  libavdevice    58. 11.103 / 58. 11.103
	  libavfilter     7.101.100 /  7.101.100
	  libswscale      5.  8.100 /  5.  8.100
	  libswresample   3.  8.100 /  3.  8.100
	  libpostproc    55.  8.100 / 55.  8.100
	[rawvideo @ 000002ee4511e840] Estimating duration from bitrate, this may be inaccurate
	Input #0, rawvideo, from 'flowers-ps50-yuv444p.raw':
	  Duration: 00:00:00.04, start: 0.000000, bitrate: 2626352 kb/s
    Stream #0:0: Video: rawvideo (444P / 0x50343434), yuv444p, 1920x1920, 2211840 kb/s, 25 tbr, 25 tbn, 25 tbc
	Stream mapping:
	  Stream #0:0 -> #0:0 (rawvideo (native) -> rawvideo (native))
	Press [q] to stop, [?] for help
	Output #0, rawvideo, to 'flowers-rgb24-ps50-yuv444p.raw':
	  Metadata:
    encoder         : Lavf58.65.101
    Stream #0:0: Video: rawvideo (RGB[24] / 0x18424752), rgb24(pc, progressive), 1920x1920, q=2-31, 2211840 kb/s, 25 fps, 25 tbn
    Metadata:
      encoder         : Lavc58.120.100 rawvideo
	Truncating packet of size 11059200 to 2072560me=00:00:00.04 bitrate=2202009.6kbits/s speed=4e+04x
	[rawvideo @ 000002ee4511e840] Packet corrupt (stream = 0, dts = 1).
	flowers-ps50-yuv444p.raw: corrupt input packet in stream 0
	[rawvideo @ 000002ee4512bd40] Invalid buffer size, packet size 2072560 < expected frame_size 11059200
	Error while decoding stream #0:0: Invalid argument
	frame=    1 fps=0.0 q=-0.0 Lsize=   10800kB time=00:00:00.04 bitrate=2211840.0kbits/s speed=0.652x
	video:10800kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.000000%

	D:\FFmpeg\ffmpeg-2021-02-02-git-2367affc2c-full_build\bin>
	
The things to note here are
	
	[rawvideo @ 000002ee4511e840] Packet corrupt (stream = 0, dts = 1).
	flowers-ps50-yuv444p.raw: corrupt input packet in stream 0
	[rawvideo @ 000002ee4512bd40] Invalid buffer size, packet size 2072560 < expected frame_size 11059200
	Error while decoding stream #0:0: Invalid argument

Now, this doesn't mean anything for us. What this means is that FFmpeg saw the empty space after processing the first "frame" of this 1-frame video, and it thought that it was corrupted because it's not the right size.

However, this message means that something's gone wrong:

	Output file is empty, nothing was encoded (check -ss / -t / -frames parameters if used)
	Conversion failed!

Translation: File not large enough, just pad out the end of the audio with (more) silence and export again.

If everything's gone right, we can now put this back into Irfanview and open it. As always, use planar, since our source image was planar.

![enter image description here](https://i.imgur.com/2N1R8D4.jpg)

Standard Paulstretch image. 

Let's do it again on the same image, but as a bog-standard 24-bit RGB raw image.

![enter image description here](https://i.imgur.com/aGP3Wyw.jpg)

I mean, it's different. You get two images from one, pretty good deal, I'd say.

## Why is interleaved bad?
I'll show you. Convert the original JPEG image into interleaved RAW, and run the following (changing file names if you're using different files):

	ffmpeg -s 1920x1920 -f rawvideo -pix_fmt rgb24 -i flowers-interleaved.raw -f rawvideo -pix_fmt yuv444p flowers-interleaved-yuv444p.raw

Opening it in YUView shows a perfectly normal image, since it's actually getting the right format this time.

![enter image description here](https://i.imgur.com/zbuzGDI.png)
Import it as raw in Audacity, as usual, then do a Paulstretch of 50 seconds and Amplify at default setttings to prevent color clipping. Never forget to pad out the end of the "audio" with silence. Export.

Run FFmpeg again, but this time converting to 24-bit RGB:

	ffmpeg -s 1920x1920 -f rawvideo -pix_fmt yuv444p -i flowers-interleaved-ps50-yuv444p.raw -f rawvideo -pix_fmt rgb24 flowers-interleaved-rgb24-ps50-yuv444p.raw

Open it in Irfanview, and we get:

![enter image description here](https://i.imgur.com/YtblGGO.jpg)
I'm not saying it's bad, but it's certainly less colorful than its planar counterpart. It's like [a VHS video undergoing generation loss](https://youtu.be/bmPoqPGitEc). If that's the look you're going for, then work in interleaved, absolutely!

## Playing with different pixel formats
Let's databend the same image with the same effect, but in different pixel formats. A list of FFmpeg's available pixel formats can be found by [looking at the source code](https://ffmpeg.org/doxygen/trunk/pixfmt_8h_source.html).

Also, these pixel formats are case-sensitive. When replacing the pix_fmt parameter to make your own, always type them in lowercase.

### RGBA64LE (packed RGBA 16:16:16:16, 64bpp, 16R, 16G, 16B, 16A, the 2-byte value for each R/G/B/A component is stored as little-endian)

![enter image description here](https://i.imgur.com/OHRkH9O.jpg)

### RGBA64LE, interleaved source
![enter image description here](https://i.imgur.com/CKUWzqu.jpg)

### RGB565 (packed RGB 5:6:5, 16bpp, (msb) 5R 6G 5B(lsb), little-endian)

![enter image description here](https://i.imgur.com/ViUyaSi.jpg)
### RGB565, interleaved source 
![enter image description here](https://i.imgur.com/yM3BI9X.jpg)

### XYZ12LE (packed XYZ 4:4:4, 36 bpp, (msb) 12X, 12Y, 12Z (lsb), the 2-byte value for each X/Y/Z is stored as little-endian, the 4 lower bits are set to 0)

![enter image description here](https://i.imgur.com/ESm4BY5.jpg)
### XYZ12LE, interleaved source

![enter image description here](https://i.imgur.com/MENxEmY.jpg)
### A note on "larger" formats

Some of the formats listed here are larger than 24 bits per pixel. Hence, the raw files they produce are longer and therefore need longer Paulstretch Time Resolutions to prevent that horizontal banding that's characteristic of too-low Time Resolutions. Or don't. Maybe that's the look that you're going for.

## Conclusion
I'm not quite sure what to say here. FFmpeg just lets us make new images out of the same image repeatedly.

I haven't even scratched the surface of what's possible with this. Try different effects to see how the different pixel formats react to different effects.

If there's anything you'd like to add or change, fork (and add a pull request) this repository or contact me on GitHub at [multiplealiases](https://github.com/multiplealiases). 



## Footnotes
### On YUV 4:4:4
"What's YUV, and why is it 4:4:4?"

You know how RGB formats store the Red, Green, and Blue components of an image? YUV (technically, since it's digital, it should be YCbCr, but they're used interchangeably by basically everyone) is a format where you store the Y, U and V components.

Y is the luminance or brightness component. This is literally a grayscale version of the image.

![enter image description here](https://i.imgur.com/70bTrp7.jpg)

U is the blue-difference chroma component.

![enter image description here](https://i.imgur.com/XoadxOo.jpg)

V is the red-difference chroma component.

![](https://i.imgur.com/GBa1AJb.jpg)
The reason why people do this is because of compression. Since our brightness/luminance/black-and-white perception is better than our color perception, smart people figured out that we can store the luminance data at a higher quality and resolution than the color parts. 

One of those methods is called "[chroma subsampling](https://en.wikipedia.org/wiki/Chroma_subsampling)". This is literally storing the color information at a lower resolution than the brightness resolution. If you have a camera, chances are, it does exactly this, and you've never noticed this unless you really went deep and zoomed in as hard as possible on the images/video it produces.

As for what 4:4:4 means, this means "store the color information at the same resolution as the brightness information". If the brightness information is 1920x1920, then the color information is also 1920x1920.

***

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
