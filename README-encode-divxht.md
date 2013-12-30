encode-divxht script
====================

Transcodes a video file from any format that MPlayer/FFmpeg can read into a
standard-definition DivX 5 AVI file that can be played back on a standalone DVD
player that has native DivX 5 Home Theatre profile support.

My primary motivation for writing this script was to allow me to archive
recorded content off a DVR onto DVD-R and play it back later on a DVD player (w/
DivX support) as a replacement for my old VHS VCR, but it can also be used if
you want to watch, say, purchased/downloaded digital video content on your TV
set.

While you could type out the MEncoder commands manually, it's quite an
error-prone process if you do it frequently, and you also have to run MEncoder
twice when doing two-pass encoding. The risk of selecting wrong arguments in
MEncoder can result in wasting hours producing a file that isn't playable on a
DivX-equipped DVD player.

This script takes all the trouble out of the encoding task - it ensures the
output file produced is playable on all hardware DivX players, and also
facilitates 2-pass encoding by default. It also renices the MEncoder encoding
process (which will be CPU-bound) so it doesn't detract from the response time
of interactive desktop applications on your system.

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

DivX Home Theatre specs and testing reference implementations
-------------------------------------------------------------

I've used a Panasonic NV-VP60 DVD/VHS combo player (mfd. 2006) as the
reference model for testing. It's taken me many hours of experimentation
and many spent DVD-R coasters, but I've determined through
trial-and-error what constraints it imposes on the files it will play
back correctly.

I've uploaded the text file `divx5ht-notes.md' as a supplementary on my
observations with what A/V format profiles the NV-VP60 will reliably
play back. Other DivX-HT DVD players may be more strict or more lenient
on these constraints. Your mileage may vary, so I advise that you please
test the output of this script on your own player using some short input
files on rewritable DVD media to avoid wasting DVD-Rs burning unplayable
files.

Please note that I'm based in Australia, which uses PAL and DVB-T as its TV and
broadcasting standards. These use a frame rate of 25fps. I haven't had the
opportunity to test this script with ATSC streams from North American DVRs, etc.
Feedback is welcome though.

If you frequently deal with content that is NTSC-based (frame rate 29.97fps or
30fps), then I recommend editing the line containing `DFL_FPS=25` in the script
to make the default more convenient for you.

Command-line options
--------------------

