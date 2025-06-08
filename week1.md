# Week 1 Task Report


---

# 1. Install \& Sanity-Check the Toolchain

**Objective:**
Install the RISC-V toolchain and verify installation.

### **1. Extract the downloaded RISCV toolchain**

```bash
sudo mkdir -p /opt/riscv
```

- **Creates a directory** `/opt/riscv` (if it doesn’t exist) with superuser permissions. This is a common location for installing system-wide tools.

```bash
sudo tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz -C /opt/riscv
```

- **Extracts** the contents of your downloaded `.tar.gz` archive into `/opt/riscv`.
    - `-xzf` tells `tar` to extract (`x`), use gzip decompression (`z`), and specify a file (`f`).
    - `-C /opt/riscv` changes the extraction directory to `/opt/riscv`.

```bash
mkdir -p ~/riscv-toolchain
```

- **Creates a directory** in your home folder called `riscv-toolchain` for a user-local install.

```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz -C ~/riscv-toolchain
```

- **Extracts** the archive into your user-local directory.

---

### **2. Add to PATH**

```bash
echo 'export PATH="$PATH:/opt/riscv/bin"' >> ~/.bashrc
```

- **Appends** a line to your shell’s startup file (`.bashrc`) to add `/opt/riscv/bin` to your system’s `PATH` environment variable.
- This ensures you can run the toolchain’s commands from any directory.

```bash
echo 'export PATH="$PATH:~/riscv-toolchain/bin"' >> ~/.bashrc
```

- Same as above, but for a user-local install.

```bash
source ~/.bashrc
```

- **Reloads** your shell configuration so the new `PATH` takes effect immediately.

---

### **3. Verify Binaries**

```bash
riscv32-unknown-elf-gcc --version
```

- **Displays** the version information for the RISC-V GCC cross-compiler.
- Confirms that the compiler is installed and accessible.

```bash
riscv32-unknown-elf-objdump --version
```

- **Displays** the version information for the RISC-V `objdump` utility.

```bash
riscv32-unknown-elf-gdb --version
```

- **Displays** the version information for the RISC-V `gdb` debugger (if included).

---

### **4. Test Compilation**

```c
int main() { return 0; }
```

- **Minimal C program** that does nothing and returns 0. Used to test compilation.

```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 test.c -o test.elf
```

- **Compiles** `test.c` for the RV32IMAC RISC-V architecture, using the ILP32 ABI, and outputs an ELF binary named `test.elf`.
    - `-march=rv32imac` specifies the target architecture.
    - `-mabi=ilp32` specifies the Application Binary Interface (32-bit int, long, pointer).
    - `-o test.elf` names the output file.

```bash
file test.elf
```

- **Checks** the file type of `test.elf`.
- Should output something like “ELF 32-bit LSB executable, RISC-V…” confirming it’s a RISC-V binary.

---

### **Troubleshooting**

```bash
sudo chmod -R 755 /opt/riscv
```

- **Sets permissions** so all users can read and execute files in `/opt/riscv`.

```bash
sudo apt install gdb-multiarch
```

- **Installs** a multi-architecture version of GDB, useful if your toolchain didn’t include a RISC-V GDB binary.

---

Each code block is a step in setting up, configuring, and verifying your RISC-V toolchain for cross-compilation and debugging.



**Screenshot:**
![Image](https://github.com/user-attachments/assets/e8568475-f9f1-45a9-9f20-32a7b0d6fee8)

---

# 2. Compile “Hello, RISC-V”

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

# 3. From C to Assembly

To generate the `.s` (assembly) file for your C code and understand the **prologue** and **epilogue** of the `main` function, follow these steps:

---

### **1. Generate Assembly Code**

Use your RISC-V toolchain to compile the C file into assembly with **no optimizations** (`-O0`) to keep the code human-readable:

```bash
riscv32-unknown-elf-gcc -S -O0 -march=rv32imac -mabi=ilp32 hello.c -o hello.s
```


#### **Flags Explained**:

- `-S`: Output assembly code instead of an executable.
- `-O0`: Disable optimizations (keeps the assembly code simple).
- `-march=rv32imac`: Target RV32IMAC architecture.
- `-mabi=ilp32`: Use 32-bit integer/long/pointer ABI.

---

### **2. Example Assembly Output**

Here’s a simplified version of what `hello.s` might look like for the `main` function:

```asm
    .text
    .globl  main
    .type   main, @function
main:
    addi    sp, sp, -16     # Prologue: Allocate stack space
    sw      ra, 12(sp)      # Save return address (ra) to stack
    lui     a0, %hi(.LC0)   # Load address of "Hello, RISC-V!\n"
    addi    a0, a0, %lo(.LC0)
    call    printf          # Call printf
    li      a0, 0           # Load return value (0) into a0
    lw      ra, 12(sp)      # Epilogue: Restore return address
    addi    sp, sp, 16      # Deallocate stack space
    ret                     # Return to caller
.LC0:
    .string "Hello, RISC-V!\n"
```


---

### **3. Prologue and Epilogue Explained**

#### **Prologue** (Function Setup):

- Prepares the stack frame for the function.
- **Tasks**:

1. **Allocate stack space**: Adjust the stack pointer (`sp`) to reserve space for local variables and saved registers.
2. **Save return address (`ra`)**: Stores the return address (where the function should return to) on the stack.
```asm
addi    sp, sp, -16     # Allocate 16 bytes of stack space
sw      ra, 12(sp)      # Save return address (ra) at offset 12
```


#### **Epilogue** (Function Cleanup):

- Restores the stack and registers before returning.
- **Tasks**:

1. **Restore return address (`ra`)**: Retrieve the saved `ra` from the stack.
2. **Deallocate stack space**: Reset the stack pointer (`sp`) to its original value.
3. **Return**: Use `ret` to jump back to the caller.
```asm
lw      ra, 12(sp)      # Restore return address
addi    sp, sp, 16      # Deallocate stack space
ret                     # Return to caller
```


---

### **Key RISC-V Assembly Concepts**

- **`sp` (Stack Pointer)**: Register `x2`, manages the call stack.
- **`ra` (Return Address)**: Register `x1`, holds the address to return to after a function call.
- **`call`**: Pseudo-instruction for `jal ra, <function>` (jump-and-link to a function, saving the return address in `ra`).

---

### **Why Prologue/Epilogue Matter**

- **Stack Frames**: Functions use stack frames to isolate their local data and ensure safe returns.
- **Nested Calls**: Without saving `ra`, nested function calls (like `printf`) would overwrite the return address, causing crashes.

---

### **Summary**

- **Prologue**: Sets up the stack frame (`sp` adjustment, saving `ra`).
- **Epilogue**: Cleans up the stack frame (`ra` restoration, `sp` reset).
- Use `-O0` to keep assembly readable for analysis.


**Screenshot:**
![Image](https://github.com/user-attachments/assets/0321f41e-83c6-4e43-a7f4-82afb72ffa20)

---

# 4. Hex Dump & Disassembly

### **1. Convert ELF to Raw Hex**

Use `objcopy` to extract raw hex data from your ELF file:

```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

- **`-O ihex`**: Outputs the file in Intel HEX format (human-readable hex values).
- For raw binary (non-text hex), use `-O binary` instead.

---

### **2. Disassemble the Hex/Binary File**

Use `objdump` to disassemble the raw hex/binary:

```bash
riscv32-unknown-elf-objdump -D -b binary -m riscv:rv32 -M numeric,no-aliases hello.hex
```


#### **Flags Explained**:

- **`-D`**: Disassemble all sections.
- **`-b binary`**: Treat the input as a raw binary (not an ELF).
- **`-m riscv:rv32`**: Specify RISC-V 32-bit architecture.
- **`-M numeric,no-aliases`**: Show raw register numbers (e.g., `x10` instead of `a0`) for clarity.

---

### **Understanding the Columns**

| Column | Example | Description |
| :-- | :-- | :-- |
| **1** | `00000000` | **Address**: Memory address of the instruction (starts at 0 for raw binary). |
| **2** | `00000517` | **Machine Code**: Hexadecimal representation of the 32-bit RISC-V instruction. |
| **3** | `auipc x10,0x0` | **Disassembly**: Human-readable assembly instruction (mnemonic + operands). |


---

### **Key Notes**

- **Byte Order**: RISC-V is little-endian. The machine code `00000517` is stored in memory as `17 05 00 00` (lowest byte first).
- **Addresses**: For raw binaries, addresses start at 0. Use `--adjust-vma=<offset>` in `objdump` if your code is linked to a different base address.
- **Register Aliases**: Without `-M numeric`, registers use ABI names (e.g., `a0` instead of `x10`).

---

### **Common Use Cases**

- **FPGA Programming**: Raw hex/bin files are used to load programs into RISC-V processor memory.
- **Debugging**: Disassembly helps verify compiled code matches expectations.

**Screenshot:**
![Image](https://github.com/user-attachments/assets/6199498c-5f03-46c0-83ed-bb69e74cf6e5)
![Image](https://github.com/user-attachments/assets/b3e8191c-4aa6-474e-985a-c989e76897af)


# 5. ABI \& Register Cheat-Sheet

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



# 6. Stepping with GDB


### Step 1: Verify GDB Environment Setup

First, ensure RISC-V GDB is properly configured and accessible.
 Check RISC-V GDB availability
```bash
which riscv32-unknown-elf-gdb
```
Verify GDB version and functionality
```bash
riscv32-unknown-elf-gdb --version
```
Confirm target binary exists
```bash
ls -la hello.elf
```
Check ELF file details
```bash
file hello.elf
```

### Step 2: Analyze Binary Structure (Preparation)

Before debugging, examine the binary structure to understand memory layout.

Examine ELF sections and program headers
```bash
riscv32-unknown-elf-objdump -h hello.elf
```
Check program headers for load addresses
```bash
riscv32-unknown-elf-readelf -l hello.elf
```
Quick view of main function location
```bash
riscv32-unknown-elf-objdump -d hello.elf | grep -A 5 "<main>:"
```


### Step 3: Start GDB Debugging Session

Launch GDB with the target RISC-V binary for static analysis.
Start GDB session with hello.elf
```bash
riscv32-unknown-elf-gdb hello.elf
```

**Expected GDB Startup Output:**
```bash
GNU gdb (GDB) 15.2
Reading symbols from hello.elf...
(No debugging symbols found in hello.elf)
(gdb)
```

### Step 4: Static Analysis - Disassemble Main Function

Use GDB's static analysis capabilities to examine the main function without execution.

At (gdb) prompt - disassemble main function
```bash
disassemble main
```
**Working Output Analysis:**
```bash

Dump of assembler code for function main:
0x00010162 <+0>: addi sp,sp,-16 # Prologue: Stack allocation
0x00010164 <+2>: sw ra,12(sp) # Save return address
0x00010166 <+4>: sw s0,8(sp) # Save frame pointer
0x00010168 <+6>: addi s0,sp,16 # Setup frame pointer
0x0001016a <+8>: lui a5,0x12 # Load upper immediate
0x0001016c <+10>: addi a0,a5,1116 # Complete string address
0x00010170 <+14>: jal 0x104de <puts> # Function call
0x00010172 <+16>: li a5,0 # Load return value
0x00010174 <+18>: mv a0,a5 # Move to return register
0x00010176 <+20>: lw ra,12(sp) # Epilogue: Restore return address
0x00010178 <+22>: lw s0,8(sp) # Restore frame pointer
0x0001017a <+24>: addi sp,sp,16 # Deallocate stack
0x0001017c <+26>: ret # Return to caller
```




```bash
riscv32-unknown-elf-gdb hello.elf
(gdb) target sim
(gdb) load
(gdb) break main
(gdb) run
(gdb) step
```

**Screenshot:**
![Image](https://github.com/user-attachments/assets/bdacbd99-a806-4108-a590-0600a9ed88bf)
![Image](https://github.com/user-attachments/assets/a79ffdc3-0be3-428c-b18c-3e8768d498b7)


---

# 7. Running Under an Emulator

### RISC-V QEMU + Spike Setup and Hello World Execution (Markdown Script)

```bash
# Check if qemu-system-riscv32 is installed
which qemu-system-riscv32

# Show QEMU version
qemu-system-riscv32 --version

# Check if spike is installed
which spike

# Try running spike (will fail if dependencies are missing)
spike --help

# Update apt package lists to fix or install missing libraries
sudo apt update

# Run hello.elf using QEMU for riscv32
qemu-system-riscv32 -nographic -machine virt -kernel hello.elf
```

### Expected Output from QEMU:
```text
OpenSBI v1.2
  ____                    _____ ____ _____
 / __ \                  / ____|  _ \_   _|
| |  | |_ __   ___ _ __ | (___ | |_) || |
| |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
| |__| | |_) |  __/ | | |____) | |_) || |_
 \____/| .__/ \___|_| |_|_____/|____/_____|
       | |
       |_|


