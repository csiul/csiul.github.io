+++
draft = false
title = 'The Signal'
category = 'Forensics'
+++

> We received an image. Just an image. Our analyst swears there's more to it, but every
> tool says it's a perfectly ordinary PNG: it opens, it renders, it looks like a picture.
> They're not wrong. But they're not seeing everything. And even when you find the second
> thing, it's locked. The key was never written down.

<!--more-->

**Flag:** `csaw{0n3_f1l3_tw0_truth5_p0lygl0t_m4g1c}`

## Challenge

Provided file: `the_signal.png` (720×400, RGB PNG).

The flavor text in the image itself reinforces the structure of the challenge:

- *"one file. two truths. one locked."*
- *"the picture holds a key, the tail holds a lock."*

Two hidden things, one of which is encrypted, and the picture contains the key.

## Step 1 — Confirm it really is "just" a PNG

```
$ file the_signal.png
the_signal.png: PNG image data, 720 x 400, 8-bit/color RGB, non-interlaced
```

Renders fine, valid header — nothing obviously wrong. Time to look past the surface.

## Step 2 — Walk the PNG chunks

Parsing the chunk stream by hand (length / type / data / CRC) turns up:

```
IHDR   13   offset 8
tEXt   77   offset 33
tEXt   28   offset 122
IDAT 8100   offset 162
IEND    0   offset 8274
```

The `tEXt` chunks are worth reading directly:

```
Comment:  Look past IEND for the lock. The key is in the pixels, not the words.
Software: signal-station v2.0
```

That's a direct pointer: **something sits after `IEND`**, and it's locked — but the
key isn't in this text metadata, it's hidden in the pixel data itself.

## Step 3 — Check for trailing data after IEND

`IEND` is supposed to be the last chunk in a PNG, and most viewers/parsers stop
reading right after it. But the file is 8612 bytes total, and the `IEND` chunk
ends at byte 8286 — leaving **326 trailing bytes**:

```python
data[8286:8286+4] -> b'PK\x03\x04'
```

`PK\x03\x04` is the local file header signature for a **ZIP archive**. This is a
classic PNG/ZIP polyglot: the file is simultaneously a valid PNG (readers stop at
`IEND`) and a valid ZIP (readers scan from the end for the central directory), which
also explains the flag's `_p0lygl0t_` fragment.

Carving out everything from that offset to EOF and inspecting it as a zip:

```
flag.txt     41 bytes   (compressed 55)   flag_bits = 0x1  (encrypted)
README.txt   53 bytes   (compressed 61)   flag_bits = 0x1  (encrypted)
```

Both entries are encrypted with traditional ZipCrypto — the "lock" from the hint.
No password was given anywhere in the visible text, so per the comment, it has to
be hidden **in the pixels**.

## Step 4 — Find the key hidden in the image

Dumping the color histogram of the actual rendered image shows it's almost
entirely one flat background color, with a second nearby color that differs by
exactly 1 in a single channel:

```
(16, 18, 28)  -> 284532 pixels   (dominant background color)
(16, 18, 29)  -> 100 pixels      (same, but blue channel +1)
```

A single-bit difference in one channel across otherwise-uniform background pixels
is the signature of **LSB steganography** — data hidden in the least significant
bit of a color channel.

Extracting the LSB of the **blue channel** for every pixel (row-major order) and
packing the bits back into bytes:

```python
from PIL import Image
import numpy as np

arr = np.array(Image.open("the_signal.png"))
lsb = (arr[:, :, 2] & 1).reshape(-1)          # blue channel LSBs
packed = np.packbits(lsb[: (len(lsb)//8)*8])
data = packed.tobytes()
```

The first bytes decode as a length-prefixed string:

```
\x17 r3ad_b3tw33n_th3_p1x3ls
```

`0x17` = 23, and `"r3ad_b3tw33n_th3_p1x3ls"` is exactly 23 characters — a clean
length-prefixed payload, not noise. That's the key.

## Step 5 — Unlock the ZIP

```python
import zipfile
z = zipfile.ZipFile("embedded.zip")
pwd = b"r3ad_b3tw33n_th3_p1x3ls"

z.read("flag.txt", pwd=pwd)
# b'csaw{0n3_f1l3_tw0_truth5_p0lygl0t_m4g1c}\n'

z.read("README.txt", pwd=pwd)
# b'You found the second truth AND the key. Nicely done.\n'
```

## Summary of the trick

| Layer | Technique |
|---|---|
| Truth #1 | PNG/ZIP polyglot — a ZIP archive appended after `IEND` |
| The lock | Both files in the ZIP are ZipCrypto-encrypted |
| The key | LSB steganography in the blue channel of near-uniform background pixels, length-prefixed ASCII |
| Payoff | Password unlocks `flag.txt` |

**Flag:** `csaw{0n3_f1l3_tw0_truth5_p0lygl0t_m4g1c}`
