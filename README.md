# Source Mappings
As part of the AST output, the compiler provides the range of the source code that is represented by the respective node in the AST. This can be used for various purposes ranging from static analysis tools that report errors based on the AST and debugging tools that highlight local variables and their uses.

Furthermore, the compiler can also generate a mapping from the bytecode to the range in the source code that generated the instruction. This is again important for static analysis tools that operate on bytecode level and for displaying the current position in the source code inside a debugger or for breakpoint handling. This mapping also contains other information, like the jump type and the modifier depth (see below).

Both kinds of source mappings use integer identifiers to refer to source files. The identifier of a source file is stored in output['sources'][sourceName]['id'] where output is the output of the standard-json compiler interface parsed as JSON.

### Developer Note

In the case of instructions that are not associated with any particular source file, the source mapping assigns an integer identifier of -1. This may happen for bytecode sections stemming from compiler-generated inline assembly statements.

## Source Map Notation 

The source mappings inside the AST use the following notation:

### s:l:f

- s is the byte-offset to the start of the range in the source file
- l is the length of the source range in bytes
- f is the source index mentioned above.

The encoding in the source mapping for the bytecode is more complicated: It is a list of s:l:f:j:m separated by ;. Each of these elements corresponds to an instruction, i.e. you cannot use the byte offset but have to use the instruction offset (push instructions are longer than a single byte). The fields s, l and f are as above. j can be either i, o or - signifying whether a jump instruction goes into a function, returns from a function or is a regular jump as part of e.g. a loop. The last field, m, is an integer that denotes the "modifier depth". This depth is increased whenever the placeholder statement (_) is entered in a modifier and decreased when it is left again. This allows debuggers to track tricky cases like the same modifier being used twice or multiple placeholder statements being used in a single modifier.

### Source Map Rules 
In order to compress these source mappings especially for bytecode, the following rules are used:

If a field is empty, the value of the preceding element is used.
If a : is missing, all following fields are considered empty.
This means the following source mappings represent the same information:

1:2:1;1:9:1;2:1:2;2:1:2;2:1:2

1:2:1;:9;2:1:2;;

### Tips and Tricks

Use delete on arrays to delete all its elements.
Use shorter types for struct elements and sort them such that short types are grouped together. This can lower the gas costs as multiple SSTORE operations might be combined into a single (SSTORE costs 5000 or 20000 gas, so this is what you want to optimise). Use the gas price estimator (with optimiser enabled) to check!

Make your state variables public - the compiler creates :ref:`getters <visibility-and-getters>` for you automatically.
If you end up checking conditions on input or state a lot at the beginning of your functions, try using :ref:`modifiers`.
Initialize storage structs with a single assignment: x = MyStruct({a: 1, b: 2});
Note

If the storage struct has tightly packed properties, initialize it with separate assignments: x.a = 1; x.b = 2;. In this way it will be easier for the optimizer to update storage in one go, thus making assignment cheaper.


## License

Document Source: Solidity Documentation
Everything else under MIT