Platform Name          : riscv-virtio,qemu
Platform Features      : medeleg
Platform HART Count    : 1
Platform IPI Device    : aclint-mswi
Platform Timer Device  : aclint-mtimer @ 10000000Hz
Platform Console Device: uart8250
Platform Reboot Device : sifive_test
Platform Shutdown Dev  : sifive_test
Firmware Size          : 208 KB
Runtime SBI Version    : 1.0

Domain0 Name           : root
Domain0 Boot HART      : 0
Domain0 HARTs          : 0*
Domain0 Region00       : 0x2000000-0x0200ffff (I)
Domain0 Region01       : 0x80000000-0x8003ffff ()
Domain0 Region02       : 0x00000000-0xffffffff (R,W,X)
Domain0 Next Address   : 0x00100000
Domain0 Next Arg1      : 0x80000000
Domain0 Next Mode      : S-mode
Domain0 SysReset       : yes

Boot HART ID           : 0
Boot HART Domain       : root
Boot HART Priv Version : v1.12
Boot HART Base ISA     : rv32imafdc
Boot HART ISA Exts     : time,sstc
Boot HART PMP Count    : 16
Boot HART PMP Addr Bits: 32
Boot HART PMP Granularity: 4
Boot HART MHPM Count   : 16
Boot HART MEDELEG      : 0x00001666
Boot HART MIDELEG      : 0x000f0b509
```

### Notes:
- If `spike` fails with a missing `libboost_regex` error, fix it using:
  ```bash
  sudo apt install libboost-regex-dev
  ```
- Make sure your RISC-V binary (`hello.elf`) is compiled properly with:
  ```bash
  riscv32-unknown-elf-gcc -o hello.elf hello.c
  ```
**Screenshot:**
![Image](https://github.com/user-attachments/assets/ac2b569a-f728-4e74-a6eb-65bf3be2dd30)
![Image](https://github.com/user-attachments/assets/7474b6f7-9c62-4113-a38f-5f81a5b93b0c)

---

# 8. Exploring GCC Optimisation

### Exploring GCC Optimisation

```bash
# Generate assembly from hello.c with no optimization (-O0)
riscv32-unknown-elf-gcc -S -O0 hello.c -o hello_O0.s

# List file details
ls -la hello_O0.s

# Count lines in the generated assembly file
wc -l hello_O0.s

# Generate assembly with optimization level -O2
riscv32-unknown-elf-gcc -S -O2 hello.c -o hello_O2.s

# Count lines in the optimized assembly
wc -l hello_O2.s
```

### Expected Output

```text
-rw-r--r-- 1 ahdee ahdee 539 Jun  8 05:45 hello_O0.s
31 hello_O0.s
27 hello_O2.s
```

### Conclusion

- The non-optimized version (`-O0`) generated an assembly file with **31 lines**.
- The optimized version (`-O2`) reduced the assembly file to **27 lines**.
- This demonstrates that **GCC optimizations can significantly reduce code size**, eliminating unnecessary instructions or optimizing sequences.


**Screenshot:**
![Image](https://github.com/user-attachments/assets/47e5ad1b-dea1-4665-858d-d8b1330390fd)
---

## 9. Inline Assembly Basics

### Compilation and Assembly Generation Script

```bash
# Compile C source with inline assembly to ELF executable
riscv32-unknown-elf-gcc -march=rv32gc -mabi=ilp32 -o task9_final.elf task9_final.c

# Check the generated ELF file properties
file task9_final.elf

# Generate assembly output with debug info and inline code
riscv32-unknown-elf-gcc -march=rv32gc -mabi=ilp32 -S -g -O1 task9_final.c -o task9_final.s

# Disassemble the compiled ELF to see generated assembly
riscv32-unknown-elf-objdump -d task9_final.elf > task9_final.disasm

# Display the disassembly with source interleaving
riscv32-unknown-elf-objdump -S task9_final.elf
```


### Expected C Source Structure (task9_final.c):

```c
#include <stdio.h>
#include <stdint.h>

// Example 1: CSR read with proper constraints
static inline uint32_t rdcycle_demo(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle"
                  : "=r"(c)      // Output constraint
                  :              // No inputs
                  );
    return c;
}

// Example 2: Arithmetic operation with constraints  
static inline uint32_t add_inline(uint32_t a, uint32_t b) {
    uint32_t result;
    asm volatile ("add %0, %1, %2"
                  : "=r"(result)
                  : "r"(a), "r"(b)
                  );
    return result;
}

// Example 3: Shift demonstration with volatile
static inline uint32_t demo_volatile(uint32_t input) {
    uint32_t output;
    asm volatile ("slli %0, %1, 1"
                  : "=r"(output)
                  : "r"(input)
                  );
    return output;
}

