MEncoder notes for converting videos into DivX 5 Home Theatre profile
=====================================================================

These are my notes produced from extensive investigation and testing playback of
DivX 5 video files produced by MPlayer on hardware DivX 5 Home Theatre players.

Devices used for testing in particular:

 * Panasonic NV-VP60 DVD/VHS combo player (rel. circa-2006).
 * Toshiba SD2010KY DVD player (mfd. Nov. 2012).

Both these devices have the "DivX" logo on their front panel.

Brief technical details on constrains for the DivX Home Theatre profile:
<http://www.divx.com/files/DivX-Profiles_Tech-Specs.pdf>

Note that the NV-VP60 manual states the player also supports MPEG-4
Simple Profile video files in an `.asf` container, however this is
constrained to 12.5fps/15fps frame-rate maximum and 352x288/352x240
pixel resolution maximum. Not really desirable for most content viewing
- rather Panasonic likely intended this feature for viewing content
produced by low-grade video cameras or mobile phones.

File naming requirements
------------------------

Use a standard AVI container format, with `.avi` extension.

The file name, including the `.avi` extension should not exceed 64 characters.
This is a limitation of the Joliet extension of the ISO 9660 file system; any
longer file names will be truncated and the file will not appear in the
playlist.

The NV-VP60 does not use the Rock-Ridge (UNIX-style) extensions which allow
longer names - please be careful not to overlook this issue when burning files
to DVD-R on a UNIX-like platform.

The SD2010KY file browser truncates the file names to 15 characters, which is a
poor design choice on Toshiba's part, but it doesn't cause problems if more than
one file has the same leading 15 characters in their name - the menu selections
appear identical but it can still access each file if you can remember their
alphabetical sort order.

I can't find out much about the technicals of the `.divx` container format.
Apparently it's a subset of the AVI format, and possibly might be used for the
DivX VOD service? The NV-VP60 manual mentions that it supports this file format
too, but MPlayer doesn't seem able to generate this container, so I'll just
disregard it for now.

Container format requirements
-----------------------------

The resulting AVI file size ideally should not exceed 2GiB; this is a limitation
of the standard Microsoft AVI indexing structures. The NV-VP60 manual explicitly
states this restriction.

The ISO 9660 file system also has an inherent file size limit of 4GiB, however
it's recommended to keep files below 2GiB if possible because some OSes or
embedded devices may mistakenly treat the file size field as a signed 32-bit
integer.

If the resulting AVI file exceeds 1GiB, then OpenDML indexes must be disabled
(MEncoder option `-noodml`). Otherwise undesired behaviour may occur on NV-VP60
once playback crosses the 1GiB mark; audio may desync, unable to jump, playback
may prematurely end. I suspect the NV-VP60 probably doesn't understand AVI files
that have more than one RIFF chunk.

FFmpeg must be patched to prevent OpenDML headers on sub-2GiB files, since it
lacks a command-line argument to disable them, and generates them by default.

For FFmpeg 0.5 (possibly later versions as well), change the preprocessor define
`AVI_MAX_RIFF_SIZE` in libavformat/avi.h to `0x7FFFFFFFLL` before compiling to
disable OpenDML headers on sub-2GiB AVI files.

### AVI container FourCC code requirements

The AVI file must have a FourCC code of either `DX50`, `DIVX` (for DivX version
5) or `DIV3` (for older DivX version 3 streams). The NV-VP60 is also known to
accept `XVID` as well.

Be aware that MEncoder by default will use `FMP4` as the FourCC code;
this FourCC code will likely *not* work.

If you forget to specify this on the command line, you can fix it after the fact
by hex-editing the resulting AVI file; the FourCC code is stored in two places
at offsets `0x00000070` and `0x000000bc` in nearly all cases.

Video frame rate
----------------

Frame rate must not exceed 30fps in any case.

If the frame rate is 25fps or less, then the video will be played back
in 576i 50Hz PAL mode (25fps).

