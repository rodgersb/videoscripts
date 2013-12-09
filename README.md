MPlayer video encoding scripts
==============================

This directory contains a collection of shell scripts that perform useful
video encoding tasks. I'm releasing the scripts under the GPL in case
anyone else finds them useful.

Please note that these scripts, although they have endured hundreds of
hours of use by myself and should be reasonably well tested, they come
with **absolutely no warranty**, **merchantability** or **fitness for a
particular purpose**. I will not be held responsible for any loss or
damage that these scripts may cause. Please use them at your own risk.

For each of these scripts, you will need
[MPlayer](http://www.mplayerhq.hu/) installed on your system, and
compiled with at least DivX 5 support through its libavcodec (lavc)
library. Some modes also require [FFmpeg](http://www.ffmpeg.org/).

These scripts are written for GNU Bash version 2 or later. They also
assume the POSIX command-line program
[bc](http://en.wikipedia.org/wiki/Bc_programming_language) is available
for real-number arithmetic, but this should be installed by default on
every Linux distribution.

These scripts were developed and tested under Linux, but should work
without problems in other UNIX-like operating systems, and should
probably work under [Cygwin](http://cygwin.com/) in Microsoft Windows,
albeit as I do not use Windows I cannot and have not tested for this.

encode-divxht
-------------

Transcodes a video file from any format that MPlayer/FFmpeg can read
into a standard-definition DivX 5 AVI file that can be played back on a
standalone DVD player that has native DivX 5 Home Theatre profile
support.

Most DVD players that support DivX Home Theatre will have a "DivX"  logo
somewhere on the front panel. There will usually also be a "DivX VOD
registration" option somewhere in the setup menu (for the ill-fated DivX
Video on Demand service).

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

### Command-line options

To get a list of options, run the script with the `-h' option:

    Usage: encode-divxht [OPTION...] VIDEOFILES...
    Converts video files to DivX Home Theatre profile AVI.

      -1      Single-pass encode only (two-pass encoding is default)
      -A      Force re-encoding of audio stream, even if it could be re-used as-is.
      -a x    Override source material aspect ratio.
              <x> must be in form `w:h', `w/h' or a decimal number.
      -B n    Attempt to limit output file to <n> megabytes or less,
              automatically selecting a video bitrate to meet this budget.
              Maximum size is 2GB.
      -b n    Output video bitrate in kbps, as a positive integer. Permitted
              maximum is 4000. By default, the source video bitrate is used.
      -C      Attempt to auto-detect black border crop region, and crop it.
              Will take several minutes for large files.
      -c w:h:x:y
              Manually specify black border crop region. <w> is width, <h>
              is height and <x>, <y> is the top-left corner (all in pixels)
              of the region to be preserved. Corresponds to MPlayer "-vf crop".
      -D      Delete input file after successful encode.
      -d dir  Directory where output files are written (default: same as source)
      -e dur  Stop encoding <dur> seconds after start of encoding. May be given as
              `mm:ss', `hh:mm:ss' or a whole number of seconds.
      -F      Use FFmpeg to decode input files rather than MPlayer. Provides
              a workaround for source files where MPlayer may decode incorrectly.
      -f n    Assume this frame rate (Hz) if source frame rate is unavailable.
              Default assumed frame rate is 25Hz. Must not exceed 30Hz.
      -I      Assume source footage is interlaced. Cannot be auto-detected.
              Progressive scan footage is assumed by default.
      -M cmd  Name of MEncoder instance used for encoding (default=mencoder)
      -o name Output file name; should only be used if one input file is given.
      -p      Crop borders off 16:9 source material for 4:3 centre extract.
              Implicitly assumes source material aspect ratio is 16:9.
      -s ts   Start encode at given timestamp in source material. Given as either
              `mm:ss', `hh:mm:ss' or a whole number of seconds from the start.
      -t      Use faster options for encode (trades off quality for more speed)

    Without `-o', output files will have the same name as the input files,
    with the extension `.avi'. Should any source file already have extension
    `.avi', then the alternate extension `.divxht.avi' will be used.

### What video bitrates to use

These observations are from my own personal experience. Your mileage may
vary. These bitrates always assume the slower default two-pass mode is
used, which gives better quality-to-bitrate than the quicker 1-pass
(`-1' option) mode.

    * -b750: Gives VHS-like quality, with a fair bit of high-frequency
      noise, minor quilting and blocking artifacts. Will fit around
      12hrs footage on a single-layer DVD-R. Max file duration is 5h30m.

    * -b1500: Very close to SD broadcast quality. Since keyframes are
      only generated every 10 seconds, sometimes there may be small
      barely-noticeable segments of the picture that periodically
      discolour slightly over a few seconds, then "snap back" to their
      proper colour. One example is human faces at moderate distance, if
      you look around the hairline fringe. Will fit around 6hrs footage
      on a single-layer DVD-R. Max file duration is ~2h45mins.

    * -b2000 or higher: Broadcast SD quality. Most of the noise you'll
      now be seeing will probably be as a result of the TV broadcaster
      throttling down their bitrate, if any. 2000kbps video will fit
      around 4.5hrs footage on a single layer DVD-R. Max file duration
      is ~2hrs.

### Caveats

I've discovered the NV-VP60 will not accept DivX AVI files that are
larger than 2GB in size. Therefore for long programs you may need to
slice up your source footage to accommodate this restriction.
Incidentally the ISO 9660 filesystem format also has a 2GB maximum file
length restriction.

Currently there is no way to detect if the source footage is interlaced
or progressive scan, because MPlayer's `-identify` option doesn't expose
that detail.

You will have to play back the video file on the computer first and
watch closely parts of the footage that have rapid motion - if you see a
"grating" or "combing" effect, then your footage is interlaced. I find
most broadcast SD DVB-T streams are interlaced.

In that case you will need to use the `-I` option, otherwise the DVD
player may exhibit picture shimmering/blurring artifacts. Interlaced
footage that is encoded correctly will appear silky smooth on a CRT TV.

Another known bug is that the bitrate budget calculations currently
don't work properly if you use the `-s` or `-e` options. I rarely use
these options (except for testing) so I haven't given the time to fix
them yet, sorry.


encode-cowon-d2
---------------

Similar to *encode-divxht*, this script transcodes any video file that
MPlayer can read into a format that the Cowon D2/D2+ portable digital
audio player can play back.

Run this script with the `-h' option to see a list of command-line
options.

I find this script useful for watching digitally recorded TV shows
on-the-go (or sometimes in bed if I can't sleep!). 

On a 2.0GHz AMD Athlon XP system (running Linux and MPlayer 1.1)
transcoding standard-definition PAL video takes around 20mins for one
hour's worth of footage (if using two-pass encode), and half that if
using single-pass.

### Cowon D2 video support

The Cowon D2 can play back DivX 5 encoded video (from the "Video"
application in the main menu), providing it's encoded to the following
A/V profile specs:

  * Horizontal resolution must be 320 pixels exactly.

  * Vertical resolution must not exceed 240 pixels.

  * Frame rate must not exceed 30Hz.

  * A square pixel aspect ratio must be used, so the video may need to
    be spatially resampled. The Cowon D2 screen is 4:3 aspect, however I
    generally find most 16:9 widescreen content can be comfortably
    watched at 4:3 centre-extracted (using the `-p` option).

  * Bitrate must not exceed 768kbps.

    Audio must be in MPEG-1 layer III stereo. If playback is stuttering
    on the D2, then try re-encoding the audio using the `-A` option.

  * Key frame interval must not exceed 250 frames.

I was able to determine the constraints for the A/V profile that the D2
will support from reading various digital audio enthusiast forums
(thanks to those that published their own findings), and some
experimentation on my own.

I personally find a video bitrate of 256kbps is acceptable quality for
two-pass encode (only minor noise and blocking artifacts), however
single-pass needs roughly 384kbps to produce comparable results. Despite
the 320x240 resolution, the D2's LCD is crisp enough that on-screen text
is still generally readable. Higher bitrates generally tend to run the
battery down a little quicker for marginal increase in quality.

### Caveats

The **big** caveat here is that the factory Cowon D2/D2+ firmware has a
known bug where if you perform a seek operation any time, the A-V sync
will then drift by 400ms for the rest of playback. I've upgraded my D2+
to the latest firmware (2.13) and this glitch unfortunately still
persists, much to my chagrin.

This script works around this issue by permanently shifting the A-V sync
by 400ms in the other direction. When you start a video playing on the
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


divxhtcheck
-----------

This is a simple script that can be used in conjunction with
*encode-divxht* to check if an AVI video file conforms to the DivX 5 Home
Theatre profile, to help ensure that files you burn to a DVD-R will be
playable on your DivX5-HT equipped standalone DVD player.

As with above, I've used the Panasonic NV-VP60 DVD/VHS combo player as a
reference model for testing DivX5-HT compatibility. Other DVD players
may be more strict or more lenient in their support, so your mileage may
vary. Feedback reports on other players are welcome, so I can improve
this script.

Usage is simply as follows:

    $ divxhtcheck file.avi ...

If all files conform to the DivX5-HT profile, then the script will print
no output (UNIX philosophy "no news is good news") and return exit code
zero.

If one or more files do not conform to the DivX5-HT profile in some way,
then error messages will be printed for each file describing what is
wrong, and a non-zero exit code will be returned (even if other files
are compliant).

Some problems can be corrected very quickly (like changing the FourCC
code); refer to the MPlayer man page to learn how to do this.

Other issues, such as the presence of MPEG-4 B-frames (causes jerky
playback on the NV-VP60 and is non-standard according to the MPlayer
team), or if the resolution is too big, can only be fixed with a full
transcode, which will be time-consuming. You can use the *encode-divxht*
script to fix these problems.

Note that I've also included a check to ensure the file name does not
exceed 64 characters. This is because I discovered the NV-VP60 only uses
the Joliet file names in ISO 9660 if present (it ignores the Rock Ridge
names), and the ISO 9660 Joliet extensions impose a 64-character limit
on the filename.

Programs like mkisofs will normally truncate file names that exceed this
limit in the Joliet namespace (printing a warning message that can
sometimes be easily overlooked), hence the file will no longer have an
.avi extension and the NV-VP60's playlist menu will fail to recognise it
as a playable file. As Rock Ridge (normally used on UNIX-like systems)
does not have such a limit, it can be easy to overlook this issue when
testing discs on a UNIX-like system.