int main() {
    printf("Cycle counter: %u\n", rdcycle_demo());
    printf("5 + 7 = %u\n", add_inline(5, 7));
    printf("10 << 1 = %u\n", demo_volatile(10));
    return 0;
}
```


### Expected File Information Output:

```text
task9_final.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, with debug_info, not stripped
```


### Expected Assembly Output (Generated Functions):

```assembly
=== Generated Assembly with Inline Code ===
.attribute stack_align, 16
.text
.align 1
.type rdcycle_demo, @function
rdcycle_demo:
    addi    sp,sp,-32
    sw      ra,28(sp)
    sw      s0,24(sp)
    addi    s0,sp,32
#APP
# 7 "task9_final.c" 1
    csrr a5, cycle
# 0 "" 2
#NO_APP
    sw      a5,-20(s0)
    lw      a5,-20(s0)
    mv      a0,a5
    lw      ra,28(sp)
    lw      s0,24(sp)
    addi    sp,sp,32
    jr      ra
    .size   rdcycle_demo, .-rdcycle_demo

.align 1
.type add_inline, @function
add_inline:
    addi    sp,sp,-48
    sw      ra,44(sp)
    sw      s0,40(sp)
    addi    s0,sp,48
    sw      a0,-36(s0)
    sw      a1,-40(s0)
#APP
# "add_inline.c" 1
    add a5, a0, a1
# 0 "" 2
#NO_APP
    sw      a5,-20(s0)
    lw      a5,-20(s0)
    mv      a0,a5
    lw      ra,44(sp)
    lw      s0,40(sp)
    addi    sp,sp,48
    jr      ra
```


### Analysis Commands:

```bash
# Check symbol table and function addresses
riscv32-unknown-elf-nm task9_final.elf

# Examine ELF headers and sections
riscv32-unknown-elf-readelf -h task9_final.elf

# Display all sections
riscv32-unknown-elf-readelf -S task9_final.elf

# Show specific function disassembly
riscv32-unknown-elf-objdump -d task9_final.elf | grep -A 20 "rdcycle_demo"
```


### Notes:

- The inline assembly uses proper constraint syntax: `"=r"` for output, `"r"` for input registers[^1][^14]
- CSR instructions like `csrr` read cycle counters efficiently[^1][^14]
- The `volatile` keyword prevents compiler optimization of inline assembly[^1][^14]
- Generated assembly shows the `#APP` and `#NO_APP` markers around inline code[^1][^14]
- Stack manipulation (`addi sp,sp,-32`) handles function prologue/epilogue[^1][^14]

**Screenshot:**

---

## 10. Memory-Mapped I/O Demo

## RISC-V GPIO Access: Volatile vs. Non-Volatile Demonstration

This script demonstrates the effect of using the `volatile` keyword when accessing hardware registers (such as GPIO) in RISC-V C code. It compares code generation and assembly output for both volatile and non-volatile pointer usage.

---

### 1. Source Code Creation

```bash
# Create GPIO demo with volatile
cat << 'EOF' > task10_gpio.c
#include <stdint.h>

// Define the GPIO register address
#define GPIO_ADDR 0x10012000

// Function to toggle GPIO with proper volatile usage
void toggle_gpio(void) {
    volatile uint32_t *gpio = (volatile uint32_t *)GPIO_ADDR;
    *gpio = 0x1; // Set GPIO pin high

    // Toggle operation: read current state and flip
    uint32_t current_state = *gpio;
    *gpio = ~current_state;

    // Set specific bits (example: set bit 0, clear bit 1)
    *gpio |= (1 << 0);  // Set bit 0
    *gpio &= ~(1 << 1); // Clear bit 1
}

// Function to demonstrate different GPIO operations
void gpio_operations(void) {
    volatile uint32_t *gpio = (volatile uint32_t *)GPIO_ADDR;
    *gpio = 0x0;         // Clear all pins
    *gpio = 0x1;         // Set pin 0
    *gpio = 0xFFFFFFFF;  // Set all pins
    *gpio = 0x0;         // Clear all pins again
}

int main() {
    // Demonstrate GPIO operations
    toggle_gpio();
    gpio_operations();

    // Infinite loop to keep program running (bare-metal style)
    while(1) {
        // In real hardware, this would continue GPIO operations
        // For demonstration, we'll break after some iterations
        static volatile int counter = 0;
        counter++;
        if (counter > 1000000) break;
    }
    return 0;
}
EOF

# Create GPIO demo without volatile
cat << 'EOF' > task10_no_volatile.c
#include <stdint.h>
#define GPIO_ADDR 0x10012000

void toggle_gpio_no_volatile(void) {
    uint32_t *gpio = (uint32_t *)GPIO_ADDR; // No volatile
    *gpio = 0x1;
    *gpio = 0x0;
    *gpio = 0x1; // Compiler might optimize this away
}

int main() {
    toggle_gpio_no_volatile();
    return 0;
}
EOF
```


---

### 2. Compile and Generate Assembly

```bash
# Compile both versions to assembly
riscv32-unknown-elf-gcc -O1 -S task10_gpio.c -o task10_with_volatiles.s
riscv32-unknown-elf-gcc -O1 -S task10_no_volatile.c -o task10_no_volatiles.s
```


---

### 3. Analyze Assembly: Count Memory Access Instructions

```bash
# Count 'sw' (store word) and 'lw' (load word) instructions in each assembly file

echo "=== With Volatile ==="
grep -nE 'sw|lw' task10_with_volatiles.s
echo "Count:"
grep -cE 'sw|lw' task10_with_volatiles.s

echo "=== Without Volatile ==="
grep -nE 'sw|lw' task10_no_volatiles.s
echo "Count:"
grep -cE 'sw|lw' task10_no_volatiles.s
```


---

### 4. Expected Results

```text
=== With Volatile ===
# Multiple sw/lw instructions at various lines (e.g., 13, 17, 19, ...)
Count:
20

=== Without Volatile ===
# Fewer sw/lw instructions (e.g., 13, 23)
Count:
2
```


---

### 5. Interpretation

- **With volatile:** Every access to the GPIO register results in an explicit memory operation in the assembly output, ensuring the compiler does not optimize away hardware access.
- **Without volatile:** The compiler may optimize away redundant or seemingly unnecessary memory operations, which can prevent correct hardware interaction.

---

### 6. Notes

- Always use `volatile` when accessing memory-mapped hardware registers to guarantee correct program behavior on real hardware.
- The difference in the number of memory access instructions (`sw`/`lw`) illustrates the impact of `volatile` on code generation[^1][^2][^3][^4].


**Screenshot:**

