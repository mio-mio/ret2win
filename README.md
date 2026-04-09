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
