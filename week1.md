<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# make it a similar fashion as this

Certainly! Here’s a Markdown outline for your RISC-V bare-metal tasks, styled to match the structure and clarity of your provided examples.
**You can insert images where indicated after exporting them from your Word file.**

---

# Week 1 Task Report


---

## 1. Install \& Sanity-Check the Toolchain

**Objective:**
Install the RISC-V toolchain and verify installation.

**Commands:**

```bash
riscv32-unknown-elf-gcc --version
```

**Screenshot:**

---

## 2. Compile “Hello, RISC-V”

**Objective:**
Create and cross-compile a minimal C program.

**C Code:**

```c
int main() {
    while(1);
    return 0;
}
```

**Command:**

```bash
riscv32-unknown-elf-gcc -nostdlib -T linker.ld hello.c -o hello.elf
```

**Screenshot:**

---

## 3. Hex Dump \& Disassembly

**Commands:**

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.asm
hexdump -C hello.elf > hello.hex
```

**Screenshot:**

---

## 4. ABI \& Register Cheat-Sheet

**Table:**


| ABI Name | Number | Role |
| :-- | :-- | :-- |
| zero | x0 | Constant zero |
| ra | x1 | Return address |
| sp | x2 | Stack pointer |
| gp | x3 | Global pointer |
| tp | x4 | Thread pointer |
| t0–t6 | x5–x7, x28–x31 | Temporaries (caller-saved) |
| s0–s11 | x8–x9, x18–x27 | Saved regs (callee-saved) |
| a0–a7 | x10–x17 | Args/return values |

**Screenshot:**

---

## 5. Stepping with GDB

**Commands:**

```bash
riscv32-unknown-elf-gdb hello.elf
(gdb) target sim
(gdb) load
(gdb) break main
(gdb) run
(gdb) step
```

**Screenshot:**

---

## 6. Running Under an Emulator

**Command:**

```bash
qemu-system-riscv32 -M virt -nographic -bios none -kernel hello.elf
```

**Screenshot:**

---

## 7. Exploring GCC Optimisation

**Command:**

```bash
riscv32-unknown-elf-gcc -O2 -S hello.c -o hello_O2.s
```

**Screenshot:**

---

## 8. Inline Assembly Basics

**Example:**

```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}
```

**Screenshot:**

---

## 9. Memory-Mapped I/O Demo

**Example:**

```c
#define GPIO_ADDR 0x10012000
volatile uint32_t *gpio = (volatile uint32_t *)GPIO_ADDR;
*gpio = 0x1; // Set GPIO pin
```

**Screenshot:**

---

## 10. Linker Script 101

**Example:**

```ld
ENTRY(_start)
MEMORY {
    ROM (rx) : ORIGIN = 0x80000000, LENGTH = 256K
    RAM (rwx) : ORIGIN = 0x10000000, LENGTH = 64K
}
SECTIONS {
    .text : { *(.text*) } > ROM
    .data : { *(.data*) } > RAM
    .bss  : { *(.bss*)  } > RAM
    .stack (NOLOAD) : { . = ALIGN(16); _stack_top = . + 0x1000; } > RAM
}
```

**Screenshot:**

---

## 11. Start-up Code \& crt0

**Example:**

```asm
.section .text.start
.global _start
_start:
    la sp, _stack_top
    call main
1:  j 1b
```

**Screenshot:**

---

## 12. Interrupt Primer

**Example:**

```c
void __attribute__((interrupt)) timer_interrupt_handler(void) {
    // handle interrupt
}
```

**Screenshot:**

---

## 13. Atomic Operations

**Example:**

```c
static inline uint32_t atomic_add(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("amoadd.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result;
}
```

**Screenshot:**

---

## 14. Endianness Demo

**Example:**

```c
union { uint32_t i; uint8_t c[^4]; } test = { .i = 0x01020304 };
printf("Bytes: %02X %02X %02X %02X\n", test.c[^0], test.c[^1], test.c[^2], test.c[^3]);
```

**Screenshot:**

---

## 15. Struct Packing Demo

**Example:**

```c
struct __attribute__((packed)) packed_struct {
    uint8_t a; uint32_t b; uint16_t c; uint8_t d;
};
printf("Packed struct size: %zu\n", sizeof(struct packed_struct));
```

**Screenshot:**

---

## 16. Using Newlib printf Without an OS

**Minimal UART printf:**

```c
void uart_putchar(char c) { *(volatile uint32_t*)0x10000000 = c; }
int _write(int fd, char *buf, int len) {
    for(int i=0; i<len; i++) uart_putchar(buf[i]);
    return len;
}
```

**Link with:** `-lc -nostdlib -T linker.ld`

**Screenshot:**

---

## 17. Troubleshooting/Verification

**Common checks:**

- Use `riscv32-unknown-elf-readelf -S file.elf` to verify section addresses.
- Use `qemu-system-riscv32` with `-serial mon:stdio` for UART output.
- Use `riscv32-unknown-elf-objdump -d file.elf` to check for expected instructions.

**Screenshot:**

---

*Replace all image filenames with your actual exported images from the Word file.*

<div style="text-align: center">⁂</div>

[^1]: Week1.docx

[^2]: README.md

[^3]: https://stackoverflow.com/questions/76455941/how-to-run-and-debug-a-simple-riscv32-bare-metal-assembly-compiled-into-elf-us

[^4]: https://github.com/schoeberl/risc-v-lab/blob/master/README.md

[^5]: https://vivonomicon.com/2020/02/11/bare-metal-risc-v-development-with-the-gd32vf103cb/