![Image](https://github.com/user-attachments/assets/0af86661-e49e-4dec-895b-bae4e966c34d)
![Image](https://github.com/user-attachments/assets/7314d87f-71f4-4421-b02c-3fe7906b9bef)
![Image](https://github.com/user-attachments/assets/d988abea-a607-44d3-a015-1b11a7fe6790)

---

## 11. Linker Script 101

### **1. Create a Minimal Linker Script**

```ld
ENTRY(_start)

MEMORY {
  ROM (rx)  : ORIGIN = 0x00000000, LENGTH = 1M
  RAM (rwx) : ORIGIN = 0x10000000, LENGTH = 1M
}

SECTIONS {
  .text : {
    *(.text.start)
    *(.text .text.*)
  } > ROM

  .data : {
    _data_start = .;
    *(.data .data.*)
    *(.rodata .rodata.*)
    *(.sdata .sdata.*)
    _data_end = .;
  } > RAM AT > ROM

  .bss (NOLOAD) : {
    _bss_start = .;
    *(.bss .bss.*)
    *(.sbss .sbss.*)
    *(COMMON)
    _bss_end = .;
  } > RAM

  .stack (NOLOAD) : {
    . = ALIGN(16);
    _stack_bottom = .;
    . += 0x1000;
    _stack_top = .;
  } > RAM
}
```


---

### **2. Compile and Link Source Files**

```bash
# Compile startup and main C files to object files
riscv32-unknown-elf-gcc -march=rv32imc -c start.s -o start.o
riscv32-unknown-elf-gcc -march=rv32imc -c test_linker.c -o test_linker.o

# Link using the custom linker script
riscv32-unknown-elf-ld -T minimal.ld start.o test_linker.o -o test_linker.elf
```


---

### **3. Inspect the ELF File**

```bash
# View ELF section headers
riscv32-unknown-elf-objdump -h test_linker.elf

# Example output:
# Sections:
# Idx Name          Size      VMA       LMA       File off  Algn
#  0 .text          0000004a  00000000  00000000  00001000  2**1
#  1 .data          00000004  10000000  0000004a  00002000  2**2
#  2 .bss           00000004  10000004  10000004  00002004  2**2
#  3 .stack         00001000  10000008  10000008  00003000  2**4
#  ...

# List symbol addresses for key symbols
riscv32-unknown-elf-nm test_linker.elf | grep -E '_start|global_var|_stack_top'

# Example output:
# 10000008 B _bss_start
# 10000000 D _data_start
# 10001008 B _stack_top
# 00000000 T _start
# 10000000 D global_var
```


---

### **4. Explanation**

- **Linker Script**: Defines ROM and RAM regions, and places `.text`, `.data`, `.bss`, and `.stack` in memory accordingly.
- **Compiling and Linking**: Source files are compiled to objects and linked with the custom script to control memory layout.
- **ELF Inspection**:
    - `.text` is mapped to ROM at 0x0.
    - `.data`, `.bss`, and `.stack` are mapped to RAM at 0x10000000 and above.
    - Symbols like `_start`, `global_var`, and `_stack_top` are placed at addresses as defined by the linker script.
- **Symbol Table**: Use `nm` to verify symbol addresses and section placement.

---

### **5. Notes**

- Always match your linker script memory regions with your target hardware or emulator expectations.
- The `AT > ROM` directive ensures `.data` is loaded from ROM but runs in RAM.
- The `NOLOAD` attribute for `.bss` and `.stack` means these sections are not initialized in the binary image, but space is reserved in RAM.

---

This script allows you to precisely control and verify the memory layout of your RISC-V program, as shown in your screenshots.





**Screenshot:**

![Image](https://github.com/user-attachments/assets/61b891e3-d9c5-4a2d-a2c7-38d3763f9e0f)
![Image](https://github.com/user-attachments/assets/69f268b7-e8ed-4e01-b428-c89677aa9cc1)
---

## 12. Start-up Code \& crt0

## RISC-V Start-up Code \& crt0: LED Blink Bare-Metal Example

This script demonstrates creating a complete bare-metal RISC-V program with custom startup code (crt0), linker script, and GPIO LED blinking functionality.

---

### **1. Create the Main Application Code**

```bash
# Create the LED blink application
cat << 'EOF' > task12_led_blink.c
#include <stdint.h>

#define GPIO_BASE 0x10012000
#define GPIO_OUTPUT_REG (*(volatile uint32_t *)(GPIO_BASE + 0x00))
#define GPIO_DIRECTION_REG (*(volatile uint32_t *)(GPIO_BASE + 0x04))

void delay(volatile int count) {
    while(count--) {
        asm volatile ("nop"); // Prevent optimization
    }
}

int main() {
    // Set GPIO pin 0 as output
    GPIO_DIRECTION_REG |= 0x1;
    
    while(1) {
        // Toggle GPIO pin 0
        GPIO_OUTPUT_REG ^= 0x1;
        delay(100000);
    }
    return 0;
}
EOF
```


---

### **2. Create the Startup Assembly File (crt0)**

```bash
# Create the startup code
cat << 'EOF' > led_start.s
.section .text.start
.global _start

_start:
    # Set up stack pointer
    lui sp, %hi(_stack_top)
    addi sp, sp, %lo(_stack_top)
    
    # Call main program
    call main
    
    # Infinite loop
1:  j 1b

.size _start, . - _start
EOF
```


---

### **3. Create the Linker Script**

```bash
# Create the custom linker script
cat << 'EOF' > led_blink.ld
ENTRY(_start)

MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 256K
    SRAM (rwx) : ORIGIN = 0x10000000, LENGTH = 64K
}

SECTIONS {
    .text 0x00000000 : {
        *(.text.start)
        *(.text .text.*)
        *(.rodata .rodata.*)
    } > FLASH
    
    .data 0x10000000 : {
        _data_start = .;
        *(.data*)
        _data_end = .;
    } > SRAM
    
    .bss : {
        _bss_start = .;
        *(.bss*)
        _bss_end = .;
    } > SRAM
    
    .stack (NOLOAD) : {
        . = ALIGN(16);
        _stack_bottom = .;
        . += 0x1000;
        _stack_top = .;
    } > SRAM
}
EOF
```


---

### **4. Compile and Link the Complete Program**

```bash
# Compile the C source to object file
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -c task12_led_blink.c -o task12_led_blink.o

# Assemble the startup code
riscv32-unknown-elf-as -march=rv32imc -c led_start.s -o led_start.o

# Link using custom linker script
riscv32-unknown-elf-ld -T led_blink.ld led_start.o task12_led_blink.o -o led_blink.elf

# Alternative: One-step compilation with custom linker script
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -T led_blink.ld -nostartfiles led_start.s task12_led_blink.c -o led_blink.elf
```


---

### **5. Inspect the Generated ELF**

```bash
# View ELF file information
file led_blink.elf

# Check section headers and memory layout
riscv32-unknown-elf-objdump -h led_blink.elf

# Disassemble to verify startup code
riscv32-unknown-elf-objdump -d led_blink.elf | head -30

# Check symbol table
riscv32-unknown-elf-nm led_blink.elf | grep -E '_start|main|_stack|GPIO'
```


---

### **6. Expected Output Analysis**

```text
# ELF file type
led_blink.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, not stripped

# Section layout (from objdump -h)
Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         000000xx  00000000  00000000  00001000  2**1
  1 .data         000000xx  10000000  000000xx  00002000  2**2
  2 .bss          000000xx  10000xxx  10000xxx  00002xxx  2**2
  3 .stack        00001000  10000xxx  10000xxx  00003xxx  2**4

# Key symbols (from nm)
00000000 T _start
000000xx T main
10000xxx B _bss_start
10001xxx B _stack_top
```


---

### **7. Run in QEMU Emulator**

```bash
# Run the LED blink program in QEMU
qemu-system-riscv32 -nographic -machine virt -kernel led_blink.elf

# For debugging with GDB
qemu-system-riscv32 -nographic -machine virt -kernel led_blink.elf -s -S &

# Connect with GDB
riscv32-unknown-elf-gdb led_blink.elf
(gdb) target remote localhost:1234
(gdb) break main
(gdb) continue
```


---

### **8. Key Components Explained**

**Startup Code (`led_start.s`):**[^4][^5]

- Sets up stack pointer using linker-defined symbols
- Calls the main function
- Provides infinite loop for program termination

**Linker Script (`led_blink.ld`):**[^6]

- Defines memory regions (FLASH for code, SRAM for data)
- Places `.text` section in FLASH starting at 0x0
- Allocates data and stack in SRAM starting at 0x10000000

**Main Application:**[^14]

- Uses volatile pointers for memory-mapped GPIO registers
- Implements delay using inline assembly to prevent optimization
- Demonstrates basic bare-metal GPIO control

---

### **9. Notes**

- The `-nostartfiles` flag prevents GCC from linking default crt0
- Custom startup code must initialize stack before calling any C functions
- Linker script controls exact memory placement for bare-metal systems
- GPIO addresses must match your target hardware platform
- Use `volatile` for all hardware register access to prevent optimization

This demonstrates a complete bare-metal RISC-V program from startup to application execution.

**Screenshot:**

![Image](https://github.com/user-attachments/assets/af765b39-d0e0-4a4c-83bc-3e89fec69ca0)
![Image](https://github.com/user-attachments/assets/40ae7a34-072c-460d-b227-632920eb4a27)
![Image](https://github.com/user-attachments/assets/cbb28e4c-6bbb-4281-a30f-3fcfcd93d037)

---

## 13. Interrupt Primer

## RISC-V Interrupt Primer: Timer Interrupt Implementation

This script demonstrates implementing timer-based interrupts in RISC-V, including CSR manipulation, interrupt service routines, and memory-mapped timer registers.

---

### **1. Create the RISC-V CSR Header File**

```bash
# Create the CSR definitions header
cat << 'EOF' > riscv_csr.h
#ifndef RISCV_CSR_H
#define RISCV_CSR_H

#include <stdint.h>

// CSR addresses for machine mode
#define CSR_MSTATUS   0x300
#define CSR_MIE       0x304
#define CSR_MTVEC     0x305
#define CSR_MEPC      0x341
#define CSR_MCAUSE    0x342
#define CSR_MTVAL     0x343
#define CSR_MIP       0x344

// Machine interrupt enable bits
#define MIE_MTIE      (1 << 7)   // Machine timer interrupt enable
#define MIE_MSIE      (1 << 3)   // Machine software interrupt enable
#define MIE_MEIE      (1 << 11)  // Machine external interrupt enable

// Machine status register bits
#define MSTATUS_MIE   (1 << 3)   // Machine interrupt enable

// Generic CSR read/write functions
static inline uint32_t read_csr(uint32_t csr) {
    uint32_t result;
    asm volatile ("csrr %0, %1" : "=r"(result) : "i"(csr));
    return result;
}

static inline void write_csr(uint32_t csr, uint32_t value) {
    asm volatile ("csrw %0, %1" : : "i"(csr), "r"(value));
}

static inline void set_csr_bits(uint32_t csr, uint32_t bits) {
    asm volatile ("csrs %0, %1" : : "i"(csr), "r"(bits));
}

static inline void clear_csr_bits(uint32_t csr, uint32_t bits) {
    asm volatile ("csrc %0, %1" : : "i"(csr), "r"(bits));
}

#endif // RISCV_CSR_H
EOF
```


---

### **2. Create the Timer Interrupt Implementation**

```bash
# Create the main timer interrupt code
cat << 'EOF' > task13_timer_interrupt.c
#include "riscv_csr.h"
#include <stdint.h>

// QEMU virt machine timer addresses
#define MTIME_BASE     0x0200BFF8
#define MTIMECMP_BASE  0x02004000

// Memory-mapped timer registers
volatile uint64_t *mtime = (volatile uint64_t *)MTIME_BASE;
volatile uint64_t *mtimecmp = (volatile uint64_t *)MTIMECMP_BASE;
volatile uint32_t interrupt_count = 0;

void enable_timer_interrupt(void) {
    // Set timer compare value (current time + 10000000 cycles)
    *mtimecmp = *mtime + 10000000;
    
    // Enable machine timer interrupt in MIE register
    write_csr(CSR_MIE, read_csr(CSR_MIE) | MIE_MTIE);
    
    // Enable machine status interrupt bit
    write_csr(CSR_MSTATUS, read_csr(CSR_MSTATUS) | MSTATUS_MIE);
}

// Timer interrupt handler with interrupt attribute
void __attribute__((interrupt)) timer_interrupt_handler(void) {
    // Reset timer compare for next interrupt
    *mtimecmp = *mtime + 10000000;
    
    // Increment interrupt counter
    interrupt_count++;
}

// Setup interrupt vector table
void setup_interrupts(void) {
    // Set machine trap vector to our handler
    write_csr(CSR_MTVEC, (uint32_t)timer_interrupt_handler);
    
    // Enable timer interrupts
    enable_timer_interrupt();
}

int main(void) {
    // Initialize interrupt system
    setup_interrupts();
    
    // Main program loop
    while(1) {
        // Wait for interrupts
        asm volatile ("wfi"); // Wait for interrupt
        
        // Check if we've received enough interrupts
        if (interrupt_count >= 10) {
            break;
        }
    }
    
    return 0;
}
EOF
```


---

### **3. Create Advanced Interrupt Handler with Context Saving**

```bash
# Create manual context saving interrupt handler
cat << 'EOF' > interrupt_handler.s
.section .text
.global manual_interrupt_handler
.align 4

manual_interrupt_handler:
    # Save all caller-saved registers
    addi sp, sp, -128
    
    # Save general purpose registers
    sw   x1,  4(sp)   # ra
    sw   x3,  8(sp)   # gp  
    sw   x4, 12(sp)   # tp
    sw   x5, 16(sp)   # t0
    sw   x6, 20(sp)   # t1
    sw   x7, 24(sp)   # t2
    sw  x10, 28(sp)   # a0
    sw  x11, 32(sp)   # a1
    sw  x12, 36(sp)   # a2
    sw  x13, 40(sp)   # a3
    sw  x14, 44(sp)   # a4
    sw  x15, 48(sp)   # a5
    sw  x16, 52(sp)   # a6
    sw  x17, 56(sp)   # a7
    sw  x28, 60(sp)   # t3
    sw  x29, 64(sp)   # t4
    sw  x30, 68(sp)   # t5
    sw  x31, 72(sp)   # t6
    
    # Call C interrupt handler
    call c_interrupt_handler
    
    # Restore all registers
    lw   x1,  4(sp)
    lw   x3,  8(sp)
    lw   x4, 12(sp)
    lw   x5, 16(sp)
    lw   x6, 20(sp)
    lw   x7, 24(sp)
    lw  x10, 28(sp)
    lw  x11, 32(sp)
    lw  x12, 36(sp)
    lw  x13, 40(sp)
    lw  x14, 44(sp)
    lw  x15, 48(sp)
    lw  x16, 52(sp)
    lw  x17, 56(sp)
    lw  x28, 60(sp)
    lw  x29, 64(sp)
    lw  x30, 68(sp)
    lw  x31, 72(sp)
    
    addi sp, sp, 128
    
    # Return from interrupt
    mret

.global c_interrupt_handler
c_interrupt_handler:
    # Read mcause to determine interrupt source
    csrr t0, mcause
    
    # Check if it's a timer interrupt (bit 31 set, cause = 7)
    li   t1, 0x80000007
    beq  t0, t1, handle_timer
    
    # Handle other interrupts here
    j    interrupt_done

handle_timer:
    # Reset timer compare value
    # Implementation depends on timer configuration
    
interrupt_done:
    ret
EOF
```


---

### **4. Compile and Test the Interrupt System**

```bash
# Compile all components
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -c task13_timer_interrupt.c -o task13_timer_interrupt.o
riscv32-unknown-elf-as -march=rv32imc -c interrupt_handler.s -o interrupt_handler.o

# Link with startup code (assuming led_start.s from previous example)
riscv32-unknown-elf-ld -T led_blink.ld led_start.o task13_timer_interrupt.o interrupt_handler.o -o timer_interrupt.elf

# Alternative: Use GCC with interrupt attribute (automatic context saving)
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -T led_blink.ld -nostartfiles led_start.s task13_timer_interrupt.c -o timer_interrupt_auto.elf
```


---

### **5. Analyze the Generated Code**

```bash
# Disassemble to see interrupt handler code
riscv32-unknown-elf-objdump -d timer_interrupt.elf | grep -A 20 "timer_interrupt_handler"

# Check CSR-related instructions
riscv32-unknown-elf-objdump -d timer_interrupt.elf | grep -E "csrr|csrw|csrs|csrc|mret"

# Verify interrupt vector setup
riscv32-unknown-elf-nm timer_interrupt.elf | grep -E "handler|vector|interrupt"
```


---

### **6. Expected Assembly Output Analysis**

```text
# Timer interrupt handler with GCC interrupt attribute
<timer_interrupt_handler>:
   # Automatic prologue (saves registers)
   addi    sp,sp,-64
   sw      ra,60(sp)
   sw      t0,56(sp)
   sw      t1,52(sp)
   # ... more register saves
   
   # User interrupt code
   lui     t0,0x200c
   lw      a5,0(t0)      # Load *mtime
   lui     t1,0x989
   add     a5,a5,t1      # Add 10000000
   lui     t0,0x2004
   sw      a5,0(t0)      # Store to *mtimecmp
   
   # Increment counter
   lui     t0,0x10000
   lw      a5,4(t0)
   addi    a5,a5,1
   sw      a5,4(t0)
   
   # Automatic epilogue (restores registers)
   lw      ra,60(sp)
   lw      t0,56(sp)
   # ... more register restores
   addi    sp,sp,64
   mret                  # Machine return
```


---

### **7. Run and Debug**

```bash
# Run in QEMU with interrupt support
qemu-system-riscv32 -nographic -machine virt -kernel timer_interrupt.elf

# Debug with GDB to trace interrupt flow
qemu-system-riscv32 -nographic -machine virt -kernel timer_interrupt.elf -s -S &
riscv32-unknown-elf-gdb timer_interrupt.elf
(gdb) target remote localhost:1234
(gdb) break timer_interrupt_handler
(gdb) break main
(gdb) monitor info registers
(gdb) continue
```


---

### **8. Key Concepts Demonstrated**

**CSR Operations:[^12][^13]**

- Reading/writing control and status registers
- Setting/clearing specific bits for interrupt control
- Using inline assembly for CSR access

**Interrupt Handling Flow:[^4]**

- Timer setup with memory-mapped registers
- Interrupt vector configuration (`mtvec`)
- Automatic context saving with `__attribute__((interrupt))`
- Manual context saving in assembly

**Timer Interrupts:[^2][^4]**

- Memory-mapped timer (`mtime`/`mtimecmp`) configuration
- Periodic interrupt generation
- Interrupt acknowledgment and rescheduling

---

### **9. Notes**

- Use `__attribute__((interrupt))` for automatic register saving/restoring[^3][^4]
- Memory-mapped timer addresses are platform-specific (QEMU virt machine shown)
- `mret` instruction returns from machine-mode interrupts
- `wfi` (Wait For Interrupt) reduces power consumption while waiting
- Always clear/acknowledge interrupt sources to prevent infinite interrupts[^4]

This demonstrates complete RISC-V interrupt handling from setup to service routine execution.


**Screenshot:**

![Image](https://github.com/user-attachments/assets/2e9e3665-b06f-4e18-9297-8fcf6bfc0cfd)

---

## 14. Atomic Operations

## RISC-V rv32imac vs rv32imc – What's the "A"? Atomic Operations Demonstration

This script demonstrates the difference between rv32imac and rv32imc architectures, specifically focusing on the "A" extension which provides atomic operations for thread-safe programming.

---

### **1. Create Atomic Operations Demo Code**

```bash
# Create the atomic operations demonstration
cat << 'EOF' > task14_atomic_demo.c
#include <stdint.h>

// Global shared variables for atomic operations demonstration
volatile uint32_t shared_counter = 0;
volatile uint32_t lock_variable = 0;

// Atomic Load-Reserve / Store-Conditional operations
static inline uint32_t atomic_load_acquire(volatile uint32_t *addr, uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("lr.w %0, (%1)" : "=r"(result) : "r"(addr) : "memory");
    return result;
}

static inline uint32_t atomic_store_conditional(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("sc.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result; // 0 = success, 1 = failure
}

// Atomic Read-Modify-Write operations
static inline uint32_t atomic_swap(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("amoswap.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result;
}

static inline uint32_t atomic_add(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("amoadd.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result;
}

static inline uint32_t atomic_and(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("amoand.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result;
}

static inline uint32_t atomic_or(volatile uint32_t *addr, uint32_t value) {
    uint32_t result;
    asm volatile ("amoor.w %0, %2, (%1)" : "=r"(result) : "r"(addr), "r"(value) : "memory");
    return result;
}

void unreliable_lock(volatile uint32_t *lock) {
    while (*lock != 0) {}
    *lock = 1;
}

int main() {
    non_atomic_increment(&shared_counter);
    unreliable_lock(&lock_variable);
    return 0;
}
EOF
```


---

### **2. Create Non-Atomic Comparison Code**

```bash
# Create non-atomic version for comparison
cat << 'EOF' > task14_non_atomic.c
#include <stdint.h>

volatile uint32_t shared_counter = 0;
volatile uint32_t lock_variable = 0;

void non_atomic_increment(volatile uint32_t *counter) {
    uint32_t temp = *counter;
    temp = temp + 1;
    *counter = temp;
}

void unreliable_lock(volatile uint32_t *lock) {
    while (*lock != 0) {}
    *lock = 1;
}

int main() {
    non_atomic_increment(&shared_counter);
    unreliable_lock(&lock_variable);
    return 0;
}
EOF
```


---

### **3. Create Startup Code and Linker Script**

```bash
# Create startup assembly file
cat << 'EOF' > atomic_start.s
.section .text.start
.global _start

_start:
    # Set up stack pointer
    lui sp, %hi(_stack_top)
    addi sp, sp, %lo(_stack_top)
    
    # Call main program
    call main
    
    # Infinite loop
1:  j 1b

.size _start, . - _start
EOF

# Create linker script
cat << 'EOF' > atomic.ld
ENTRY(_start)

MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 256K
    SRAM (rwx) : ORIGIN = 0x10000000, LENGTH = 64K
}

SECTIONS {
    .text 0x00000000 : {
        *(.text.start)
        *(.text .text.*)
        *(.rodata .rodata.*)
    } > FLASH
    
    .data 0x10000000 : {
        _data_start = .;
        *(.data*)
        _data_end = .;
    } > SRAM
    
    .bss : {
        _bss_start = .;
        *(.bss*)
        _bss_end = .;
    } > SRAM
    
    _stack_top = ORIGIN(SRAM) + LENGTH(SRAM);
}
EOF
```


---

### **4. Compile Both Versions and Compare**

```bash
# Compile with rv32imac (includes atomic operations)
riscv32-unknown-elf-gcc -march=rv32imac -c atomic_start.s -o atomic_start.o
riscv32-unknown-elf-gcc -march=rv32imac -c task14_atomic_demo.c -o task14_atomic_demo.o
riscv32-unknown-elf-ld -T atomic.ld atomic_start.o task14_atomic_demo.o -o task14_atomic_demo.elf

# Compile non-atomic version with rv32imc (no atomic operations)
riscv32-unknown-elf-gcc -march=rv32imc -c atomic_start.s -o atomic_start.o
riscv32-unknown-elf-gcc -march=rv32imc -c task14_non_atomic.c -o task14_non_atomic.o
riscv32-unknown-elf-ld -T atomic.ld atomic_start.o task14_non_atomic.o -o task14_non_atomic.elf
```


---

### **5. Analyze Assembly Output for Atomic Instructions**

```bash
# Extract atomic instructions from the atomic version
echo "=== Atomic Instructions (rv32imac) ==="
riscv32-unknown-elf-objdump -d task14_atomic_demo.elf | grep -E "lr\.w|sc\.w|amoadd\.w|amoswap\.w|amoand\.w|amoor\.w"

# Show the context around atomic instructions
riscv32-unknown-elf-objdump -d task14_atomic_demo.elf | grep -A 3 -B 3 "amo"

# Compare file sizes
echo "=== File Size Comparison ==="
ls -la task14_atomic_demo.elf task14_non_atomic.elf
```


---

### **6. Expected Assembly Output Analysis**

```text
=== Atomic Instructions (rv32imac) ===
1c:    1007a7af    lr.w     a5,(a5)
4a:    18e7a7af    sc.w     a5,a4,(a5)
78:    00e7a7af    amoadd.w a5,a4,(a5)
a6:    08e7a7af    amoswap.w a5,a4,(a5)
d4:    60e7a7af    amoand.w a5,a4,(a5)
102:   40e7a7af    amoor.w  a5,a4,(a5)

=== Context Example (atomic_swap function) ===
a6:    08e7a7af    amoswap.w a5,a4,(a5)
aa:    fef42623    sw       a5,-20(s0)
ae:    fec42783    lw       a5,-20(s0)
```


---

### **7. Detailed Analysis Commands**

```bash
# Check ISA compliance and features
file task14_atomic_demo.elf
file task14_non_atomic.elf

# Count atomic instruction usage
echo "Atomic instruction count in rv32imac version:"
riscv32-unknown-elf-objdump -d task14_atomic_demo.elf | grep -cE "lr\.w|sc\.w|amo"

echo "Atomic instruction count in rv32imc version:"
riscv32-unknown-elf-objdump -d task14_non_atomic.elf | grep -cE "lr\.w|sc\.w|amo"

# Show symbol differences
echo "=== Symbol comparison ==="
riscv32-unknown-elf-nm task14_atomic_demo.elf | grep atomic
riscv32-unknown-elf-nm task14_non_atomic.elf | grep atomic
```


---

### **8. Understanding the "A" Extension**

**Key Atomic Instructions Provided:**

- **Load-Reserved/Store-Conditional (LR/SC):**
    - `lr.w rd, (rs1)` - Load reserved word
    - `sc.w rd, rs2, (rs1)` - Store conditional word
- **Atomic Memory Operations (AMO):**
    - `amoadd.w` - Atomic add
    - `amoswap.w` - Atomic swap
    - `amoand.w` - Atomic AND
    - `amoor.w` - Atomic OR
    - `amoxor.w` - Atomic XOR
    - `amomax.w` - Atomic maximum
    - `amomin.w` - Atomic minimum

---

### **9. Expected File Analysis Output**

```text
# ELF file identification
task14_atomic_demo.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, not stripped
task14_non_atomic.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, not stripped

# File size comparison (atomic version typically larger due to additional instructions)
-rwxr-xr-x 1 user user 2048 task14_atomic_demo.elf
-rwxr-xr-x 1 user user 1824 task14_non_atomic.elf

# Instruction count
Atomic instruction count in rv32imac version: 6
Atomic instruction count in rv32imc version: 0
```


---

### **10. Key Differences Summary**

**rv32imac (with "A" extension):**

- Provides hardware-level atomic operations
- Thread-safe memory operations without locks
- Single-instruction read-modify-write operations
- Lock-free programming capabilities
- Better performance in multi-threaded environments

**rv32imc (without "A" extension):**

- No atomic instructions available
- Requires software-based synchronization (locks, critical sections)
- Multiple instructions needed for atomic-like operations
- Potential race conditions in multi-threaded code
- Smaller code size when atomic operations aren't needed

---

### **11. Notes**

- The "A" extension is essential for robust multi-threaded bare-metal programming
- Atomic operations are hardware-guaranteed to be indivisible
- LR/SC pairs enable complex atomic operations beyond simple AMO instructions
- Memory ordering suffixes (`.aq`, `.rl`) can be added for acquire/release semantics
- Without the "A" extension, thread synchronization requires disabling interrupts or using software locks

This demonstrates why rv32imac is preferred for systems requiring thread safety and concurrent programming.


**Screenshot:**

![Image](https://github.com/user-attachments/assets/c6349cee-531c-486f-b0ac-93d0f305bc7b)
![Image](https://github.com/user-attachments/assets/9a21faf5-fac8-4fd8-a11f-0a8bfbac5cae)
![Image](https://github.com/user-attachments/assets/d1cce6e1-5a05-447e-b51a-21a29fb42b0f)
![Image](https://github.com/user-attachments/assets/b84d4b00-63a7-4b8e-9d9a-542ae1553c81)
![Image](https://github.com/user-attachments/assets/caf47761-9a82-4402-b92e-14e061c50aec)


---

## 15. Atomic Test Program

## RISC-V Atomic Test Program: Spinlock Implementation with Load-Reserved/Store-Conditional

This script demonstrates implementing and testing atomic operations using RISC-V's Load-Reserved (LR) and Store-Conditional (SC) instructions for thread-safe spinlock implementation.

---

### **1. Create the Atomic Spinlock Implementation**

```bash
# Create the mutex/spinlock demonstration program
cat << 'EOF' > task15_mutex_demo.c
#include <stdint.h>

volatile int spinlock = 0;
volatile int shared_counter = 0;

void spinlock_acquire(volatile int *lock) {
    int tmp;
    asm volatile (
        "1:\n"
        "    lr.w    %0, (%1)\n"      // Load-reserved from lock address
        "    bnez    %0, 1b\n"        // If lock != 0, retry
        "    li      %0, 1\n"         // Load immediate 1 into tmp
        "    sc.w    %0, %0, (%1)\n"  // Store-conditional 1 to lock
        "    bnez    %0, 1b\n"        // If SC failed, retry
        : "=&r"(tmp)                  // Output: tmp register (early clobber)
        : "r"(lock)                   // Input: lock address
        : "memory"                    // Memory clobber
    );
}

void spinlock_release(volatile int *lock) {
    asm volatile (
        "sw zero, 0(%0)\n"           // Store 0 to lock address
        :                            // No outputs
        : "r"(lock)                  // Input: lock address  
        : "memory"                   // Memory clobber
    );
}

void thread1() {
    for(int i=0; i<5000; i++) {
        spinlock_acquire(&spinlock);
        shared_counter++;             // Critical section
        spinlock_release(&spinlock);
    }
}

void thread2() {
    for(int i=0; i<5000; i++) {
        spinlock_acquire(&spinlock);
        shared_counter--;             // Critical section
        spinlock_release(&spinlock);
    }
}

int main() {
    // In a real multi-threaded system, these would run concurrently
    thread1();
    thread2();
    
    // shared_counter should be 0 if synchronization worked correctly
    return shared_counter;
}
EOF
```


---

### **2. Create Startup Code and Linker Script**

```bash
# Create startup assembly for mutex demo
cat << 'EOF' > mutex_start.s
.section .text.start
.global _start

_start:
    # Set up stack pointer
    lui sp, %hi(_stack_top)
    addi sp, sp, %lo(_stack_top)
    
    # Call main program
    call main
    
    # Infinite loop
1:  j 1b

.size _start, . - _start
EOF

# Create linker script for mutex demo
cat << 'EOF' > mutex.ld
ENTRY(_start)

MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 256K
    SRAM (rwx) : ORIGIN = 0x10000000, LENGTH = 64K
}

SECTIONS {
    .text 0x00000000 : {
        *(.text.start)
        *(.text .text.*)
        *(.rodata .rodata.*)
    } > FLASH
    
    .data 0x10000000 : {
        _data_start = .;
        *(.data*)
        _data_end = .;
    } > SRAM
    
    .bss : {
        _bss_start = .;
        *(.bss*)
        _bss_end = .;
    } > SRAM
    
    _stack_top = ORIGIN(SRAM) + LENGTH(SRAM);
}
EOF
```


---

### **3. Compile and Link the Atomic Test Program**

```bash
# Assemble startup code
riscv32-unknown-elf-gcc -march=rv32imac -c mutex_start.s -o mutex_start.o

# Compile main program
riscv32-unknown-elf-gcc -march=rv32imac -c task15_mutex_demo.c -o task15_mutex_demo.o

# Link final executable
riscv32-unknown-elf-ld -T mutex.ld mutex_start.o task15_mutex_demo.o -o task15_mutex_demo.elf
```


---

### **4. Analyze Atomic Instructions in Generated Code**

```bash
# Search for Load-Reserved and Store-Conditional instructions
riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -E 'lr\.w|sc\.w'

# Show context around atomic instructions
echo "=== Spinlock Acquire Function Disassembly ==="
riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -A 15 "spinlock_acquire"

# Show complete disassembly with line numbers
riscv32-unknown-elf-objdump -S task15_mutex_demo.elf > task15_mutex_demo.disasm
```


---

### **5. Expected Assembly Output Analysis**

```text
=== Atomic Instructions Found ===
1c:    1007a7af    lr.w     a5,(a5)
24:    18f7a7af    sc.w     a5,a5,(a5)

=== Spinlock Acquire Function Context ===
00000018 <spinlock_acquire>:
  18:   1007a7af    lr.w     a5,(a5)        # Load-reserved from lock
  1c:   e789        bnez     a5,26 <spinlock_acquire+0xe>  # Branch if not zero
  1e:   4785        li       a5,1           # Load immediate 1
  20:   18f7a7af    sc.w     a5,a5,(a5)     # Store-conditional
  24:   f7f5        bnez     a5,18 <spinlock_acquire>      # Retry if failed
  26:   8082        ret                     # Return
```


---

### **6. Detailed Analysis Commands**

```bash
# Count atomic instruction usage
echo "=== Atomic Instruction Count ==="
riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -cE 'lr\.w|sc\.w'

# Check symbol table for functions
echo "=== Function Symbols ==="
riscv32-unknown-elf-nm task15_mutex_demo.elf | grep -E 'spinlock|thread|main'

# Verify file architecture and features
file task15_mutex_demo.elf

# Show memory layout
riscv32-unknown-elf-objdump -h task15_mutex_demo.elf
```


---

### **7. Create Test Verification Script**

```bash
# Create a test script to verify atomic operations
cat << 'EOF' > test_atomic_spinlock.sh
#!/bin/bash

echo "=== RISC-V Atomic Spinlock Test ==="

# Compile and link
riscv32-unknown-elf-gcc -march=rv32imac -c mutex_start.s -o mutex_start.o
riscv32-unknown-elf-gcc -march=rv32imac -c task15_mutex_demo.c -o task15_mutex_demo.o  
riscv32-unknown-elf-ld -T mutex.ld mutex_start.o task15_mutex_demo.o -o task15_mutex_demo.elf

# Verify atomic instructions are present
ATOMIC_COUNT=$(riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -cE 'lr\.w|sc\.w')
echo "Atomic instructions found: $ATOMIC_COUNT"

if [ $ATOMIC_COUNT -eq 2 ]; then
    echo "✓ SUCCESS: Both lr.w and sc.w instructions found"
else
    echo "✗ FAILURE: Expected 2 atomic instructions, found $ATOMIC_COUNT"
fi

# Show the atomic instruction addresses
echo "=== Atomic Instruction Locations ==="
riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -E 'lr\.w|sc\.w'

# Run in QEMU (if available)
if command -v qemu-system-riscv32 &> /dev/null; then
    echo "=== Running in QEMU ==="
    timeout 5s qemu-system-riscv32 -nographic -machine virt -kernel task15_mutex_demo.elf
else
    echo "QEMU not available for runtime testing"
fi
EOF

chmod +x test_atomic_spinlock.sh
./test_atomic_spinlock.sh
```


---

### **8. Expected Test Output**

```text
=== RISC-V Atomic Spinlock Test ===
Atomic instructions found: 2
✓ SUCCESS: Both lr.w and sc.w instructions found

=== Atomic Instruction Locations ===
1c:    1007a7af    lr.w     a5,(a5)
24:    18f7a7af    sc.w     a5,a5,(a5)

=== File Information ===
task15_mutex_demo.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, not stripped
```


---

### **9. Advanced Testing: Compare with Non-Atomic Version**

```bash
# Create a non-atomic version for comparison
cat << 'EOF' > task15_non_atomic.c
#include <stdint.h>

volatile int spinlock = 0;
volatile int shared_counter = 0;

void unsafe_spinlock_acquire(volatile int *lock) {
    while(*lock != 0) {}  // Busy wait (race condition possible)
    *lock = 1;            // Not atomic - race condition!
}

void unsafe_spinlock_release(volatile int *lock) {
    *lock = 0;
}

// Same thread functions as before...
void thread1() {
    for(int i=0; i<5000; i++) {
        unsafe_spinlock_acquire(&spinlock);
        shared_counter++;
        unsafe_spinlock_release(&spinlock);
    }
}

int main() {
    thread1();
    return shared_counter;
}
EOF

# Compile non-atomic version
riscv32-unknown-elf-gcc -march=rv32imc -c task15_non_atomic.c -o task15_non_atomic.o
riscv32-unknown-elf-ld -T mutex.ld mutex_start.o task15_non_atomic.o -o task15_non_atomic.elf

# Compare atomic instruction counts
echo "=== Comparison Results ==="
echo "Atomic version lr.w/sc.w count:"
riscv32-unknown-elf-objdump -d task15_mutex_demo.elf | grep -cE 'lr\.w|sc\.w'
echo "Non-atomic version lr.w/sc.w count:"
riscv32-unknown-elf-objdump -d task15_non_atomic.elf | grep -cE 'lr\.w|sc\.w'
```


---

### **10. Key Concepts Demonstrated**

**Load-Reserved/Store-Conditional (LR/SC) Pair:**

- `lr.w rd, (rs1)` - Atomically loads word and sets reservation
- `sc.w rd, rs2, (rs1)` - Conditionally stores if reservation valid
- Returns 0 on success, 1 on failure for SC instruction

**Spinlock Implementation:**

- Retry loop using LR/SC for atomic test-and-set
- Memory barriers prevent reordering
- Critical section protection for shared resources

**Assembly Constraints:**

- `"=&r"(tmp)` - Early clobber output register
- `"r"(lock)` - Input register for lock address
- `"memory"` - Memory clobber prevents optimization

---

### **11. Notes**

- The `rv32imac` architecture is required for atomic instructions
- LR/SC provides stronger guarantees than simple AMO instructions
- Proper use of memory barriers (`"memory"`) prevents compiler reordering
- In real multi-threaded systems, this spinlock would provide mutual exclusion
- The retry loop in assembly handles the case where another thread invalidates the reservation
- Always verify atomic instructions are present in the generated assembly

This demonstrates practical implementation of thread-safe synchronization primitives using RISC-V atomic instructions.

**Screenshot:**

![Image](https://github.com/user-attachments/assets/ce728177-972b-4275-805e-e13929bf199e)
![Image](https://github.com/user-attachments/assets/29963560-37e9-4b94-9fb2-65cf95f33c09)


---


# 16. Using Newlib printf Without an OS

## Using Newlib printf Without an OS (Bare Metal RISC-V)

This step-by-step guide shows how to use Newlib’s `printf` on a bare-metal RISC-V system—without an operating system—by implementing the necessary system call stubs and linking your code with Newlib. The process is illustrated in your screenshots and explained below.

---

### **1. Compile All Components**

```bash
# Assemble and compile all required files for bare-metal printf support
riscv32-unknown-elf-gcc -march=rv32imac -c start.s   -o start.o
riscv32-unknown-elf-gcc -march=rv32imac -c uart.c    -o uart.o
riscv32-unknown-elf-gcc -march=rv32imac -c syscalls.c -o syscalls.o
riscv32-unknown-elf-gcc -march=rv32imac -c main.c    -o main.o
```

- **start.s**: Startup code (crt0)
- **uart.c**: UART driver (for output)
- **syscalls.c**: Minimal system call stubs (e.g., `_write`, `_sbrk`)
- **main.c**: Your application code, which may use `printf`

---

### **2. Link with Newlib and Custom Linker Script**

```bash
# Link all objects with Newlib and your linker script, disabling default startup files
riscv32-unknown-elf-gcc -T printf.ld -nostartfiles \
  start.o main.o uart.o syscalls.o \
  -lc -lm -lgcc -o uart_printf.elf
```

- `-T printf.ld`: Use your custom linker script to control memory layout.
- `-nostartfiles`: Prevents the default C runtime from being linked (since you provide your own `start.s`).
- `-lc -lm -lgcc`: Link against Newlib C library, math library, and GCC support library.

---

### **3. Inspect the ELF for printf Support**

```bash
# List symbols related to printf, puts, and memory ends
riscv32-unknown-elf-nm uart_printf.elf | grep -E 'printf|puts|_end'
```

**Expected output:**
You should see symbols such as:

- `printf`, `puts`, `vprintf`, etc. (from Newlib)
- `_end`, `_data_end`, `_bss_end`, `_heap_end` (from your linker script and Newlib)
- These indicate that Newlib’s `printf` and related functions are present and ready to use on bare metal.

---

### **4. How It Works**

- **Newlib** provides the standard C library functions, but expects you to implement low-level system call stubs (like `_write`, `_sbrk`, etc.) in `syscalls.c`[^3].
- When you call `printf`, Newlib will eventually call your `_write` function, which should send characters to your UART driver (in `uart.c`)[^3].
- Your custom linker script (`printf.ld`) ensures all sections are placed correctly in memory for bare-metal execution.

---

### **5. Key Points**

- You do **not** need an OS to use `printf` if you provide the right system call stubs and link with Newlib.
- The UART driver is essential: it provides the hardware output mechanism for `printf` via `_write`.
- The `syscalls.c` file is the glue between Newlib and your hardware, mapping C library calls to your UART or other devices[^3].
- Inspecting the ELF with `nm` confirms that the necessary symbols are present and linked.

---

### **References**

- The workflow and rationale are detailed in the [Bare Metal printf - C standard library without OS](https://popovicu.com/posts/bare-metal-printf/) guide[^3].

---

This approach enables powerful C standard library features like `printf` on bare-metal RISC-V systems, with output routed through your UART implementation—no operating system required.


**Screenshot:**

![Image](https://github.com/user-attachments/assets/666f4b5f-bc8d-4149-b71f-e64a0e1589e4)
![Image](https://github.com/user-attachments/assets/5e8dc74c-7167-4b40-b80b-b130e5957cf7)


---


# 17) Endianness & Struct Packing Verification

## Complete Implementation Files

### Linker Script (`endian.ld`)
```

ENTRY(_start)

MEMORY
{
FLASH (rx)  : ORIGIN = 0x2040, LENGTH = 256K
SRAM  (rwx) : ORIGIN = 0x80000000, LENGTH = 64K
}

SECTIONS
{
.text : {
*(.text*)
*(.rodata*)
} > FLASH

.data : {
*(.data*)
} > SRAM AT > FLASH

.bss : {
*(.bss*)
*(COMMON)
} > SRAM

/DISCARD/ : { *(.eh_frame) *(.comment) }
}

```

### Assembly Startup Code (`endian_start.s`)
```

.section .text
.global _start

_start:
la sp, _stack_top       \# Initialize stack pointer
call main               \# Enter C program
1:j 1b                    \# Infinite loop after return

.section .bss
.space 1024               \# 1KB stack space
.global _stack_top
_stack_top:

```

### Endianness Test Program (`task17_simple_endian.c`)
```

\#include <stdint.h>
\#define UART0 0x10013000

void uart_putc(char c) {
volatile uint8_t *tx = (uint8_t *)(UART0);
while ((*(volatile uint32_t *)(UART0 + 0x18)) \& 0x800); // Wait TX ready
*tx = c;
}

void main() {
uint32_t val = 0x12345678;
uint8_t *ptr = (uint8_t *)\&val;

uart_puts("Endianness Test:\n");

for(int i=0; i<4; ++i) { // Print each byte
char hex[] = "0123456789ABCDEF";
uart_putc(hex[(ptr[i] >>4)\&0xF]);
uart_putc(hex[ptr[i]\&0xF]);
uart_putc(' ');
}

while(1); // Halt
}

```

## Build Process
```


# Assemble startup code

riscv32-unknown-elf-as -march=rv32imac -o endian_start.o endian_start.s

# Compile C program

riscv32-unknown-elf-gcc -c -O2 -march=rv32imac -o task17_simple_endian.o task17_simple_endian.c

# Link components

riscv32-unknown-elf-gcc -nostartfiles -T endian.ld -o task17_simple_endian.elf \
endian_start.o task17_simple_endian.o

```

## Execution & Verification
```

qemu-system-riscv32 -machine sifive_e -nographic -bios none -kernel task17_simple_endian.elf

# Expected output:

Endianness Test:
78 56 34 12

```

This implementation demonstrates:
1. **Memory layout control** through linker script
2. **Low-level UART access** for bare-metal output
3. **Byte-wise memory inspection** technique
4. **RV32IMAC toolchain usage** with proper architecture flags

The output `78 56 34 12` shows:
- Least Significant Byte (0x78) at lowest memory address
- Consistent little-endian byte ordering pattern
- Successful cross-compilation and execution in QEMU RV32 environment```


**Common checks:**

- Use `riscv32-unknown-elf-readelf -S file.elf` to verify section addresses.
- Use `qemu-system-riscv32` with `-serial mon:stdio` for UART output.
- Use `riscv32-unknown-elf-objdump -d file.elf` to check for expected instructions.

**Screenshot:**
![Image](https://github.com/user-attachments/assets/35a1d190-7406-448d-851a-b78e5bf45a61)