If the frame rate exceeds 25fps but not 30fps, then the video will be
played back in either 480i 59.94Hz NTSC mode (29.97fps) or 480i 60Hz PAL
mode (30fps), depending on player configuration and TV support.

The NV-VP60 uses nearest neighbour algorithm for temporal upscaling when
playing back footage with oddball frame rates; e.g. for 24fps material,
it will duplicate one frame each second to meet PAL's 25fps requirement.

Interlaced scan modes are supported. Use the `ildct:ilme` options in `-lavcopts`
for MEncoder libavcodec when encoding interlaced footage.

Note that MPlayer currently has no way of auto-detecting if the source
footage is interlaced via the `-identify` option; if interlaced source
material is mistakenly encoded as progressive, the NV-VP60 will show
shimmering artifacts on fast movement scenes.

Image resolution and aspect ratio constraints
---------------------------------------------

Horizontal resolution must not exceed 720 pixels in any case.

Vertical resolution must not exceed 576 lines if frame rate is 25Hz or less.

Vertical resolution must not exceed 480 lines if frame rate is greater than
25fps but less than or equal to 30fps.

#Aspect terminology
 * SAR: Storage Aspect Ratio (pixel array width:pixel array height)
 * PAR: Pixel Aspect Ratio (pixel width:pixel height)
 * DAR: Display Aspect Ratio (projected picture width:projected picture height)

Formula:
 * DAR = SAR * PAR
 * PAR = DAR / SAR
 * SAR = DAR / PAR

#Aspect ratio constraints

According to the DivX tech specs document, the PAR must conform to one of the
predefined standard MPEG-4 PARs:

 * 1:1 (typically used on computer monitors)
 * 10:11 (4:3 NTSC)
 * 40:33 (16:9 NTSC)
 * 12:11 (4:3 PAL)
 * 16:11 (16:9 PAL)

The SD2010KY will usually display any video file that doesn't use one of the
following PARs in the wrong aspect. In the case of 720x576 16:9 DAR, it will simply
strech to fill the whole screen, which will work okay for a 16:9 TV set, but not
on a 4:3 one.

The NV-VP60 however appears to support custom PARs fine, and will scale them
on-the-fly correctly. This can be very useful for downscaling HD content to SD,
as it enables the full 720x576 space to be used and preserve as much detail as
possible.

Note that the above PAR constraint means it's not possible to represent the
common DVD-Video or DVB profiles of 720x576 or 720x480 at either 4:3 or 16:9
aspect. One of the following approaches must be taken to use an above PAR:

 * Crop off the leftmost/rightmost 8px columns to give 704px width, but leave
   the aspect unchanged.

 * Multiply the aspect by 45/44 (1.33:1 becomes 1.36:1, 1.78:1 becomes 1.82:1),
   but leave the resolution unchanged.

 * Rescale each row of the picture from 720px to 704px, but keep the same aspect.

#Brief background on 704px/720px discrepancy

The reason for the 704px vs. 720px discrepancy between DVD-Video/DVB MPEG-2 and
MPEG-4 is to do with ITU-R BT.601 standards for digitising analogue video
signals. The width of the active picture is technically 702px, but 704px is
chosen to be a multiple of 16, which is feasible for MPEG-2 motion compensation.

The reason for adding 8px on either side of each scanline to make 720px wide
rows is to allow for timing instability tolerances when digitising analogue
video signals from electromechanical mediums such as tape, laserdisc etc. If the
active line start and end timings are off slightly by a few microseconds, then
720px allows the video editor on a digital workstation to correct the picture by
shifting it left or right slightly, without having to do a re-capture.

Typically the left/rightmost 8px columns in a 720x576 or 720x480 DVD/DVB picture
should not be visible on a conventional CRT TV set. Some TV or DVD producers put
content there anyway, while others will leave these sections black (or fill them
with a solid colour).

#MPlayer crop/aspect issues

When transcoding content from DVD-Video, it's usually a
good idea to crop the black letterbox borders if present (see MPlayer
`-vf cropdetect` and MEncoder `-vf crop`) so the codec doesn't need to
waste bits trying to encode the sharp transitions from the edge of the
active picture to the black regions.

