r[expr.call]
# Call expressions

r[expr.call.syntax]
```grammar,expressions
CallExpression -> Expression `(` CallParams? `)`

CallParams -> Expression ( `,` Expression )* `,`?
```

r[expr.call.intro]
A *call expression* calls a function.
The syntax of a call expression is an expression, called the *function operand*, followed by a parenthesized comma-separated list of expression, called the *argument operands*.

r[expr.call.convergence]
If the function eventually returns, then the expression completes.

r[expr.call.trait]
For [non-function types], the expression `f(...)` uses the method on one of the following traits based on the function operand:

- [`Fn`] or [`AsyncFn`] --- shared reference.
- [`FnMut`] or [`AsyncFnMut`] --- mutable reference.
- [`FnOnce`] or [`AsyncFnOnce`] --- value.

r[expr.call.autoref-deref]
An automatic borrow will be taken if needed.
The function operand will also be [automatically dereferenced] as required.

Some examples of call expressions:

```rust
# fn add(x: i32, y: i32) -> i32 { 0 }
let three: i32 = add(1i32, 2i32);
let name: &'static str = (|| "Rust")();
```

r[expr.call.desugar]
## Disambiguating Function Calls

r[expr.call.desugar.fully-qualified]
All function calls are sugar for a more explicit [fully-qualified syntax].

r[expr.call.desugar.ambiguity]
Function calls may need to be fully qualified, depending on the ambiguity of a call in light of in-scope items.

> [!NOTE]
> In the past, the terms "Unambiguous Function Call Syntax", "Universal Function Call Syntax", or "UFCS", have been used in documentation, issues, RFCs, and other community writings. However, these terms lack descriptive power and potentially confuse the issue at hand. We mention them here for searchability's sake.

r[expr.call.desugar.limits]
Several situations often occur which result in ambiguities about the receiver or referent of method or associated function calls.
These situations may include:

* Multiple in-scope traits define methods with the same name for the same types
* Auto-`deref` is undesirable; for example, distinguishing between methods on a smart pointer itself and the pointer's referent
* Methods which take no arguments, like [`default()`], and return properties of a type, like [`size_of()`]

r[expr.call.desugar.explicit-path]
To resolve the ambiguity, the programmer may refer to their desired method or function using more specific paths, types, or traits.

For example,

```rust
trait Pretty {
    fn print(&self);
}

trait Ugly {
    fn print(&self);
}

struct Foo;
impl Pretty for Foo {
    fn print(&self) {}
}

struct Bar;
impl Pretty for Bar {
    fn print(&self) {}
}
impl Ugly for Bar {
    fn print(&self) {}
}

fn main() {
    let f = Foo;
    let b = Bar;

    // we can do this because we only have one item called `print` for `Foo`s
    f.print();
    // more explicit, and, in the case of `Foo`, not necessary
    Foo::print(&f);
    // if you're not into the whole brevity thing
    <Foo as Pretty>::print(&f);

    // b.print(); // Error: multiple 'print' found
    // Bar::print(&b); // Still an error: multiple `print` found

    // necessary because of in-scope items defining `print`
    <Bar as Pretty>::print(&b);
}
```

Refer to [RFC 132] for further details and motivations.

[RFC 132]: https://github.com/rust-lang/rfcs/blob/master/text/0132-ufcs.md
[`default()`]: std::default::Default::default
[`size_of()`]: std::mem::size_of
[automatically dereferenced]: field-expr.md#automatic-dereferencing
[fully-qualified syntax]: ../paths.md#qualified-paths
[non-function types]: ../types/function-item.md
