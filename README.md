# ret2win　Write-up
## 1. Overview

This write-up explains the ret2win challenge from ROP Emporium.
This challenge is a basic introduction to buffer overflow, using an existing ret instruction, and overwriting the return address to redirect execution.

## 2. Binary Analysis
### 2.1 Function Overview

First, *checksec* and *file* provide general information about the binary.
Using nm, we can identify functions such as main, pwnme, and ret2win.

From *objdump*, we can observe the following:

- The **pwnme** function allocates 0x20 (**32 bytes**) on the stack but reads 0x38 (**56 bytes**) of input, making it vulnerable to buffer overflow.
- The **ret2win** function prints a message and provides the desired functionality.

### 2.2 Stack Layout

The stack layout is as follows:

[ Buffer (32 bytes) ]

[ Saved RBP (8 bytes) ]

[ Return address ]

The return address is located after the buffer and saved RBP.

### 2.3 Vulnerability

The pwnme function allocates 32 bytes for the buffer but reads 56 bytes from user input.
Since there is no bounds checking, the extra input overwrites the saved RBP and eventually the return address.

This allows an attacker to control the execution flow.

## 3. Offset Calculation

A cyclic pattern (cyclic(100)) was generated and used as input in *GDB*.
After the program crashed, the overwritten value was identified from the stack trace.

The offset to the return address was determined to be **40 bytes**.

This also matches the stack structure: 32 (buffer) + 8 (saved RBP) = 40

## 4. Exploitation Strategy

The goal is to overwrite the return address with the address of the **ret2win** function.

- Fill the buffer up to the return address

- Fix stack alignment using a ret gadget

- Overwrite the return address with the target function
