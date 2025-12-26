---
layout: post
title: "Steganowarmup Writeup"
subtitle: "File-format Steganography Warmup for ECSC 2025"
description: "File-format Steganography Warmup for ECSC 2025"
author: "jxkxl"
header-style: text
tags:
  - CTF
  - Writeup
  - Steganography
  - ECSC2025
---

A file-format steganography warmup - enjoy!

## Tools Used

- file
- pngcheck
- strings
- xxd
- Python

## Approach

### Initial Analysis

I started by checking the basic file properties:

```bash
$ file stegochall.png
stegochall.png: PNG image data, 4096 x 4096, 8-bit/color RGB, non-interlaced

$ ls -la stegochall.png
-rw-rw-r-- 1 jxkxl jxkxl 232861 Oct  7 09:06 stegochall.png
```

So it's a 4096x4096 PNG image, which is pretty large. The file size of 232KB seemed reasonable for that resolution, so nothing immediately suspicious there.

### PNG Structure Analysis

Since the challenge mentioned "file-format steganography," I figured the flag was probably hidden in the file structure itself rather than in the image content. I ran `pngcheck` to analyze the PNG structure:

```bash
$ pngcheck -v stegochall.png
File: stegochall.png (232861 bytes)
  chunk IHDR at offset 0x0000c, length 13
    4096 x 4096 image, 24-bit RGB, non-interlaced
  chunk IDAT at offset 0x00025, length 333
  chunk IDAT at offset 0x0017e, length 444
  chunk IDAT at offset 0x00346, length 333
  ...
  chunk IDAT at offset 0x198d3, length 128182
  chunk IEND at offset 0x38d95, length 0
No errors detected in stegochall.png (267 chunks, 99.5% compression).
```

This was interesting. The file had **267 total chunks**, which is extremely unusual for a normal PNG. Most PNGs only have a handful of chunks. I also noticed that most of the IDAT chunks had alternating lengths of 333 and 444 bytes, with one massive IDAT chunk at the end (128,182 bytes). The 99.5% compression ratio was also pretty high.

### Pattern Recognition

The alternating chunk lengths (333 and 444 bytes) looked suspicious. Since this was a steganography challenge and the description mentioned "file-format steganography," I figured the chunk lengths themselves might be encoding the flag.

I hypothesized that:
- 333 bytes = binary `0`
- 444 bytes = binary `1`

This would allow the flag to be encoded in the file structure itself, which is pretty clever.

### Extraction Process

I wrote a Python script to extract the PNG chunks and decode the binary pattern:

```python
import struct

def extract_png_chunks(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    
    offset = 8
    chunks = []
    
    while offset < len(data) - 8:
        length = struct.unpack('>I', data[offset:offset+4])[0]
        chunk_type = data[offset+4:offset+8]
        chunk_data = data[offset+8:offset+8+length]
        crc = data[offset+8+length:offset+12+length]
        
        chunks.append({
            'type': chunk_type,
            'length': length,
            'data': chunk_data,
            'crc': crc
        })
        
        offset += 12 + length
    
    return chunks

chunks = extract_png_chunks('stegochall.png')

# Extract IDAT chunks and analyze the pattern
idat_chunks = [chunk for chunk in chunks if chunk['type'] == b'IDAT']

# The pattern: 333 = 0, 444 = 1
binary_data = []
for chunk in idat_chunks[:-1]:  # Skip the last large chunk
    if chunk['length'] == 333:
        binary_data.append('0')
    elif chunk['length'] == 444:
        binary_data.append('1')

binary_string = ''.join(binary_data)

# Convert binary to ASCII
ascii_text = ''
for i in range(0, len(binary_string), 8):
    if i + 8 <= len(binary_string):
        byte_str = binary_string[i:i+8]
        try:
            char = chr(int(byte_str, 2))
            ascii_text += char
        except:
            pass

print(f'ASCII text: {ascii_text}')
```

The script extracted all the IDAT chunks (excluding the final large one), converted the chunk lengths to binary (333 = 0, 444 = 1), and then decoded the binary string to ASCII.

### Decoding the Flag

After running the script, the binary data decoded to the flag. The technique worked because PNG allows multiple IDAT chunks, and their lengths can vary. By encoding binary data in the chunk lengths, the flag was hidden in the file structure itself rather than in the image content.

The image itself appeared blank (hence the flag text "ABlankPageYetItHidesTheFlag"), but the flag was encoded in the PNG chunk structure.

## Solution

- **Flag:** `ECSC{ABlankPageYetItHidesTheFlag}`

