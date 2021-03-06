# The lowering module in rustc

The program clauses described in the
[lowering rules](./traits-lowering-rules.html) section are actually
created in the [`rustc_traits::lowering`][lowering] module.

[lowering]: https://github.com/rust-lang/rust/tree/master/src/librustc_traits/lowering.rs

## The `program_clauses_for` query

The main entry point is the `program_clauses_for` [query], which --
given a def-id -- produces a set of Chalk program clauses. These
queries are tested using a
[dedicated unit-testing mechanism, described below](#unit-tests).  The
query is invoked on a `DefId` that identifies something like a trait,
an impl, or an associated item definition. It then produces and
returns a vector of program clauses.

[query]: ./query.html

<a name=unit-tests>

## Unit tests

Unit tests are located in [`src/test/ui/chalkify`][chalkify]. A good
example test is [the `lower_impl` test][lower_impl]. At the time of
this writing, it looked like this:

```rust
#![feature(rustc_attrs)]

trait Foo { }

#[rustc_dump_program_clauses] //~ ERROR Implemented(T: Foo) :-
impl<T: 'static> Foo for T where T: Iterator<Item = i32> { }

fn main() {
    println!("hello");
}
```

The `#[rustc_dump_program_clauses]` annotation can be attached to
anything with a def-id.  (It requires the `rustc_attrs` feature.) The
compiler will then invoke the `program_clauses_for` query on that
item, and emit compiler errors that dump the clauses produced. These
errors just exist for unit-testing, as we can then leverage the
standard [ui test] mechanisms to check them. In this case, there is a
`//~ ERROR Implemented` annotation which is intentionally minimal (it
need only be a prefix of the error), but [the stderr file] contains
the full details:

```
error: Implemented(T: Foo) :- ProjectionEq(<T as std::iter::Iterator>::Item == i32), TypeOutlives(T \
: 'static), Implemented(T: std::iter::Iterator), Implemented(T: std::marker::Sized).
  --> $DIR/lower_impl.rs:15:1
   |
LL | #[rustc_dump_program_clauses] //~ ERROR Implemented(T: Foo) :-
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error: aborting due to previous error
```

[chalkify]: https://github.com/rust-lang/rust/tree/master/src/test/ui/chalkify
[lower_impl]: https://github.com/rust-lang/rust/tree/master/src/test/ui/chalkify/lower_impl.rs
[the stderr file]: https://github.com/rust-lang/rust/tree/master/src/test/ui/chalkify/lower_impl.stderr
[ui test]: https://rust-lang-nursery.github.io/rustc-guide/tests/adding.html#guide-to-the-ui-tests
