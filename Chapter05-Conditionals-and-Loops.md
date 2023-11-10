# Chapter 5 -- Conditions and Loops
Conditions are necessary to implement *control structures* in a program. Most programs are not very practical without the ability to change the course of execution based on evaluation of input data.

Software also implements *loops* to repeat a series of instructions until a break condition is met. This is useful in many applications, especially ones with interactive user-facing interfaces.

## Labels
A label is a location in your program which can be *jumped* to from another location using the `JMP`
instruction.

An example of an unconditional jump:
```asm
top:
    mov eax, 10
    jmp bottom
middle: ; line under middle never runs, eax is still 10
    add eax, 5
bottom:
    invoke ExitProcess, 0
```

## Conditions/Branching

Labels may be used to skip execution of blocks of code based off of conditional evaluation. Conditions may
be evaluated using the `TEST` or `CMP` instructions. Conditional checks can be used to change what code is executed on the stack. This is called [*branching*](https://en.wikipedia.org/wiki/Branch_(computer_science)).

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

> *Visual Studio note: You can find information on how to use the `Register` window in debug mode [here](https://learn.microsoft.com/en-us/visualstudio/debugger/debugging-basics-registers-window?view=vs-2022).*

Below is a list of jump instructions which check the flag registers:

| Flag   | Instruction | Description                   | Semantic (Using `CMP`)                           |
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
| CF = 0 | JAE/JNB/JNC | Jump if carry flag is `0`.    | Jump if `op1` is greater than or equal to `op2`. |

The following instructions *compare* flag registers:
| Flag               | Instruction | Description                      | Semantic (Using `CMP`)                                   |
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

> **Every *jump* instruction has a single operand, which is the *LABEL* at which to jump.**

There also exists conditional jump instructions for checking the `CX/ECX/RCX` register:
| Register | Instruction   | Description                     |
| -------- | ------------- | ------------------------------- |
| *`CX`*   | JCXZ *LABEL*  | Jump to *LABEL* if CX is `0`.   |
| *`ECX`*  | JECXZ *LABEL* | Jump to *LABEL* if ECX is `0`.  |
| *`RCX`*  | JRCXZ *LABEL* | Jump to *LABEL* if RCX is `0`.  |

An example of checking whether two values are equal:
```asm
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
> *Note: This print operation probably wouldn't work, I just haven't yet found a way to actually print text that would be intuitive for this section.*

A rough equivalent in C++ might look like:
```cpp
int main() {
    auto a = 5;
    a += 3;
    if (a == 8) {
        std::cout << "a is 8!\n";
    }
    return 0;
}
```

Or more accurately (but not as intuitively):
```cpp
int main() {
    auto a = 5;
    a += 3;
    if (a != 8) { goto end; }
middle:
    std::cout << "a is 8!\n";
end:
    return 0;
}
```

## Loops
Labels may also be used to indicate the beginning of a repeating block of code, such that the code after the label can be called multiple times.

An example of an unconditional loop:
```asm
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
```asm
mov ecx, 5
mov eax, 1
L1:
    inc eax
    dec ecx
    cmp ecx, 0
    je L1
```

A rough equivalent in C++ might look like:
```cpp
auto val = 1;
for (int i = 0; i < 5; ++i) {
    ++val;
}
```

## Looping using CX/ECX/RCX

#### `LOOP `*`LABEL`*
The `LOOP` instruction internally decrements `CX/ECX/RCX` *then* jumps to the operand *`LABEL`* if `CX/ECX/RCX` is not zero.

An example of a loop that repeats five times, using `EAX` to keep track of iterations:
```asm
.code
_main PROC
	mov eax, 0
	mov ecx, 5
L1:
	inc eax
	loop L1

    ; By this point, eax is equal to 5

	INVOKE ExitProcess, 0
_main ENDP
```

You can use `ECX` directly as a counter register without the use of the `LOOP` instruction. The code above is semantically equivalent to:
```asm
.code
_main PROC
    mov eax, 0
    mov ecx, 5
L1:
    inc eax

    dec ecx
    cmp ecx, 0
    jne L1

    ; By this point, eax is equal to 5

_main ENDP
```

A rough C++ equivalent might look like:
```cpp
int main() {
    int a = 0;
    int i = 5;
    do {
        ++a;
    } while (--i != 0);

    std::cout << a << '\n';
}
```

## Extras -- Compound Conditionals
A *compound conditional* statement is simply a boolean statement which tests multiple conditions in one expression. You've undoubtedly seen (if not written) your own before. Examples include boolean *AND* and boolean *OR* conditional statements:
```cpp
if (a > b && b > c) {
    ...
}

if (a > b || b > c) {
    ...
}
```

An intrinsic of boolean AND is that if the first condition of `A && B` (`A`) evalutes to *false*, the second condition (`B`) is never evaluated, since the entire statement will evaluate as *false* anyway. This is called [*short-circuit evaluation*](https://en.wikipedia.org/wiki/Short-circuit_evaluation). 

A rough equivalent of the boolean AND operation in MASM might look like:
```asm
cmp ax, bx
jbe next    ;  skip to `next` if `ax > bx` is false
cmp bx, cx
jbe next    ;  skip to `next` if `bx > cx` is false

...         ;  will only run if ax > bx and bx > cx

next:
```

The inverse is true for boolean OR. If `A` in the statement `A || B` evaluates to true, `B` is not evaluated, since `true || X` is true whether *`X`* is true or false.

A rough equivalent of the boolean OR operation in MASM might look like:
```asm
cmp  ax, bx
ja   or_true  ;  if `ax > bx` is true, don't bother checking second condition
cmp  bx, cx
jbe  next     ;  skip past or_true if `bx > cx` is false.

or_true:
...           ;  will run if `ax > bx` is true or `bx > cx` is true

next:
```

## Contributing/Issues
If anything is missing, or anything from these notes are confusing, feel free to reach out. You can open a GitHub issue or message me on Discord at @`zaruhev`.
___
Copyleft (Ↄ) Sarah Evans 
