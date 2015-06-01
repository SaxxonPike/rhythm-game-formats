Format used by [Dance Dance Revolution S+](https://itunes.apple.com/us/app/dancedancerevolution-s+-us/id300655935?mt=8) on iOS.

Not much is known about this format internally. It appears to contain graphics (PVR format), an MP3 preview, an MP3 of the full song, and the song information all in one file (stored at the end).

When downloaded from the store, a hashing algorithm is used likely for file verification purposes (checksum) and then the first 8 bytes are of the file are stripped. The file is then a valid `.gen` file.

# Header

The header consists of absolute file offsets and sizes. Each value is a 32-bit integer (4 bytes) in little-endian format.

The game, probably for performance reasons, only decrypts what is absolutely necessary at the time. For example, the game will not decrypt any part of the MP3 contained until you need to hear it, such as when you start playing the song.

The header starts at `0x00` and ends `0x3F`, leaving room for 7 file offsets and sizes.

Almost all sections are encrypted using a Blowfish implementation. They all start with the characters `b"KDEI"`. Each format described is *after* the decryption process.

Since all sections in encrypted form start with `b"KDEI"`, it is likely these 4 bytes must be removed prior to beginning decryption.

Offsets and descriptions of content at said offset:

* `0x00`: song (MP3), MP3 format
* `0x08`: preview song, MP3 format
* `0x10`: song banner in PVR format. This may not be a very high quality image and is optimised for use with the game engine. As a result, it appears the texture is stored upside-down and mirrored horizontally. To get the correct image, a 180 degree rotation and mirror is required.
* `0x18`: steps in [SSQ format](/ddr/ssq.md), with a chunk type 0x09 at the end storing metadata
* `0x20`: unknown
* `0x28`: song information including title and artist. There are usually 3 fields that can hold a maximum of 70 characters. The first 3 bytes of each field is the length of the string (even though the fields are zero-padded, this is not what determines the end and these are not C strings). The string length is in big-endian (example: `00 00 0e` for 15). This field is not encrypted. All strings are encoded in UTF-8
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

    uint32_t ssq_offset;
    uint32_t ssq_size;

    uint32_t unk1_offset;
    uint32_t unk1_size;

    uint32_t song_info_offset;
    uint32_t song_info_size;

    uint32_t unk2_offset;
    uint32_t unk2_size;

    uint32_t unk3_offset;
    uint32_t unk3_size;
};
```

# Reading song information

If the length read does not make sense (exceeds 70) with the 3 bytes, it should be ignored and the data should be read until `\0` with a limit of 70.

The length value is still the same, even if the data is in Japanese (or a multi-byte sequence of any type). The length in the case of *ＴЁЯＲＡ* is still 5, although the actual length of the data in UTF-8 encoding is 13. So for the purpose of data extraction, it may be better to assume to read until `\0` in every case, with a maximum length of 70.

```c
const unsigned int SECTION_SIZE = 8;
const unsigned int SONG_SECTION_NUMBER = 5;
struct gen_hdr gen_info;
char *data, artist[70], title[70], alt_title[70];
unsigned int length;

fseek(file_handle, SONG_SECTION_NUMBER * SECTION_SIZE, 0);
fread(&gen_info.song_info_offset, 4, 1, file_handle);
fread(&gen_info.song_info_size, 4, 1, file_handle);

fseek(file_handle, gen_info.song_info_offset, 0);
data = malloc(gen_info.song_info_size);
fread(data, gen_info.song_info_size, 1, file_handle);

// Get title
// data + 73 for 'alternate' title (may be a romaji title)
// data + 143 for artist
memcpy(&length, data, 3);
endian_swap_if_necessary(&length);
memcpy(&title, data + 3, length);

free(data);
```

# Banner

The PVR format used in this game is readable with [QuickPVR](https://github.com/Volcore/quickpvr). The exact PVR format used is not yet known (there are [many choices](https://github.com/Volcore/quickpvr/blob/master/pvr.cc#L70)).
