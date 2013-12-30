encode-cowon-d2
===============

Similar to *encode-divxht*, this script transcodes any video file that MPlayer
can read into a format that the Cowon D2/D2+ portable digital audio player can
play back.

The Cowon D2 normally comes with a MS-Windows program called JetAudio that can
do this task, but this replacement script enables you to do the same under Linux
or any other UNIX-like OS via MPlayer.

Run this script with the `-h' option to see a list of command-line options. In
many ways it works similar to the `encode-divxht` script. The simplest
invocation is:

    $ encode-cowon-d2 file.mp4

It will produce a similar-named filed with an `.d2.avi' extension, that you can
copy to the internal memory or SD card to load into the Cowon D2 to play it
back.

I find this script useful for watching digitally recorded TV shows
on-the-go (or sometimes in bed if I can't sleep!).

On a 2.0GHz AMD Athlon XP system (running Linux and MPlayer 1.1) transcoding
standard-definition PAL video takes around 20mins for one hour's worth of
footage (if using two-pass encode), and half that if using single-pass.

I've tested this script on vast ranges of input files, including HD content,
high-framerate content (50 or 60fps), regular Youtube videos and DVB-T/S/C
broadcast streams. It handles all these types almost (see below for a few notes)
transparently for you.

For this script you will need [MPlayer](http://www.mplayerhq.hu/) installed on
your system, and compiled with at least DivX 5 support through its libavcodec
(lavc) library and MPEG-1 layer III audio encoding support via the LAME library.
Some modes also require [FFmpeg](http://www.ffmpeg.org/).

This script also assume the POSIX command-line program
[bc](http://en.wikipedia.org/wiki/Bc_programming_language) is available for
real-number arithmetic, but this should be installed by default on every Linux
distribution.


Explanation of script command line options
------------------------------------------

    Usage: encode-cowon-d2 [OPTION...] VIDEO-FILES...
       or: encode-cowon-d2 [OPTION...] -o OUTPUT-AVI-FILE INPUT-VIDEO-FILE
    Converts video files to Cowon D2-playable DivX AVI using MPlayer.
    
    Input/output file options
    -------------------------
      -D, --delete-input-file
            Delete input file after successful encode.
      -d, --output-dir=dir
            Directory where output files are written (default: same as source)
      -o, --output-file=name
            Output file name; can only be used if one input file is given.
            
    Timecode selection options
    --------------------------
      -s, --start=ts
            Start encode at given timestamp in source material. Given as either
            `mm:ss', `hh:mm:ss' or an absolute number of seconds from start.
      -e, --duration=dur
            Stop encoding <dur> seconds after start of encoding. May be given as
            `mm:ss', `hh:mm:ss' or an absolute number of seconds.
            
    Input video file decoding options
    ---------------------------------
      -a, --aspect=x
            Override source material aspect ratio (if incorrect or missing). <x>
            must be in form `w:h', `w/h' or a positive real number.
      -F, --ffmpeg-decode
            Use FFmpeg to decode input files rather than MPlayer. Provides a
            workaround for input files where MPlayer may decode incorrectly
            or crashes.
      -f, --fps=n
            Assume this frame rate (fps) for output if source frame rate is
            unavailable or if the input file is variable frame rate. Default
            assumed frame rate is 25fps. Must not exceed 30fps.
            
            As a special case, if a frame rate of 50fps, 59.94fps or 60fps (or an
            input file with this frame rate) is given, then a filter that halves
            the frame rate will be used when producing the output file.
            
      -Y, --noskip
            Use basic MEncoder A/V sync (eqv. "-noskip -mc 0") to suppress
            excessive "Skipping frame" messages.  Should only be used on
            well-formed input files. Should not be used with variable frame
            rate files.
            
    Picture cropping/scaling/adjust options
    ---------------------------------------
      -C, --auto-crop
            Attempt to auto-detect size of black borders around edge of picture
            and crop them. Will take several minutes for large files.
      -c, --crop=w:h:x:y
            Manually specify black border crop region. <w> is width, <h> is height
            and <x>, <y> is the top-left corner (all in pixels) of the region to
            be preserved. Corresponds to MPlayer "-vf crop".
      -G, --gamma=n
            Adjust gamma levels by factor of n (default=1.0, meaning leave as-is).
      -p, --panscan
            Crop sides of 16:9 source material for 4:3 centre extract. This option
            implicitly assumes source material aspect ratio is 16:9.
      -r, --rotate90
            Rotate picture 90 degrees clockwise. The rotation is done after the
            cropping stage, so any cropping coordinates must be given pre-rotation.
      -Z, --zoom-to-aspect=x
            Zooms in about the centre of the picture and crops it so the remaining
            active region of the picture is of aspect ratio <x>. <x> must be in
            form `w:h', `w/h' or a positive real number, and should be closer
            to 1.333 (4:3) than the footage's original aspect. No action taken
            if <x> is further away from 1.333 than the film's original aspect
            (never zooms out).
            
            For example, specifying `-Z14:9' for a 16:9-aspect input file would
            zoom-in the picture by a factor of 1.14x (8/7). The visible area of
            the film will be shown letterboxed 14:9 aspect, with 1/16th-width
            columns on the left and right edges of the original picture cropped
            off. The resulting picture would then be 206px tall, rather than
            180px if no zooming/cropping was performed at all and the full 16:9
            image made visible.
            
            As the Cowon D2's native screen is 320x240px (4:3 aspect), this option
            can provide a useful compromise for watching widescreen content on
            the D2.
            
    Video bitrate management options
    --------------------------------
      -S, --size=n
            Attempt to limit output file to <n> megabytes or less, automatically
            selecting a video bitrate to meet this budget. Maximum size is 2GiB
            (2147.483647MB).
      -b, --vbitrate=n
            Output video bitrate in kbps. Permitted maximum is 768kbps.
            Default for 1-pass encode is 384kbps, 2-pass encode is 256kbps.
            
    Video encoding options
    ----------------------
      -1, --single-pass
            Single-pass encode only (two-pass encoding is default)
      -g, --greyscale
            Greyscale mode; use if source footage is B&W to reduce encoding time.
      -I, --interlaced
            Assume source footage is interlaced. The odd/even fields of the
            footage will be combined through a de-interlace filter to create
            progressive-scan footage that can be displayed on the Cowon D2.
            
            NOTE: Progressive-scan footage is assumed by default. MEncoder
            is unable to detect the presence of interlaced footage; you need
            to examine the source material yourself by pausing at scenes
            containing fast motion. If you see horizontal combing effects on
            fast-moving portions of the still image, your footage is likely
            interlaced. Typically SD-DVB footage and DVDs of older TV programmes
            (pre-2000s) are interlaced. Some film DVDs may be hard-telecined.
            
      -k, --keyint=n
            Generate keyframes every <n> frames. Maximum permitted is 250.
            Shorter keyframe intervals make the video more bit-error resilient
            at the expense of lesser compression ratio. Default is 250.
      -t, --turbo
            Turbo mode; trades off some output quality for faster encode.
      -X, --xvid
            Use xvid library to encode video, instead of MPlayer's
            libavcodec. Requires MPlayer to be compiled with xvid support.
            
    Audio encoding options
    ----------------------
      -A, --acodec-mp3=abr
            Re-encode non-Cowon-D2-compatible audio streams to MPEG-1 layer
            III VBR, using a LAME preset to specify quality goal. <abr>
            is one of `medium' (150-180kbps), `standard' (170-210kbps),
            `extreme' (200-240kbps), `insane' (320kbps) or an integer between
            8-320 for custom ABR goal given in kbps. The default action for
            non-Cowon-D2-compatible audio streams is to re-encode using the
            `medium' profile.
      -x, --reencode-audio
            Force re-encoding of the source audio stream, even if it is
            Cowon-D2-compatible, instead of muxing it verbatim into the output. Use
            this option if the source audio stream is triggering decoding bugs
            on the Cowon D2 (e.g. playback stuttering, playback of some files
            ending prematurely).
            
            
    MPlayer/MEncoder process control options
    ----------------------------------------
      -M, --mencoder-cmd=cmd
            Name of MEncoder instance used for encoding (default="mencoder")
            If <cmd> is surrounded by double-quotes, can also include extra
            command-line options to supply to MEncoder.
      -N, --renice=pri
            UNIX renice priority when running MEncoder (0-20, default=20).
      -n, --dry-run
            Dry-run mode: don't encode; only print commands that would be run.
      -T, --threads=n
            Number of concurrent threads to use for encoding (1-8, default
            1). Additional threads speed up encoding on multi-core systems but
            may produce slightly lesser quality than single-threaded mode.
            
    All output files will take on the extension `.d2.avi'.
    
    Note that output file A-V sync will be offset by -400ms; this is to
    compensate for a bug in Cowon D2 video player firmware where A-V sync
    irreparably drifts +400ms after the first seek operation is performed.
    
    To play a file encoded by this script properly on the Cowon D2, start the
    video playing, then immediately perform at least one seek operation that
    lands anywhere but the start of the file (timecode 0:00:00). After this
    point, correct A-V sync is maintained despite further seeks and resumes.


Note that the long arguments will not be available if you don't have the GNU
version of the getopt(1) program on your system; this is because the standard
POSIX version has a design flaw where it cannot reliably process arguments with
spaces in them. You'll have to use the short 1-letter options in this case.
Nearly all Linux distributions in the last 10 years will come with the GNU
version of the getopt(1) program.


Example uses
------------

Encodes a batch of videos in the one directory:

    $ encode-cowon-d2 programme_episode*.mp4

Each output file will be named `program_episode<x>.d2.avi`. The output
files will be encoded in 2-pass mode and will use a bitrate of 256kbps.

Encodes a typical MPEG-2 video stream captured from a standard-definition DVR,
deinterlacing it so it can be displayed properly on the Cowon D2's internal LCD
display.

    $ encode-cowon-d2 -I recorded_dvr_programme.mpg


Cowon D2 video support
----------------------

The Cowon D2 can play back DivX 5 encoded video (from the "Video"
application in the main menu), providing it's encoded to the following
A/V profile specs:

  * Container format must be AVI.

  * Horizontal resolution must be a multiple of 16 pixels, and must not exceed
    320 pixels.

  * Vertical resolution must not exceed 240 pixels.

  * Frame rate must not exceed 30fps.

  * A square pixel aspect ratio must be used, so the video may need to
    be spatially resampled. The Cowon D2 screen is 4:3 aspect, however I
    generally find most 16:9 widescreen content can be comfortably
    watched at 4:3 centre-extracted (using the `-p` option).
    
  * Bitrate must not exceed 768kbps.

  * Audio must be in MPEG-1 layer III stereo. If playback is stuttering
    on the D2, then try re-encoding the audio using the `-x` option.

  * Key frame interval must not exceed 250 frames.

I was able to determine the constraints for the A/V profile that the D2
will support from reading various digital audio enthusiast forums
(thanks to those that published their own findings), and some
experimentation on my own.

I personally find a video bitrate of 256kbps is acceptable quality for two-pass
encode (only minor noise and blocking artifacts), however single-pass needs
roughly 384kbps to produce comparable results. Despite the 320x240px resolution,
the D2's LCD is crisp enough that on-screen text is still generally readable.
Higher bitrates generally tend to run the battery down a little quicker for
marginal increase in quality.

The Cowon D2 apparently also supports Microsoft's WMV format, but MPlayer's
support for this format is currently not as mature as that for DivX, so I'll
ignore that capability for now. 

Caveats
-------

The **big** caveat here is that the factory Cowon D2/D2+ firmware has a
known bug where during video playback, if you perform a seek operation
at any time, the A-V sync will then drift by -400ms for the rest of
playback. I've upgraded my D2+ to the latest firmware (2.13) and this
glitch unfortunately still persists, much to my chagrin.

This script works around this issue by permanently shifting the A-V sync
by +400ms in the other direction. When you start a video playing on the
D2, you will need to let it play for a few seconds, then immediately
jump to any location in the stream that is not the absolute beginning
(0:00:00). This will correct the lip-sync so you can watch the rest of
the programme without having to worry about this issue.

The trade-off here is that you now will not be able to play the output
files on other video player software without having to manually
re-correct the playback A-V sync.

If Cowon ever fix this problem (unlikely, as they've moved onto the D3
and other successors as of 2011), then edit the ADELAY variable at the
start of this script to remove the intentional A-V shift.

Reporting bugs
--------------

Since the `encode-cowon-d2` script is basically a front-end for MEncoder, it's
at the mercy of MEncoder to produce the correct output streams for the options
given.

The script will print the `mencoder` commands as it runs them. If the output
files you're getting are not what you expected, or there's problems with them
(e.g. A/V desync), then first confer with the MPlayer documentation if the
`mencoder` commands that this script chose to ran are appropriate and relevant
for what you were trying to achieve.

If the commands executed are wrong, then feel free to e-mail me a bug report at
[rodgersb at it dot net dot au] and mention the following:

    * Operating system/distribution version and system hardware specs
    * MPlayer and FFmpeg versions installed
    * What you were trying to do
    * What you were expecting to happen

If the commands executed are correct according to the MPlayer docs, then it's
likely that there's a bug in MPlayer/MEncoder that is fouling up the encode, and
you should report the bug directly to the MPlayer maintainers at
<http://mplayerhq.hu/>.

As of MPlayer 1.1, there are still various quibbles with transcoding from
Matroska (MKV) containers (mainly problems with A/V sync in the output)
especially when variable framerates are involved.

If you're cut for time and want your files encoded sooner
---------------------------------------------------------

By default 2-pass encoding is enabled. This will give much better quality than
single-pass encoding, but obviously takes twice as long.

If you're cut for time and willing to trade off quality, or more disk space to
maintain a similar level of quality, then consider looking at the following
options:

    -1, --single-pass
        Single-pass encode only (two-pass encoding is default). Halves
        encoding time but won't allocate more bits to scenes w/ complex motion,
        resulting in more significantly degraded picture quality unless very
        high bitrates are used.

    -t, --turbo
        Turbo mode; trades off some output quality for faster encode.

    -T, --threads=n
        Number of concurrent threads to use for encoding (1-8, default=1).
        Additional threads speed up encoding on multi-core systems but may
        produce slightly lesser quality than single-threaded mode.

    -g, --greyscale
        Greyscale mode; use if source footage is B&W to reduce encoding time.
