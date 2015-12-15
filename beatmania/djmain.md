# beatmania DJMAIN Formats

This document serves to explain the formats used on DJMAIN-based beatmania
games from Konami.

## About these formats

The software for any version of these games consists of the ROM images and the
HDD image. The ROM images store the program code, graphics and song metadata.
The HDD image stores the charts and audio for each song.

## In a nutshell

TODO: Do something with the following:
```
BMFINAL

Song metadata: 0x035444 @ 0x1E each
  00-03: Song name offset
  07-0D: Difficulties
  0E-0F: HDD Song ID A
  10-11: HDD Song ID B
  12-13: HDD Song ID C

Song metadata offset table: 0x03E06C
```
