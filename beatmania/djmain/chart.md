# Beatmania 5-key Chart Format (DJMAIN)

This document serves to explain the chart format used by Konami's Beatmania
series of arcade games.

## About this format

This format is used in all of the 5-key Beatmania games that run on the DJMAIN
hardware. It started with 1st mix, and goes all the way to The Final.

The hard drives in this game allocate 16 megabytes of space for each song
entry on the disk. Sometimes songs are duplicated, and sometimes the offsets of
the chart and audio data are different between mixes.

This document will not go into detail about the offsets of charts. It will
document the format itself.

## In a nutshell

Charts are meant to be read as a stream of data. You don't really know how long
the data is, but you will know when you've reached the end of it. There are two
sections to a chart. The first section is the note count. The second section is
the event data.

## Event types

Before we go too much further, you'll need to know about the different kinds
of events you can expect to encounter:

- *Note Event*: These are the notes you see in columns during gameplay.
- *Sound Event*: These change what sound a key plays.
- *BPM Event*: These set the current BPM.
- *End of Song Event*: When this is encountered, gameplay ends.
- *BGM Event*: This is a sound effect that is auto-played in the background.
- *Judgment Event*: This determines the judgment timing window.

##

If you're using raw MAME images, chances are you'll need to perform a 16-bit
byteswap on the data as you read it.
