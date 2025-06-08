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

