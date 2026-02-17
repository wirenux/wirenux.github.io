# 💾 Reverse Engineering the .GBR File Format

The `.GBR` format was introduced in 1997 by [Gameboy Tile Designer](https://www.devrs.com/gb/hmgd/gbtd.html) (GBTD), a classic tool for creating tile graphics for Game Boy games.
Despite being nearly three decades old, this format is still used.
However, comprehensive documentation on the file format internals has been scarce.
This article documents my journey reverse engineering the .GBR format specification.

## Why Reverse Engineer .GBR?

While GBTD remains functional on modern systems through emulation, I wanted to:

- Create modern conversion tools that don’t require running legacy software
- Understand how tile data was stored and could be manipulated programmatically
- Build a pipeline for converting between .GBR and modern formats like PNG
- Enable batch processing of tile sets

## Initial Analysis

Opening a .GBR file in a hex editor immediately reveals some structure. Here’s what a typical .GBR file header looks like:

```css
		  00 01 02 03 04 05 06 07  08 09 0A 0B 0C 0D 0E 0F
00000000: 47 42 4f 30 01 00 00 00  78 00 00 00 47 61 6d 65  GBO0....x...Game
00000010: 62 6f 79 20 54 69 6c 65  20 44 65 73 69 67 6e 65  boy Tile Designe
00000020: 72 00 00 00 00 00 00 00  00 00 32 2e 32 00 1d 01  r.........2.2...
00000030: 02 00 00 00 48 6f 6d 65  3a 20 77 77 77 2e 63 61  ....Home: www.ca
00000040: 73 65 6d 61 2e 6e 65 74  2f 7e 68 70 6d 75 6c 64  sema.net/~hpmuld
00000050: 65 72 00 00 00 00 00 00  00 00 00 00 00 00 00 00  er..............
00000060: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00000070: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00000080: 00 00 00 00 00 00 00 00  02 00 01 00 28 20 00 00  ............( ..
00000090: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
000000a0: 00 00 00 00 00 00 00 00  00 00 08 00 08 00 80 00  ................
000000b0: 00 01 02 03                                       ....
```

## File Structure Breakdown

Through experimentation with multiple .GBR files and comparing outputs, I mapped out the structure:

### Header (180 bytes)

The header contains metadata about the tile set. Here’s what I’ve discovered:

| Offset | Size | Description       | Example Value                    |
| ------ | ---- | ----------------- | -------------------------------- |
| `0x00` | 4    | Identifier ???    | `47 42 4f 30` (`GBO0`)           |
| `0x08` | 4    | ???               | `78 00 00 00` (`x...`)           |
| `0x0C` | 21   | Producer string   | "Gameboy Tile Designer"          |
| `0x21` | 9    | ???               | `00 00 00 00 00 00 00 00 00 00`  |
| `0x2A` | 3    | Version string    | `2.2`                            |
| `0x2D` | 2    | ???               | `00 1d 01`                       |
| `0x30` | 4    | ???               | `02 00 00 00`                    |
| `0x34` | 30   | Author/URL string | "Home: www.casema.net/~hpmulder" |
| `0x52` | 94   | ???               | Zeroed out                       |
| `0xB0` | 4    | Pallette          | `00 01 02 03`                    |

The total header is exactly **180 bytes** (`0xB4` in hex).

### Tile Data Format

After the header, the actual tile data begins. This is where it gets interesting.

#### Game Boy Hardware Format (2bpp)

Game Boy tiles in hardware/exported C code use a compressed 2-bits-per-pixel format:

- Each tile is **16 bytes**
- 8 rows per tile
- 2 bytes per row (low bitplane + high bitplane)

Example of a solid black tile in hardware format:

```c
0xFF, 0xFF,  // Row 0
0xFF, 0xFF,  // Row 1
0xFF, 0xFF,  // Row 2
0xFF, 0xFF,  // Row 3
0xFF, 0xFF,  // Row 4
0xFF, 0xFF,  // Row 5
0xFF, 0xFF,  // Row 6
0xFF, 0xFF   // Row 7
```

#### .GBR Storage Format (Unpacked)

However, .GBR stores tiles in an **unpacked format**:

- Each tile is **64 bytes**
- 8 rows × 8 pixels
- Each pixel is a full byte (values 0-3)

The same solid black tile in .GBR format:

```css
03 03 03 03 03 03 03 03  // Row 0: 8 pixels, each value 3
03 03 03 03 03 03 03 03  // Row 1
03 03 03 03 03 03 03 03  // Row 2
03 03 03 03 03 03 03 03  // Row 3
03 03 03 03 03 03 03 03  // Row 4
03 03 03 03 03 03 03 03  // Row 5
03 03 03 03 03 03 03 03  // Row 6
03 03 03 03 03 03 03 03  // Row 7
```

## Decoding the Bitplane Format

The conversion from 2bpp to unpacked format requires understanding bitplane encoding:

```c
void unpack_tile(uint8_t* in_2bpp, uint8_t* out_raw) {
    for (int row = 0; row < 8; row++) {
        uint8_t low = in_2bpp[row * 2];      // Low bitplane
        uint8_t high = in_2bpp[row * 2 + 1];  // High bitplane
        
        for (int bit = 0; bit < 8; bit++) {
            int shift = 7 - bit;
            // Combine bits from both planes
            out_raw[row * 8 + bit] = 
                ((low >> shift) & 0x01) |       // Bit 0
                (((high >> shift) & 0x01) << 1); // Bit 1
        }
    }
}
```

Each pixel’s color (0-3) comes from combining corresponding bits in the low and high bitplanes.

## Practical Example

Let’s look at a checkerboard pattern tile:

**Hardware format (16 bytes):**

```c
0xAA, 0xAA,  // 10101010, 10101010 = alternating 0,2,0,2...
0x55, 0x55,  // 01010101, 01010101 = alternating 1,3,1,3...
0xAA, 0xAA,
0x55, 0x55,
0xAA, 0xAA,
0x55, 0x55,
0xAA, 0xAA,
0x55, 0x55
```

**After unpacking (64 bytes):**

```css
00 02 00 02 00 02 00 02  // Row 0
01 03 01 03 01 03 01 03  // Row 1
00 02 00 02 00 02 00 02  // Row 2
01 03 01 03 01 03 01 03  // Row 3
00 02 00 02 00 02 00 02  // Row 4
01 03 01 03 01 03 01 03  // Row 5
00 02 00 02 00 02 00 02  // Row 6
01 03 01 03 01 03 01 03  // Row 7
```

This creates the classic checkerboard pattern!

## Tools I Built

Based on this reverse engineering work, I created two utilities:

### [C2GBR](https://github.com/wirenux/C2GBR)

Converts Game Boy tile data from C array format back to .GBR format. Useful for:

- Regenerating .GBR files from exported code
- Modifying tiles programmatically then importing back to GBTD
- Creating .GBR files from scratch

### [GBR2PNG](https://github.com/wirenux/GBR2PNG)

Converts .GBR files to modern PNG format. Features:

- Batch conversion support
- Preserves tile grid structure
- Custom palette options
- Perfect for documentation and modern workflows

## Conclusion

Reverse engineering the .GBR format was a very cool dive into retro game development tools. The format is elegantly simple once understood: a fixed header followed by unpacked tile data. This structure made it straightforward to build modern conversion tools while maintaining compatibility with the original GBTD software.

The tools and knowledge gained from this project help keep these classic development workflows accessible for modern retro game developers and preservationists.

-----

## 👀 🫵 ⏭️ ⏰ ヾ(＾ ∇ ＾).

## Boring stuff

### Links

https://github.com/wirenux/GBR2PNG

https://github.com/wirenux/C2GBR

https://www.devrs.com/gb/hmgd/gbtd.html

This project is released under the [Unlicense](https://github.com/wirenux/ASM-VS-C-GB/blob/main/UNLICENSE).

Created by [@Wirenux](https://github.com/wirenux)

Read more on [my blog](https://wirenux.github.io)
