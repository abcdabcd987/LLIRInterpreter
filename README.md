# LLIRInterpreter
Single file interpreter (or naive virtual machine) for my intermediate representation. SSA support has been added.

## How to use?

You can use [`LLIRInterpreter::main`](https://github.com/abcdabcd987/LLIRInterpreter/blob/master/src/LLIRInterpreter.java#L490).

```bash
cd src                      # enter the source directory
javac LLIRInterpreter.java  # compile
java LLIRInterpreter        # run the VM (read from stdin)

# or my favourite: paste from the pasteboard and run the VM in SSA Mode
pbpaste | java LLIRInterpreter +ssa
```

Or you can copy [`src/LLIRInterpreter.java`](https://github.com/abcdabcd987/LLIRInterpreter/blob/master/src/LLIRInterpreter.java) to your code and use it as a library.

## API

#### `LLIRInterpreter(InputStream in, boolean isSSAMode) throws IOException`
Read IR from `in` and build internal data structures. Enable SSA mode if `isSSAMode` is true. If nothing goes wrong, mark `isReady()` true.

#### `void run()`
Run the virtual machine.

#### `void setInstructionLimit(int instLimit)`
Set the maximum instructions that the virtual machine can operate.

#### `boolean isReady()`
Check if the virtual machine is ready to run.

#### `int getExitcode()`
Get the exitcode of the virtual machine.

#### `boolean exitException()`
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

## Brief Introduction to SSA Mode

In SSA Mode,

- A register can only be defined once statically (not dynamically).
- A `phi` node must have source register for all possible incoming blocks.
- A source register in a `phi` node can take a special value `undef`, which says any value is acceptable in that context.
- All `phi` nodes should be placed in the front of a block, i.e., `phi` nodes are forbidden in the middle of a block.

## Brief Introduction to VM

- All registers are 32-bit integer register.
- All integers are signed integer.
- Functions do not share any register.
- Will terminate if memory access violation occurs.
- Will terminate if arithmetic error occurs.
- Will terminate if you try to read a register that has no value.
- Execution starts at `main` function.
- A random padding `(< 4KB)` is added after `alloc` in order to help detect memory access violation.

## Instruction Set

You can guess the meaning from their names. All register except `$dest` and `$reg*` can be replaced by immediate number. `$reg*` can also be replaced by `undef`.

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

Phi Instruction:
    $dest = phi %block1 $reg1 %block2 $reg2 ...

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
    $dest = not $src

Condition Set Instruction:
    $dest = slt $src1 $src2
    $dest = sgt $src1 $src2
    $dest = sle $src1 $src2
    $dest = sge $src1 $src2
    $dest = seq $src1 $src2
    $dest = sne $src1 $src2
```

## Sample IR 1

```llvm
func min $a $b {
%min_entry:
    $t = sle $a $b
    br $t %if_true %if_merge

%if_true:
    ret $a

%if_merge:
    ret $b
}

func main {
%main_entry:
    $x = move 10
    $y = move 20
    $x = call min $x $y 
    ret $x
}
```

## Sample IR 2

```llvm
func main {
%main.entry:
    $n.1  = move 10
    $f0.1 = move 0
    $f1.1 = move 1
    $i.1  = move 1
    jump %for_cond

%for_cond:
    $f2.1 = phi %for_step $f2.2  %main.entry undef
    $f1.2 = phi %for_step $f1.3  %main.entry $f1.1
    $i.2  = phi %for_step $i.3   %main.entry $i.1
    $f0.2 = phi %for_step $f0.3  %main.entry $f0.1
    $t.1  = slt $i.2 $n.1
    br $t.1 %for_loop %for_after

%for_loop:
    $t_2.1 = add $f0.2 $f1.2
    $f2.2  = move $t_2.1
    $f0.3  = move $f1.2
    $f1.3  = move $f2.2
    jump %for_step

%for_step:
    $i.3   = add $i.2 1
    jump %for_cond

%for_after:
    ret $f2.2
}
```

## To-do List

- Void function support
- Static data
- `print`
- `read`

