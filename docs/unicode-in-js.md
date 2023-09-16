## Code points vs. code units 

Two concepts are crucial for understanding Unicode:

  **Code points** are numbers that represent the atomic parts of Unicode text. Most of them represent visible symbols but they can also have other meanings such as specifying an aspect of a symbol (the accent of a letter, the skin tone of an emoji, etc.).
  
  **Code units** are numbers that encode code points, to store or transmit Unicode text. One or more code units encode a single code point. Each code unit has the same size, which depends on the encoding format that is used. The most popular format, UTF-8, has 8-bit code units.


## Code points 
The first version of Unicode had 16-bit code points. Since then, the number of characters has grown considerably and the size of code points was extended to 21 bits. These 21 bits are partitioned in 17 planes, with 16 bits each:

  > **Plane 0:** Basic Multilingual Plane (BMP), `0x0000–0xFFFF, in decimal 0 - 65535, in binary 0000 0000 0000 0000 - 1111 1111 1111 1111`
  Contains characters for almost all modern languages (Latin characters, Asian characters, etc.) and many symbols.

  > Plane 1: Supplementary Multilingual Plane (SMP), `0x10000–0x1FFFF, in decimal 65536 - 131071, in binary 1 0000 0000 0000 0000 - 1 1111 1111 1111 1111`
  >> Supports historic writing systems (e.g., Egyptian hieroglyphs and cuneiform) and additional modern writing systems.
  >> Supports emojis and many other symbols.

  > Plane 2: Supplementary Ideographic Plane (SIP), 0x20000–0x2FFFF
  Contains additional CJK (Chinese, Japanese, Korean) ideographs.

  > Plane 3–13: Unassigned

  > Plane 14: Supplementary Special-Purpose Plane (SSP), 0xE0000–0xEFFFF
  Contains non-graphical characters such as tag characters and glyph variation selectors.

  > Plane 15–16: Supplementary Private Use Area (S PUA A/B), `0x0F0000–0x10FFFF, in decimal 983040 - 1114111, in binary 1111 0000 0000 0000 0000 - 1 0000 1111 1111 1111 1111`
  >> Available for character assignment by parties outside the ISO and the Unicode Consortium. Not standardized.

  **Planes 1-16 are called supplementary planes or astral planes.** Remember plane 0 is BMP (Basic Multilingual Plane)

  >> Look closely the number of bits needed to represent the code points as we move towards the higher planes.

## Encoding Unicode code points: UTF-32, UTF-16, UTF-8 

The main ways of encoding code points are three Unicode Transformation Formats (UTFs): UTF-32, UTF-16, UTF-8. The number at the end of each format indicates the size (in bits) of its code units.

- UTF-32 (Unicode Transformation Format 32) 

UTF-32 uses 32 bits to store code units, resulting in one code unit per code point. This format is the only one with fixed-length encoding; all others use a varying number of code units to encode a single code point.

- 
