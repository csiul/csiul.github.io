+++
draft = false
title = 'Masterkey'
category = 'Reverse Engineering'
+++

> Every lock in the building answers to one key. We burned the checker into
> a little machine of our own design — it speaks a language you won't find
> in any disassembler's opcode table. Give it the master key and it'll tell
> you. 

<!--more-->

**Flag:** `csaw{cl1mb1ng_th3_v1rtu4l_st4ck_0n3_0pc0d3_4t_4_t1m3}`

### 1. Initial recon

```
$ file masterkey
masterkey: ELF 64-bit LSB pie executable, x86-64, ..., stripped

$ readelf -S masterkey
...
[16] .text     PROGBITS  ...   size 0x259   <- tiny
[18] .rodata   PROGBITS  ...   size 0x7ba   <- unusually large for a stripped CLI tool
```

A ~600-byte `.text` next to a ~2KB `.rodata` is the classic shape of a
**bytecode interpreter** with the program data stored as a big blob of
constants — "a little machine of our own design" and "a language you won't
find in any disassembler's opcode table" are both direct hints at this.

### 2. Reverse-engineering the interpreter

`main()`:
1. Checks `argc == 2`.
2. Checks `strlen(argv[1]) == 0x35` (53).
3. Runs an interpreter loop over a bytecode array at `.rodata+0x40`, 3 bytes
   per instruction: `[opcode, reg_a, operand_b]`.

The dispatcher is a cascade of `cmp`/`ja`/`je` on the opcode byte. Tracing
each branch target gives this instruction set (`buf[]` is a small working
register file on the stack, indexed by `reg_a` / the byte at `operand_b`
when it's used as a register index):

| Opcode | Mnemonic       | Effect                                   |
|-------:|----------------|-------------------------------------------|
| `0x91` | `LOAD_INPUT`   | `reg[a] = password[b]`                    |
| `0x7a` | `XOR_IMM`      | `reg[a] ^= b`                             |
| `0x24` | `ADD_IMM`      | `reg[a] += b`                             |
| `0xb2` | `ROL_IMM`      | `reg[a] = rotl8(reg[a], b)`               |
| `0x1d` | `MUL_IMM`      | `reg[a] *= b` (mod 256)                   |
| `0x4e` | `XOR_REG`      | `reg[a] ^= reg[b]`                        |
| `0x6d` | `ADD_REG`      | `reg[a] += reg[b]`                        |
| `0xa7` | `CHECK_EQ_IMM` | clears a "success" flag if `reg[a] != b`  |
| `0xe0` | `HALT`         | prints "Correct!"/"Wrong." based on flag  |

### 3. Extracting and parsing the bytecode

```python
mnem = {0x1d:"MUL_IMM",0x24:"ADD_IMM",0x4e:"XOR_REG",0x6d:"ADD_REG",
        0x7a:"XOR_IMM",0x91:"LOAD_INPUT",0xa7:"CHECK_EQ_IMM",
        0xb2:"ROL_IMM",0xe0:"HALT"}

data = open('masterkey','rb').read()
start = 0x2040
i = 0; instrs = []
while True:
    op, a, b = data[start+i], data[start+i+1], data[start+i+2]
    instrs.append((mnem[op], a, b)); i += 3
    if op == 0xe0: break
```

This decodes into 638 instructions: one 8-instruction "round" per password
character (53 rounds total), each following an identical template:

```
LOAD_INPUT 0, i        ; reg0 = password[i]
XOR_IMM    0, K1[i]
ADD_IMM    0, K2[i]
ROL_IMM    0, K3[i]
XOR_IMM    0, 195
MUL_IMM    0, 27
XOR_REG    0, 1         ; reg0 ^= chain_state
CHECK_EQ_IMM 0, K4[i]   ; must equal target constant

LOAD_INPUT 2, i
ADD_REG    1, 2          ; chain_state += password[i]
ROL_IMM    1, 3           ; chain_state = rotl8(chain_state, 3)
XOR_IMM    1, 158          ; chain_state ^= 158
```

`chain_state` (register 1) is initialized to `0 ^ 60 = 60` before round 0,
and carries forward across rounds — so each character's check also depends
on every character before it (a simple hash-chain, not just a per-byte
substitution).

### 4. Inverting instead of brute-forcing

Every operation in a round (`XOR`, `ADD mod 256`, `ROL`, `XOR`, `MUL by 27
mod 256`) is invertible over a byte, so instead of brute-forcing 53
characters we can solve directly, one round at a time, using the running
`chain_state`:

```python
inv27 = pow(27, -1, 256)     # modular inverse of 27 mod 256

def rol8(x, n): n %= 8; return ((x << n) | (x >> (8 - n))) & 0xFF
def ror8(x, n): n %= 8; return ((x >> n) | (x << (8 - n))) & 0xFF

state = 60
flag = []
for (idx, K1, K2, K3, K4) in rounds:       # parsed from the bytecode
    t4 = K4 ^ state
    t3 = (t4 * inv27) % 256
    t2 = t3 ^ 195
    t1 = ror8(t2, K3)
    t0 = (t1 - K2) % 256
    ch = t0 ^ K1
    flag.append(ch)
    state = rol8((state + ch) & 0xFF, 3) ^ 158

print(bytes(flag))
```

Output:

```
csaw{cl1mb1ng_th3_v1rtu4l_st4ck_0n3_0pc0d3_4t_4_t1m3}
```

### 5. Verification

```
$ ./masterkey 'csaw{cl1mb1ng_th3_v1rtu4l_st4ck_0n3_0pc0d3_4t_4_t1m3}'
Correct! That's the master key.
```

### Summary of technique
- Spot the interpreter shape (tiny `.text`, oversized `.rodata`) before
  even reading a single instruction.
- Reverse-engineer the dispatcher once to recover the full custom ISA.
- Parse the bytecode blob into a structured instruction list programmatically
  rather than reading raw hex.
- Notice each per-character "round" is a chain of invertible byte operations
  plus a running state — so the whole thing can be solved by algebraic
  inversion instead of brute force or emulation.
