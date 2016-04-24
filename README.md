# LLIRInterpreter
Single file interpreter (or naive virtual machine) for my intermediate representation.

## How to use?

You can use [`LLIRInterpreter::main`](https://github.com/abcdabcd987/LLIRInterpreter/blob/master/src/LLIRInterpreter.java#L380), or you can copy [`src/LLIRInterpreter.java`](https://github.com/abcdabcd987/LLIRInterpreter/blob/master/src/LLIRInterpreter.java) to your code and use it as a library.

## API

### `LLIRInterpreter(InputStream in) throws IOException`
Read IR from `in` and build internal data structures. If nothing goes wrong, mark `isReady()` true.

### `void run()`
Run the virtual machine.

### `void setInstructionLimit(int instLimit)`
Set the maximum instructions that the virtual machine can operate.

### `boolean isReady()`
Check if the virtual machine is ready to run.

### `int getExitcode()`
Get the exitcode of the virtual machine.

### `boolean exitException()`
Return true if the virtual machine is terminated by an exception.

## Brief Introduction to IR

- Infinite number of registers.
- Registers are writeable if not in SSA Mode.
- Registers' name start with `$`. e.g. `$t1`, `$arr_addr`, `$arr.addr`, ...
- Blocks' name starts with `%`. e.g. `%entry`, `%if_true`, `%if.true`, ...
- A block must end with a jump instruction, i.e., fall-through is not allowed.
- Function call: `$dest = call name $arg1 $arg2 ...`.
- Use `func name $arg1 $arg2 ... {` to start a function definition. `func` can be replaced with `void` if no return value.
- The entry block of a function is the first block in it.
- Heap allocation: `$dest = alloc $size` will acquire `$size` bytes from heap.

## Brief Introduction to VM

- All registers are 32-bit integer register.
- All integers are signed integer.
- Functions do not share any register.
- Will terminate if memory access violation occurs.
- Will terminate if arithmetic error occurs.
- Will terminate if you try to read a register that has no value.
- Execution starts at `main` function.

## Instruction Set

You can guess the meaning from their names. All register except `$dest` can be replaced by immediate number.

```
Jump Instruction:
    ret $src
    jump %target
    br $cond %ifTrue %ifFalse

Memory Access Instruction:
    store size $addr $src offset    // M[$addr+offset : $addr+offset+size-1] <- $src
    $dest = load size $addr offset  // $dest <- M[$addr+offset : $addr+offset+size-1]
    $dest = alloc $size

Function Call Instruction:
    call funcname $op1 $op2 $op3 ...
    $dest = call funcname $op1 $op2 $op3 ...

Register Transfer Instruction:
    $dest = move $src

Arithmetic Instruction:
    $dest = neg $src
    $dest = add $src1 $src2
    $dest = sub $src1 $src2
    $dest = mul $src1 $src2
    $dest = div $src1 $src2
    $dest = rem $src1 $src2

Bitwise Instruction:
    $dest = shl $src1 $src2
    $dest = shr $src1 $src2
    $dest = and $src1 $src2
    $dest = xor $src1 $src2
    $dest = or $src1 $src2
    $dest = not $src         // bitwise not

Condition Set Instruction:
    $dest = slt $src1 $src2
    $dest = sgt $src1 $src2
    $dest = sle $src1 $src2
    $dest = sge $src1 $src2
    $dest = seq $src1 $src2
    $dest = sne $src1 $src2
```

## Sample IR

```
func min $a $b {
%min_start:
    $t = sle $a $b
    br $t %if_true %if_merge

%if_true:
    ret $a

%if_merge:
    ret $b
}

func main {
%main_start:
    $x = move 10
    $y = move 20
    $t = call min $x $y 
    ret $t
}
```

## To-do List

- Void function support
- `print` function
- SSA Mode

