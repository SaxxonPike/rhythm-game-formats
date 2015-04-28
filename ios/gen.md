Format used by [Dance Dance Revolution S+](https://itunes.apple.com/us/app/dancedancerevolution-s+-us/id300655935?mt=8) on iOS.

Not much is known about this format interanlly. It appears to contain graphics (PVR format), an MP3 preview, an MP3 of the full song, and the song information all in one file (stored at the end).

When downloaded from the store, a hashing algorithm is used likely for file verification purposes (checksum) and then the first 8 bytes are of the file are stripped. The file is then a valid `.gen` file.
