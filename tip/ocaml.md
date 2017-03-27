# ocaml manual

---

http://caml.inria.fr/pub/docs/manual-ocaml/

---

- int / float / bool / char / string
    - `0`
    - `1.1`
    - `true` / `false`
    - `'c'`
    - `"string"`
- tuple / list / array
    - `(1, 2)`
    - `[1; 2]`
    - `1::2::[]`
    - `[|1; 2|]`
- function
- record / variant
    - `type r = { x : float; y : int; }`
    - `type v = Int | Float`
- array assign / mutable record / ref (single-field mutable record)
    - `arr.(idx) <- 10`
    - `type m = { mutable x : float };; let mm = { x=10. };; mm.x <- 10;;`
    - `ref i`, `!i`, `i := 0`
- exception
    - `exception`
    - `raise`
    - `try ... with ...`

---

- structure
- signature
- functor
- module

---

- object

---

- label
- polymorphic variant