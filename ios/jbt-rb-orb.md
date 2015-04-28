# Zip files with special extensions

`.jbt`, `.orb`, and `.rb` are formats used generally by the iOS versions of BEMANI games *jubeat plus*, *jukebeat*, *Reflec Beat Plus*, *Rb+*, and *pop'n rhythmin*. The `.jbt` format is also used with the Android version of *jubeat plus*.

All 3 formats are zip files with a non-standard extension. You can use any program capable of unzipping zip files to unzip them. There is nothing special about the zip file itself.

*pop'n rhythm* also has `.acv` files that are for arcade play data.

## Files inside

All files inside are encrypted using Blowfish (in all games including Android (C++) the class name is `BFCodec`) with a custom set of P and S boxes. The IV seems to be static in all cases. Known keys or IVs will not be disclosed here.

There are at least 2 keys in existence which are calculated like so, from an obfuscated string stored in the binary (which contains both keys at different offsets: 0 and 8 characters in):

```python
from hashlib import md5

unobfuscated = []

# where obfuscated_str is a bytearray instance
for i in range(0, 26):  # For alternate key, range(0, 22) and in the loop add 7 to each result
    unobfuscated.append(obfuscated_str[i] + i)

# unobfuscated is a list of integers but it is human-readable: map(chr, unobfuscated)

key = md5(unobfuscated).digest() # 16 integers (the MD5 bytes) which are used as the key (passed as const char *)
```

Which key to use may be determined in the encrypted file itself.

# While encrypted

All files inside the zip file contain 8 bytes at the end that are used for verification purposes prior to decryption. These appear to be offsets. The last 4 bytes contain the start of the verification bytes.

The decryption algorithm will remove the last 8 bytes off the read file data before decryption.

# File formats and purpose

*Note: These are only the formats described once decrypted.*

For note format files, it is generally assumed that the notes themselves are encoded the same way as their arcade counterparts, except in separate files. There is no arcade format to compare to for *pop'n rhythmin'*.

## artwork

Artwork image. This is a PNG inside. It is Apple's version so [`pngdefry`](http://www.jongware.com/pngdefry.html) is necessary. Assume so for all PNGs.

## bgm

An M4A (AAC) file of the song.

## info

[Property list](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/PropertyLists/AboutPropertyLists/AboutPropertyLists.html) in XML. Contains song information such as artist, title, etc.

# Jubeat-specific

## artwork_s

Artwork image (purpose unknown). This may be a retina version of `artwork`.

## index

Unknown. The name sounds like it might be used for sorting purposes.

## name_b

Unknown.

## name_w

Unknown.

## seq_adv

Advanced level notes.

## seq_bas

Basic level notes.

## seq_ext

Extreme level notes.

# Reflec Beat-specific

## artist_b

Unknown. Assumed to be an image.

## artist_b2x

Unknown purpose. Assumed to be *@2x* (retina) version of `artist_b`.

## artist_w

Same as `artist_b`.

## artist_w2x

Same as `artist_w`.

## artwork_2x

Retina version of `artwork`.

## note_bas

Basic level notes.

## note_har

Hard level notes.

## note_med

Medium level notes.

## pre

Preview in M4A (AAC) format.

## title_b

Title card image in Apple's PNG format.

## title_b2x

*@2x* (retina) version of `title_b`.

## title_w

Image. Unknown purpose.

## title_w2x

*@2x* (retina) version of `title_w`.

# Popn'n Rhythmin-specific

## sheet_n

Normal level notes.

## sheet_h

Hard level notes.

## sheet_ex

Extreme level notes.

## title_2x

Title card. Rhythmin' is a new game and does not support pre-iOS 7 as of version 2.0.0. As a result all images are retina optimised and no regular size images are provided.

## artist_2x

Artist image? Unknown purpose.
