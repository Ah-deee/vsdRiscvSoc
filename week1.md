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
![Image](https://github.com/user-attachments/assets/e8568475-f9f1-45a9-9f20-32a7b0d6fee8)

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


![Image](https://github.com/user-attachments/assets/ca19f894-00a5-4a11-900a-4b23dd5169ec)
![Image](https://github.com/user-attachments/assets/233bfbfa-2844-47a7-8f7d-bd86210875e3)


---

## 3. From C to Assembly

**Commands:**

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.asm
hexdump -C hello.elf > hello.hex
```

**Screenshot:**
![Image](https://github.com/user-attachments/assets/0321f41e-83c6-4e43-a7f4-82afb72ffa20)

---

## 4. 

**Commands:**

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.asm
hexdump -C hello.elf > hello.hex
```

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

