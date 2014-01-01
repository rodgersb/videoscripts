Video transcoding scripts
=========================

This directory contains a collection of shell scripts that perform useful
video encoding tasks. I'm releasing the scripts under the GPL in case
anyone else finds them useful.

Please note that these scripts, although they have endured hundreds of hours of
use by myself and should be reasonably well tested and robust, they come with
**absolutely no warranty**, **merchantability** or **fitness for a particular
purpose**. I will not be held responsible for any loss or damage that these
scripts may cause. Please use them at your own risk.

For each of these scripts, you will need
[MPlayer](http://www.mplayerhq.hu/) installed on your system, and
compiled with at least MPEG-4 Part 2 (Advanced Simple Profile) support
through its libavcodec (lavc) library and MPEG-1 layer III audio
encoding support via the LAME library. Some modes also require
[FFmpeg](http://www.ffmpeg.org/).

These scripts are written for the standard POSIX shell `/bin/sh`. I've tested
them with GNU Bash 2.x and also Almquist shell.

They also assume the POSIX command-line program
[bc](http://en.wikipedia.org/wiki/Bc_programming_language) is available for
real-number arithmetic, but this should be installed by default on every Linux
distribution.

These scripts were developed and tested under Linux, but should work
without problems in other UNIX-like operating systems, and should
probably work under [Cygwin](http://cygwin.com/) in Microsoft Windows,
albeit as I do not use Windows I cannot and have not tested for this.


encode-divxht
-------------

Transcodes a video file from any format that MPlayer/FFmpeg can read into a
standard-definition DivX 5 AVI file that can be played back on a standalone DVD
player that has native DivX 5 Home Theatre profile support.

My primary motivation for writing this script was to allow me to archive
recorded content off a DVR onto DVD-R and play it back later on my DVD
player (which supports DivX Home Theatre profile) as a replacement for
my old VHS VCR, but it can also be used if you want to watch, say,
purchased or downloaded digital videos on your TV set.

While you could type out the MEncoder commands manually each time to do
the transcoding, it's quite an error-prone process if you do it
frequently, and you also have to run MEncoder twice when doing two-pass
encoding. The risk of selecting wrong arguments in MEncoder can result
in wasting hours of CPU time producing a file that isn't playable on a
DivX-equipped DVD player.

This script takes all the trouble out of the transcoding task - it
ensures the output file produced is playable on all hardware DivX
players, and also does 2-pass encoding by default. Invocation is as
simple as:

    $ encode-divxht file.mp4 ...

It will produce a similar named file with an `.avi' extension as the output.

Most DVD players that support DivX Home Theatre will have a "DivX"  logo
somewhere on the front panel. There will usually also be a "DivX VOD
registration" option somewhere in the setup menu (for the seldom-used
DivX Video on Demand service).

See the file README-encode-divxht.md for more information on how to use it. You
can also run it with the `-h` option to see a brief summary of each option.


encode-cowon-d2
---------------

Similar to *encode-divxht*, this script transcodes any video file that
MPlayer can read into a format that the Cowon D2/D2+ portable digital
audio player can play back.

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

See the file README-encode-cowon-d2.md for more details.


divxhtcheck
-----------

This is a quality-assurance check script that can be used in conjunction with
*encode-divxht* to verify if an AVI video file conforms to the DivX 5 Home
Theatre profile, to help ensure that files you burn to a DVD-R will be playable
on your DivX5-HT equipped standalone DVD player.

As with above, I've used the Panasonic NV-VP60 and Toshiba SD2010KY DVD players
as reference models for testing DivX5-HT compatibility. Other DVD players may be
more strict or more lenient in their support, so your mileage may vary. Feedback
reports on other players are welcome, so I can improve this script.

Run the script with the `-h` option to see a list of command-line options:

    $ divxhtcheck -h

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

Other issues, such as the presence of multiple consective MPEG-4 B-frames, or if
the resolution is too big, can only be fixed with a full transcode, which will
be time-consuming. You can use the `encode-divxht` script to fix these problems.


divxhtbitrate
-------------

This script is a video bitrate calculator for transcoding video files to DivX
Home Theatre profile and storing them on optical media for archival.

My motivation for writing this script was to make it much easier to calculate
the optimal bitrate I could transcode a set of programmes recorded on my DVR

Run the script with the `-h' option to see a list of options.

The script also allows you to reserve a fixed or per-second amount of overhead,
in case you are later muxing in additional audio tracks or subtitle data.

You will need to either have the source videos on disk ready for encoding (but
you can also type in the durations on the command line as an alternative).

The simplest invocation is:

    $ divxhtbitrate <storage-budget> videofiles...

<storage-budget> is given either in megabytes or can also be a pre-set keyword
that specifies a common optical media type:

    bluray      A single-layer Blu-ray disc (25GB)
    bluraydl    A dual-layer Blu-ray disc (50GB)
    dvd         A single-layer DVD-R/+R disc (4.7GB)
    dvddl       A dual-layer DVD-R/+R disc (8.5GB)
    cd          An 80min CD-R disc (700MB)

The program output will be an integer on a single line; this is the proposed
video bitrate (in kbps) that you should encode the videos at to make them fill
up the storage budget. Zero exit status will be returned in this case.

If the videos are too long for the storage budget (or if there's too much
overhead), then an error message will be printed and non-zero exit status will
be returned.

I've tested this with MPlayer 1.1 and I'm quite confident on the accuracy of the
estimation. After doing 2-pass encoding of several hours of video on a DVD-R DL
disc with MEncoder at the bitrate this script suggested, it was able to fill up
the whole disc with around 2MB to spare.

This script is not necessarily tied to DivX Home Theatre format - it can easily
be adapted for other video formats.