The aspect ratio must be specified in the MPEG-4 stream itself (MEncoder option
`-lavcopts aspect=w/h`). Using MEncoder's `-force-avi-aspect` option won't work,
as the NV-VP60 does not honour the AVI `vprp` header.

The MP4Box utility can be used to correct the aspect ratio on existing
encoded `.avi` files after the fact, however currently it only supports
DivX version 5.

The NV-VP60 uses some form of linear interpolation when rescaling the
video spatially.

Audio codec requirements
------------------------

Audio should either be MPEG-1 Layer III or Dolby AC-3 (with either mono,
stereo or up to 5.1 channels). MPEG-4 AAC support is optional according
to the DivX Home Theatre technical specs document.

The NV-VP60 and SD2010KY don't support AAC, however they also support
MPEG-1 Layer II audio as well, which is the audio format for standard
definition DVB - useful as it avoids a re-encode when archiving DVB
content. It's not clear if the DivX HT spec permits MPEG-1 Layer II, but
every DVD player that plays DivX ought to support it, as MPEG-1 Layer II
is a required part of the DVD-Video specification.

All bitrates for AAC and MPEG-1 Layer II/III seem to work fine, as well as
variable bitrate for MPEG-1 Layer III. For most DVD-Video streams, you
can just copy the audio as-is (MEncoder `-oac copy`).

Up to 8 audio tracks may be used.

Currently FFmpeg must be used to encode multi-audio AVI files since
MEncoder does not support writing them.

FFmpeg command line to merge an additional audio track (in this case
`commentary.aac`) into an existing AVI file (`infile.avi`) that already
has one video and one audio stream:

    $ ffmpeg -i infile.avi -i commentary.aac \
        -vcodec copy -acodec copy -acodec copy outputfile.avi -newaudio

(Note the order and repetition of the arguments is significant.)

Video codec requirements
------------------------

The video codec that should be used is "MPEG-4 Part 2 (Advanced Simple
Profile) level 5"; to select this in MEncoder use the option `-lavcopts
vcodec=mpeg4`.

The DivX Home Theatre spec states B-type frames (bidirectional predictive) may
only occur between other P (forward predictive) or I (intra) frames. I've also
read online forum posts stating that B-frames should not be used in interlaced
content.

The NV-VP60 will simply refuse to display multiple consecutive B-type frames,
causing jerky motion. MEncoder's libavcodec will not include them by default.
MPlayer will print a warning message if it detects the presence of them in an
AVI file, as apparently they are non-standard.

The SD2010KY usually displays significant tearing or block-glitching artifacts
if B-frames are used in interlaced footage. B-frames in progressive scan content
are fine though, and usually result in a slightly smaller file size.

I've also noted that MPlayer 1.1 libavcodec MPEG-4 encoding also causes minor
macroblock glitching on fast-moving interlaced scenes (e.g. time-lapse footage)
- this was observed independently on computer in both MPlayer and FFmpeg. May be
a MEncoder bug?

Average video bitrate must not exceed 4Mbps. The peak video bitrate must not
exceed 4.854Mbps for longer than one second.

For most cases, two-pass 1.5Mbps will provide near broadcast-SD quality; roughly
6 hours of footage on a single-layer DVD-R.

The maximum keyframe interval is 250 frames; MEncoder selects this by
default.

There's a certain bitrate (or macroblock-per-second) threshold where
files below this rate will play back smoothly like a VCR when being
fast-forwarded at double-speed on the NV-VP60, instead of
frame-dropping. I'm not sure what this threshold is.

Subtitle requirements
---------------------

According to the DivX Home Theatre specs document, the DivX HT profile includes
support for XSUB format subtitles. It's not clear how to instruct MPlayer to
include subtitles in this exact format - will need to research.

Surprisingly the SD2010KY supports SRT format subtitles as well - it will pop up
a menu as you start playing allowing you to select a subtitle file.
