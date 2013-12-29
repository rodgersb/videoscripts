MEncoder notes for converting videos into DivX 5 for Panasonic NV-VP60
======================================================================

These are my notes produced from extensive investigation and testing
playback of DivX 5 video files produced by MPlayer on the standalone
Panasonic NV-VP60 DVD/VHS combo player (rel. circa-2006).

I suspect the NV-VP60 likely conforms to the DivX 5 Home Theatre
profile. 
<http://www.divx.com/files/DivX-Profiles_Tech-Specs.pdf>

Note that the NV-VP60 manual states the player also supports MPEG-4
Advanced Simple Profile video files (in an `.asf` container), however
this is constrained to 12.5Hz/15Hz frame-rate maximum and
352x288/352x240 pixel resolution maximum. Not really desirable for most
content viewing - rather Panasonic likely intended this feature for
viewing content produced by low-grade video cameras or mobile phones
(perhaps they didn't want to license the full MPEG-4 codec for cost
reasons?).

File naming requirements
------------------------

Must use AVI container format, with `.avi` extension.

The file name, including the `.avi` extension should not exceed 64
characters. This is a limitation of the Joliet extension of the ISO 9660
file system; any longer file names will be truncated and the file will
not appear in the playlist.

The NV-VP60 does not use the Rock-Ridge (UNIX-style) extensions which
allow longer names - please be careful not to overlook this issue when
burning files to DVD-R on a UNIX-like platform.

Container format requirements
-----------------------------

The resulting AVI file size ideally should not exceed 2GB; this is a
limitation of both the standard Microsoft AVI indexing structures and
the ISO 9660 file system.

If the resulting AVI file exceeds 1GB, then OpenDML indexes must be
disabled (MEncoder option `-noodml`). Otherwise undesired behaviour may
occur on NV-VP60 once playback crosses the 1GB mark; audio may desync,
unable to jump, playback may prematurely end.

FFmpeg must be patched to prevent OpenDML headers on sub-2GB files,
since it lacks a command-line argument to disable them, and generates
them by default.

For FFmpeg 0.5 (possibly later versions as well), change the
preprocessor define `AVI_MAX_RIFF_SIZE` in libavformat/avi.h to
`0x7FFFFFFFLL` before compiling to disable OpenDML headers on sub-2GB
AVI files.

### AVI container FourCC code requirements

The AVI file must have a FourCC code of either `DX50`, `DIVX`, `XVID`
(for DivX version 5) or `DIV3` (for older DivX version 3 streams).

Be aware that MEncoder by default will use `FMP4` as the FourCC code;
this FourCC code will *not* work on the NV-VP60.

If you forget to specify this on the command line, you can fix it after
the fact by hex-editing the resulting AVI file; the FourCC code is
stored in two places at offsets `0x00000070` and `0x000000bc` in nearly
all cases.

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

Interlaced scan modes are supported. Use the `ildct:ilme` options in
`-lavcopts` for MEncoder when encoding interlaced footage.

Note that MPlayer currently has no way of auto-detecting if the source
footage is interlaced via the `-identify` option; if interlaced source
material is mistakenly encoded as progressive, the NV-VP60 will show
shimmering artifacts on fast movement scenes.

Image resolution and aspect ratio constraints
---------------------------------------------

Horizontal resolution must not exceed 720 pixels in any case.

Vertical resolution must not exceed 576 lines if frame rate is 25Hz or
less.

Vertical resolution must not exceed 480 lines if frame rate is greater
than 25Hz but less than or equal to 30Hz.

The aspect ratio can be anything, unlike DVD-Video's fixation on only
16:9 or 4:3. When transcoding content from DVD-Video, it's usually a
good idea to crop the black letterbox borders if present (see MPlayer
`-vf cropdetect` and MEncoder `-vf crop`) so the codec doesn't need to
waste bits trying to encode the sharp transitions from the edge of the
active picture to the black regions.

The aspect ratio must be specified in the MPEG-4 stream itself (MEncoder
option `-lavcopts aspect=w/h`).

Using MEncoder's `-force-avi-aspect` option won't work, as the NV-VP60
does not honour the AVI `vprp` header.

If no aspect ratio is given, then square pixels are assumed, which
will typically result in slight pillarboxing on most 4:3 footage.

The MP4Box utility can be used to correct the aspect ratio on existing
encoded `.avi` files after the fact, however currently it only supports
DivX version 5.

The NV-VP60 uses some form of linear interpolation when rescaling the
video spatially.

Audio requirements
------------------

Audio may either be MPEG-1 Layer II/III or MPEG-2 AAC (with either mono,
stereo or 5.1 channels). MPEG-4 AAC is not supported and must be
transcoded.

If using MPEG-1 Layer II for audio, the bitrate must not exceed 256kbps
(?). I found the NV-VP60 sometimes wouldn't play back Layer II audio at
some higher threshold bitrate, although this needs to be investigated
further.

All bitrates for AAC and MPEG-1 Layer III seem to work fine, as well as
variable bitrate for MPEG-1 Layer III. For most DVD-Video streams, you
can just copy the audio as-is (`-oac copy`).

Multiple audio tracks may be used; use the "Audio" button on the NV-VP60
remote to switch between them.

Currently FFmpeg must be used to encode multi-audio AVI files since
MEncoder does not support writing them.

FFmpeg command line to merge an additional audio track (in this case
`commentary.aac`) into an existing AVI file (`infile.avi`) that already
has one video and one audio stream:

    $ ffmpeg -i infile.avi -i commentary.aac \
        -vcodec copy -acodec copy -acodec copy outputfile.avi -newaudio

(Note the order and repetition of the arguments is significant.)

Codec requirements
------------------

B-type frames (bidirectional predictive) should not be used. The NV-VP60
will simply refuse to display B-type frames, causing jerky motion.
MEncoder will not include them by default. MPlayer will print a warning
message if it detects the presence of them in an AVI file, as apparently
they are non-standard.

Average video bitrate must not exceed 4Mbps. The peak video bitrate must
not exceed 8Mbps for longer than one second. For most cases, two-pass
1.5Mbps will provide near broadcast-SD quality; roughly 6 hours of
footage on a single-layer DVD-R.

The maximum keyframe interval is 250 frames; MEncoder selects this by
default.

There's a certain bitrate (or macroblock-per-second) threshold where
files below this rate will play back smoothly like a VCR when being
fast-forwarded at double-speed on the NV-VP60, instead of
frame-dropping. I'm not sure what this threshold is.

Subtitle requirements
---------------------

I'm yet to find out if the DivX 5 Home Theatre profile allows for
soft-subtitles. I know there are ways to store soft-subtitles in the AVI
container, as I've seen MPlayer play back such streams. 

Will need to investigate - as a workaround, hard-subs can be used if
acceptable.
 