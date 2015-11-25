# Dance Dance Revolution SSQ Format

This document serves to explain the SSQ format used by Konami's Dance Dance
Revolution series of video games.

## About this format

To date, SSQ is the most recent format used by Konami's Dance Dance Revolution
series. Its earliest occurrence is in unreferenced files in DDR 3rd Mix Plus.
They are first officially used in DDR 4th Mix.

SSQ appears to have been originally developed without freeze notes in mind.
The reasoning is how early the format has been found and the hacky way it
was implemented. Shock arrows are also implemented in a hacky way, although
in a less intrusive manner (see below.)

It supports BPM changes and multiple variable sized chunks, much like a WAV
file.

## In a nutshell

SSQ is a series of "chunks". Each chunk has a header describing what it is
along with some chunk-specific metadata.

Often, you'll find at least three chunks. They are the tempo data, some
unknown yet consistent data, and the note data. One chunk exists per chart
difficulty. Additional data is likely video scripting data; see below for
details.

## Chunk format

There is no indication how long an SSQ file actually is. The typical method is
to keep reading until there's no more bytes available to read, or until the
chunk length doesn't make sense (it's less than a value of 0x0C for example.)

```
Offset(h) Type      Length    Descrption
+00       int32     4         length of the data in bytes including this header
+04       int16     2         parameter 1: type of chunk
+06       int16     2         parameter 2: type-dependent metadata
+08       int16     2         parameter 3: type-dependent metadata
+0A       int16     2         parameter 4: type-dependent metadata
+0C       ?         ?         the rest of the data
```

## Chunk types

The format changes depending on parameter 1 found above. Data will have all its
time offsets first, then all its corresponding data second.

### Type 01: tempo changes

- Parameter 2: Time frames per second
- Parameter 3: Number of BPM change/stop entries

After the time offsets, tempo data is `int32` type and will be 4 bytes per
entry. The time offset determines what point the song needs to change tempo
or stop, and the data determines how many time frames are supposed to have
passed.

You can use the following formula to convert this kind of time table into BPM.
Note that `i` must be at least 1 because deltas are used.

```
DeltaOffset = TimeOffset[i] - TimeOffset[i - 1];
DeltaTicks = TempoData[i] - TempoData[i - 1];
TicksPerSecond = Parameter2;
MeasureLength = 4096;

BPM = (DeltaOffset / MeasureLength) / ((DeltaTicks / TicksPerSecond) / 60);
```

Stops will be encoded as two consecutive entries with the same time offset, but
different data. You can use the following formula to convert this kind of
time into seconds.

```
DeltaTicks = TempoData[i] - TempoData[i - 1];
TicksPerSecond = Parameter2;

StopLengthInSeconds = DeltaTicks / TicksPerSecond;
```

### Type 02: unknown

- Parameter 3: Number of entries

After the time offsets, this data is `int16` type and will be 2 bytes per
entry. Not much else is known about this chunk type. It appears in every
observed SSQ file so far. It could possibly be linked to what tells the game
when to end the song and show the results screen.

The observed pattern in hex:
```
0104
0201
0202
0205
0203
0204
```

### Type 03: steps

- Parameter 2: Chart difficulty type
- Parameter 3: Number of steps

After the time offsets, this data is `byte` type and will be at least one byte
per entry (see below for details about why this varies.)

#### Difficulty types

Use Parameter 2 with the following table to determine the chart type.

```
Value(h)  Type
0114      Single Basic
0214      Single Standard
0314      Single Heavy
0414      Single Beginner
0614      Single Challenge

0118      Double Basic
0218      Double Standard
0318      Double Heavy
0418      Double Beginner
0618      Double Challenge
```

#### Decoding steps

Each arrow is represented by one bit of the data. Assuming the least
significant bit is 0, and the most significant bit is 7, you can use this table
to determine which arrow is to be pressed:

```
Bit       Arrow
0         Player 1 Left
1         Player 1 Down
2         Player 1 Up
3         Player 1 Right
4         Player 2 Left
5         Player 2 Down
6         Player 2 Up
7         Player 2 Right
```

**Note:** As of DDR X, shock arrows are part of the format, and are indicated
by having all bits set (a value of 0xFF).

On DDR MAX and later mixes, it is possible to encounter data where the step
data has no bits set (0x00). This indicates a freeze arrow. Freeze arrow data
is present after all the normal data. For example: a chunk with 300 steps would
consist of 300 time offsets, 300 steps, and the freeze step data would come
afterward, if any. You can easily determine the number of freeze steps by using
this formula:

```
StepCount = Parameter3;
FreezeStepCount = ChunkLength - 0x0C - (StepCount * 5)
```

The freeze data indicates which arrows are to start the freeze, and should be
read one by one as zero-bytes are encountered in the regular step data.

### Type 04: background change script

No parameters are known about this chunk type. It exists here for speculation
only.

### Type 09: song metadata

This section contains at least the title and artist of a song. This kind of chunk was discovered by looking at Dance Dance Revolution S+. Further details pending for this section.
