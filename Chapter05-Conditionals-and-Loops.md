# Labels
A label is a location in your program which can be *jumped* to from another location using the `JMP`
instruction.

An example of an unconditional jump:
```c
top:
    mov eax, 10
    jmp bottom
middle: ; line under middle never runs
    add eax, 5
bottom:
    invoke ExitProcess, 0
```

## Conditions/Branching

Labels may be used to skip execution of blocks of code based off of conditional evaluation. Conditions may
be evaluated using the `TEST` or `CMP` instructions.

#### `TEST M/R L/M/R`
The `TEST` instruction performs a `bitwise AND` on two operands. 
> The first operand may be a memory address or a register. The second operand may be a memory address, a
register, or an immediate value.

The result of the operation is discarded; however, various *flags* are set based on the result of the operation:

* `SF` (Sign Flag) is set to the most significant bit of the resulting `AND` operation.
* `ZF` (Zero Flag) is set to `1` if the result of the `AND` operation is `0`.
* `PF` (Parity Flag) is set to `1` if the result of the `AND` operation is even, otherwise `0`.
* `CF` (Carry Flag) is set to `0`.
* `OF` (Overflow Flag) is set to `0`.

> *We'll get to why this is important shortly.*

#### `CMP M/R L/M/R`
The `CMP` instruction performs an arithmetic subtraction of the second operand from the first operand. This is analogous to `SUB op1, op2`.

The result of the subtraction is discarded; however, the following flags are set:

* `SF` (Sign Flag) is set to the most significant bit of the resulting `SUB` operation.
* `ZF` (Zero Flag) is set if the result of the `SUB` operation is `0`, that is, if `op1` and `op2` are equal.
* `PF` (Parity Flag) is set to `1` if the *least significant byte* is even.
* `CF` (Carry Flag) is set to `1` if unsigned `op2` > unsigned `op1`.
* `OF` (Overflow Flag) is set to 1 if the sign bit of `op1` is not the same as the sign bit of the result.

The values of these flags are used in various jump operations. Rather than having conditional directives (like with `if/else` in C++), assembly languages contain numerous *jump* instructions that perform conditional branching based on the value of the flag registers.

Below is a list of jump instructions which check the flag registers:

| Flag   | Instruction | Description                   | Semantic                                         |
| ------ | ----------- | ----------------------------- | ------------------------------------------------ |
| OF = 1 | JO          | Jump if overflow flag is `1`. | Jump if overflow has occurred.                   |
| OF = 0 | JNO         | Jump if overflow flag is `0`. | Jump if overflow has not occurred.               |
| PF = 1 | JP          | Jump if parity flag is `1`.   | Jump if the result is even.                      |
| PF = 0 | JNP         | Jump if parity flag is `0`.   | Jump if the result is odd.                       |
| SF = 1 | JS          | Jump if sign flag is `1`.     |                                                  |
| SF = 0 | JNS         | Jump if sign flag is `0`.     |                                                  |
| ZF = 1 | JE/JZ       | Jump if zero flag is `1`.     | Jump if `op1` is equal to `op2`.                 |
| ZF = 0 | JNE/JNZ     | Jump if zero flag is `0`.     | Jump if `op1` is not equal to `op2`.             |
| CF = 1 | JB/JC/JNAE  | Jump if carry flag is `1`.    | Jump if `op1` is less than `op2`.                |
| CF = 0 | JAE/JNB/JNC | Jump if carry flag is `0`.    | Jump uf `op1` is greater than or equal to `op2`. |

The following instructions *compare* flag registers:
| Flag               | Instruction | Description                      | Semantic                                                 |
| ------------------ | ----------- | -------------------------------- | -------------------------------------------------------- |
| CF = 1 OR ZF = 1   | JBE/JNA     | Jump if CF is `1` or ZF is `1`.  | (Unsigned) Jump if `op1` is less than or equal to `op2`. |
| CF = 0 AND ZF = 0  | JA/JNBE     | Jump if CF is `0` and ZF is `0`. | (Unsigned) Jump if `op1` is greater than `op2`.          |
| SF != OF           | JL/JNGE     | Jump if SF is not equal to OF.   | (Signed) Jump if `op1` is less than `op2`.               |
| SF = OF            | JGE/JNL     | Jump if SF and OF are equal.     | (Signed) Jump if `op1` is greater than `op2`.            |
| ZF = 1 or SF != OF | JLE/JNG     | Jump if ZF is `1` and SF != OF   | (Signed) Jump if `op1` is less than or equal to `op2`.   |
| ZF = 0 and SF = OF | JG/JNLE     | Jump if ZF is `0` and SF = OF    | (Signed) Jump if `op1` is greater than `op2`.            |

> Instructions that are separated by forwardslashes on the same entry
correspond to the exact same opcodes in machine code. That is, they
are virtually the same instructions.

An example of checking whether two values are equal:
```c
main:
    mov eax, 5
    add eax, 3
    cmp eax, 8
    je middle
    jne end ; necessary to skip the code under `middle`
middle:
    print "eax is 8!",13,10,0 ; gets printed, since eax is 8.
end:
    invoke ExitProcess, 0
```

## Loops
Labels may also be used to indicate the beginning of a repeating block of code, such that the code after the label can be called multiple times.

An example of an unconditional loop:
```c
top:
    mov eax, 10
middle: ; No break condition, runs forever
    add eax, 5
    jmp middle
bottom:
    invoke ExitProcess, 0
```

This code isn't very useful unless you want `middle` to execute forever. We can define a control structure for conditional loops using the `TEST/CMP` and `JMP` instructions, the way we do for conditional branching.

An example of a loop that counts to 5:
```c
mov ecx, 6
mov eax, 0
L1:
    inc eax
    dec ecx
    cmp ecx, 0
    je L1
```
