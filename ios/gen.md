Format used by [Dance Dance Revolution S+](https://itunes.apple.com/us/app/dancedancerevolution-s+-us/id300655935?mt=8) on iOS.

Not much is known about this format interanlly. It appears to contain graphics (PVR format), an MP3 preview, an MP3 of the full song, and the song information all in one file (stored at the end).

When downloaded from the store, a hashing algorithm is used likely for file verification purposes (checksum) and then the first 8 bytes are of the file are stripped. The file is then a valid `.gen` file.

# Header

The header consists of absolute file offsets and sizes. Each value is a 32-bit integer (4 bytes) in little-endian format.

The game, probably for performance reasons, only decrypts what is absolutely necessary at the time. For example, the game will not decrypt any part of the MP3 contained until you need to hear it, such as when you start playing the song.

The header starts at `0x00` and ends `0x3F`, leaving room for 7 file offsets and sizes.

Each section is encrypted using a Blowfish implementation. They all start with the characters `b"KDEI"`. Each format described is *after* the decryption process.

Since all sections in encrypted form start with `b"KDEI"`, it is likely these 4 bytes must be removed prior to beginning decryption.

* `0x00`: song (MP3), MP3 format
* `0x08`: preview song, MP3 format
* `0x10`: texture, PVR format, unknown purpose
* `0x18`: unknown, step format? Class-dumping the game suggests steps are in SSQ format
* `0x20`: unknown
* `0x28`: unknown but appears to have only text data, the start byte appears to be `0x0E` (song title?) and `0x15` (artist?) and the strings are null-terminated and padded with zeroes. Each field is allowed 72 bytes. End of the file's purpose is unknown.
* `0x30`: unknown; size is always small (0x24 (36) bytes as an example)
* `0x38`: unknown; same idea as what is at `0x30`

```c
const unsigned int SECTION_SIZE = 8;
uint32_t abs_file_offset, expected_size;

fseek(file_handle, SECTION_SIZE * section_number, 0);

fread(&abs_file_offset, 4, 1, file_handle);
fread(&expected_size, 4, 1, file_handle);
```

```c
struct gen_hdr {
    uint32_t song_offset;
    uint32_t song_size;

    uint32_t song_preview_offset;
    uint32_t song_preview_size;

    uint32_t unk_texture1_offset;
    uint32_t unk_texture1_size;

    uint32_t unk1_offset;
    uint32_t unk1_size;

    uint32_t unk2_offset;
    uint32_t unk2_size;

    uint32_t unk3_offset;
    uint32_t unk3_size;

    uint32_t unk4_offset;
    uint32_t unk4_size;
};
```
