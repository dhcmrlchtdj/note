# compiling with continuations

---

## 01.overview

---

- CPS
    - makes every aspect of control flow and data flow explicit
    - easy for optimizing compilers to manipulate and transform

---

- representation
    - λ-calculus (lambda calculus)
    - CPS (continuation-passing style)
        - a restricted form of λ-calculus
    - PDG (program-dependence graphs)
    - QUAD (quadruples, register transfers)
    - SSA (static single-assignment form)
        - a restricted form of quadruples

(variables have a single-binding property in both CPS/SSA

- transformation
    - in-line expansions
    - closure representations
    - dataflow analysis
    - register allocation
    - vectorizing
    - instruction scheduling

---

(how to summary a language

- strict vs lazy
- higher-order function support
- parametric polymorphic types vs overloading
- static type, type inference
- garbage collection
- lexical scope
- side effects vs purely functional
- formally defined semantics

---

- a multipass compiler
    - lexing, parsing, type checking => annotated AST
    - => λ-calculus-like representation
    - => continuation-passing style representation
    - => optimize to a better CPS expression
    - => closure conversion
    - => elimination of nested scopes
    - => "register spilling"
    - => target-machine assembly-language
    - => target-machine instructions

(focus on the use of continuations for optimization and code generation.

---

## 02.continuation-passing style

---

in a well-formed CPS expressions
- the arguments to a function are always atomic (variables or constants)
- a function application can never be an argument of another application

- 会有这种性质，是因为这是在对冯诺伊曼机进行建模，每次操作的参数都是寄存器的值
- 程序的控制流（control flow）在转换到 CPS 时确定了，比如参数的求职顺序。
- 所有的函数调用，都变成了尾调用。

---

## 03. semantics of the cps

---

> In the CPS language it is possible to write syntactically correct but meaningless programs.

这就是类型系统想解决的问题呀

---

## 04. ml-specific optimizations

https://dev.realworldocaml.org/runtime-memory-layout.html

---

> most optimizations (partial evaluation, dataflow, register allocation, etc.) are done in the CPS representation.

在编译成 CPS 之前，可以进行一些语法、类型相关的优化。

---

## 05. CONVERSION INTO CPS

---

---

## 16. THE RUNTIME SYSTEM

---

> The most important job of a runtime system is to manage and garbage collect
> the heap, and do so as efficiently as possible.

这节其实主要是讲 GC。

- Cheney's copying collection algorithm
    - 广度优先遍历
    - to space 充当了 queue 的角色
    - scan 之后值待处理，next 之后的空余空间，scan 和 next 重合，就处理完了
- generational garbage collection
- data format
