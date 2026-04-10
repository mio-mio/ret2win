# ret2win　write-up
## 1. Overview

This write-up explains the ret2win challenge from ROP Emporium. (https://ropemporium.com/challenge/ret2win.html)
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

This vulnerability exists because the program writes more data than the allocated buffer size without validation.

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

## 5. Payload Construction

The payload consists of three parts: 'A' * offset + ret gadget + ret2win address
This ensures correct alignment and redirects execution to **ret2win**.

## 6. Mitigation

Stack canaries can prevent this type of attack.
A canary is a random value placed before the return address, and the program checks whether it has been modified before returning.

If the canary is altered, the program terminates, preventing exploitation.

## 7. Result

The exploit successfully redirects execution and reveals the flag: ROPE{a_placeholder_32bytes_flag}

## 8. Lessons Learned

- Understanding stack layout is essential for exploitation
- Input size must always be validated
- Stack canaries are an effective mitigation technique
- Intel and AT&T assembly syntaxes use different operand orders
