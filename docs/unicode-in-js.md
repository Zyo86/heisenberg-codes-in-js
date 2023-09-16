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

- UTF-32 (Unicode Transformation Format 32) (UCS - 4)

UTF-32 uses 32 bits to store code units, resulting in one code unit per code point. This format is the only one with fixed-length encoding; all others use a varying number of code units to encode a single code point.

- UTF-16 (Unicode Transformation Format 16) (UCS - 2)

UTF-16 uses 16-bit code units. It encodes code points as follows:

The BMP (first 16 bits of Unicode) is stored in single code units.

Astral planes: The BMP comprises `0x10_000 (65,536) code points (16 bits)`. Given that Unicode has a total of 0x110_000 code points, we still need to encode the remaining `0x100_000 (1,048,576) code points (20 bits)`. The BMP has two ranges of unassigned code points that provide the necessary storage:

`Most significant 10 bits (leading surrogate): 0xD800-0xDBFF, 55296 - 56319`
`Least significant 10 bits (trailing surrogate): 0xDC00-0xDFFF, 56314 - 57343`

Now, for each of the hex numbers between these range will either start with D8,D9,DA,DB or DC,DD,DE,DF.

In other words, the two hexadecimal digits at the end contribute 8 bits. But we can only use those 8 bits if a BMP starts with one of the following 2-digit pairs:

D8, D9, DA, DB
DC, DD, DE, DF
Per surrogate, we have a choice between 4 pairs, which is where the remaining 2 bits come from.` 4 choices means 2 * 2 i.e 2 bits`

`This way, using 2 surrogate pairs we will be able to encode 20 bits long code points from the Astral planes.`

As a consequence, each UTF-16 code unit is always either a leading surrogate, a trailing surrogate, or encodes a BMP code point.

These are two examples of UTF-16-encoded code points:

Code point 0x03C0 (π) is in the BMP and can therefore be represented by a single UTF-16 code unit: 0x03C0.
Code point 0x1F642 (🙂) is in an astral plane and represented by two code units: 0xD83D and 0xDE42.

## How to convert a more than 16 bits code point into surrogate pairs?
> Will explain later.

- UTF-8 (Unicode Transformation Format 8) 

UTF-8 has 8-bit code units. It uses 1–4 code units to encode a code point:

Code points	Code units
```
0000–007F	         0bbbbbbb (7 bits)
0080–07FF	         110bbbbb, 10bbbbbb (5+6 bits)
0800–FFFF	         1110bbbb, 10bbbbbb, 10bbbbbb (4+6+6 bits)
10000–1FFFFF	         11110bbb, 10bbbbbb, 10bbbbbb, 10bbbbbb (3+6+6+6 bits)
```
Notes:

The bit prefix of each code unit tells us:
Is it first in a series of code units? If yes, how many code units will follow?
Is it second or later in a series of code units?
The character mappings in the 0000–007F range are the same as ASCII, which leads to a degree of backward compatibility with older software.
Three examples:

Character	Code point	Code units
A	0x0041	01000001
π	0x03C0	11001111, 10000000
🙂	0x1F642	11110000, 10011111, 10011001, 10000010


## Encodings used in web development: UTF-16 and UTF-8 #
The Unicode encoding formats that are used in web development are: UTF-16 and UTF-8.

Source code internally: UTF-16 
The ECMAScript specification internally represents source code as UTF-16.