To get a list of options, run the script with the `-h' option:

    $ encode-divxht -h

    Usage: encode-divxht [OPTION...] VIDEO-FILES...
       or: encode-divxht [OPTION...] -o OUTPUT-AVI-FILE INPUT-VIDEO-FILE
    Converts video files to DivX Home Theatre profile AVI.

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
    ----------------------------------
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

            As a special case, if a frame rate of 50fps, 59.94fps or 60fps (or a
            progressive-scan input file with this frame rate) is given, then each
            consecutive pair of frames will be converted into odd/even fields of
            one interlaced frame, and the effective framerate halved. This allows
            such content to be played on a conventional PAL/NTSC TV set.

      -Y, --noskip
            Use basic MEncoder A/V sync (eqv. "-noskip -mc 0") to suppress
            excessive "Skipping frame" messages.  Should only be used on
            well-formed input streams. Should not be used with variable frame
            rate files.

    Picture cropping/scaling options
    --------------------------------
      -7, --crop-720-704
            Crops left/rightmost 8-pixel columns off 720px-wide PAL/NTSC
            DVD-Video or SD-DVB footage to create a 704px-wide picture as per
            ITU-R BT.601. This hack eliminates a costly resample operation
            and allows a standard MPEG-4 Pixel Aspect Ratio (PAR) to be used,
            ensuring better DivX H/W player compatibility. No action taken if
            picture is not 720px wide.
      -C, --auto-crop
            Attempt to auto-detect size of black borders around edge of picture
            and crop them. Will take several minutes for large files.
      -c, --crop=w:h:x:y
            Manually specify black border crop region. <w> is width, <h> is height
            and <x>, <y> is the top-left corner (all in pixels) of the region to
            be preserved. Corresponds to MPlayer "-vf crop".
      -E, --expand-aspect-720
            Widens aspect ratio by 45/44 if picture is 720px wide (most DVD-Video
            or SD-DVB footage) so a standard MPEG-4 Pixel Aspect Ratio can be used,
            ensuring better DivX H/W player compatibility. 4:3 (1.33:1) aspect
            becomes 1.36:1, 16:9 (1.78:1) becomes 1.82:1. This is an alternative
            hack to the `-7' option, also avoids a costly resample operation but
            preserves the left/rightmost 8px columns. No action taken if picture
            is not 720px wide.
      -H, --height=n
            Rescale picture height to no larger than <n> pixels. Maximum <n>
            is 576 for PAL (fps <= 25), 480 for NTSC (25 < fps <= 30).
      -p, --panscan
            Crop sides off 16:9 source material for 4:3 centre extract. This
            option implicitly assumes source material aspect ratio is 16:9.
      -Q, --square-pixels
            Rescale picture as needed so that square pixels are used. Needed for
            buggy H/W DivX players that ignore the MPEG-4 aspect ratio field.
      -W, --width=n
            Rescale picture width to no larger than <n> pixels (max: 720)

    Video bitrate management options
    --------------------------------
      -S, --size=n
            Attempt to limit output file to <n> megabytes or less, automatically
            selecting a video bitrate to meet this budget. Maximum size is 2GiB
            (2147.483647MB) unless OpenDML is enabled (`-O').
      -b, --vbitrate=n
            Output video bitrate in kbps. Permitted maximum is 4000kbps.
            By default, the source video bitrate is used.

    Video encoding options
    ----------------------
      -1, --single-pass
            Single-pass encode only (two-pass encoding is default). Halves
            encoding time but won't allocate more bits to scenes w/ complex motion,
            resulting in more significantly degraded picture quality unless very
            high bitrates are used.
      -g, --greyscale
            Greyscale mode; use if source footage is B&W to reduce encoding time.
      -I, --interlaced
            Assume source footage is interlaced. Progressive-scan footage is
            assumed by default.

            WARNING: MEncoder cannot auto-detect interlaced footage; you need
            to examine the source material yourself by pausing at scenes
            containing fast motion. If you see horizontal combing effects
            on fast-moving portions of the still image, your footage is
            likely interlaced. Typically SD-DVB footage and DVDs of older
            TV programmes (pre-2000s) are interlaced. Some film DVDs may be
            hard-telecined. Failure to use `-I' option on interlaced footage may
            result in shimmering/tearing artifacts on H/W DivX players.

      -k, --keyint=n
            Generate keyframes every <n> frames. Maximum permitted is 250.
            Shorter keyframe intervals make the output file more bit-error
            resilient at the expense of lesser compression ratio. Default is 250.
      -t, --turbo
            Turbo mode; trades off some output quality for faster encode.
      -X, --xvid
            Use xvid library to encode video, instead of MPlayer's
            libavcodec. Requires MPlayer to be compiled with xvid support.

    Audio encoding options
    ----------------------
      -A, --acodec-mp3=abr
            Re-encode non-DivXHT-compliant audio streams to MPEG-1 layer III VBR,
            using a LAME preset to specify quality goal. <abr> is one of `medium'
            (150-180kbps), `standard' (170-210kbps), `extreme' (200-240kbps),
            `insane' (320kbps) or an integer between 8-320 for custom ABR goal
            given in kbps. This is the default action for non-DivXHT-compliant
            audio streams.
      -L, --acodec-ac3=cbr
            Re-encode non-DivXHT-compliant audio streams to Dolby AC-3, using
            MPlayer's libavcodec library. <cbr> specifies the constant bitrate
            used. Maximum permitted bitrate is 448kbps.
      -x, --reencode-audio
            Force re-encoding of the source audio stream, even if it is
            DivXHT-compliant, instead of muxing it verbatim into the output. Use
            this option if the source audio stream is triggering decoding bugs
            on the target player hardware.

    Non-DivXHT-compliant options (*might* work on some H/W DivX players, but YMMV!)
    -------------------------------------------------------------------------------
      -B, --bframes-interlaced
            Allow encoding B-frames in interlaced (`-I') mode. May improve
            compression slightly. Not officially supported in the DivX HT
            spec. Some H/W players will exhibit block-glitching, tearing or jerky
            motion artifacts if B-frames are used in interlaced mode.
      -G, --gmc
            Use Global Motion Compensation. May improve picture quality in slow
            panning scenes. Many H/W players do *not* implement this feature.
      -J, --acodec-aac=abr
            Re-encode non-DivXHT-compliant audio streams to MPEG-4 Advanced Audio
            Coding (AAC) format using the FAAC library. Note that AAC support is
            *optional* in the DivX HT spec; many H/W players lack AAC support.
            <abr> is average bitrate used, in kbps. Requires MPlayer be built w/
            libfaac support. Maximum permitted bitrate is 1152kbps.
      -j, --copy-aac
            Do not re-encode source audio stream if it is MPEG-4 AAC; mux it into
            the output file verbatim. See warning for `-J' option above.
      -P, --custom-par
            Allow custom MPEG-4 Pixel Aspect Ratios (PAR); can preserve more
            detail for wider-aspect (e.g. 2.35+:1) HD content. By default,
            picture is rescaled if needed to conform to a standard MPEG-4 PAR
            (1:1, 10:11, 40:33, 12:11 or 16:11). DivX HT spec doesn't require
            custom PARs, but some H/W players support them anyway. H/W players
            w/o custom-PAR support will show custom-PAR video at wrong aspect
            (stretched/squished).
      -q, --qpel
            Enable quarter-pixel motion compensation (qpel). Sometimes this
            might improve picture quality slightly, but generally only at higher
            bitrates. Not available on interlaced video modes.
      -O, --opendml
            Allow OpenDML headers to be used in the AVI container, enabling the
            output file to exceed 2GiB. Some H/W DivX players (e.g. Panasonic
            NV-VP60) cannot reliably play back files greater than 2GiB.

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
            Number of concurrent threads to use for encoding (1-8, default=1).
            Additional threads speed up encoding on multi-core systems but may
            produce slightly lesser quality than single-threaded mode.

    Without `-o', output files will have the same name as the input files,
    with the extension `.avi'. Should any source file already have extension
    `.avi', then the alternate extension `.divxht.avi' will be used.


Note that the long arguments will not be available if you don't have the GNU
version of the getopt(1) program on your system; this is because the standard
POSIX version has a design flaw where it cannot reliably process arguments with
spaces in them. You'll have to use the short 1-letter options in this case.
Nearly all Linux distributions in the last 10 years will come with the GNU
version of the getopt(1) program.

Example uses
------------

Encodes a batch of videos in the one directory:

    $ encode-divxht programme_episode*.mp4

Each DivX HT output file will be named `program_episode<x>.avi`. The output
files will have the same video bitrate as the input files.

Encodes a typical MPEG-2 video stream captured from a standard-definition DVR
keeping it at broadcast quality:

    $ encode-divxht -I -b2500 recorded_dvr_programme.mpg

What video bitrates to use
--------------------------

These observations are from my own personal experience. Your mileage may
vary. These bitrates always assume the slower default two-pass mode is
used, which gives better quality-to-bitrate than the quicker one-pass
(`-1' option) mode.

    * -b750 to -b1000: Gives VHS-like quality, with a fair bit of high-frequency
      noise, minor quilting and blocking artifacts. Will fit around 10-12hrs
      footage on a single-layer DVD-R. Max file duration is ~5hrs.

    * -b1500 to -b2000: Very close to SD broadcast quality. Since keyframes are
      only generated every 10 seconds, sometimes there may be small
      barely-noticeable segments of the picture that periodically discolour
      slightly over a few seconds, then "snap back" to their proper colour. One
      example is human faces at moderate distance, if you look around the
      hairline fringe. Will fit around 5-6hrs footage on a single-layer DVD-R. Max
      file duration is ~2.5hrs.

    * -b2500 or higher: Broadcast SD quality. Most of the noise you'll now be
      seeing will probably be as a result of the TV broadcaster throttling down
      their bitrate, if any. 2500kbps video will fit around 4hrs footage on a
      single layer DVD-R. Max file duration is ~2hrs.


The options offered aren't flexible enough
------------------------------------------

Look at the `-n` option. This is a "dry-run" mode where it simply prints out the
commands that the script would run to standard output, instead of running them.

    $ encode-divxht -n -b2500 -I rage-20131228-satnight-pt1.mpg
    mencoder -ffourcc DX50 -noodml \
        -ofps 25.000   -vf-add scale=704:576:1 \
        -ovc lavc -lavcopts vcodec=mpeg4:v4mv:vrc_maxrate=4854:vrc_buf_size=3145:keyint=250:vhq:trell:aspect=1.7778:ildct:ilme:vmax_b_frames=0:vbitrate=2500:vpass=1:turbo -passlogfile rage-20131228-satnight-pt1.passlog \
        -oac copy \
        -o /dev/null \
         rage-20131228-satnight-pt1.mpg && \
    mencoder -ffourcc DX50 -noodml \
        -ofps 25.000   -vf-add scale=704:576:1 \
        -ovc lavc -lavcopts vcodec=mpeg4:v4mv:vrc_maxrate=4854:vrc_buf_size=3145:keyint=250:vhq:trell:aspect=1.7778:ildct:ilme:vmax_b_frames=0:vbitrate=2500:vpass=2 -passlogfile rage-20131228-satnight-pt1.passlog \
        -oac copy \
        -o rage-20131228-satnight-pt1.avi \
         rage-20131228-satnight-pt1.mpg && \
    rm -f  rage-20131228-satnight-pt1.passlog

What you can do is redirect this output to a file, make it into a shell script,
edit it as you please and then run the script directly.

    $ encode-divxht -n -b2500 -I rage-20131228-satnight-pt1.mpg >encode-rage.sh

    $ vim encode-rage.sh        # make your edits here

    $ sh -x encode-rage.sh


Caveats
-------

The Panasonic NV-VP60 and many other hardware players will not accept DivX AVI
files that are larger than 2GiB in size. Therefore for long programs you may
need to slice up your source footage to accommodate this restriction.

Currently there is no way to detect if the source footage is interlaced
or progressive scan, because MPlayer's `-identify` option doesn't expose
that detail.

You will have to play back the video file on the computer first and watch
closely parts of the footage that have rapid motion - if you see a "grating" or
"combing" effect, then your footage is interlaced. Typically most broadcast
standard definition DVB-T/S/C streams are interlaced, as well as DVDs of older
(pre-2000) television progams. Some films may also be hard-telecined.

In that case you will need to use the `-I` option, otherwise the DVD
player may exhibit picture shimmering/blurring artifacts. Interlaced
footage that is encoded correctly will appear silky smooth on a CRT TV.

I've decided performing inverse-telecining is outside the scope of this script,
since the objective is to enable a video file to be played on a conventional
standard-definition PAL/NTSC TV set (which is interlaced by nature). This script
will leave the interlacing intact when the `-I` option is present.


Reporting bugs
--------------

Since the `encode-divxht` script is basically a front-end for MEncoder, it's at
the mercy of MEncoder to produce the correct output streams for the options
given.

The script will print the `mencoder` commands as it runs them. If the output
files you're getting are not what you expected, or there's problems with them
(e.g. A/V desync), then first confer with the MPlayer documentation if the
`mencoder` commands that this script chose to ran are appropriate and relevant
for what you were trying to achieve.

If the commands executed are wrong, then feel free to e-mail me a bug report at
<rodgersb at it dot net dot au> and mention the following:

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


If you want to try and save on storage
--------------------------------------

If you have a scenario where you're severely constrained for storage and
want to lower the bitrate significantly (e.g. by 50%), usually
downsampling the resolution of the picture instead of just dropping the
bitrate yields better results; the picture will only look a bit softer,
as opposed to more high-frequency noise and quilting/blocking artifacts.

Explanation: DivX 5 has an upper limit on MPEG macroblock size (32x32
pixels). Spatial downsampling enables the motion-compensation
macroblocks to cover larger portions of the image, so the codec doesn't
need to use as many macroblocks to describe inter-frame changes.

Use the `-H` and `-W` options to reduce the size of the picture. For example:

    $ encode-divxht -I -b1000 -W352 -H288 recorded_dvr_programme.mpg

will encode the programme at quarter-PAL resolution (picture will look softer,
like VHS or VCD) but there'll be much less high-frequency noise in the picture,
and it'll look quite reasonable at 1000kbps bitrate IMO.


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

If you're technically inclined, you can also read the next section if you're
re-encoding DVD-Video or DVB-T/S/C streams.


What's the deal with the 720px and 704px horizontal resolutions?
----------------------------------------------------------------

The DVD-Video and standard-definition DVB-T/S/C specs allow the video stream to
be transmitted as MPEG-2 at resolutions up to 720x576 (PAL) or 720x480 (NTSC),
and at display aspect ratios of 4:3 or 16:9.

The DivX file format is based on the MPEG-4 standard (there are a few minor
differences in DivX from other MPEG-4 based video formats like H.264, such as
limited or no support for Global Motion Compensation, etc).

When describing the aspect ratio of a video, there are three important ratios:

 * SAR: Storage Aspect Ratio (pixel array width:pixel array height)
 * PAR: Pixel Aspect Ratio (pixel width:pixel height)
 * DAR: Display Aspect Ratio (projected picture width:projected picture height)

There's a relationship between these ratios. If you know two of them, you can
solve for the third one:

 * DAR = SAR * PAR
 * PAR = DAR / SAR
 * SAR = DAR / PAR

Unlike still images in computer graphics, where pixels are normally considered
square, in digital video terminology, pixels are usually *not* square. The
reason for this is to do with sampling an analogue signal at an optimal sampling
rate, and integrating digital video equipment (editing workstations, storage
systems) with existing older analogue equipment.

ITU-R BT.601 (a television engineering standard) defined an industry standard
sampling rate of 13.5MHz for digital TV equipment that dealt with standard
definition TV content. They defined the active region of a digitised PAL/NTSC TV
signal to be 704 pixels wide, but allowed for an 8px margin on either side to
accommodate for timing inaccuracies in analogue video equipment, hence 720 pixels
per line, with the leftmost/rightmost 8px not intended to be visible (falls in
the overscan region of an analogue TV signal, and most CRT TV sets won't show
that area).

The MPEG-4 file format normally encodes the aspect ratio of the picture in a
video stream by giving the PAR instead of the DAR, whereas MPEG-2 instead has a
few predefined DARs (1:1, 4:3, 16:9, 2.21:1).

MPEG-4 offers 5 pre-defined PARs:

 * 1:1 (typically used on computer monitors)
 * 10:11 (4:3 NTSC)
 * 40:33 (16:9 NTSC)
 * 12:11 (4:3 PAL)
 * 16:11 (16:9 PAL)

MPEG-4 streams can also have custom PARs, but the DivX Home Theatre spec states
that only the above PARs should be used. Some DivX-capable DVD players
unofficially  support custom PARs properly, while others don't (they'll display
the picture at the wrong aspect - stretched or squished).

The reason for this constraint is that the MPEG-4 standard (like its
predecessors MPEG-1/2) are designed to be implemented in hardware as well as
software, and need to be reasonably simple in design.

In a hardware DVD player, the video stream's PAR is directly linked to the clock
rate of the output video DAC used to generate the TV signal. TV encoder chipsets
used in hardware DVD players vary wildy in design, however quite often for cost
reasons low-end chipsets typically only support limited clock rates (allowing
for simpler circuitry).

A DVD-Video player video chipset DAC technically only needs to support a small
handful of clock rates, as the resolutions and DARs of DVD-Video content are
tightly constrained.

The problem with adapting DVB-T/S/C or DVD-Video content to DivX HT arises in
that you cannot describe a 720x576 or 720x480 video stream as having a DAR of
exactly 4:3 or 16:9 while using one of the above pre-defined MPEG-4 PARs. You
would have to use a custom MPEG-4 ratio to achieve this, i.e:

  * 720x576 16:9 DAR widescreen PAL would require a PAR of 64:45
  * 720x576 4:3 DAR standard PAL would require a PAR of 16:15
  * 720x480 16:9 DAR widescreen NTSC would require a PAR of 32:27
  * 720x480 4:3 DAR standard NTSC would require a PAR of 8:9

How can we deal with this problem?

If you have a DVB-T/S/C recording or DVD-Video content that is 720px wide and
either 16:9 or 4:3 DAR, then to make it fit a standard MPEG-4 PAR you can do one
of three things:

1. Rescale the image horizontally from 720px to 704px, keeping the same aspect
ratio. This is the default course of action in the `encode-divxht` script. The
caveat with this is that it slows down the encode slightly, since there's an
extra step of having to resample the image before compressing it.

2. Crop the leftmost/rightmost 8px columns off the side of the image to make it
704px wide, while keeping the same aspect. The ITU-R BT.601 standard recommends
that these regions of the screen shouldn't be used, and most TV content will
leave them blank, however some programmes may fill out the full 720px horizontal
width. This is the `-7` option in `encode-divxht`.

3. Alternatively you can slightly expand the DAR of the video by multiplying it
by 45/44 (~1.02) to then satisfy the DAR=SAR*PAR equation above. This keeps the
full 720px width of the image intact. This is the `-E' option in
`encode-divxht`.

Solutions #2 and #3 above will avoid the resampling operation, so they will
allow for a slightly faster encode in MEncoder. They have the caveat of slightly
distorting the picture DAR by 2%, however this should not be noticeable to most
viewers.
