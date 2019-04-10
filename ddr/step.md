# Dance Dance Revolution STEP Format

This document serves to explain the STEP format used by Konami's Dance Dance
Revolution series of video games.

## About this format

STEP isn't the official name. It is, however, the format exported using DDR
Utility. It is featured in 2nd Mix through 3rd Mix Plus, including Solo Bass
Mix and Solo 2000.

The Solo mixes have a slight variation in step panel configuration. See below
for more information.

## In a nutshell

The STEP format consists of variable length "chunks". Each chunk's header
contains its total length. The end of the file is indicated by a chunk with
zero length.

## Chunk format

```
Offset(h) Type      Length    Descrption
+00       int32     4         length of the data in bytes including this header
+04       ?         ?         the rest of the data
```

## First chunk: timing data

```
Offset(h) Type      Length    Description
+00       int32     4         metric offset
+04       int32     4         linear offset, in ticks (75 ticks/sec)
```

You can use the following formula to convert this kind of time table into BPM.
Note that `i` must be at least 1 because deltas are used.

```
DeltaOffset = TimeOffset[i] - TimeOffset[i - 1];
DeltaTicks = TempoData[i] - TempoData[i - 1];
TicksPerSecond = Parameter2;
MeasureLength = 4096;

BPM = (DeltaOffset / MeasureLength) / ((DeltaTicks / TicksPerSecond) / 240);
```
