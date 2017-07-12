# Multi-value Extension

## Introduction

### Background

* Currently, functions and instructions consume multiple operands but can produce at most one result
  - functions: `value* -> value?`
  - instructions: `value* -> value?`
  - blocks: `[] -> value?`

* In a stack machine, these asymmetries are artifical restrictions
  - were imposed to simplify the initial WebAssembly release (multiple results deferred to post-MVP)
  - can easily be lifted by generalising to value* -> value*

* Generalised semantics is well-understood
  - https://github.com/WebAssembly/spec/tree/master/papers/pldi2017.pdf

* Semi-complete implementation of multiple results in V8


### Motivation

* Multiple return values for functions:
  - enable unboxing of tuples or structs returned by value
  - efficient compilation of multiple return values

* Multiple results for instructions:
  - enable instructions producing several results (divmod, arithmetics with carry)

* Inputs to blocks:
  - loop labels can have arguments
  - can represent phis on backward edges
  - enable future `pick` operator to cross block boundary
  - macro definability of instructons with inputs
    * `i32.select3` = `dup if ... else ... end`


## Spec Changes

### Structure

The structure of the language is mostly unaffected. The only changes are to the type syntax:

* *resulttype* is generalised from \[*valtype*?\] to \[*valtype*\*\]
* block types (in `block`, `loop`, `if` instructions) are generalised from *resulttype* to *functype*


### Validation

Arity restrictions are removed:

* no arity check is imposed for valid *functype*
* all occurrences of superscript "?" are replaced with superscript "\*" (e.g. blocks, calls, return)

Validation for block instructions is generalised:

* The type of `block`, `loop`, and `if` is the *functype* \[t1\*\] -> \[t2\*\] given as the block type
* The type of the label of `block` and `if` is \[t2\*\]
* The type of the label of `loop` is \[t1\*\]


### Execution

Nothing much needs to be done for multiple results:

* replace all occurrences of superscript "?" with superscript "\*".

The only non-mechanical change involves entering blocks with operands:

* The operand values are popped of the stack, and pushed right back after the label.
* See paper for formulation of formal reduction rules


### Binary Format

The binary requires a change to allow function types as block types. That requires extending the current ad-hoc encoding to allow references to function types.

* `blocktype` is extended to the following format:
  ```
  blocktype ::= 0x40       => [] -> []
             |  t:valtype  => [] -> [t]
             |  ft:typeidx => ft
  ```

### Text Format

The text format is mostly unaffected, except that the syntax for block types is generalised:

* `resulttype` is replaced with `blocktype`, whose syntax is
  ```
  blocktype ::= vec(param) vec(result)
  ```

* `block`, `loop`, and `if` instructions contain a `blocktype` instead of `resulttype`.

* The existing abbreviations for functions apply to block types.


### Soundness Proof

The typing of admininstrative instructions need to be generalised, see the paper.


## Possible Alternatives and Extensions

### More Flexible Block and Function Types

* Instead of inline function types, could use references to the type section
  - more bureaucracy in the semantics, but otherwise no problem

* Could also allow both
  - inline function types are slightly more compact for one-off uses

* Could even unify the encoding of block types with function types everywhere
  - allow inline types even for functions, for the same benefits


## Open Questions

* Destructuring or reshuffling multiple values requires locals, is that enough?
  - could add `pick` instruction (generalised `dup`)
  - could add `let` instruction (if you squint, a generalised `swap`)
  - different use cases?

