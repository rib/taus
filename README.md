# TAUS, et al

[Source Repository](https://github.com/ejona86/taus)

The main project is the Actually Useful Statistics Tetris mod. However, the
repository also contains disassembly knowledge for tetris, a structure for
building NES ips/nes files, and a LUA-based unit/integration test helpers.

## How to run

Pre-built IPS mod files are available in the
[releases section of GitHub](https://github.com/ejona86/taus/releases). They
can be applied to the USA Tetris ROM, commonly known as "Tetris (U) [!].nes".
The ROM has CRC32 `6d72c53a`, MD5 `ec58574d96bee8c8927884ae6e7a2508`, and
SHA1 `77747840541bfc62a28a5957692a98c550bd6b2b`. Ignoring the 16 byte iNES
header (`tail -c +17 tetris.nes > no-header.nes`), it has CRC32 `1394f57e`,
MD5 `5b0e571558c8c796937b96af469561c6`, and
SHA1 `fd9079cb5e8479eb06d93c2ae5175bfce871746a`. It is generally okay if the
iNES header is different.

## TAUS

[![In-game stats](media/stats-ingame.thumb.png)](media/stats-ingame.aspect.png)
[![Post-game stats](media/stats-postgame.thumb.png)](media/stats-postgame.aspect.png)

TAUS provides the in-game statistics:
 * DHT: The drought. The number of pieces since the last line
 * BRN: The burn. The number of lines cleared since the last tetris
 * EFF: The efficiency. The score per lines ignoring the level multiplier.
   Individual clears have score per lines of: 40 for a single, 50 for a double,
   100 for a triple, 300 for a tetris
 * TRT: The tetris rate. The percentage of lines cleared that were tetrises
 * TRNS: The transition score. The score achieved when leaving the starting
   level

TAUS provides the post-game statistics:
 * EFF LOG: A chart for the EFF progression through the game with each bar
   cooresponding to the EFF of 10 lines cleared. The EFF presented is not the
   cumulative EFF to that point, but the EFF of the 10 lines in isolation.
   It tracks each line clear separately, so if you have cleared 8 lines and
   then clear a tetris that is considered two lines followed by two more lines

The mod also allows skipping the legal screen.

EFF is similar in purpose to the conventional TRT. Both assume you die because
you reach too high of a level (e.g., level 29) and both let you know for "how
well are you doing" in the middle of the game. But EFF is a strong predictor
of your final score. For example, let's say you start on level 9 and
die after 100 lines. If you alternate between tetrises and sets of three
singles, your final score will be ~188k. But if you alternate between tetrises
and triples, your final score will be ~214k. TRT for each will be 57%, but your
EFF was 188 and 214 agreeing with the 14% better score on the second game. If
your EFF increases by 10%, then your score increases ~10% for the same number
of lines.

## Two Player

[![Next on side](media/twoplayer-side.thumb.png)](media/twoplayer-side.aspect.png)
[![Next on top](media/twoplayer-top.thumb.png)](media/twoplayer-top.aspect.png)

The twoplayer mod allows two simultaneous players to play while receiving the
same pieces. Garbage is not sent between players.

There are two variations of the mod. One shows the next piece on the side,
which makes it easier to see with peripheral vision. The other shows it on top, which makes it clearer at-a-glance which next piece belongs to which player.

## How to build

You only need to build if you are making changes or want to try out changes
that have not yet been included in a release.

Dependencies (should be in PATH):
1. [Flips](https://github.com/Alcaro/Flips). [Flips
   1.31](https://www.smwcentral.net/?p=section&a=details&id=11474) is fine
2. [cc65](https://cc65.github.io/getting-started.html)
3. GNU Make. Windows users can use `make.exe` (just the one file) from the
   `bin/` folder of `make-*-without-guile-w32-bin.zip` available at
   [ezwinports](https://sourceforge.net/projects/ezwinports/files/)
4. GNU Coreutils and Sed. These are standard on Linux. On Windows they are
   provided by [Git for Windows](https://git-scm.com/download/win) when using
   the "Git Bash" command line. Note that it uses a Unix directory structure;
   the Windows directory structure is within the `/c/` directory

On Windows, to modify your PATH, run `SystemPropertiesAdvanced.exe`. On the
"Advanced" tab click "Environment Variables" and then change `Path` in your
"User variables" and hit Okay. You will need to restart any terminals for the
changes to take effect.

Manual prep:
1. Copy tetris ROM to `tetris.nes` in the `taus` folder. If the iNES header is
   different than mentioned above you can still use the ROM, but you need to
   adjust the header in `tetris.s` to match your rom to make `$ make test`
   happy

Use `$ make` to build artifacts into `build/`, which includes disassembling
into `build/tetris-PRG.s`. `$ make test` verifies the reassembled version
matches the original. The mod will be generated at `build/taus.ips` and will
have been applied to `build/taus.nes`.

## Structure

tetris-PRG.info is the location for all tetris ROM knowledge. It is used to
disassemble tetris into build/tetris-PRG.s. tetris.s and tetris.nes.info
contain the pieces to reassemble tetris into a iNES file. Reassembly is able to
output debug information.

The main debug output is the .lbl file. It is basic and just contains the
labels with their addresses, so doesn't have any more information than
tetris-PRG.info. However, it is easy to parse so the file format is used for
several other tasks; it is transformed into build/tetris.inc using sed and can
be read directly by the LUA testing tools.

NES and IPS files are output directly by the linker, because our .s files
define the headers for the formats and the .cfg files specify the ordering of
the headers/chunks. The linker is fairly well suited to the job and provides
the ability to mix-and-match source files when generating an IPS file, only
needing to manually sort the hunks. It is useful to have understanding of the
IPS format and how it works. It is basically the simplest possible patch
format, only supporting 1:1 replacing, so should be easy to learn.

The [Nesdev Wiki](https://wiki.nesdev.com/w/index.php/NES_reference_guide) has
good resources for the various file formats. The .info file format is described
in the [da65 (disassembler)
documentation](https://www.cc65.org/doc/da65-4.html). The .cfg file format is
described in the [ld65 (linker)
documentation](https://www.cc65.org/doc/ld65-5.html).

## Creating new NES/IPS files

To create a new NES file output (IPS/NES file), simply create a new .cfg file,
mirroring one of the existing ones (depending on what you are making), and any
.s files for the source. Then modify the Makefile to list the object files you
want to include within your IPS/NES file. These will be in the form
build/ORIGNAME.o corresponding to each ORIGINAL.s file. If your .cfg file was
named `myfile.ext.cfg`, then run `$ make build/myfile.ext` to build the output.
The extension used does not matter to the Makefile nor linker.

For any modifications to the original ROM, creating the IPS hunks directly
works well for 1) mods with few changes and 2) injecting a jsr to a custom
function. If you need to make changes that don't reference any new labels, then
you can manage a .diff file. The .diff is intended to be checked in and is
applied to build/diffhead-YOURFILENAME. You can change the diffhead and `make`
will re-generate the .diff. Similarly, changing the .diff will re-generate the
diffhead.
