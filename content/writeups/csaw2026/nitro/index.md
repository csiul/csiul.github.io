+++
draft = false
title = 'Nitro'
category = 'Reverse Engineering'
+++

> Our engine only fires with the right password. We built it tough: pop it open
> in a disassembler and the important part is pure noise. It only makes sense
> once the engine is running.

<!--more-->

**Flag:** `csaw{c0d3_th4t_rewr1t3s_1ts3lf_c4nt_b3_tru5t3d}`

### 1. Initial recon

```
$ file nitro
nitro: ELF 64-bit LSB executable, x86-64, ..., not stripped

$ nm -C nitro
...
000000000040130d T __start_enccode
000000000040146b T __stop_enccode
0000000000402020 r enc_flag
00000000004020a8 r smc_key.0
000000000040130d T secret_check
```

Two things stand out immediately:

- There's a custom section, `enccode`, holding `secret_check` — separate from `.text`.
- There are symbols named `enc_flag` and `smc_key`, strongly hinting at
  self-modifying / self-decrypting code ("SMC").

Disassembling `secret_check` directly produces garbage — that's the "pure
noise" the prompt mentions, because the bytes on disk are XOR-encrypted.

### 2. Reading `main()`

```
call sysconf(0x1e)              ; get page size
lea rax, [secret_check]
lea rdx, [__stop_enccode]
sub rax, rdx                    ; size of enccode section
...
call mprotect(page, size, PROT_READ|WRITE|EXEC)
```

`main()`:
1. Computes the page-aligned range covering the `enccode` section.
2. Calls `mprotect()` to make it writable (it's normally just `r-x`).
3. Runs a decryption loop:
   ```c
   for (i = 0; i < enc_size; i++)
       secret_check[i] ^= smc_key[i % 8];
   ```
4. Calls the now-decrypted `secret_check(argv[1])`.

So the code only becomes valid x86 *after* this XOR pass runs at startup —
hence "it only makes sense once the engine is running."

### 3. Decrypting statically instead of executing

No need to actually run it — we can replicate the XOR in Python using the
raw file bytes, since `enccode`'s file offset equals `vaddr - 0x400000` for
this non-PIE binary.

```python
data  = bytearray(open('nitro','rb').read())
base  = 0x400000
enc   = data[0x40130d - base : 0x40130d - base + 0x15e]   # __start/__stop_enccode
key   = data[0x4020a8 - base : 0x4020a8 - base + 8]        # smc_key
dec   = bytes(b ^ key[i % 8] for i, b in enumerate(enc))
```

`key` turned out to be `13 37 C0 DE BA AD F0 0D`.

Disassembling the decrypted bytes (`objdump -D -b binary -m i386:x86-64
--adjust-vma=0x40130d`) gives real, readable code:

```c
char check[10] = "n2o_boost";
bool ok = true;
for (i = 0; i <= 8; i++)
    if (password[i] != check[i]) { ok = false; break; }
if (password[9] != 0) ok = false;          // length must be exactly 9

if (ok) {
    // decrypt enc_flag and printf("NITRO ENGAGED: %s\n", flag);
} else {
    puts("Access denied. The engine stays cold.");
}
```

So the required password is simply **`n2o_boost`**.

### 4. Decrypting the flag

The flag bytes (`enc_flag`, 47 bytes at `0x402020`) are XOR'd with a
keystream that is itself derived from bytes of the *decrypted* `secret_check`
function — another small self-referential trick:

```c
key_byte[i] = 0x6B + secret_check_bytes[(7*i + 3) % 0x15e] + 5*i;
flag[i]     = enc_flag[i] ^ key_byte[i];
```

Reimplementing that in Python with the already-decrypted `secret_check`
bytes:

```python
key_size = len(dec)                     # 0x15e
enc_flag = data[0x402020-base : 0x402020-base+0x2f]
flag = bytearray()
for i in range(0x2f):
    idx = (7*i + 3) % key_size
    kb  = (0x6b + dec[idx] + 5*i) & 0xff
    flag.append(enc_flag[i] ^ kb)
print(bytes(flag))
```

Output:

```
csaw{c0d3_th4t_rewr1t3s_1ts3lf_c4nt_b3_tru5t3d}
```

### 5. Verification

```
$ ./nitro n2o_boost
NITRO ENGAGED: csaw{c0d3_th4t_rewr1t3s_1ts3lf_c4nt_b3_tru5t3d}
```

### Summary of technique
- Recognize the `mprotect` + XOR-decrypt-in-place pattern as classic SMC.
- Extract the static key and encrypted bytes directly from the file instead
  of single-stepping in a debugger.
- Decrypt offline with a short script, then disassemble the *plaintext*
  bytes as raw binary at the correct virtual address.
- The "flag" blob used a second, self-referential XOR keystream built from
  the decrypted code itself — solvable the same way once you have the
  decrypted bytes in hand.
